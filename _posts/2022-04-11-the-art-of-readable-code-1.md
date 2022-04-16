---
title: "[책] 읽기 좋은 코드가 좋은 코드다 - 1,2장"
excerpt_separator: "<!--more-->"
categories:
  - book
tags:
  - 클린코드
date: 2022-04-11T21:27
---

계속 읽던 책을 마무리 하지 못 하고 다른 책을 펼치게 되는것 같네요.

이번에는 '읽기 좋은 코드가 좋은 코드다' 라는 책을 읽으며 내용을 정리 해보려고 합니다.

이번에는 반드시 완독 할 수 있기를 🤣

![책 정보](/images/posts/2022/04/06871807.jpeg)  
[책 링크](https://book.naver.com/bookdb/book_detail.nhn?bid=6871807)

<!--more-->

### 1 코드는 이해하기 쉬워야 한다

#### 가독성의 기본 정리

코드는 다른 사람이 그것을 이해하는 데 들이는 시간을 최소화하는 방식으로 자성되어야 한다.

#### 분량이 적으면 항상 더 좋은가?

더 분량이 적은 코드로 똑같은 무제를 해결할 수 있다면 그것이 더 낫다. 하지만, 이해를 위한 시간을 최소화하는 게 더 좋은 목표다.

#### 이해를 위한 시간은 다른 목표와 충돌하는가?

고도로 최적화된 코드조차도 이해하기 쉽게 만드는 방법이 있다. 그리고 코드를 읽기 쉽게 만드는 노력은 종종 잘 구성된 아키텍처와 테스트하기 쉬운 코드를 작성하게 도와주기도 한다.

### PART ONE 표먼적 수준에서의 개선

### 2 이름에 정보 담기

#### 특정한 단어 고르기

매우 구체적인 단어를 선택하여, '무의미한' 단어를 피하는 것이다.

> (인터넷에서 페이지를 가져오는 경우) GetPage(url) -> FetchPage(url) or DownloadPage(url)  
> BinaryTree.size() -> Height(), NumNodes(), MemoryBytes() ..  
> Thread.stop() -> Thread.kill()

#### tmp 나 retval 같은 보편적인 이름 피하기

개체의 값이나 목적을 정확하게 설명하는 이름을 골라야 한다.
> retval 이라는 이름은 정보를 제대로 담고 있지 않다. 대신 변수값을 설명하는 이름을 사용하라.  
> tmp 라는 이름은 대상이 짧게 임시적으로만 존재하고, 임시적 존재 자체가 변수의 가장 중요한 용도일때에 한해서 사용해야 한다.    
> tmp, it, retval 같은 보편적인 이름을 사용하려면, 꼭 그렇게 해야하는 이유가 있어야 한다.

#### 추상적인 이름보다 구체적인 이름을 선호하라
변수나 함수 혹은 다른 요소에 이름을 붙일 때, 추상적인 방식이 아니라 구체적인 방식으로 묘사하라
> 서버가 어느 TCP/IP 포트를 사용할 수 있는지 검사하는 메소드의 경우    
> ServerCanStart() -> CanListenOnPort()  
> DISALLOW_EVIL_CONSTRUCTOR -> DISALLOW_COPY_AND_ASSIGN  
> --run_locally -> --extra_logging

#### 추가적인 정보를 이름에 추가하기
> 16진수 문자열을 담고 있는 변수
> string id; -> string hex_id; 

단위를 포함하는 값들  
> var start = (new Date()).getTime(); -> var `start_ms` = (new Date()).getTime();  
> Start(int delay) -> Start(int `delay_secs`)  
> CreateCache(int size) -> CreateCache(int `size_mb`)  
> ThrottleDownload(flot limit) -> ThrottleDownload(flot `max_kbps`)  
> Rotate(float angle) -> Rotate(float `degrees_cw`)

다른 중요한 속성 포함하기

| 상황                                                     | 변수명      | 더 나은 이름            |
|--------------------------------------------------------|----------|--------------------|
| 패스워드가 'plaintext' 에 담겨있고, 추가적인 처리를 하기전에 반드시 암호화 되야 한다. | password | plaintext_password |
| 사용자에게 보여지는 설명문이 화면에 나타나기 전에 이스케이프 처리가 되어야 한다.          | comment  | unescaped_comment  |
| html의 바이트가 UTF-8 으로 변환되었다.                             | html     | html_utf8          |
| 입력데이터가 url encoded 되었다.                                | data     | data_urlenc        |

#### 이름은 얼마나 길어야 하는가?
좁은 범위(scope) 에서는 짧은 이름이 괜찮다. 즉, 변수의 타입이 무엇인지, 초기값이 무엇인지 그것이 어떻게 사라지는지 등과 같은 변수가 담고 있는 모든 정보가 쉽게 한누에 보이므로 짧은 이름을 사용해도 상관없다.

반대로, 어떤 이름이 큰 범위를 갖는다면, 이름에 의미를 분명하게 만들기 위한 정보를 충분히 포함해야 한다.

```
    if (debug) {
	    Map<String, int> m;
	    LookUpNamesNumbers(m);
	    Print(m);
    }
```

약어와 축약형 

특정 프로젝트에 국한된 의미를 가진 약어나 축약형 사용은 좋은 생각이 아니다.   
단, 팀에 새로 함류한 사람이 이름이 의미하는 바를 이해할 수 있다면 그 이름은 약어나 축약형을 사용해도 괜찮은 것이다.

| 단어         | 약어   |
|------------|------|
| evaluation | eval |
| document   | doc  |
| string     | str  |

잘못된 예)
> BackEndManager -> BEM 

불필요한 단어 제거하기

경우에 따라, 아무런 정보를 손실하지 않으면서 이름에 포함된 단어를 제거할 수도 있다.

| 단어                | 불필요 단어 제거   |
|-------------------|-------------|
| ConvertToString() | ToString()  |
| DoServeLoop()     | ServeLoop() |

#### 이름 포맷팅으로 의미를 전달하라

밑줄과 대시, 대문자를 잘 이용하면 이름에 더 많은 정보를 담을 수 있다.
```
static const int kMaxOpenFiles = 100;

class LogReader {
    public:
        void OpenFile(string local_file);
    private:
        int offset_;
        DISALLOW_COPY_AND_ASSIGN(LogReader);
};
```
문법적 차이가 드러나게 서로 다른 개체의 이름에 각자 다른 포맷팅 방법을 적용하는 방법은 코드를 더 읽기 쉽게 해준다

| 단어       | 불필요 단어 제거       |
|----------|-----------------|
| 클래스명     | CamelCase       |
| 변수명      | lower_separated |
| 클래스 멤버변수 | _ 로 끝남          |

다른 포맷팅 관습으로 이름에 더 많은 정보를 담을 수 있는 경우가 있다.
```javascript
var x = new DatePicker(); // DatePicker 는 '생성자' 함수
var y = pageHeight(); // pageHeight() 는 일반 함수
var $all_images = $("img") // $all_images 는 jquery 객체
```
```html
<div id="middle_column" class="main-content"></div> <!-- id는 밑줄, class 는 대시를 사용 -->
```

### 마무리

코드의 가독성에 대해서 많은 고민이 필요함과 변수, 메소드, 클래스 명 하나 짓는 데에도 많은 고민이 필요함을 다시 느끼게 되었던 것 같습니다.

그럼 이만. 🥕👋🏼🖐🏼