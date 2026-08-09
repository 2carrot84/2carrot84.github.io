---
title: "[JPA] @Convert + @IdClass 복합키에서 발생한 암호화 트러블슈팅"
excerpt_separator: "<!--more-->"
categories:
  - development

tags:
  - jpa
  - encrypt
  - aes256
  - attributeConverter
  - compositeKey
  - idClass
  - querydsl
---

특정 개인정보 필드를 AES-256으로 암호화 전환하는 작업 중, JPA `@Convert`와 `@IdClass` 복합키를 함께 사용할 때 겪은 트러블슈팅을 정리합니다.

암호화 자체보다 **JPA가 @Id 필드를 다루는 방식**이 일반 필드와 다르다는 점에서 꽤 시간을 잡아먹었고, 같은 서비스 내에서도 JPA/MyBatis 혼용 구조 때문에 혼선이 있었기에 기록해둡니다.

<!--more-->

### 배경: 민감정보 암호화 전환

개인정보 보호 정책 강화에 따라 기존에 평문으로 저장하던 특정 식별번호 필드를 AES-256-ECB로 암호화하는 작업을 진행했습니다.

핵심 전략은 **JPA `@Convert`를 활용한 자동 암/복호화**였습니다.

```java
@Convert(converter = SensitiveDataConverter.class)
@Column(name = "identifier")
private String identifier;
```

이렇게 하면 Entity를 통한 모든 read/write에서 자동으로 encrypt/decrypt가 수행되니까, Service 코드에서 수동으로 `encrypt()`/`decrypt()`를 호출할 필요가 없어집니다. 누락 위험도 없고, 코드도 깔끔해지죠.

대부분의 Entity에서는 이 전략이 잘 동작했습니다. **한 군데를 제외하고.**

### 문제: 복합키 Entity — @Id + @Convert 조합

특정 매핑 테이블은 `userId`(회원번호)와 `identifier`(식별번호)를 복합키로 사용하고 있었습니다.

```java
@Entity
@Table(name = "user_identifier")
@IdClass(UserIdentifierId.class)
public class UserIdentifierEntity {

    @Id
    @Column(name = "user_id")
    private Integer userId;

    @Id
    @Convert(converter = SensitiveDataConverter.class)
    @Column(name = "identifier")
    private String identifier;
    
    // ...
}
```

```java
@EqualsAndHashCode
public class UserIdentifierId implements Serializable {
    private Integer userId;
    private String identifier;
}
```

여기에 `@Convert`를 달았으니, 당연히 조회할 때도 자동으로 동작할 거라 생각했습니다. 

실제로 **저장(INSERT)과 읽기(SELECT 결과 매핑)는 정상 동작**했습니다. DB에 암호문이 잘 들어가고, Entity로 읽으면 평문이 잘 나왔습니다.

문제는 **검색 조건(WHERE)** 이었습니다.

### 증상: 분명 데이터가 있는데 조회가 안 된다

식별번호로 회원을 조회하는 Querydsl 코드가 있었습니다.

```java
public UserEntity findUserBy(String identifier) {
    return queryFactory.selectFrom(USER_ENTITY)
        .innerJoin(IDENTIFIER_ENTITY)
        .on(USER_ENTITY.userId.eq(IDENTIFIER_ENTITY.userId))
        .where(IDENTIFIER_ENTITY.identifier.eq(identifier))
        .fetchOne();
}
```

`@Convert`가 적용되어 있으니 평문을 전달하면 Hibernate가 자동으로 `convertToDatabaseColumn()`을 호출해서 암호문으로 비교할 거라 예상했는데... **결과가 null**이었습니다.

로그를 확인해보니 회원이 존재하지 않는 것으로 판정되어 후속 로직이 중단되고 있었습니다.

DB에 해당 식별번호가 암호화되어 잘 들어가 있는 것도 확인했는데, 조회만 안 되는 상황.

### 원인: Querydsl JOIN 조건에서 @Id + @Convert 필드의 Converter가 적용되지 않는다

원인을 정확히 파악하기 위해 테스트 코드를 작성하여 케이스별로 검증했습니다.

**테스트 환경**: Spring Boot 2.3.12, Hibernate 5.4.32, Querydsl 5.0.0, H2

```java
// Converter: 문자열 reverse (ABC → CBA). 동작 여부를 눈으로 확인하기 쉽게.
// save("ABC123") → DB에 "321CBA" 저장
// findById("ABC123") → Converter 적용 시 WHERE = '321CBA' → 매칭 성공
```

#### 결과

| PK 방식 | `selectFrom(Entity).where(.eq(평문))` | **`join(Entity).on(...).where(.eq(평문))`** |
|---|---|---|
| 단일 `@Id` + `@Convert` | ✅ Converter 적용됨 | ❌ **Converter 미적용** |
| `@IdClass` + `@Id` + `@Convert` | ✅ Converter 적용됨 | ❌ **Converter 미적용** |
| `@EmbeddedId` + `@Embeddable` 내부 `@Convert` | ✅ Converter 적용됨 | ✅ **Converter 정상 적용** |

추가로 `findById`, `existsById`, JPQL WHERE도 전부 테스트했는데, 이 경로들은 3가지 방식 모두 정상 동작했습니다.

#### 핵심 차이

- **`selectFrom(Entity).where`**: 해당 Entity를 직접 조회 → Hibernate가 Entity 메타데이터에서 타입 정보를 참조 → Converter 적용됨
- **`join(Entity).on(...).where`**: 다른 Entity 기준 조회 중 JOIN → Querydsl이 JOIN 대상 필드의 타입을 **`@Id` 경로로 해석** → Converter를 건너뜀
- **`@EmbeddedId`**: `@Embeddable` 클래스 내부 필드는 `@Id`가 아닌 일반 필드로 처리됨 → JOIN에서도 Converter 정상 적용

즉, **Querydsl이 JOIN 쿼리를 생성할 때 `@Id`가 붙은 필드는 Converter 변환 없이 raw 값으로 파라미터를 바인딩**하는 것이 원인입니다. Hibernate 버전과 무관하게 `@Id` 어노테이션이 붙어 있는지 여부가 핵심입니다.

같은 서비스 내에서도:
- `OrderEntity.identifier` (일반 `@Column` + `@Convert`) → **평문 전달 OK** (자동 변환)
- `UserIdentifierEntity.identifier` (`@Id` + `@Convert`) → JOIN 조건에서 사용하면 **수동 encrypt 필요**

### 대응: 수동 encrypt + 평문 fallback

```java
public UserEntity findUserBy(String identifier) {
    String encrypted = CryptoUtil.encrypt(identifier, secretKey);
    UserEntity user = queryFactory.selectFrom(USER_ENTITY)
        .innerJoin(IDENTIFIER_ENTITY)
        .on(USER_ENTITY.userId.eq(IDENTIFIER_ENTITY.userId))
        .where(IDENTIFIER_ENTITY.identifier.eq(encrypted))
        .fetchOne();
    // 마이그레이션 과도기: 아직 평문인 데이터 대응
    if (user == null) {
        user = queryFactory.selectFrom(USER_ENTITY)
            .innerJoin(IDENTIFIER_ENTITY)
            .on(USER_ENTITY.userId.eq(IDENTIFIER_ENTITY.userId))
            .where(IDENTIFIER_ENTITY.identifier.eq(identifier))
            .fetchOne();
    }
    return user;
}
```

`existsById`, `findById`, Spring Data의 `findByIdentifier` 등 `@Id` 필드를 조건으로 사용하는 모든 경로에서 동일한 패턴을 적용해야 했습니다.

### 같은 테이블, 다른 모듈, 다른 복합키 구현 — 진짜 혼선

사실 위 규칙까지 정리하고 나면 깔끔할 것 같지만, 현실은 그렇지 않았습니다.

우리 서비스는 여러 모듈(마이크로서비스)이 **같은 DB 테이블을 각자의 Entity로 매핑**하고 있었는데, 문제는 **모듈마다 복합키 구현 방식이 달랐다**는 것입니다.

**모듈 A** — `@IdClass` 방식:
```java
@Entity
@IdClass(UserIdentifierId.class)
public class UserIdentifierEntity {

    @Id
    private Integer userId;

    @Id
    @Convert(converter = SensitiveDataConverter.class)
    private String identifier;
}
```

**모듈 B** — `@EmbeddedId` 방식:
```java
@Entity
public class UserIdentifierEntity {

    @EmbeddedId
    private UserIdentifierId id;
}

@Embeddable
public static class UserIdentifierId implements Serializable {

    @Column(name = "user_id")
    private int userId;

    @Convert(converter = SensitiveDataConverter.class)
    @Column(name = "identifier")
    private String identifier;
}
```

같은 테이블, 같은 필드, 같은 `@Convert`인데 모듈에 따라 **Entity 구조가 다릅니다**.

모듈 A에서 `@Id` + `@Convert` 이슈를 발견하고 "수동 encrypt 필요"라는 규칙을 만들었는데, 모듈 B에서도 같은 이슈가 있을 거라 생각하고 동일한 대응을 하려 했습니다.

그런데 실제로 확인해보니 **모듈 B(`@EmbeddedId`)는 문제가 없었습니다.**

```java
// 모듈 B — 평문 전달해도 정상 동작
carRepository.existsById(
    UserIdentifierId.builder().userId(userId).identifier(plainText).build()
);
// → Hibernate가 @Embeddable 내부 @Convert를 정상 적용하여 암호문으로 비교 ✅
```

정리하면:

| 복합키 방식 | selectFrom WHERE .eq() | **JOIN 조건 .eq()** | 수동 encrypt 필요 |
|---|---|---|---|
| `@IdClass` + `@Id` + `@Convert` | ✅ 정상 | ❌ Converter 미적용 | ✅ JOIN 사용 시 필요 |
| `@EmbeddedId` + `@Embeddable` 내부 `@Convert` | ✅ 정상 | ✅ 정상 | ❌ 불필요 |

같은 `@Id` + `@Convert`인데 **구현 방식에 따라 동작이 다릅니다**. `@EmbeddedId`는 Hibernate가 `@Embeddable` 클래스 내부의 `@Convert`를 BasicType으로 정상 처리하는 반면, `@IdClass`는 ID 처리 경로가 달라서 Converter를 건너뛰는 것으로 보입니다.

처음에는 "복합키면 다 문제"라고 생각하고 모듈 B에도 수동 encrypt를 넣으려 했는데, 실제 테스트해보니 불필요했습니다. 오히려 넣었으면 이중 암호화가 됐을 거예요.

교훈은:
- **`@IdClass`와 `@EmbeddedId`는 같은 복합키라도 Hibernate 내부 처리 경로가 다르다**
- "복합키니까 다 수동 encrypt"가 아니라, **실제 동작을 확인**해야 한다
- 멀티 모듈에서 같은 테이블을 다를 때, 한 모듈의 규칙을 다른 모듈에 기계적으로 적용하면 오히려 문제가 생길 수 있다

### 한 가지 더: 이중 암호화

반대로 "JPA에 암호문을 전달하면 안 된다"는 것도 중요했습니다.

일반 `@Column` + `@Convert` 필드에 이미 encrypt된 값을 전달하면, Hibernate가 그 값을 **또 한번** encrypt합니다.

```
Service: encrypt("원본값") → "aBcDeFgHiJkLmNoPqRsT=="  (24자)
Entity.setIdentifier("aBcDeFgHiJkLmNoPqRsT==")
JPA save() → @Convert가 "aBcDeFgHiJkLmNoPqRsT=="를 또 encrypt
DB: "xYzAbCdEfGhIjKlMnOpQrStUvWxYzAbCdEfGhIjKlMn=" (44자, 비정상!)
```

정상이면 24자(Base64, 1블록)인데 44자가 나오면 이중 암호화를 의심해야 합니다. 실제로 이 문제를 테스트 서버에서 먼저 발견했는데, DB 값의 **길이**로 바로 진단할 수 있었습니다.

### 정리

테스트로 확인한 케이스별 `@Convert` 동작 여부:

| PK 방식 | INSERT/UPDATE | findById / existsById | selectFrom WHERE .eq() | **JOIN 조건 WHERE .eq()** |
|---|---|---|---|---|
| 일반 `@Column` + `@Convert` | ✅ | — | ✅ | ✅ |
| 단일 `@Id` + `@Convert` | ✅ | ✅ | ✅ | ❌ |
| `@IdClass` + `@Id` + `@Convert` | ✅ | ✅ | ✅ | ❌ |
| **`@EmbeddedId`** + `@Embeddable` 내부 `@Convert` | ✅ | ✅ | ✅ | **✅** |

**복합키에서 `@Convert`를 사용해야 한다면 `@EmbeddedId` 방식을 권장합니다.**

`@IdClass`는 Entity 필드에 `@Id`를 직접 붙이기 때문에 Querydsl JOIN 쿼리 생성 시 Converter 경로를 타지 않습니다. `@EmbeddedId`는 `@Embeddable` 내부 필드가 `@Id`가 아닌 일반 필드로 처리되므로 모든 경로에서 Converter가 정상 동작합니다.

그리고 가장 중요한 점 — `selectFrom(Entity).where(.eq())`는 정상 동작하기 때문에 **단위 테스트에서는 문제가 발견되지 않습니다**. 실제 서비스에서 여러 Entity를 JOIN해서 사용할 때 비로소 문제가 발생하므로, 통합 테스트 또는 실제 쿼리 패턴과 동일한 조건으로 검증해야 합니다.

그럼 이만. 🥕👋🏼🖐🏼

### 참고자료
- [Hibernate @Convert with @Id 이슈 관련 Stackoverflow](https://stackoverflow.com/questions/29856053/hibernate-jpa-attributeconverter-not-applied-to-id-field)
- [JPA AttributeConverter 공식 문서](https://jakarta.ee/specifications/persistence/3.0/jakarta-persistence-spec-3.0.html#a16056)
- [Hibernate 공식 포럼: @Convert ignored when @Id is used](https://discourse.hibernate.org/t/convert-ignored-when-id-is-used/2281)
