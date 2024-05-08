---
title: "kotlin + Swagger 3.0 + spring boot 3.x 세팅"
excerpt_separator: "<!--more-->"
categories:
- development

tags:
- kotlin 
- Swagger
- Spring
- SpringBoot
---

지난번 포스팅에 이어 kotlin 프로젝트 세팅 과정 중 저를 힘들게한 swagger 설정 부분을 정리 하려고 합니다.
<!--more-->

지난번 프로젝트때는 손쉽게 설정했단 swagger 설정으로 꽤나 많은 시간을 보내 다음번 또는 다른 분들은 저같은 삽질을 덜 했으면 하는 마음에 설정 부분을 포스팅 합니다. 

### 구현 내용

신규 프로젝트인 만큼 Swagger 3.0 버전을 사용하자는 의견이 있어, 구글링을 하며 세팅을 해보았습니다. 

삽질의 포인트는 spring boot 버전과 그에 맞는 swagger 버전 세팅이 젤 이슈 였던 것 같습니다.

우선 최종 세팅된 버전을 공유 하고, 삽질 과정을 공유 하도록 하겠습니다.


#### build.gradle.kts
```groovy
dependencies {
    ...
    // Swagger
    implementation("org.springdoc:springdoc-openapi-starter-webmvc-ui:2.3.0")
    ...
}
```
#### config
```kotlin
import io.swagger.v3.oas.models.Components
import io.swagger.v3.oas.models.OpenAPI
import io.swagger.v3.oas.models.info.Info
import org.springframework.context.annotation.Bean
import org.springframework.context.annotation.Configuration

@Configuration
class SwaggerConfig {
    @Bean
    fun openAPI(): OpenAPI {
        return OpenAPI()
            .components(Components())
            .info(configurationInfo())
    }

    private fun configurationInfo(): Info {
        return Info()
            .title("타이틀")
            .description("상세 설명 Swagger")
            .version("1.0.0")
    }
}
```
우선 위 2가지 의존성 추가와 Config 설정을 통해 [http://localhost:8080/swagger-ui/index.html](http://localhost:8080/swagger-ui/index.html) 로 UI 가 뜬다면, 설정은 성공한 것입니다.

### 첫번째 이슈 발생

첫번째 이슈는 의존성 추가와 어노테이션을 어떤걸 해야 하는지 였습니다. 

블로그 글대로 세팅을 해봤지만, 계속 404 에러가 발생하여 여러 글을 보다가 아래 글을 보게 되었습니다.

[Springboot 3.x에 Swagger를 적용시켜보자!](https://velog.io/@kjgi73k/Springboot3%EC%97%90-Swagger3%EC%A0%81%EC%9A%A9%ED%95%98%EA%B8%B0)  

블로그를 보시면 여러 가지 세팅을 하며 삽질을 해나가는 과정을 아주 자세히 기록해두셨으니, 참고 부탁드립니다.

```groovy
    implementation("org.springdoc:springdoc-openapi-starter-webmvc-ui:2.3.0")
```

위 블로그 대로 설정 의존성 추가 후 swagger 페이지가 드디어 떴습니다!

기존에 사용하던 `Springfox 대신 Springdoc 을 이용하여 세팅`을 해야 한다는점 유의 하시기 바랍니다.

### 두번째 이슈 발생

어렵게 첫번째 이슈를 해결 하였으나, 아래와 같은 화면이 떴습니다.

![Error](/images/posts/2024/05/image.png)

또 열심히 구글링을 해본 결과 아래와 같은 글을 보고 뭔가 의존성에 이슈가 있을꺼 같다는 생각으로 다른 예제를 조금 더 찾아보았습니다.

[[SpringBoot/Swagger] Failed to load API definition.](https://blog.naver.com/4off4/223132968907)

다른 예제를 보니 아래 의존성을 같이 사용하는게 있어 추가를 했더니 드디어 swagger 페이지가 정상적으로 구동 되었습니다.

```groovy
	implementation("org.springdoc:springdoc-openapi-starter-common:2.3.0")
```

### 그 후 삽질

성공에 기쁨에 찬 후 아래 부분 텍스트를 프로젝트에 맞게 변경하려고 하였으나, 변경이 되지 않는 부분이 발견 되었습니다.

![Error](/images/posts/2024/05/image2.png)

그 당시 Config 파일은 아래와 같이 세팅이 되어 있었고, 이상하다는 생각에 다시 이전 블로그 글들을 읽어 보며 Config 와 의존성 추가가 잘 못 된 것을 발견 하게 되었고,

위에 공유 드린 최종 코드의 형태로 정리를 하게 되었습니다.

```groovy
	// Swagger
	implementation("io.springfox:springfox-boot-starter:3.0.0")
	implementation("org.springdoc:springdoc-openapi-starter-webmvc-ui:2.3.0")
	implementation("org.springdoc:springdoc-openapi-starter-common:2.3.0")
```

```kotlin
import org.springframework.context.annotation.Bean
import org.springframework.context.annotation.Configuration
import org.springframework.web.servlet.config.annotation.EnableWebMvc
import springfox.documentation.builders.ApiInfoBuilder
import springfox.documentation.builders.PathSelectors
import springfox.documentation.builders.RequestHandlerSelectors
import springfox.documentation.spi.DocumentationType
import springfox.documentation.spring.web.plugins.Docket


@Configuration
@EnableWebMvc
class SwaggerConfig {
    @Bean
    fun swaggerApi(): Docket = Docket(DocumentationType.OAS_30)
        .consumes(getConsumeContentTypes())
        .produces(getProduceContentTypes())
        .apiInfo(swaggerInfo())
        .select()
        .apis(RequestHandlerSelectors.basePackage("com.colabear754.swagger_example.controllers"))
        .paths(PathSelectors.any())
        .build()
        .useDefaultResponseMessages(false)

    private fun swaggerInfo() = ApiInfoBuilder()
        .title("스웨거 테스트")
        .description("스웨거로 API를 테스트")
            .version("1.0.0")
        .build()

    private fun getConsumeContentTypes(): Set<String> {
        val consumes = HashSet<String>()
        consumes.add("multipart/form-data")
        return consumes
    }

    private fun getProduceContentTypes(): Set<String> {
        val produces = HashSet<String>()
        produces.add("application/json;charset=UTF-8")
        return produces
    }
}
```

즉, 2번째 이슈는 springfox 의존성과 구버전 config 설정에 의해 나는 에러 였으며 그걸 해결 하기 위해 불필요한 의존성을 추가하여 해결을 하는 과정을 거친 것이 였습니다.

이렇게 또 한번 반성의 시간을 가지며, 깔끔히 정리를 한 후 이 포스팅을 남깁니다.

### 마무리

이렇게 여러 블로그의 글을 읽으며 세팅을 하다 보니 내용이 섞여서 불필요한 설정이 포함될 수 있겠다 라는걸 느꼈으며, 원하던 결과를 얻었더라도 한번 복기 하는 마음으로 다시 해보는 습관을 가져야 겠다고 생각했습니다.

swagger 3 어노테이션 관련 해서는 마지막 참고 자료를 참고 하시면 될 것 같습니다.

다소 틀린 내용이 있을 수 있으니, 발견하시는 분은 댓글 부탁드립니다.

그럼 이만. 🥕👋🏼🖐🏼

### 참고자료🤣
[Springboot 3.x에 Swagger를 적용시켜보자!](https://velog.io/@kjgi73k/Springboot3%EC%97%90-Swagger3%EC%A0%81%EC%9A%A9%ED%95%98%EA%B8%B0)  
[[Spring Boot] Springdoc 라이브러리를 통한 Swagger 적용](https://colabear754.tistory.com/99#Springfox%EC%99%80%20Springdoc%EB%8A%94%20%EB%AC%B4%EC%97%87%EC%9D%B4%20%EB%8B%A4%EB%A5%B8%EA%B0%80?)    
[[SpringBoot/Swagger] Failed to load API definition.](https://blog.naver.com/4off4/223132968907)
[DynamoDB Enhanced Client for Java: Missing Setters Cause Misleading Error or Unexpected Behavior](https://davidagood.com/dynamodb-enhanced-client-java-missing-setters/)
[Swagger 3.x 어노테이션 정리](https://velog.io/@mj3242/Swagger-3.x-%EC%96%B4%EB%85%B8%ED%85%8C%EC%9D%B4%EC%85%98-%EC%A0%95%EB%A6%AC)