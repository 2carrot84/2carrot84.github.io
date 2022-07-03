---
title: "IntelliJ IDEA(인텔리제이) 꿀 단축키 🍯"
excerpt_separator: "<!--more-->"
categories:
  - development
tags:
  - IntelliJ
  - 단축키
  - shortcut key
last_modified_at: 2022-01-05T22:20:00
---

오늘은 현재 사용 중인 IDE인 [IntellJ](https://namu.wiki/w/IntelliJ%20IDEA) 의 단축키에 대해 포스팅을 해보려고 합니다.

> IDE 란?   
> Integrated Development Environment 의 약어로 개발자에 사용하는 툴을 말합니다.

과거 군대에서 행정병으로 근무한 경험 때문인지, 개인적으로 개발 업무뿐 아니라 컴퓨터 작업(문서 등)을 할 때 단축키를 많이 알고 잘 사용하면 남들보다 손이 빠르다는 얘기를 듣게 되는 거 같더라구요.

물론 작업 속도도 빨라지겠죠? 마우스를 욺직이고 클릭을 몇 번 할 때 단축키 한방으로 샤샥!⚡️ 

얼마 전 팀 막내(부럽..😔)분이 알려준 너무 좋은 단축키를 듣고 반성을 하며 앞으로 이 포스팅을 계속 업데이트해보려 합니다. 

<!--more-->


### command(⌘) + shift(⇧) + \

`/main/main.do` 가 매핑된 메소드를 찾고자 할때 기존에는 아래 처럼 검색을 했습니다. 😅🙄

![기존 방식](/images/posts/2021/10/findUrl_asis.png)

1. command(⌘) + shift(⇧) + f
1. File Mask 를 .java 로 설정
1. `/main/main.do`가 아닌 `/main.do` 붙혀넣기

> 3번의 사유는 컨트롤러 클래스와 메소드에 URL 패스가 분리되어 있다보니, `/main/main.do` 로 검색이 불가했습니다.
>     
> ```java
> @RequestMapping("/main")
> public class MainController { 
> 
>   @RequestMapping("/main.do")
>   public ModelAndView main() { 
>   } 
> }
> ```

하지만, 현재는 아래와 같이 검색을 합니다.

![현재 방식](/images/posts/2021/10/findUrl_tobe.png)

1. command(⌘) + shift(⇧) + \
2. `/main/main.do` 붙혀넣기

> 검색한 URL 가 포함된 모든 패턴의 메소드를 검색 결과로 보여주는 놀라움 ️👍🏼👍🏼


### 그외 유용한 단축키 - 지속 업데이트 예정

| 단축키                             | 설명                    |복사|
|---------------------------------|-----------------------|---|
| command(⌘) + shift(⇧) + \       | URL 검색                |`copy`|
| command(⌘) + option(⌥) + ← or → | Navigate back/forward |`copy`|
| command(⌘) + k                  | Commit project to VCS |`copy`|
| command(⌘) + shift(⇧) + k       | Push commits          |`copy`|
| command(⌘) + e                  | 최근 파일 팝업 |`copy`|


복사 기능 구현하고 싶어요..🤔😱

### 마무리

요즘 드는 생각이지만, 꾸준히 뭔가를 하는게 참 어려운 것 같습니다..🙄

이 포스팅도 꾸준히 업데이트 해보자 다짐하며, 그럼 이만. 🥕👋🏼🖐🏼
