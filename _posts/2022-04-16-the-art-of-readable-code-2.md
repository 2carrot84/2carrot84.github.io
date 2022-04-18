---
title: "[책] 읽기 좋은 코드가 좋은 코드다 - 3장"
excerpt_separator: "<!--more-->"
categories:
  - book
tags:
  - 클린코드 
---

지난번 포스팅에 이어, '읽기 좋은 코드가 좋은 코드다' 책의 3장 '오해할 수 없는 이름들' 에 대한 내용을 정리해 봅니다.

[책 링크](https://book.naver.com/bookdb/book_detail.nhn?bid=6871807)

<!--more-->

### 3 오해할 수 없는 이름들

본인이 지은 이름을 "다른 사람들이 다른 의미로 해석할 수 있을까?" 라는 질문을 던져보며 철저하게 확인해야 한다.

#### 예: Filter()
```java
results = Database.all_objects.filter("year <= 2011")
```
위 results 는 어떤 데이터를 담고 있는가?

- year <= 2011 인 객체들 -> select("year <= 2011") 
- year <= 2011 이 아닌 객체들 -> exclude("year <= 2011")

#### 예: Clip(text, length)
```kotlin
def Clip(text, length)
```
여기에서 Clip()이 동작하는 방식을 다음 두 가지로 이해할 수 있다.

- 문단의 끝에서부터 거꾸로 length 만큼 제거한다. 
- 문단의 처음부터 최대 length 만큼 잘라낸다.

문단의 처움부터 잘라내는 함수라면 `Truncate(text, length)` 로 짓는 것이 더 좋을 것이다. 
legnth 도 max_length 로 하면, 의미를 더욱 뚜렷하게 만들어 줄 것이다. 하지만 이뿐만 아니라 max_length 도 여러 의미로 해설될 수 있다.
- 바이트의 수 (max_bytes)
- 문자의 수 (max_chars)
- 단어의 수 (max_words)

#### 경계를 포함하는 한계값을 다룰 때는 min 과 max 를 사용하라

쇼핑카트 애플리케이션은 고객이 한번에 10개 이상의 품목을 구매하지 못한다면,

```java
CART_TOO_BIG_LIMIT = 10

if shopping_cart_num_items() >= CART_TOO_BIG_LIMIT:
    Error("Too many intes in cart")
```
CART_TOO_BIG_LIMIT 이라는 이름이 '그 수까지'를 의미하는지 아니면 '해당 수를 포함하면서 그 수까지'를 의미하는지 분명하지 않다.
```java
MAX_ITEMS_IN_CART = 10

if shopping_cart_num_items() > MAX_ITEMS_IN_CART:
    Error("Too many intes in cart")
```

#### 경계를 포함하는 범위에는 first 와 last 를 사용하라
```java
print integer_range(start = 2, stop = 4)
# 이 코드는 [2,3]과 [2,3,4] 중에서 무엇을 출력하는가? 
```
경계의 양 끝 점이 포함된다는 의미에서 경계를 포함하는 범위에는 first/last 가 좋은 선택이다.
```java
set.PrintKeys(first = "Bart", last = "Maggie")
```

#### 경계를 포함하고/배제하는 범위에는 begin 와 end 를 사용하라
10월 16일에 일어난 일을 모두 출력하고 싶을때
```java
PringEventsInRange("Oct 16 12:00am", "Oct 17 12:00am") 
```
라고 쓰는 것이 아래보다 더 편리하다
```java
PringEventsInRange("Oct 16 12:00am", "Oct 16 11:59:59.9990pm")
```
이렇게 '포함/배제가' 동시에 일어나면 begin/end 를 사용하는 전형적인 프로그래밍 관행이 있다.

#### 불리언 변수에 이름 붙이기
```java
bool read_password = true;
```
위 코드는 읽는 방법에 따라서 두 가지 상반된 해석이 가능하다.
- 우리는 패스워드를 읽을 필요가 있다.
- 패스워드가 이미 읽혔다.  

'read' 대신 'need_password' 혹은 'user_is_authenticated' 를 사용한다.  
일반적으로 is, has, can, should 와 같은 단어를 더하면 불리언 값의 의미가 더 명확해 진다.

예를 들어 SpaceLeft() 같은 함수는 숫자값을 반환할 것처럼 보인다. 불리언 값을 반환한다면 HasSpaceLeft() 가 더 적합할 것이다.

끝으로 이름에서는 의미를 부정하는 용어를 피하는 것이 좋다.
```java
bool disable_ssl = false; -> bool use_ssl = true;
```

#### 사용자의 기대에 부응하기

**예: get*()**  
프로그래머들은 대개 get으로 시작되는 이름의 메소드를 '가벼운 접근자'로서 단순히 내부 멤버를 반환한다고 관행적으로 생각한다.  

**예: list::size()**  
list.size() 는 O(1) 연산 또는 미리 계산된 카운트를 반환할 것이라고 생각한다. 만약 O(n) 연산을 하는 size() 함수가 있다면, 버그를 유발할 수 있다.

#### 예: 이름을 짓기 위해 복수의 후보를 평가하기

좋은 이름을 정할 때 마음속으로 떠올리는 후보가 여럿 있게 마련이다. 어느 하나를 최종적으로 선택하기 전에 각 이름의 장점으 ㄹ따저보기 마련이다.

### 마무리

변수명을 지을때 많은 고민이 필요한 것과 영어 공부가 필요하다고 느낀 장이였습니다. 🤔

그럼 이만. 🥕👋🏼🖐🏼