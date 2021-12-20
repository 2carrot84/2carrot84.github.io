---
title: "UriComponentBuilder를 이용하여 쉽게 URI 만드는 법"
excerpt_separator: "<!--more-->"
categories:
  - development
tags:
  - spring
  - uri
---

회사에서 작업 중 URI 를 Map 의 Key, Value 로 동적으로 구성하는 코드를 작성하게 되었습니다.

별 생각없이 String 또는 StringBuffer 를 이용하여 코드를 작성 하였으나, 동료 개발자분의 코드 리뷰 후 깔끔해진 제 코드를 보고

내가 모르는 다양한 클래스들이 유용한 기능을 제공 해주고 있다는걸 또 한번 깨닫고 되도록 기록하기 위해 포스팅을 올립니다.
 
<!--more-->

먼저 들어가기 앞서 간략하게 URI 란 무엇인지? 더 자주 접하는 용어인 URL 은 무엇인지 알아보도록 하겠습니다.

### URI (Uniform Resource Identifier) 이란?

> 통합 자원 식별자는 인터넷에 있는 자원을 나타내는 유일한 주소이다.   
> URI 의 존재는 인터넷에서 요구되는 기본조건으로서 인터넷 프로토콜에 항상 붙어 다닌다. URI 의 하위개념으로 **URL, URN** 이 있다.

#### URL (Uniform Resource Locator) 

> 네트워크 상에서 자원이 어디 있는지를 알려주기 위한 규약이다.   
> 즉, 컴퓨터 네트워크와 검색 메커니즘에서의 위치를 지정하는, 웹 리소스에 대한 참조이다.   
> 흔히 웹 사이트 주소로 알고 있지만, URL 은 웹 사이트 주소뿐만 아니라 컴퓨터 네트워크상의 자원을 모두 나타낼 수 있다.   
> 그 주소에 접속하려면 해당 URL 에 맞는 프로토콜을 알아야 하고, 그와 동일한 프로토콜로 접속해야 한다.

URI 와 URL 의 차이에 대해서는 아래 그림과 링크를 보시면 이해에 도움이 될 것이라고 생각됩니다.

#### [참고링크](https://mygumi.tistory.com/139)
![참고그림](https://t1.daumcdn.net/cfile/tistory/2416C94158D62B9E11)  


### UriComponentBuilder 

UriComponentBuilder 는 Spring Framework 5.3.14 API 부터 사용되었으며, URI 를 만드는데 약간의 편리함과 명확함을 제공해 준다고 생각됩니다.

기존에 제가 작성한 코드와 UriComponentBuilder 를 사용한 코드를 예로 들어 간략히 설명해 보겠습니다.

#### AS-IS 코드
```java
    public String makeQueryString(Map parameterMap) {
        StringBuilder sb = new StringBuilder(this.httpUrl + "?1=1");
        parameterMap.forEach((k, v) -> sb.append("&").append(k).append("=").append(v));
        return sb.toString();
    }
```

#### TO-BE 코드
```java
    public String makeQueryString(Map parameterMap) {
        UriComponentsBuilder uriComponentsBuilder = UriComponentsBuilder.fromHttpUrl(this.httpUrl);
        queryParameterMap.forEach((key, value) -> {uriComponentsBuilder.queryParam(key,value)});
        return uriComponentsBuilder.encode().toUriString();
    }
```

소스 양으로 봤을때 크게 차이가 없지만, 동적으로 Query String 을 만들때 `제일 번거로운 ? 와 & 에 대한 고민을 하지 않아도 되는 점이 인상적`이였습니다.

?, & 처리를 위한 불필요한 소스 없이 깔끔한 모습일 볼 수 있습니다.

추가로 타 블로그에서 본 예제 소스를 통해 명확성에 대한 장점을 보도록 하겠습니다.


#### AS-IS 코드
```java
    @RequestMapping("/login/kakao")
    public void kakao(HttpServletResponse httpServletResponse) throws IOException {

        // 1안 String 사용
        String kakaoAuthUri = "https://kauth.kakao.com/oauth/authorize?client_id=insertYourId&redirect_uri=http://localhost:8080/oauth&response_type=code";
    
		// 2안 StringBuilder 사용
        StringBuilder sb = new StringBuilder();
        sb.append("https://");
        sb.append("kauth.kakao.com");
        sb.append("/oauth/authorize?");
        sb.append("client_id=insertYourId");
        sb.append("&");
        sb.append("redirect_uri=http://localhost:8080/oauth");
        sb.append("&");
        sb.append("response_type=code");
    
        httpServletResponse.sendRedirect(sb.toString());
    }
```

#### TO-BE 코드
```java
    @RequestMapping("/login/kakao")
    public void kakao(HttpServletResponse httpServletResponse) throws IOException {

        UriComponents uriComponents = UriComponentsBuilder.newInstance()
        .scheme("https")
        .host("kauth.kakao.com")
        .path("/oauth/authorize")
        .queryParam("client_id", "Insert your Id")
        .queryParam("redirect_uri", "http://localhost:8080/oauth")
        .queryParam("response_type", "code")
        .build(true);
    
        httpServletResponse.sendRedirect(uriComponents.toString());
    }
```

![URI 구성 요소](https://media.vlpt.us/images/jch9537/post/88b0c8ac-5870-4cbc-b613-7dd39f510f31/image.png)

위와 같이 URI 구성 요소(위 이미지)에 대해 좀 더 명확하게 코드로 구분 지을 수 있고, 불필요한 기호들(://, ?, &, = 등)을 사용하지 않아도 되는 장점이 눈에 보입니다.

코드에 대한 고민이 점점 많아지면서 다른 사람이 봤을 때 명확하고, 깔끔하게 의도를 이해할 수 있는 코드가 좋은 코드가 아닐까 싶습니다.

그런 코드를 위해 이런 클래스를 찾아보고 사용하는 것도 좋은 방법이지 않나 생각한 하루였습니다.

그럼 이만. 🥕👋🏼🖐🏼

### 참고자료
[https://velog.io/@jch9537/URI-URL](https://velog.io/@jch9537/URI-URL)
[https://mygumi.tistory.com/139](https://mygumi.tistory.com/139)  
[https://www.baeldung.com/spring-uricomponentsbuilder](https://www.baeldung.com/spring-uricomponentsbuilder)
[https://youngwonhan-family.tistory.com/71](https://youngwonhan-family.tistory.com/71)
