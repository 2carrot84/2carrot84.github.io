---
title: "@RequiredArgsConstructor 와 @Qualifier 사용시 주의점"
excerpt_separator: "<!--more-->"
categories:
- development

tags:
- RequiredArgsConstructor 
- Qualifier
- ElasticSearch
- Spring
---

팀 자체 개선 프로젝트 진행 중 트러블슈팅을 하면서 알게된 내용을 포스팅 하고자 합니다.

이론상으로는 아무 문제 없이 구현이 되었어야할 이슈로 이틀간 삽질을 하여, 저 같은 삽질을 하지 않았으면 하는 마음에 포스팅을 합니다.
<!--more-->

### 구현 내용
현재 프로젝트에서 ElasticSearch(ES) 에 통계 데이터를 저장하여, 조회를 하고 있으나, 추가적인 정보 조회를 위해 ES 의 EndPoint 를 추가해야 할 일이 생겼습니다.

ES 접속을 위해서는 아래와 같이 RestHighLevelClient 객체를 빈으로 등록하여, 필요한 Service 단에서 사용을 하고 있었습니다.

```java
	@Bean
	public RestHighLevelClient restHighLevelClient() {
		return new RestHighLevelClient(
			RestClient.builder(esSearchProperties.hosts())
				.setHttpClientConfigCallback(
					httpClientBuilder -> httpClientBuilder.setDefaultIOReactorConfig(
						IOReactorConfig.custom().setSoKeepAlive(true).build()
					)
				)
		);
	}
```

EndPoint 를 추가하기 위해서 yaml 파일에 새로운 EndPoint Host 항목과 esSearchProperties 클래스 추가하고, RestHighLevelClient 객체를 리턴 하는 빈을 추가로 생성하였습니다.

### 첫번째 이슈 발생

위 구현을 끝내고 서버를 기동하니, 아래와 같은 에러가 발생하였습니다.

```shell
***************************
APPLICATION FAILED TO START
***************************

Description:

Parameter 0 of constructor in xxx required a single bean, but 2 were found:
	- restHighLevelClient: defined by method 'restHighLevelClient' in class path resource [xxx/ESSearchConfig.class]
	- restHighLevelClient2: defined by method 'restHighLevelClient2' in class path resource [xxx/ESSearchConfig.class]


Action:

Consider marking one of the beans as @Primary, updating the consumer to accept multiple beans, or using @Qualifier to identify the bean that should be consumed
```

`같은 타입의 객체를 리턴하는 빈이 2개가 생기다 보니, 어떤 빈을 주입 해야할지 모른다는 내용`이었고, 해당 이슈가 발생할 것을 예상하여, 빈의 method 명으로 명시를 해줬으나, 소용이 없는 것 같았습니다.

에러 메시지 안내와 같이 `@Primary 어노테이션을 사용`해 에러가 발생하지 않도록 조치를 하였고, 테스트를 진행 하였으나, 아래와 같은 이슈가 발생 하였습니다.

### 두번째 이슈 발생
```java
// 빈 생성 부
	@Bean
	@Primary
	public RestHighLevelClient restHighLevelClient() {
    ...
	}

	@Bean
	public RestHighLevelClient restHighLevelClient2() {
    ...
	}
    
// 빈 사용 부
@Service
@RequiredArgsConstructor
public class XXService {

    private final RestHighLevelClient restHighLevelClient2;
    ...
}

```

위와 같이 구현하여 빈을 주입 시켜봤지만, 2번째 빈이 아닌 @Primary 가 붙은 빈이 주입이 되는 현상이 발생하였고, 이를 해결 하기 위해 `@Qualifier 어노테이션을 사용`하여, 두번째 빈을 명확하게 명시해주기로 했습니다.

### 세번째 이슈 발생
```java
// 빈 생성 부
	@Bean
	@Primary
	public RestHighLevelClient restHighLevelClient() {
    ...
	}

	@Bean
	@Qualifier("restHighLevelClient2")
	public RestHighLevelClient restHighLevelClient2() {
    ...
	}
    
// 빈 사용 부
@Service
@RequiredArgsConstructor
public class XXService {
    @Qualifier("restHighLevelClient2")
    private final RestHighLevelClient restHighLevelClient2;
    ...
}

```

위와 같이 구현 후 당연히 모든 이슈가 해결 되었을꺼라고 생각하고 테스트를 해보았지만, 2번째 빈이 주입되지 않는 현상은 해소가 되지 않았다.

주변 개발자분 들에게도 현상에 대해 얘기를 하고 같이 원인을 찾아봤지만 쉽게 원인이 밝혀지지 않았다.

그러던 중 전혀 다른 방향에서 원인을 찾게 되었다.

### 원인

원인은 Qualifier 사용법이 잘못된 것이 아니라, @RequiredArgsConstructor 어노테이션 사용이 원인이였다.

Lombok 에서 @RequiredArgsConstructor 어노테이션을 인지하여, 생성자를 자동으로 생성 해주나, @Qualifier 어노테이션을 생성자 파라미터에 붙혀주지는 않는것 이였습니다.

그렇기 때문에 위와 같이 @Qualifier 를 명시했지만, 자동 생성된 생성자에는 @Qualifier 가 없는 코드가 생성되어, @Primary 가 붙은 빈이 주입되는 현상이 발생하는 것이 였습니다.

### 해결 방법

인터넷에 몇가지 해결 방법이 있어서 진행 해봤지만, 저는 해결이 안되는 케이스도 있어 아래 방법으로 해결 하였습니다. 

@RequiredArgsConstructor 어노테이션을 사용하는 대신 직접 생성자를 만들고, 

@Qualifier 를 붙혀주었더니 제가 원하는 대로 2번째 빈이 주입되는 것을 확인할 수 있었습니다.

```java
@Service
public class XXService {

    private final RestHighLevelClient restHighLevelClient2;

    public XXService(@Qualifier("restHighLevelClient2") RestHighLevelClient restHighLevelClient2) {
        this.restHighLevelClient2 = restHighLevelClient2;
    }
    ...
}
```

### 마무리
위 이슈를 겪고 해결하면서 습관처럼 사용하는 코드들에 대해서도 제가 생각하지 못한 문제를 야기시킬 수 있다는 생각을 늘 품고 의심해봐야 한다는 점을 잊지 말아야 겠다고 생각했습니다.

코드를 간단히 만들어 주던 @RequiredArgsConstructor 어노테이션이 이런 오류를 유발시킬 줄은 생각 못했죠. 

그럼 이만. 🥕👋🏼🖐🏼

### 참고자료🤣
[@Qualifier와 @Primary 어노테이션 사용법](https://bestinu.tistory.com/58)  
[@RequiredArgsConstructor과 @Qualifier 같이 사용 시 이슈 해결법](https://www.podo-dev.com/blogs/224)