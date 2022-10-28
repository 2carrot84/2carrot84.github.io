---
title: "[책] 읽기 좋은 코드가 좋은 코드다 - 8장"
excerpt_separator: "<!--more-->"
categories:
  - book
tags:
  - 클린코드, clean code, 읽기 좋은 코드, 좋은코드
---

지난번 포스팅에 이어, '읽기 좋은 코드가 좋은 코드다' 책의 8장 '거대한 표현을 잘게 쪼개기' 에 대한 내용을 정리해 봅니다.

[책 링크](https://book.naver.com/bookdb/book_detail.nhn?bid=6871807)

<!--more-->

## 8 거대한 표현을 잘게 쪼개기

**거대한 표현을 더 소화하기 쉬운 여러 조각으로 나눈다.**

이 장에서는 코드를 수정해서 삼키기 쉬운 작은 조각으로 나누는 방법을 알아볼 것이다.

### 설명 변수
커다란 표현을 쪼개는 가장 쉬운 방법은 작은 하위표현의 의미를 설명하는 '설명 변수'를 만드는 것이다.
```javascript
if line.split(':')[0].strip() == "root":
```
다음은 설명변수를 사용하는 위와 동일한 코드의 예다.
```javascript
username = line.split(':')[0].strip()
if username == "root":
```

### 요약 변수
커다란 코드의 덩어리를 짧은 이름으로 대체하여 더 쉽게 관리하고 파악하는 목적을 가진 변수를 요약 변수라고 한다.
```java
if (request.user.id == document.owner_id) {
// 사용자가 이 문서를 수정할 수 있다...
}
...
if (request.user.id != document.owner_id) {
// 문서는 읽기전용이다...
}
```
위 코드의 핵심 개념은 '사용자가 이 문서를 소유하는가?'다. 이러한 개념은 요약 변수를 더하면 더 명확하게 표현될 수 있다.
```java
final boolean user_owns_document = (request.user.id == document.owner_id); 
if (user_owns_document) {
// 사용자가 이 문서를 수정할 수 있다...
}
...
if (!user_owns_document) {
// 문서는 읽기전용이다...
}
```
대단한 개선처럼 보이지 않을지 몰라도 더 읽기 쉽다. 또한 user_owns_document 라는 표현을 맨 위에 두어 주된 개념이라는 생각이 들게 한다.

### 드모르간의 법칙 사용하기
동일한 불리언 표현은 다음과 같이 두가지 방법으로 작성할 수 있다.
```java
1) not (a or b or c) <-> (not a) and (not b) and (not c)
2) not (a and b and c) <-> (not a) or (not b) or (not c)
```
이러한 법칙을 떠올리는 게 힘들면 "not을 분배하고 and/or를 바꿔라"만 기억하자. 혹은 거꾸로 not을 밖으로 빼내기도 한다.
```java
if (!(file_exists && !is_protected)) Error("미안합니다. 파일을 읽을 수 없습니다.")
```
이를 다음과 같이 수정할 수 있다.
```java
if (!file_exists || is_protected) Error("미안합니다. 파일을 읽을 수 없습니다.")
```

### 쇼트 서킷 논리 오용 말기
대부분의 프로그래밍 언어에서 불리언 연산은 쇼트 서킷 평가를 수행한다. 예를 들어 if (a || b) 에서 a가 참이면 b는 평가하지 않는다.
```javascript
assert((!(bucket = FindBucket(key))) || !bucket->IsOccupied());
```
이 코드는 한 줄에 불과하지만 대부분의 프로그래머는 의미를 이해하기 위해서 손을 멈추고 생각해야 한다. 앞에서 살펴본 예를 다음 코드와 비교해보라.
```javascript
bucket = FindBucket(key);
if (bucket != NULL) assert(!bucket->IsOccupied());
```
코드가 두 줄로 늘어났지만 훨씬 이해하기 쉬워졌다.

**'영리하게' 작성된 코드에 유의하라. 나중에 다른 사람이 읽으면 그런 코드가 종종 혼란을 초래한다.**

다음과 같이 깔끔하게 사용할 수 있는 경우에 쇼트 서킷 연산을 얼마든지 사용해도 된다.
```javascript
if (object && object->method()) ...
```
파이썬, 자바스크립트, 루비 같은 언어는 'or' 연산자가 인수 중 하나를 반환한다(해당 값은 불리언으로 변환되지 않는다).
```javascript
x = a || b || c
```
a, b, c 라는 세 값 중에서 첫 번째 '참'값을 반환하는 데 사용할 수 있다.

### 예: 복잡한 논리와 씨름하기
```javascript
struct Range {
    int begin;
    int end;
    bool OverlapsWith(Range other);
}
```
다음은 주어진 범위의 양쪽 경계값이 other의 범위에 속하는지 확인하는 OverlabsWith() 한수를 구현하는 한가지 방법이다.
```javascript
bool Range::OverlabWith(Range other) {
    return (begin >= other.begin && begin <= other.end) ||
        (end >= other.begin && end <= other.end);
}
```
코드가 두 줄밖에 되지 않지만, 안에서 많은 일이 일어나고 있다.  
새각해야 하거나 조건이 너무나 많으므로 버그가 발생할 확률이 매우 높다.  
사실은 위 코드에 버그가 있다. 앞선 코드는 범위 [0,2]가 [2,4]와 겹친다고 말한다. 사실은 겹치지 않는데 말이다.
```javascript
    return (begin >= other.begin && begin < other.end) ||
        (end > other.begin && end <= other.end);
```
이제는 정확한가? 사실은 또 다른 버그가 있다. 이 코드는 begin/end가 other를 완전히 포함하는 경우를 무시한다.
```javascript
    return (begin >= other.begin && begin < other.end) ||
        (end > other.begin && end <= other.end) ||
        (begin <= other.begin && end >= other.end);
```
**더 우아한 접근방법 발견하기**

해결책은 똑같은 문제를 '반대되는' 방법으로 해결할 수 있는지 확인하는 것이다. 이는 상황에 따라 배열을 역순으로 반복하거나 어떤 데이터 구조를 앞이 아니라 뒤로 가면서 채워 넣는 것을 의미한다.

여기서 OverlabWith()의 반대는 '겹치지 않는 것'이다. 이를 확인하는 방법에는 두가지 가능성만 존재한다. 

1. 다른 범위가 이 범위 시작보다 전에 끝난다. 
2. 다른 범위가 이 범위가 끝난 후에 시작된다.

```javascript
bool Range::OverlabWith(Range other) {
    if (other.end <= begin) return false;
    if (other.begin >= end) return false;
    return true;
}
```
코드의 각 줄은 전보다 훨씬 더 간단하다. 한 번의 비교만 포함할 뿐이다.

### 거대한 구문 나누기
```javascript
var update_highlight = function (message_num) {
    if ($("#vote_value" + message_num).html() === "Up") {
        $("#thumbs_up" + message_num).addClass("highlighted");
        $("#thumbs_down" + message_num).removeClass("highlighted");
    } else if ($("#vote_value" + message_num).html() === "Down") {
        $("#thumbs_up" + message_num).removeClass("highlighted");
        $("#thumbs_down" + message_num).addClass("highlighted");
    } else {
        $("#thumbs_up" + message_num).removeClass("highighted");
        $("#thumbs_down" + message_num).removeClass("highlighted");
    }
}
```
이 코드에 있는 개별적인 표현은 그렇게 크지 않지만, 모두 한 곳에 있어서 코드를 읽는 사람의 머리를 강타하는 거대한 구문을 형성한다.

다행히도 동일한 부분을 요약 변수로 추출해서 함수의 앞부분에 놓아 둘 수 있다.

```javascript
var update_highlight = function (message_num) {
    var thumbs_up = $("#thumbs_up" + message_num); 
    var thumbs_down = $("#thumbs_down" + message_num); 
    var vote_value = $("#vote_value" + message_num).html(); 
    var hi = "highlighted"; 
        
    if (vote_value === "Up") {
        thumbs_up.addClass(hi);
        thumbs_down.removeClass(hi);
    } else if (vote_value === "Down") {
        thumbs_up.removeClass(hi);
        thumbs_down.addClass(hi);
    } else {
        thumbs_up.removeClass(hi);
        thumbs_down.removeClass(hi);
    }
}
```
var hi = "highlighted"; 처럼 변수를 만들면 다음과 같은 여러 이점을 갖는다.

- 타이핑 실수를 피할 수 있다. (위 예제 아홉 번째 줄에서 "highighted" 라는 잘못된 철자를 사용하였다.)
- 코드를 한눈에 훑어보는 게 용이하도록 코드의 길이를 조금이라도 더 줄여준다.
- 클래스명을 변경해야 할 때 한곳만 바꾸면 된다.

### 표현을 단순화하는 다른 창의적인 방법들

```javascript
void AddStats(const Stats& add_from, Stats* add_to) {
    add_to->set_total_memory(add_from.total_memory() + add_to->total_memory());
    add_to->set_free_memory(add_from.free_memory() + add_to->free_memory());
    add_to->set_swap_memory(add_from.swap_memory() + add_to->swap_memory());
    add_to->set_status_string(add_from.status_string() + add_to->status_string());
    add_to->set_num_processes(add_from.num_processes() + add_to->num_processes());
}
```
위 코드는 길고 비슷한 코드로 보겠지만 완전히 똑같지는 않다. 하지만 모두 다른 변수를 이용할 뿐, 결국 같은 일을 수행하고 있음을 파알할 것이다.
```javascript
add_to->set_XXX(add_from.XXX() + add_to.XXX());
```
C++ 에서는 매크로를 정의하여 이러한 표현을 구현할 수 있다.
```javascript
void AddStats(const Stats& add_from, Stats* add_to) {
    #define ADD_FIELD(field) add_to->set_##field(add_from.field() + add_to->field());

    ADD_FIELD(total_memory);
    ADD_FIELD(free_memory);
    ADD_FIELD(swap_memory);
    ADD_FIELD(status_string);
    ADD_FIELD(num_processes);
    ...
    #undef ADD_FIELD
}
```
이제 코드를 다시 읽으면 핵심을 즉시 이해할 수 있다. 각각의 줄이 같은 일을 수행한다는 사실도 매우 명확해진다.

### 마무리

코드를 짜다 보면 머릿속에 떠오른 순서대로 작성하는 일이 빈번하죠. 본문에 나온 것처럼 '반대되는' 방법으로 간단히 해결되는 경우를 저도 종종 경험해보았습니다.

본인이 짠 코드를 보며 남들이 읽을때 어렵지 않은지, 어렵다면 그걸 보기 쉽게 고칠 방법은 없는지 고민이 늘 필요해 보입니다.

그럼 이만. 🥕👋🏼🖐🏼