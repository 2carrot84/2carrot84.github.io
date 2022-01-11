---
title: "Spring Boot 외부 설정파일 우선순위"
excerpt_separator: "<!--more-->"
categories:
  - development
tags:
  - Spring Boot
  - 외부 설정 파일
  - 우선순위
---

Spring Boot 외부 설정파일에 대한 우선순위에 대해 팀원의 저에게 낸 퀴즈 정답을 까먹지 않게 기록해두려고 작성합니다.
<!--more-->

Spring Boot에서 외부설정의 종류 

|외부설정|
|---|
|properties|
|YAML|
|환경 변수|
|커맨드 라인 Argument|

#### Properties 우선순위
> 1. 유저 홈 디렉토리에 있는 spring-boot-dev-tools.properties
> 2. 테스트에 있는 @TestPropertySource
> 3. @SpringBootTest 애노테이션의 properties 애트리뷰트
> 4. 커맨드 라인 아규먼트
> 5. SPRING_APPLICATION_JSON (환경 변수 또는 시스템 프로티) 에 들어있는 프로퍼티
> 6. ServletConfig 파라미터
> 7. ServletContext 파라미터
> 8. java:comp/env JNDI 애트리뷰트
> 9. System.getProperties() 자바 시스템 프로퍼티
> 10. OS 환경 변수
> 11. RandomValuePropertySource
> 12. JAR 밖에 있는 특정 프로파일용 application properties
> 13. JAR 안에 있는 특정 프로파일용 application properties
> 14. JAR 밖에 있는 application properties
> 15. JAR 안에 있는 application properties
> 16. @PropertySource
> 17. 기본 프로퍼티 (SpringApplication.setDefaultProperties)

#### application.properites & appliation.yml 우선순위 ( 높은게 낮을걸 덮어 씁니다 ) 최종적으로 1개의 File만 남습니다.
#### 경로 관련 우선 순위
1. file:./config/
2. file:./
3. classpath:/config/
4. classpath:/

#### Spring Boot 2.4 외부 설정 파일 변화
1. .properties 파일도 마치 .yaml 파일에서처럼 다중 문서를 지원한다.
2. 한 설정 파일 내에 있는 하위 문서가 상위 문서 내용을 덮어쓴다.
3. spring.config.activate.on-profile 로 해당 문서가 어떤 프로파일용 문서인지 지정한다. 
4. 프로파일 지정(spring.config.activate.on-profile)과 사용할 프로파일(spring.profiles.active)을 한 문서 내에서 같이 사용할 수 없다. (혼란을 줄이고자.)
5. 프로파일 그룹 기능 지원. (특정 프로파일을 활성화 했을 때 같이 활성화 해야 하는 세부 프로파일을 설정하는 기능)

### 마무리
짤막 하더라도 잊어버리거나 다시 검색 시 내 블로그 안에서 검색할 수 있도록 포스팅을 하는 게 어떨까 싶네요.

그럼 이만. 🥕👋🏼🖐🏼

### 참고자료
[https://www.whiteship.me/spring-boot-external-config/](https://www.whiteship.me/spring-boot-external-config/)
[https://toycoms.tistory.com/33](https://toycoms.tistory.com/33)

