---
title: "[책] 읽기 좋은 코드가 좋은 코드다 - 6장"
excerpt_separator: "<!--more-->"
categories:
  - book
tags:
  - 클린코드
---

지난번 포스팅에 이어, '읽기 좋은 코드가 좋은 코드다' 책의 6장 '명확하고 간결한 주석 달기' 에 대한 내용을 정리해 봅니다.

[책 링크](https://book.naver.com/bookdb/book_detail.nhn?bid=6871807)

<!--more-->

## 6 명확하고 간결한 주석 달기

주석은 명확하게, 최대한 구체적이고 자세하게 작성해야 한다. 반면 주석은 화면에서 추가적인 면적을 차지하고, 읽는 데 추가적인 시간을 요구하므로 간결해야 한다.

#### 주석은 높은 '정보 대 공간' 비율을 갖춰야 한다.

### 주석을 간결하게 하라
```javascript
// int 는 CategoryType 이다
// 내부 페어의 첫 번째 float는 'scroe' 다
// 두 번째는 'weight' 다
typedef hash_map<int, pair<float, float> > ScoreMap;
```
위의 예제의 주석은 한 줄이면 충분하다.
```javascript
// CategoryTpye -> (score, weight)
typedef hash_map<int, pair<float, float> > ScoreMap;
```

### 모호한 대명사는 피하라
코드를 읽는 사람이 대명사를 '해석'하려면 추가적인 노력이 필요하다.
```javascript
// Insert the data into the cache, but check if it's too big first.
// 데이터를 캐시에 넣어라, 하지만 그것이 너무 큰지 먼저 확인하라
```
이 주석에서 'it'은 데이터를 가리킬 수도 있고 캐시를 가리킬 수도 있다. 가장 안전한 방법은 대명사를 원래 명사로 대체 하는 것이다.
```javascript
// Insert the data into the cache, but check if **the data** is  too big first.
// 데이터를 캐시에 넣어라, 하지만 **데이터가** 너무 큰지 먼저 확인하라
```
또한 'it'을 완전히 명확하게 하기 위해서 문장을 고칠 수도 있다.
```javascript
// if the data is small enough, insert it into the cache.
// 데이터가 충분히 작으면, 이를 캐시에 넣어라.
```

### 엉터리 문장을 다듬어라
주석을 명확하게 하는 작업과 간결하게 하는 작업은 대부분 한번에 이루어진다.
```javascript
# 이 URL을 전에 이미 방문했는지에 따라서 다른 우선순위를 부여한다.
```
이 문장도 어느 정도 괜찮지만, 다음 문장과 비교해보자.
```javascript
# 전에 방문하지 않은 URL에 높은 우선순위를 부여하라.
```
아래 문장이 더 간단하고, 짧고, 직접적이다. 또한, 아직 방문하지 않은 URL에 높은 우선순위가 부여된다는 사실까지 설명하고 있다.

### 함수의 동작을 명확하게 설명하라
```javascript
// 이 파일에 담긴 줄 수를 반환하라
int CountLines(string filename) { ... }
```
이 주석은 그다지 명확하지 않다. '줄'을 정의하는 방법은 여러 가지이기 때문이다.

- "" (빈 파일)은 줄 수가 0인가 1인가?
- "hello"은 줄 수가 0인가 1인가?
- "hello\n"은 줄 수가 1인가 2인가?
- "hello\n world"은 줄 수가 1인가 2인가?
- "hello\n\r cruel\n world\r"은 줄 수가 2,3,4 중 어느 것인가?

가장 간단한 구현은 단순희 개행문자(\n)를 세는 것이다.
```javascript
// 파일 안에 새 줄을 나타내는 바이트('\n')가 몇 개 있는지 샌다.
int CountLines(string filename) { ... }
```

### 코너케이스를 설명해주는 입/출력 예를 사용하라
주석을 작성하는 데 신중하게 선택된 입/출력 예는 천 마디 말보다 위력적이다.
```javascript
// 입력된 'src'의 'chars'라는 접두사와 접미사를 제거한다.
String Strip(String src, String chars) { ... }
```
이 주석은 다음과 같은 질문에 답하지 않으므로 명확하지 않다.
- chars가 제거되어야 하는 정확한 부분 문자열을 의마하는가 아니면 특정한 순서가 정해지지 않은 문자의 집합을 의미하는가?
- src의 끝에 chars가 여러 번 있으면 어떻게 되는가?

이러한 주석에 비해서 잘 선택된 입출력 예는 위 질문의 대답을 제공한다.
```javascript
// 예: Strip("abba/a/ba", "ab") 은 '/a/' 를 반환한다.
int CountLines(string filename) { ... }
```

지나치게 간단한 입출력 예는 별로 유용하지 않다는 점에 유의하라
```javascript
// 예: Strip("ab", "a") 은 "b"를 반환한다.
```

다음은 설명을 이런 식으로 활용하는 함수의 또 다른 예다.
```javascript
// pivot보다 작은 요소가 pivot과 크거나 같은 요소들보다 앞에 오도록 'v'를 재배열한다.
// 그 다음 v[i] < pivot을 만족시키는 것 중에서 가장 큰 'i'를 (혹은 pivot보다 작은 것이 없으면 -1 을) 반환한다.
int Partition(vector<int>* v, int pivot);
```
다음은 함수의 내용을 더 잘 설명해주는 입출력 예다.
```javascript
// 예: Partition([8 5 9 8 2], 8) 은 [ 5 2 | 8 9 8] 를 만들고 1을 반환할 것이다.
int Partition(vector<int>* v, int pivot);
```
이번 예는 입/출력을 보여주는데 몇 군데 짚고 넘어갈 부분이 있다.

- 벡터 안에 존재하는 값을 pivot으로 상용하여 경계가 분활되는 방식을 설명한다.
- 벡터가 중복된 값을 허용한다는 사실을 보여주기 위해서 중복된 값(8)을 포함시켰다.
- 중복된 값(8)을 포함시켜 벡터가 중복된 값을 허용한다는 사실을 보여준다.
- 결과값을 담은 벡터를 일부러 정렬하지 않았다. 만약 정렬하면 혼동을 초래할 것이다.
- 반환된 값이 1이므로, 벡터에 1이 포함되지 않게 했다. 1이 포함되면 혼동을 초래할 것이다.

### 코드의 의도를 명시하라
주석 달기는 코드를 작성하면서 생각했던 바를 나중에 코드를 읽는 사람에게 전달해주는 것이다.
```java
void DisplayProducts(list<Product> products) {
	products.sort(CompareProductByPrice);
	
	// 리스트를 역순으로 반복한다.
    for (list<Product>::reverse_interator it = products.rbegin(); it != products.rend(); ++it)
		DisplayPrice(it -> price);
    ...
}
```
이 주석은 바로 아래에 있는 코드의 묘사만 전달할 뿐이다. 대신 더 좋은 주석을 생각해보자.
```javascript
// 각 가격을 높은 값에서 낮은 값 순으로 나타낸다.
for (list<Product>::reverse_interator it = products.rbegin(); it != products.rend(); ++it)
```
이 주석은 프로그램이 수행하는 동작을 개략적인 높은 수준에서 설명한다. 프로그래머가 코드를 작성하는 동안 생각했던 것에 더 가까운 설명인 것이다.  
이런 주석은 실질적으로 중복검사의 역활을 수행한다. 궁극적으로 가장 최선의 중복검사는 유닛테스트다. 하지만 프로그램의 의도를 설명해주는 주석을 다는 행위에는 의미가 있다.

### 이름을 가진 함수 파라미터(Named Function Prameter) 주석
다음과 같은 함수 호출을 만났다고 하자.
```javascript
Connect(10, false);
```
이 함수 호출은 함수에 주어진 정수와 불리언값이 무엇을 뜻하는지 불분면하기 때문에 명확하지 않다.  
파이썬 같은 언어는 이름과 함께 인수를 전달할 수 있다.
```javascript
def Connect(timeout, use_encryption): ...
# 이름을 가진 파라미터를 이용해서 함수를 호출한다.
Connect(timeout = 10, use_encryption = False)
```
C++ 와 자바 같은 언어는 이렇게 사용할 수 없으므로, 아래와 같은 방법을 사용할 수 있다.
```java
void Connect(timeout, use_encryption) { ... }
// 주석을 가진 파라미터를 이용해서 함수를 호출한다.
Connect(/* timeout_ms = */ 10, /*use_encryption = */ False)
```

### 정보 축양형 단어를 사용하라
```javascript
// 이 클래스는 데이터베이스와 동일한 정보를 담는 멤버를 가지고 있는데, 
// 이는 속도를 향상시키는 데 사용된다. 나중에 이 클래스가 읽히면, 멤버들이 어떤 값을 
// 가졌는지 확인하고, 만약 값이 있으면 그 값이 반환된다. 값이 없으면
// 데이터베이스에서 값이 읽혀져서 나중에 이용될 수 있게 멤버에 저장된다.
```
이러한 표현 대신 다음처럼 간단하게 할 수도 있다.
```javascript
// 이 클래스는 데이터베이스에 대한 캐시 계층으로 가능하다.
```
혹은 다음과 같은 주석이 있다고 하자.
```javascript
// 주소값에서 불필요한 빈 칸을 제거한다. 그리고 "Avenue" 를 "Ave."로 바꾸는 것과 
// 같은 정리 작업을 수행한다. 이러한 과정으로 사실상 같지만 다르게 입력된 
// 주소는 동일한 방식으로 정리된 값을 갖게 되어 동일한 주소를 가지는지를 
// 값들을 서로 비교해서 확인할 수 있다.
```
이것도 다음과 같이 다시 작설할 수 있다.
```javascript
// 주소값을 표준화한다(불필요한 빈칸을 제거하고, "Avenue" -> "Ave." 등의 정리 작업을 수행한다)
```
길게 늘어지는 주석을 써야 하는 상황이라면 프로그래밍에 전형적인 상황을 묘사하는 표현이 있는지 확인하는 편이 좋다.

### 마무리

개인적으로 주석보단 코드를 명확하게 짜는게 좋다는 생각을 가지고 있는 입장에서 이번 장과 같이 주석을 다는 내용에 대해서는 다시 한번 생각해볼 필요가 있는 것 같습니다.

조금 과하다 싶은 부분도 있는 것 같기도 하고, 공감되는 부분도 있는 장이였습니다.

그럼 이만. 🥕👋🏼🖐🏼