---
title: "[책] 읽기 좋은 코드가 좋은 코드다 - 7장"
excerpt_separator: "<!--more-->"
categories:
  - book
tags:
  - 클린코드
---

지난번 포스팅에 이어, '읽기 좋은 코드가 좋은 코드다' 책의 PART TWO '루프와 논리를 단순화하기' 7장 '읽기 쉽게 흐름제어 만들기' 에 대한 내용을 정리해 봅니다.

[책 링크](https://book.naver.com/bookdb/book_detail.nhn?bid=6871807)

<!--more-->

## PART TWO 루프와 논리를 단순화 하기

## 7 읽기 쉽게 흐름제어 만들기

흐름을 제어하는 조건과 루프 그리고 여타 요소를 최대한 '자연스럽게' 만들도록 노력하라. 코드를 읽다가 다시 되돌아가서 코드를 읽지 않아도 되게끔 만들어야 한다.

### 조건문에서의 인수의 순서
다음 두 코드 중에서 어떤 코드가 더 읽기 쉬운지 생각해보라
```javascript
if (length >= 10)
```
혹은
```javascript
if (10 <= length)
```
```javascript
while (bytes_received < bytes_expected)
```
혹은
```javascript
while (bytes_expected > bytes_received)
```
두 경우 모두 첫 번째 코드가 더 읽기 편하다. 이와 관련하여 우리가 발견한 유용한 규칙은 다음과 같다.

| 왼쪽 | 오른쪽 |
| --- | --- |
|값이 더 유동적인 '질문을 받는' 표현 | 더 고정적인 값으로, 비교대상으로 사용되는 표현|

### if/else 블록의 순서
if/else 문은 블록의 순서를 자유롭게 바꿔 작성할 수 있다.
```javascript
if (a == b) {
    // 첫 번째 경우
} else {
    // 두 번째 경우
}
```
```javascript
if (a != b) {
    // 두 번째 경우
} else {
    // 첫 번째 경우
}
```
- 부정이 아닌 긍정을 다루어라. 즉 if(!debug) 가 아니라 if(debug) 를 선호하자.
- 간단한 것을 먼저 처리하라. 이렇게 하면 동시에 같은 화면에 if 와 else 구문을 나타낼 수도 있다. 두개의 주문을 동시에 보는 게 더 좋다.
- 더 흥미롭고, 확실한 것을 먼저 다루어라.

예를 들어 URL이 expand_all이라는 질의 파리미터 포함 여부에 따라 response를 만드는 웹 서버가 있다고 하자.
```javascript
if (!URL.HasQueryParameter("expand_all")) {
    response.Render(items);
    ...
} else {
    for (int i = 0; i < items.size(); i++) {
        items[i].Expand();
    }
    ...
}
```
코드를 읽는 사람은 첫 번째 줄을 읽자마자 expand_all이 무엇인지 궁금할 것이다. 이것이 더 흥미롭고 긍정하는 부분이기도 하므로 이를 먼저 다루는 것이 좋다.
```javascript
if (URL.HasQueryParameter("expand_all")) {
    for (int i = 0; i < items.size(); i++) {
        items[i].Expand();
    }
    ...
} else {
    response.Render(items);
    ...
}
```
### ?:를 이용하는 조건문 표현
삼항 연산자와 같은 표현이 가독성에 미치는 영향은 논쟁의 대상이다. 다음은 삼항 연산자가 읽기 편하고 간결한 경우다.

```javascript
time_str += (hour >= 12) ? "pm" : "am";
```
```javascript
if (hour >= 12) {
    time_str += "pm";
} else {
    time_str += "am";
}
```
이는 다소 산만하고 중복적이다. 이 경우에는 앞의 조건문이 더 그럴듯하다.  
하지만, 이러한 표현은 쉽게 복잡해진다.
```javascript
return exponent >= 0 ? mantissa * (1 << exponent) : mantissa / (1 << -exponent);
```
#### 줄 수를 최소화하는 일보다 다른 사람이 코드를 읽고 이해하는 데 걸리는 시간을 최소화하는 일이 더 중요하다
이러한 코드는 if/else 문으로 작성하는 편이 더 자연스럽다.
```javascript
if (exponent >= 0){
    return mantissa * (1 << exponent);
} else {
    return mantissa / (1 << -exponent);
}
```
#### 기본적으로 if/else 를 이용하라. ?:를 이용하는 삼항 연산은 매우 간단할 때만 사용해야 한다.

### do/while 루프를 피하라
```java
// 'node' 부터 리스트를 검색하여 주어진 'name'을 찾는다.
// 'max_length' 이상의 노드는 고려하지 않는다.
public boolean ListHasNode(Node node, String name, int max_length) {
    do {
		if (node.name().equals(name)) 
			return true;
		node = node.next();
    } while (node != null && --max_length > 0);
	
	return false;
}
```
do/while 루프는 코드 블록이 아래에 있는 조건에 따라서 다시 실행될 수도 있다. 일반적으로 논리적 조건은 그것이 감싸는 코드 위에 놓인다.  
따라서 그 역순인 do/while문은 부자연스럽다. 코드를 두번 읽기 때문이다.
```java
public boolean ListHasNode(Node node, String name, int max_length) {
	while (node != null && max_length-- > 0); {
		if (node.name().equals(name)) 
			return true;
		node = node.next();
    }
	return false;
}
```
이 버전은 만약 max_length가 0이거나 node가 null일 때도 여전히 동작한다는 장점이 있다. do/while 루프에서 countinue를 사용하면 혼란을 초래하므로 피하는 것이 좋다.  
C++의 창시자인 반얀 스트라우스트럽은 이러한 사실을 잘 정리하였다.
#### "내 경험으로 에러와 혼동의 원인은 do문에 있다. 그래서 나는 조건이 '눈에 뜨이는 곳에 미리' 나타나도록 만드는 것을 선호한다. 결과적으로 나는 do문을 피하는 경향이 있다."

### 함수 중간에서 반환하기
어떤 프로그래머는 한 함수에서 반환하는 곳이 여러 곳이면 안 된다고 생각한다. 함수 중간에서 반환하는 것은 완전히 허용되어야 한다. 이는 종종 바람직할 때도 있다.
```java
public boolean Contains(String str, String substr) {
	if (str == null || substr == null) return false;
	if (substr.equals("")) return false;
    ...    
}
```
이 함수를 위와 같이 '보호 장치' 없이 구현하면 매우 부자연스러워질 것이다.
반환 포인트를 하나만 두려는 건 함수의 끝부분에서 실행되는 클린업 코드의 호출을 보장하려는 의도다.

|언어|클린업 코드를 위한 관용적 구조|
|---|---|
|C++|destructors|
|자바, 파이썬|try finally|
|파이썬|with|
|C#|using|

### 악명 높은 goto
C를 제외한 다른 언어에는 goto를 사용하는 것보다 더 좋은 방법이 있으므로 이를 이용할 필요성이 거의 없다. goto를 쓰면 코드가 쉽게 엉망진창이 되어버려, 코드의 흐름을 따라가기 어렵게 하는 걸로 악명이 높다.
goto를 사용하는 가장 간단하고 순진무구한 방법은 함수의 맨 밑에 하나의 exit 포인트만 두는 것이다.
```javascript
if (p == NULL) goto exit;
...
exit:
    fclose(file1);
    fclose(file2);
    ...
    return;
```
위 예가 goto를 사용하는 유일한 상황이라면, goto는 그렇게까지 심각한 문제아로 여겨지지 않을 것이다.  
문제는 goto가 이동할 수 있는 장소가 여러 곳으로 늘어나면서 시작되어, 경로가 서로 교차할 때 더욱 심각해진다. 특히 goto의 목표 위치가 위로 향하면 스파게티 코드가 양산된다.  
어잿든 goto는 피할 수 있다면 피하는 게 낫다.

### 중첩을 최소화하기
코드의 중첩이 심할수록 이해하기 어렵다. 중첩이 일어날 때마다 코드를 읽는 사람의 마음 속에 존재하는 '정신적 스택'에 추가적인 조건이 입력된다.
```javascript
if (use_result == SUCCESS) {
    if (permission_result != SUCCESS) {
        reply.WriteErrors("error reading perimissions");
        reply.Done();
        return;
    }
    reply.WriteErrors("");
} else {
    reply.WriteErrors(user_result);
}
replay.Done();
```
이런 코드는 더 좋지 못한데, 그 이유는 바로 코드를 읽는 사람이 SUCCESS인 경우와 SUCCESS가 아닌 경우를 계속해서 왔다 갔다 하기 때문이다.

#### 중첩이 축적되는 방법
앞의 에제 코드를 수정하기 전에, 코드가 그렇게 된 이유부터 살펴보자
```javascript
if (use_result == SUCCESS) {
    reply.WriteErrors("");
} else {
    reply.WriteErrors(user_result);
}
replay.Done();
```
이 코드는 완벽하게 이해할 수 있다. 하지만 어떤 프로그래머가 두 번째 동작을 집어넣었다.
```javascript
if (use_result == SUCCESS) {
    if (permission_result != SUCCESS) {
        reply.WriteErrors("error reading perimissions");
        reply.Done();
        return;
    }
    reply.WriteErrors("");
...
```
수정해야 하는 상황이라면 여러분의 코드를 새로운 관점에서 바라보라. 뒤로 한걸음 물러서서 코드 전체를 보라.

#### 함수 중간에서 반환하여 중첩을 제거하라
이렇게 특정한 조건을 만나면 함수를 반환하기 위해서 삽입된 중첩은 '실패한 경우들'을 최대한 빠르게 처리하고 함수에서 반환하여 제거할 수 있다.
```javascript
if (use_result != SUCCESS) {
    reply.WriteErrors(user_result);
    reply.Done();
    return;
}
     
if (permission_result != SUCCESS) {
    reply.WriteErrors("error reading perimissions");
    reply.Done();
    return;
}

reply.WriteErrors("");
replay.Done();
```
이 코드는 이제 두단계가 아닌 한 단계 중첩을 가진다. 모든 if 블록은 return과 함께 끝나기 때문이다.

#### 루프 내부에 있는 중첩 제거하기
중간에 반환하는 기술은 항상 적용할 수 있는 게 아니다.
```javascript
for (int i = 0; i < results.size(); i++) {
    if (results[i] != NULL) {
        non_null_count++;
        if (results[i] -> name != "") {
            cout << "Considering candidate..." << endl;
            ...
        }
    }
}
```
루프 내부에 중첩된 코드가 있다면, 밖으로 빠져나가지 않고 루프에서 중간에 반환할 때는 continue를 사용한다. 
```javascript
for (int i = 0; i < results.size(); i++) {
    if (results[i] == NULL) continue;
    non_null_count++;
    
    if (results[i] -> name == "") continue; 
    cout << "Considering candidate..." << endl;
    ...
}
```
### 실행 흐름을 따라 올 수 있는가
프로그램의 전체 실행 경로를 쉽게 따라갈 수 있게 만드는 게 궁극의 목표다. main()에서 시작해서 프로그램이 종료할 때까지 함수의 호출과 같이 코드에 존재하는 각 단계를 마음 속으로 밟아나가는 것이다.

하지만 실제로는 프로그래밍 언어와 라이브러리 코드가 눈에 보이는 코드의 '뒤에서'실행되므로 흐름을 완전히 따라가기가 녹녹하지는 않다.

| 프로그래밍 구조 | 상위수준의 프로그램 흐림이 혼란스러워지는 방식 |
|----|----|
|스레딩|어느 코드가 언제 실행되는지 불분명하다.|
|시그널/인터럽트 핸들러|어떤 코드가 어떤 시점에 실행될지 모른다.|
|예외|예외처리가 여러 함수 호출을 거치면서 실행될 수 있다.|
|함수 포인터 & 익명 함수|실행할 함수가 런타임에 결정되기 때문에 컴파일 과정에서는 어떤 코드가 실행될지 알기 어렵다|
|가상 메소드 | object.virtualMethod()는 알려지지 않은 하위클래스의 코드를 호출할지도 모른다|

위 예시 중 어떤 것은 매우 유용하다. 코드를 더 읽기 편하고 덜 중복되게 한다. 하지만 프로그래머는 나중에 코드를 읽는 사람이 얼마나 어렵게 느낄지 생각하지 않은 채 이러한 구조들을 과도하게 사용하기도 한다.  
결국 핵심은 코드를 작성할 때 이러한 구조가 차지하는 비율이 너무 높지 않아야 한다는 데 있다.

### 마무리

클린코드를 위해서 메소드내 중첩은 가능하면 한단계만 사용하도록 만드려는 노력을 해야 한다는 내용을 본적 있습니다.

early return 과 메소드 분리, 조건문 변경 등을 통해서 이번 장에 설명된 읽기 쉽게 흐름 제어 하는 코드를 만들도록 노력해 보아야겠습니다.

그럼 이만. 🥕👋🏼🖐🏼