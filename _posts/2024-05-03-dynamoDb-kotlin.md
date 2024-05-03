---
title: "kotlin 을 이용한 DynamoDB 조회 방법 "
excerpt_separator: "<!--more-->"
categories:
- development

tags:
- kotlin 
- DynamoDB
- Spring
- SpringBoot
---

작년 한해 동안 만든 자체 프로젝트 언어를 Java 에서 Kotlin 으로 변환하기로 하여, 샘플 코드를 만들면서 삽질한 내용을 포스팅 해봅니다.

Kotlin 의 새로운 문법에 대한 적응도 쉽지 않은 상태에서, DynamoDB 조회 로직을 구현하면서 겪은 내용이며, 저와 같은 삽질의 시간을 보내지 않았으면 하는 마음에 기록을 남겨두니 많은 도움이 되면 좋겠습니다.
<!--more-->

현재 맡고 있는 전시 업무 중 가장 기본이 되는 단위인 모듈 데이터가 DynamoDB에 위치하여, kotlin 프로젝트를 신규 생성하여, 모듈번호를 이용하여 모듈을 조회하는 로직과 검증 로직을 만들어 테스트 하고 싶었습니다.

Java 에서 kotlin 전환은 간단하다는 얘기를 많이 듣고, kotlin 강의도 듣고 처음 세팅을 시작 하였으나, 처음부터 다양한 어려움을 겪게 되었습니다.

구현한 예제 코드를 아래 첨부 드리니 참고 부탁드립니다.

### 구현 내용

여러 레퍼런스를 찾아 보던 중 kotlin 을 위한 AWS SDK 와 Java 용 AWS SDK 2가지 방안을 테스트 삼아 구연해 보았습니다.

#### build.gradle.kts
```groovy
dependencies {
    ...
    // AWS SDK Kotlin
    implementation("aws.sdk.kotlin:dynamodb-jvm:1.2.3")
    implementation("com.squareup.okhttp3:okhttp:5.0.0-alpha.11")

    // AWS Java SDK V2
    implementation("software.amazon.awssdk:dynamodb-enhanced")
    implementation("io.reactivex.rxjava2:rxjava:2.2.21")
    ...
}
dependencyManagement {
    imports {
        // AWS Java SDK V2
        mavenBom("software.amazon.awssdk:bom:2.15.22")
    }
}
```
#### config
```kotlin
import aws.sdk.kotlin.runtime.auth.credentials.DefaultChainCredentialsProvider
import aws.sdk.kotlin.services.dynamodb.DynamoDbClient
import org.springframework.context.annotation.Bean
import org.springframework.context.annotation.Configuration

// AWS SDK Kotlin
@Configuration
class DynamoDbClientProvider {
    @Bean
    fun client() = DynamoDbClient {
        region = "ap-northeast-2"
        credentialsProvider = DefaultChainCredentialsProvider()
    }
}
```

```kotlin
import org.springframework.context.annotation.Bean
import org.springframework.context.annotation.Configuration
import software.amazon.awssdk.auth.credentials.DefaultCredentialsProvider
import software.amazon.awssdk.enhanced.dynamodb.DynamoDbEnhancedAsyncClient
import software.amazon.awssdk.regions.Region
import software.amazon.awssdk.services.dynamodb.DynamoDbAsyncClient

// AWS Java SDK V2
@Configuration
class DynamoDBConfig {
    @Bean
    fun dynamoDbAsyncClient(): DynamoDbAsyncClient {
        return DynamoDbAsyncClient.builder()
            .region(Region.AP_NORTHEAST_2)
            .credentialsProvider(DefaultCredentialsProvider.builder().build())
            .build()
    }

    @Bean
    fun dynamoDbEnhancedAsyncClient(): DynamoDbEnhancedAsyncClient {
        return DynamoDbEnhancedAsyncClient.builder()
            .dynamoDbClient(dynamoDbAsyncClient())
            .build()
    }
}
```
#### DTO
> DTO 구현에 사용된 Annotation 들은 AWS Java SDK V2 을 위한 것이며, AWS SDK Kotlin 버전에선 필요없으니 참고 부탁드립니다.

```kotlin
import software.amazon.awssdk.enhanced.dynamodb.mapper.annotations.DynamoDbAttribute
import software.amazon.awssdk.enhanced.dynamodb.mapper.annotations.DynamoDbBean
import software.amazon.awssdk.enhanced.dynamodb.mapper.annotations.DynamoDbPartitionKey
import software.amazon.awssdk.enhanced.dynamodb.mapper.annotations.DynamoDbSortKey

@DynamoDbBean
class DisplayModuleDTO(
    @get:DynamoDbPartitionKey
    var hk: String? = null,
    @get:DynamoDbSortKey
    var sk: String? = null,
    @get:DynamoDbAttribute("dcorn_no")
    var dcornNo: String? = null,
    @get:DynamoDbAttribute("dcorn_id")
    var dcornId: String? = null
)
```

#### Repository
```kotlin
import aws.sdk.kotlin.services.dynamodb.DynamoDbClient
import aws.sdk.kotlin.services.dynamodb.model.AttributeValue
import aws.sdk.kotlin.services.dynamodb.model.QueryRequest
import io.reactivex.Flowable
import kotlinx.coroutines.runBlocking
import org.example.kotlin_test.repository.dto.DisplayModuleDTO
import org.springframework.stereotype.Repository
import software.amazon.awssdk.enhanced.dynamodb.DynamoDbEnhancedAsyncClient
import software.amazon.awssdk.enhanced.dynamodb.Expression
import software.amazon.awssdk.enhanced.dynamodb.Key
import software.amazon.awssdk.enhanced.dynamodb.TableSchema
import software.amazon.awssdk.enhanced.dynamodb.model.QueryConditional
import software.amazon.awssdk.services.dynamodb.model.AttributeValue as A

@Repository
class TestRepository(
    private val client: DynamoDbClient,
    private val dynamoDbAsyncClient: DynamoDbEnhancedAsyncClient
) {
    companion object {
        const val TABLE_NAME = "dp_dp"
        private val tableSchema = TableSchema.fromBean(DisplayModuleDTO::class.java)
    }

    private val table = dynamoDbAsyncClient.table(TABLE_NAME, tableSchema)

    // AWS SDK Kotlin
    fun findModuleByDcornNo(dcornNo: String): List<DisplayModuleDTO> {
        val list = mutableListOf<DisplayModuleDTO>()

        runBlocking {
            val attr = mutableMapOf<String, AttributeValue>()
            attr[":hk"] = AttributeValue.S("DCORN$${dcornNo}")
            attr[":sk"] = AttributeValue.S("DCORN")

            val request = QueryRequest {
                tableName = TABLE_NAME
                keyConditionExpression = "hk = :hk AND sk = :sk"
                expressionAttributeValues = attr
            }
            client.use { ddb ->
                ddb.query(request).items?.forEach {
                    list.add(
                        DisplayModuleDTO(
                            hk = it.getValue("hk").asS(),
                            sk = it.getValue("sk").asS(),
                            dcornNo = it.getValue("dcorn_no").asS(),
                            dcornId = it.getValue("dcorn_id").asS(),
                        )
                    )
                }
            }
        }

        return list
    }

    // AWS Java SDK V2
    fun findModule2ByDcornNo(dcornNo: String): List<DisplayModuleDTO> {
        val values = mutableMapOf<String, A>()
        values[":del_yn"] = A.builder().s("N").build()

        val expression = Expression.builder()
            .expression("del_yn = :del_yn")
            .expressionValues(values)
            .build()

        val queryConditional = QueryConditional
            .keyEqualTo(Key.builder().partitionValue("DCORN$${dcornNo}").sortValue("DCORN").build())

        val result = table.query { r ->
            r.queryConditional(queryConditional)
                .filterExpression(expression)
        }.items()

        return Flowable.fromPublisher(result).toList().blockingGet()
    }
}
```

#### Test
```kotlin
import org.assertj.core.api.Assertions.assertThat
import org.junit.jupiter.api.Test
import org.springframework.beans.factory.annotation.Autowired
import org.springframework.boot.test.context.SpringBootTest

@SpringBootTest
class TestRepositoryTest @Autowired constructor(
    private val testRepository: TestRepository
) {
    @Test
    fun findByDcornNoTest() {
        val modules = testRepository.findModuleByDcornNo("M000001")
        assertThat(modules.size).isGreaterThan(0)
        val module = modules.get(0)
        assertThat(module.dcornNo).isEqualTo("M000001")
        assertThat(module.dcornId).isEqualTo("b_image_banner_01")
    }

    @Test
    fun findByDcornNoTest2() {
        val modules = testRepository.findModule2ByDcornNo("M000001")
        assertThat(modules.size).isGreaterThan(0)
        val module = modules.get(0)
        assertThat(module.dcornNo).isEqualTo("M000001")
        assertThat(module.dcornId).isEqualTo("b_image_banner_01")
    }
}
```

위 코드는 테스트 성공까지 완료되었던 코드들이며, 이 코드를 작성하며 겪은 몇가지 이슈와 해결 방안에 대해 아래 작성하였습니다.


### 첫번째 이슈 발생

처음 AWS SDK Kotlin 방식으로 코드 작성 후 Test 코드를 돌렸을때 아래와 같은 에러 메시지가 발생하였습니다. 

```kotlin
import org.assertj.core.api.Assertions.assertThat
import org.junit.jupiter.api.Test
import org.springframework.boot.test.context.SpringBootTest

@SpringBootTest
class TestRepositoryTest (
    private val testRepository: TestRepository
) {
    @Test
    fun findByDcornNoTest() {
        val modules = testRepository.findModuleByDcornNo("M000001")
        assertThat(modules.size).isGreaterThan(0)
        val module = modules.get(0)
        assertThat(module.dcornNo).isEqualTo("M000001")
        assertThat(module.dcornId).isEqualTo("b_image_banner_01")
    }
}
```

```shell
org.junit.jupiter.api.extension.ParameterResolutionException: No ParameterResolver registered for parameter [org.example.kotlin_test.repository.TestRepository testRepository] in constructor [public org.example.kotlin_test.repository.TestRepositoryTest(org.example.kotlin_test.repository.TestRepository)].
```

에러 메시지에 나와 있듯이 TestRepository 를 이용해, TestRepositoryTest 클래스를 만들어야 하는데, TestRepository 파라미터를 위해 등록된 ParameterResolver 가 없다는 에러 였습니다.

해결법은 kotilne 클래스 명 뒤에 에 `@Autowired constructor` 를 붙혀주는 것으로 해결이 가능했습니다.
 
```kotlin
class TestRepositoryTest @Autowired constructor
```

제가 이해한 원인은 테스트 프레임워크에서의 생성자 매개변수 관리는 스프링 컨테이너가 아닌 Jupiter가 담당하기 때문에, @Autowired 를 명시하지 않으면 의존성 주입이 되지 않는다 였습니다.

해당 이슈에 대한 자세한 설명은 [JUnit 5 + Kotlin 테스트 클래스에서 생성자 주입 이슈](https://minkukjo.github.io/framework/2020/06/28/JUnit-23/) 에 자세히 설명 되어 있으니, 궁금하신 분들은 참고 부탁드립니다.

### 두번째 이슈 발생

첫번째 이슈를 간단히(?) 해결 하고 2번째 방법으로 구현을 다 하였으나, 계속 아래의 메시지가 발생 하였습니다.
```kotlin
import software.amazon.awssdk.enhanced.dynamodb.mapper.annotations.DynamoDbAttribute
import software.amazon.awssdk.enhanced.dynamodb.mapper.annotations.DynamoDbBean
import software.amazon.awssdk.enhanced.dynamodb.mapper.annotations.DynamoDbPartitionKey
import software.amazon.awssdk.enhanced.dynamodb.mapper.annotations.DynamoDbSortKey

@DynamoDbBean 
class DisplayModuleDTO(
    @get:DynamoDbPartitionKey
    val hk: String? = null,
    @get:DynamoDbSortKey
    val sk: String? = null,
    @get:DynamoDbAttribute("dcorn_no")
    var dcornNo: String? = null,
    @get:DynamoDbAttribute("dcorn_id")
    var dcornId: String? = null
)
```
```shell
java.lang.IllegalArgumentException: Attempt to execute an operation that requires a primary index without defining any primary key attributes in the table metadata.
```
열심히 구글링을 해본 결과 [DynamoDB Enhanced Client for Java: Missing Setters Cause Misleading Error or Unexpected Behavior](https://davidagood.com/dynamodb-enhanced-client-java-missing-setters/) 찾게 되었고 아래 문구에 대해 이해가 되지 않았습니다.

> Make sure every DynamoDB attribute on your entity has a publicly-accessible setter with the same name (case-sensitive) as the getter.

Java 와 달리 kotlin 에서는 필드만 만들면 Lombok 을 사용하지 않아도 getter/setter 를 자동으로 생성해 준다고 알고 있는데, 왜 이런 이슈가 발생하는지 이해가 되지 않았습니다.

결국 DTO 클래스를 Decompile 해보자고 생각을 했고, 아래 방법으로 Decompile 해본 결과 이마를 치며 깨달았습니다.

> Tools > Kotlin > Show Kotlin Bytecode 선택 > Decompile 버튼 클릭
 
Decompile 된 파일에는 setter 가 PartitionKey 로 지정한 필드에 대한 setter 메소드가 없었습니다 이유는 바로 PartitionKey 와 SortKey 가 null 일리가 없으니, `val` 로 설정한게 문제였습니다.

불변 객체다 보니, 생성자를 주입이 아닌 경우 set 을 할 수 없기 때문에 당연히 setter 메소드가 자동 생성되지 않는 것을 생각하지 못한 kotlin 초보의 실수 였습니다.

### 마무리
이렇게 크고 작은 이슈들을 겪으며 2가지 방법 모두 Test 에 성공하였습니다.

해당 샘플 코드를 보시면 아시겠지만 비동기 모듈을 사용해 구현되어 앞으로 많은 숙제를 남겨줄 것 같은 느낌입니다..

개발자로 일을 시작하고, 실무에서 Java 가 아닌 새로운 언어를 사용하는 것은 처음이라 많이 기대하며 진행한 샘플 코드 작성은 자책과 반성만 남기게 되었습니다.

그리고 역시 실제로 개발을 해보지 않으면, 그 언어를 안다고 하면 안되는구나 느꼈습니다.

다소 틀린 내용이 있을 수 있으니, 발견하시는 분은 댓글 부탁드립니다.

그럼 이만. 🥕👋🏼🖐🏼

### 참고자료🤣
[JUnit 5 + Kotlin 테스트 클래스에서 생성자 주입 이슈](https://minkukjo.github.io/framework/2020/06/28/JUnit-23/)  
[[Spring Boot] DynamoDB](https://dico.me/back-end/articles/343/ko)  
[AWS Java SDK V2](https://betterprogramming.pub/aws-java-sdk-v2-dynamodb-enhanced-client-with-kotlin-spring-boot-application-f880c74193a2)   
[DynamoDB Enhanced Client for Java: Missing Setters Cause Misleading Error or Unexpected Behavior](https://davidagood.com/dynamodb-enhanced-client-java-missing-setters/)