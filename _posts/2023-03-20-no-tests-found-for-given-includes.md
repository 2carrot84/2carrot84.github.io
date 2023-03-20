---
title: "[IntelliJ] execution failed for task ' test'. no tests found for given includes (--tests filter)"
excerpt_separator: "<!--more-->"
categories:
- development

tags:
- IntelliJ
- execution failed for task ' test'
- no tests found for given includes
---

오늘은 IntelliJ 로 Gradle 프로젝트 생성 후 테스트 코드 실행 시 발생한 오류와 해당 오류를 해결한 내용을 포스팅 하고자 합니다. 

팀 자체 개선 프로젝트에 테스트 코드 작성을 위해 PowerMock 이라는 라이브러리가 필요해 신규 프로젝트 생성하여, 몇가지 확인을 해보고자 하였습니다.

하지만 테스트 코드 실행시 아래와 같은 에러가 발생 하였습니다.
![](/images/posts/2023/03/no tests found for given includes.png)
<!--more-->

해결 방법은 아주 간단하였습니다. 바로 IntelliJ 설정 변경으로 해결이 가능 하였습니다.

> Prefereness (⌘ + ,) > Build, Execution, Deployment > Build Tool > Gradle  
> Gradle Project > Build and run > Run test using   
> Gradle 을 IntelliJ 로 변경
![](/images/posts/2023/03/스크린샷 2023-03-20 오전 9.22.11.png)


### 마무리
간단히 해결 되는 오류지만, 저와 같이 헤매는 분이 있을까봐 포스팅 하니 도움이 되었으면 좋겠습니다.

그럼 이만. 🥕👋🏼🖐🏼