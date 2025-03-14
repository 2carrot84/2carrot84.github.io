---
title: "[책] 만들면서 배우는 클린 아키텍처 - 9,10,11,12장"
excerpt_separator: "<!--more-->"
categories:
- book

tags:
- clean architecture
- 만들면서 배우는 클린 아키텍처
---

오늘은 지난 포스팅에 이어서 [**만들면서 배우는 클린 아키텍처**](https://search.shopping.naver.com/book/catalog/32445096226?cat_id=50010921&frm=PBOKPRO&query=%ED%81%B4%EB%A6%B0+%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98&NaPm=ct%3Dm563nl00%7Cci%3Dec22aa8a6e7e0fbfe9cf535d0e581d8e0fcc930b%7Ctr%3Dboknx%7Csn%3D95694%7Chk%3D9d122eaa1d5795ef48b7d85e6cf5ff1f476412ed) 에 대해 정리해둔 내용을 계속 포스팅 하려고 합니다.

이번 포스팅에서는 9,10,11,12장 내용을 다루고자 합니다.
> 09장 애플리케이션 조립하기  
> 10장 아키텍처 경계 강제하기  
> 11장 의식적으로 지름길 사용하기  
> 12장 아키텍처 스타일 결정하기    
<!--more-->

## 09장 애플리케이션 조립하기

### 왜 조립까지 신경 써야 할까?
- 유스케이스와 어댑터를 그냥 필요할 때 인스턴스화하면 안 될까?
  - 모든 의존성은 안쪽(도메인 코드 방향)으로 향해야, 도메인 코드가 바깥 계층의 변경으로부터 안전
- 유스케이스는 인터페이스만 알아야 하고, 런타임에 이 인터페이스의 구현을 제공 받아야 한다.
- 코드를 훨씬 더 테스트하기 쉽다는 점
  - 클래스가 필요로 하는 모든 객체를 생성자로 전달할 수 있다면 실제 객체 대신 mock 으로 전달 가능
    - 격리된 단위 테스트를 생성하기 쉬워진다.
- 아키텍처에 대해 중립적이고 인스턴스 생성을 위해 모든 클래스에 대한 의존성을 가지는 설정 컴포넌트가 있어야 한다는 것
  - 우리가 제공한 조각들로 애플리케이션을 조립 하는 것을 책임진다.

### 평범한 코드로 조립하기
- 평범함 코드 방식의 단점
  - 완전한 enterprise application 을 실행하기 위해 만들어야 하는 코드 양이 너무 많다.
  - 각 클래스가 속한 패키지 외부에서 인스턴스 생성 모든 클래스가 public
    - 유스케이스가 영속성 어댑터에 직접 접근하는 것을 막지 못한다.

### 스프링의 클래스패스 스캐닝으로 조립하기
- 클래스패스에서 접근 가능한 모든 클래스를 확인해서 @Component 애너테이션이 붙은 클래스를 찾는다.
  - 이 애너테이션이 붙은 각 클래스의 객체를 생성
  - 클래스들의 인스턴스를 만들어 application context 에 추가
- 클래스에 프레임워크에 특화된 애너테이션을 붙여야 한다는 점에서 침투적이다.
  - 특정한 프레임워크와 결합
  - 다른 개발자들이 사용할 라이브러리나 프레임워크를 만드는 입장에서는 기피해야 한다.
- 예상하지 못한 side-effect 가 발생할 수 있다.

### 스프링의 자바 컨피그로 조립하기
- application context 에 추가할 빈을 생성하는 설정 클래스를 만든다.
- @Configuration, @Bean 애너테이션을 통해 생성

### 유지보수 가능한 소프트웨어를 만드는데 어떻게 도움이 될까?
- 스프링과 스프링부트는 개발을 편하게 만들어주는 다양한 기능들을 제공한다.
  - 클래스패스 스캐닝은 아주 편리한 기능
- 어떤 빈이 application context 에 올라오는지 정확히 알 수 없다.
  - 테스트에서 application context 의 일부만 독립적으로 띄우기가 어려워진다.
- application 조립을 책임지는 전용 설정 컴포넌트를 만들면 application 이 이런 책임으로부터 자유로워진다.

## 10장 아키텍처 경계 강제하기
### 경계와 의존성
- 각 계층 사이, 안쪽 인접 계층과 바깥쪽 인접 계층 사이에 경계가 있다.
- 의존성 규칙에 따르면 계층 경계를 넘는 의존성은 항상 안쪽 방향으로 향해야 한다.

### 접근 제한자
- 자바 패키지를 통해 클래스들을 응집적인 **'모듈'**로 만들어 주기 때문이다.
  - package-private 제한자는 모듈내에 있는 클래스들을 서로 접근가능하지만, 패키지 바깥에서는 접근할 수 없다.
  - 모듈의 진입점으로 활용될 클래스들만 골라서 public 으로 만들면 된다.
  - 의존성이 잘못된 방향을 가리켜서 의존성 규칙을 위반할 위험이 줄어든다.
- 몇 개 정도의 클래스로만 이뤄진 작은 모듈에서 가장 효과적
  - 패키지 내의 클래스가 특정 개수를 넘어가기 시작하면 하나의 패키지에 너무 많은 클래스를 포함하는 것이 혼란스럽다.
  - 하위 패키지를 만들면 다른 패키지로 취급하기 때문에 package-private 멤버에 접근 X

### 컴파일 후 체크
- 의존성 규칙을 위반했는지 확인을 위해 컴파일된 후 런타임에 체크한다.
- ArchUnit 을 통해 의존성 방향이 기대한 대로 잘 설정돼 있는지 체크
  - Junit 과 같은 단위 테스트 프레임워크 기반에서 가장 잘 동작하며 의존성 규칙을 위반할 경우 테스트를 실패시킨다.
- 언제나 코드와 함께 유지보수해야 한다 (refactoring 등)

### 빌드 아티팩트
- 각 모듈 혹은 계층에 대해 전용 코드 베이스와 빌드 아티팩트로 분리된 빌드 모듈을 만들 수 있다.
  - 각 모듈의 빌드 스크립트에서는 아키텍처에서 허용하는 의존성만 지정한다.
- 빌드 모듈로 아키텍처 경계를 구분하는 것이 장점
  - 순환 의존성을 허용하지 않는다.
  - 특정 모듈의 코드를 격리한 채로 변경할 수 있다.
    - 특정 모듈의 컴파일러 에러 때문에 빌드가 실패 나는 것을 막을 수 있다.
  - 모듈 간 의존성이 빌드 스크립트에 선언돼 있기 때문에 새로 의존성 추가 시 의식적인 행동이 된다.
- 빌드 스크립트를 유지보수하는 비용을 수반

### 유지보수 가능한 소프트웨어를 만드는데 어떻게 도움이 될까?
- 소프트웨어 아키텍처는 아키텍처 요소 간의 의존성을 관리하는 게 전부다.
- 새로운 코드를 추가하거나 리팩토링할 때 패키지 구조를 항상 염두
  - package-private 가시성을 이용해 패키지 바깥에서 접근하면 안되는 클래스에 대한 의존성을 피해야 한다.
- 하나의 빌드 모듈안에서 아키텍처 경계를 강제, package-private 제한자를 사용할 수 없다면 ArchUnit 같은 컴파일 후 체크 도구를 이용
- 아키텍처가 충분히 안정적 -> 아키텍처 요소를 독립적인 모듈로 추출
  - 의존성을 분명하게 제어

## 11장 의식적으로 지름길 사용하기
### 왜 지름길은 깨진 창문 같을까?
- 품질이 떨어진 코드에서 작업할 때 더 낮은 품질의 코드를 추가하기가 쉽다.
- 코딩 규칙을 많이 어긴 코드에서 작업할 때 또 다른 규칙을 어기기도 쉽다.

### 깨끗한 상태로 시작할 책임
- 가능한 한 지름길을 거의 쓰지 않고 기술 부채를 지지 않은 채로 프로젝트를 깨끗하게 시작하는 것이 중요
- 때때로 지름길을 취하는 것이 더 실용적인 때도 있다.
  - 프로젝트 전체로 봤을때 그리 중요하지 않은 경우
  - 프로토타이핑 작업 중
  - 경제적 이유

### 유스케이스간 모델 공유하기
- 유스케이스들이 기능적으로 묶여 있을때 유효
  - 특정 세부사항을 변경할 경우 실제로 두 유스케이스 모두에 영향을 주고 싶은 경우

### 도메인 엔티티를 입출력 모델로 사용하기
- 간단한 생성이나 업데이트 유스케이스에서는 도메인 엔티티가 있는 것도 괜찮을지도 모른다.
  - 복잡한 도메인 로직 구현 시 전용 입출력 모델 사용 권장
  - 유스케이스의 변경이 도메인 엔티티까지 전파되길 바라지 않는다.
- 많은 유스케이스가 간단한 생성 또는 업데이트 유스케이스로 시작, 시간이 지나면 복잡한 도메인 로직 괴물이 되어간다.(특히 애자일 환경)

### 인커밍 포트 건너뛰기
- 아웃고잉 포트는 어플리케이션 계층과 아웃고잉 어댑터 사이의 의존성 역전을 위한 필수 요소
- 인커밍 포트는 어플리케이션 중심에 접근하는 진입점을 정의
  - 이를 제거하면 특정 유스케이스를 구현하기 위해 어떤 서비스 메서드를 호출해야 할지 알기 위해 어플리케이션의 내부 동작에 대해 더 잘 알아야 한다.
  - 인커밍 포트를 유지하면 한 눈에 진입점을 식별할 수 있다.
- 아키텍처를 쉽게 강제할 수 있다.
  - 인커밍 어댑터가 인커밍 포트만 호출하게 할 수 있다. (실수 방지)

### 애플리케이션 서비스 건너뛰기
- 아웃고잉 어댑터에 있는 클래스는 직접 인커밍 포트를 구현 -> 어플리케이션 서비스를 대체한다.
  - 인커밍 어댑터와 아웃고잉 어댑터 사이에 모델을 공유해야 한다.(도메인 모델 공유)
  - 시간이 지나 유스케이스가 점점 복잡해지면 도메인 로직을 그대로 아웃고잉 어댑터에 추가하고 싶은 생각이 들것이다.
  - 도메인 로직이 흩어져서 도메인 로직을 찾거나 유지보수하기가 어려워진다.

### 유지보수 가능한 소프트웨어를 만드는데 어떻게 도움이 될까?
- 모든 어플리케이션은 처음에는 작게 시작
  - 유스케이스가 단순한 CRUD 상태에서 벗어나는 시점이 언제인지에 대해 팀이 함의하는 것이 매우 중요
- 어떤 경우든 아키텍처에 대해, 그리고 왜 특정 지름길을 선택했는가에 대한 기록을 남겨두자.

## 12장 아키텍처 스타일 결정하기
### 도메인이 왕이다.
- 외부의 영향을 받지 않고 도메인 코드를 자유롭게 발전시킬 수 있다는 것은 육각형 아키텍처 스타일이 내세우는 가장 중요한 가치다.
  - 육각형 아키텍처 스타일이 도메인 주도 설계 방식과 정말 잘 어울리는 이유다.
- 도메인을 중심에 두는 아키텍처 없이는, 또 도메인 코드를 향한 의존성을 역전시키지 않고서는 DDD 를 제대로 할 가능성이 없다.
- 도메인 코드가 어플리케이션에서 가장 중요한 것이 아니라면 이 아키텍처 스타일은 필요하지 않을 것이다.

### 경험이 여왕이다.
- 습관이 저절로 결정을 내리기 때문에 우리는 무언가를 결정할 때 시간을 들일 필요가 없다.
- 아키텍처 스타일에 대해서 괜찮은 결정을 내리는 유일한 방법은 다른 아키텍처 스타일을 경험해 보는 것
  - 어플리케이션의 작은 모듈에 먼저 시도해 보라.

### 마무리

지난 포스팅 이후 한참 동안 신경을 못 쓰다가 오늘 해당 책에 관련해 마지막 포스팅을 올리게 되었네요.

아직 새로운 회사에서 많은 개발건을 처리하고 있지 않아서, 실무적인 경험을 많이 하지는 못했지만, 
클린 아키텍처를 사용했을 때 장점이 뭔지 왜 사용해야 하는지 저 스스로도 깨우치는 날이 오면 좋겠다 싶네요.

그럼 이만. 🥕👋🏼🖐🏼

### 참고자료🤣
[만들면서 배우는 클린 아키텍처](https://search.shopping.naver.com/book/catalog/32445096226)
