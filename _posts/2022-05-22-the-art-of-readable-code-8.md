---
title: "[책] 읽기 좋은 코드가 좋은 코드다 - 9장"
excerpt_separator: "<!--more-->"
categories:
  - book
tags:
  - 클린코드
---

지난번 포스팅에 이어, '읽기 좋은 코드가 좋은 코드다' 책의 9장 '변수와 가독성' 에 대한 내용을 정리해 봅니다.

[책 링크](https://book.naver.com/bookdb/book_detail.nhn?bid=6871807)

<!--more-->

## 9 변수와 가독성

1. 변수의 수가 많을수록 기억하고 다루기 더 어려워진다.
2. 변수의 범위가 넓어질수록 기억하고 다루는 시간이 더 길어진다.
3. 변수값이 자주 바뀔수록 현재값을 기억하고 다루기가 더 어려워진다.

이번 장에서는 위 문제를 다루는 방법을 알아볼 것이다.

### 변수 제거하기
이번 절에서는 가독성에 도움되지 않는 변수를 제거하는 방법을 알아볼 것이다. 이러한 변수를 제거하면 새로운 코드는 더 간결하고 이해하기 쉬워진다.

**불필요한 임시 변수들**
```javascript
now = datetime.datetime.now()
root_message.last_view_time = now
```
now 변수가 필요하지 않은 이유는 다음과 같다.
- 복잡한 표현을 잘게 나누지 않는다.
- 명확성에 도움이 되지 않는다.
- 한 번만 사용되어 중복된 코드를 압축하지 않는다.
```javascript
root_message.last_view_time = datetime.datetime.now()
```

**중간 결과 삭제하기**
```javascript
var remove_one = function (array, value_to_remove) {
    var index_to_remove = null;
    for (var i = 0; i < array.length; i += 1) {
        if (array[i] === value_to_remove) {
            index_to_remove = i;
            break;
        }
    }
    if (index_to_remove !== null) {
        array.splice(index_to_remove, 1);
    }
};
```
변수 index_to_remove는 단지 중간 결과를 저장할 뿐이다. 이러한 변수는 결과를 얻자마자 곧바로 처리하는 방식으로 제거할 수 있다.
```javascript
var remove_one = function (array, value_to_remove) {
    var index_to_remove = null;
    for (var i = 0; i < array.length; i += 1) {
        if (array[i] === value_to_remove) {
            array.splice(i, 1);
            return;
        }
    }
};
```
할 수만 있다면 이처럼 함수를 최대한 빨리 반환하는게 좋다.

**흐름 제어 변수 제거하기**  
때로는 루프에서 다음과 같은 패턴을 만나기도 한다.
```javascript
boolean done = false;

while (/* 조건 */ && !done) {
    ...
    if (...) {
        done = true;
        continue;
    }
}
```
이러한 흐름 제어 변수는 프로그램의 구조를 잘 설계하면 제거할 수 있다.
```javascript
boolean done = false;

while (/* 조건 */) {
    ...
    if (...) {
        break;
    }
}
```
중첩된 여러 루프 때무에 break 가 추가로 필요하다면, 루프 안에서 반복되는 코드를 새로운 함수로 만들면 된다.

### 변수의 범위를 좁혀라
**변수가 적용되는 범위를 최대한 좁게 만들어라**  
코드를 읽는 사람이 한꺼번에 생각해야 하는 변수 수를 줄여주기 때문에 범위를 좁게 만들면 좋다.
```java
class LargeClass {
	string str_;
	
	void Method1() {
		str_ = ...;
		Method2();
    }
	
	void Method2() {
		// Uses str_
    }
	// str_ 을 이용하지 않는 다른 메소드들
}
```
위와 같은 경우 str_ 을 지역 변수로 '강등'시키는 편이 좋다
```java
class LargeClass {
	void Method1() {
		string str = ...;
		Method2(str);
    }
	
	void Method2(string str) {
		// str를 이용한다.
    }
	// 이제 다른 메소드는 str을 볼 수 없다.
}
```

**많은 메소드를 정적 static으로 만들어서** 클래스 멤버 접근을 제한해라.   
**커다란 클래스를 여러 작은 클래스로 나누는 방법**도 있다. 이 방법은 작은 클래스들이 서로 독립적일때 유용하다. 만약 두 개의 작은 클래스가 서로의 멤버를 참조한다면, 의미가 없게 된다.  
커다란 파일이나 함수를 여러 개의 작은 단위로 나눌때는 데이터, **즉 변수를 서로 분리하는 데 있기 때문**이다.

#### C++ 에서 if문의 범위
```java
PaymentInfo* info = database.ReadPaymentInfo();
if (info) {
	cout << "User paid: " << info->amount() << endl;
}
// 아래에 더 많은 줄이 있다..
```
이 코드를 읽는 사람은 info가 나중에 사용될 수 있다고 생각하고 기억할 것이다. 하지만, 이 경우에 info는 단지 if문 안에서만 사용될 뿐이다.
C++ 에서는 이러한 info를 조건문 표현으로 나타낼 수 있다.
```java
if (PaymentInfo* info = database.ReadPaymentInfo()) {
	cout << "User paid: " << info->amount() << endl;
}
// 아래에 더 많은 줄이 있다..
```
이제 코드를 읽는 사람은 if문을 빠져 나오는 순간 더 이상 info를 생각하지 않아도 된다.

#### 자바스크립트에서 프라이빗 변수 만들기
함수 하나에서만 사용되는 전역 변수가 있다고 해보자
```javascript
submitted = false;

var submit_form = function (form_name) {
    if (submitted) {
        return; // 폼을 두번 제출하지 말라.
    }
    ...
    submitted = true;
};
```
submitted와 같은 전역 변수는 코드를 읽는 사람에게 고민을 안겨 줄 것이다. 다른 파일에서 이와는 다른 목적으로 submitted라는 이름이 붙은 전역 변수를 사용할지도 모르는 일이다.  
submitted 변수를 클로저 내부에 집어넣어 이러한 문제를 해결할 수 있다.
```javascript
var submit_form = (function () {
    var submitted = false;  // 주의: 아래에 있는 함수만 접근할 수 있다.
    return function (form_name) {
        if (submitted) {
            return; // 폼을 두번 제출하지 말라.
        }
    ...
        submitted = true;
    };
}());    
```
마지막 줄에 있는 괄호에 주목하라. 익명의 바깥 함수는 즉각적으로 실행되고 내부의 함수를 반환한다.

#### 자바스크립트 전역 범위 
변수를 정의할때 var를 생략하면 해당 변수는 전역 변수로 모든 자바스크립트 파일과 `<script>` 블록에 접근할 수 있다.
```html
<script>
    var f = function () {
        // 위험: 'i'는  'var'와 함께 선언되지 않았다!
        for (i = 0; i < 10; i += 1) ...
    };
    
    f();
</script>
```
```html
<script>
    alert(i); // '10'을 나타낸다. 'i'가 전역 변수이기 때문이다.
</script>
```
#### 파이썬과 자바스크립트에는 없는 중첩된 범위
C++나 자바 같은 언어는 선언된 변수의 범위를 if, for, try 혹은 그와 비슷하게 중첩된 블록 안으로 제한하는 블록 범위가 있다.
```java
if (...) {
	int x = 1;
}
x++; // 컴파일 에러!
```
하지만 파이썬이나 자바스크립트는 블록 안에서 정의된 변수가 전체 함수로 '흘러나온다'
```javascript
# 이 지점까지는 example_value 를 사용하지 않는다.
if requset:
    for value in request.values:
        if value > 0:
            example_value = value
            break

for logger in debug.loggers:
    logger.log("Example : ", example_value)
```
범위에 대한 이러한 규칙은 많은 프로그래머를 당황스럽게 만들고, 코드를 읽기 어렵게 한다.  
앞서 살펴본 예는 심지어 버그도 있다. 만약 example_value 가 코드의 첫 번째 부분에서 값이 설정되지 않으면, 두 번째 부분은 "NameError: 'example_value' is not defined" 라는 예외를 발생시킨다.
```javascript
example_value = None

if requset:
    for value in request.values:
        if value > 0:
            example_value = value
            break

if example_value:
    for logger in debug.loggers:
        logger.log("Example : ", example_value)
```
하지만 이 예제에서 example_value를 완전히 제거할 수도 있다.
```javascript
def LogExample(value):
    for logger in debug.loggers:
        logger.log("Example : ", example_value)

if requset:
    for value in request.values:
        if value > 0:
            LogExample(value) # 'value' 를 즉시 처리한다.
            break
```

#### 정의를 아래로 옮기기
```javascript
def ViewFilteredReplies(original_id):
    filtered_replies = []
    root_messages = Messages.objects.get(original_id)
    all_replies = Messages.objects.select(root_id = original_id)
    root_messages.view.count += 1
    root_messages.last_view_time = datetime.daatetime.now()
    root_messages.save()

    for reply in all_replies:
        if reply.spam_vote <= MAX_SPAM_VOTES:
            filtered_replies.append(reply)
    return filtered_replies
```
이러한 코드는 코드를 읽는 사람이 한꺼번에 세 개의 변수를 생각하면서 그 사이에서 왔다갔다 하게 만드는 문제가 있다.  
코드를 읽는 사람은 이러한 변수를 당장 염두에 둘 필요가 없으므로, 실제로 사용하기 바로 직전 위치로 옮기는 게 좋다.
```javascript
def ViewFilteredReplies(original_id):
    root_messages = Messages.objects.get(original_id)
    root_messages.view.count += 1
    root_messages.last_view_time = datetime.daatetime.now()
    root_messages.save()

    all_replies = Messages.objects.select(root_id = original_id)
    filtered_replies = []
    for reply in all_replies:
        if reply.spam_vote <= MAX_SPAM_VOTES:
            filtered_replies.append(reply)
    return filtered_replies
```

### 값을 한 번만 할당하는 별수를 선호하라
```java
static final int NUM_THREADS = 10;
```
위와 같은 상수는 코드를 읽는 사람에게 별다른 추가적인 생각을 요구하지 않는다. 이와 같은 이유로 C++에서 const 사용을, 자바에서 final 사용을 권장한다. 
**변수값이 달라지는 곳이 많을수록 현재값을 추측하기 더 어려워진다.**

### 마지막 예
다음과 같이 순서대로 정렬된 여러 개의 입력 텍스트 필드를 가지는 웹페이지가 있다고 하자.
```html
<input type="text" id="input1" value="Dustin">
<input type="text" id="input2" value="Trevor">
<input type="text" id="input3" value="">
<input type="text" id="input4" value="Melissa">
```
페이지에 있는 텍스트 필드 중 첫 번째로 비어 있는 `<input>`에 집어넣는 함수인 setFirstEmptyInput()을 작성 하는 것이다. 이 장에서 논의했던 원리를 적용하지 않는 코드 예다.
```javascript
var setFirstEmptyInput = function (new_value) {
    var found = false;
    var i = 1;
    var elem = document.getElementById('input' + i);
    while (elem !== null) {
        if (elem.value === '') {
            found = true;
            break;
        }
        i++;
        elem = document.getElementById('input' + i);
    }
    if (found) elem.value = new_value;
    return elem;
}
```
- var found
- var i
- var elem

앞의 변수 세 개가 존재하는 범위는 함수 전체며 여러 차례 값이 변경된다.
이 장의 앞에서 설명한 바와 같이 found처럼 중간 결과값을 저장하는 변수는 중간에 반환하는 전략으로 제거할 수 있다.
```javascript
var setFirstEmptyInput = function (new_value) {
    var i = 1;
    var elem = document.getElementById('input' + i);
    while (elem !== null) {
        if (elem.value === '') {
            elem.value = new_value;
            return elem
        }
        i++;
        elem = document.getElementById('input' + i);
    }
    return null;
}
```
다음은 elem을 살펴보자. 이 변수는 저장하는 값이 무엇인지 기억하기 쉽지 않게끔 매우 '반복적인' 방식으로 코드 전체에 여러 번 사용되었다.
```javascript
var setFirstEmptyInput = function (new_value) {
    for (var i = 1; elem !== null; i++) {
        var elem = document.getElementById('input' + i);
        if (elem.value === '') {
            elem.value = new_value;
            return elem;
        }
    }
    return null;
}
```
이 코드에서 elem 범위가 루프의 안쪽으로 국한되었으며 값이 한 번만 할당되는 변수로 기능하고 있음에 주목하라

### 마무리

프로그래밍을 하면서 변수를 최대한 덜 사용하고, 변수의 범위를 좁게, 변수 값이 자주 변하지 않도록 고민을 하면서 코드의 가독성을 높일 수 있도록 노력해야 할 것 같네요.

그럼 이만. 🥕👋🏼🖐🏼