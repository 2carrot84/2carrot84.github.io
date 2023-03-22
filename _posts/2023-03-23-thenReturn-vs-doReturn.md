---
title: "[Test Code] thenReturn vs doReturn 차이점"
excerpt_separator: "<!--more-->"
categories:
- development

tags:
- 테스트 코드 
- thenReturn
- doReturn
---

요즘 팀 자체 개선 프로젝트에 테스트 코드를 작성하느라 이것저것 검색하며 하던 중 알게된 내용을 포스팅 하고자 합니다. 

단위 테스트 코드 작성을 위해서 mocking 이 필수적으로 사용됩니다. 특정 클래스를 mocking 하고, 해당 클래스의 메소드 실행시 저희가 예상하는 값을 리턴 하도록 설정을 많이 하게 되는데요.

이때 사용하는 문법이 when(클래스.메소드(파라미터)).thenReturn(응답값) 을 주로 사용해왔습니다. 
<!--more-->

저는 위와 같이 사용하면 해당 메소드를 호출하는 대신 응답값을 리턴해준다고 생각하고 있었는데, 실제 메소드가 실행되어, NPE 를 뱉는 현상을 발견하고 혼란스러워 졌습니다.

그래서 구글링을 해봤더니 thenReturn 과 doReturn 두가지 문법이 존재하고 각 차이점은 아래와 같다는걸 알 수 있었습니다.

### 차이점
> thenReturn  
- 메소드를 실제 호출하지만 리턴 값은 임의로 정의 할 수 있다.  
- 메소드 작업이 오래 걸릴 경우 끝날때까지 기다려야함  
- 실제 메소드를 호출하기 때문에 대상 메소드에 문제점이 있을 경우 발견 할 수 있다.

> doReturn  
- 메소드를 실제 호출하지 않으면서 리턴 값을 임의로 정의 할 수 있다.  
- 실제 메소드를 호출하지 않기 때문에 대상 메소드에 문제점이 있어도 알수가 없다.

### 마무리
위 차이점과 같이 실제 메소드를 실행 시키고 싶을때는 thenReturn, 메소드 호출을 생략하고 싶을때는 doReturn 을 사용하면 됩니다.

늘 느끼는 거지만, 테스트 코드는 참 어려운것 같습니다. 🤣

그럼 이만. 🥕👋🏼🖐🏼

### 참고자료
[https://royleej9.tistory.com/entry/Mockito-doReturn-thenReturn](https://royleej9.tistory.com/entry/Mockito-doReturn-thenReturn)