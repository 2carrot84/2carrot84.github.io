---
title: "[책] 읽기 좋은 코드가 좋은 코드다 - 4장"
excerpt_separator: "<!--more-->"
categories:
  - book
tags:
  - 클린코드, clean code, 읽기 좋은 코드, 좋은코드
---

지난번 포스팅에 이어, '읽기 좋은 코드가 좋은 코드다' 책의 4장 '미학' 에 대한 내용을 정리해 봅니다.

[책 링크](https://book.naver.com/bookdb/book_detail.nhn?bid=6871807)

<!--more-->

### 4 미학

좋은 소스코드는 '눈을 편하게' 해야 한다. 우리는 이번 장에서 빈칸, 정렬, 코드의 순서를 이용하여 읽기 편한 소스코드를 작성하는 방법을 살펴볼 것이다.

- 코드를 읽는 사람이 이미 친숙한, 일관성 있는 레이아웃을 사용하라
- 비슷한 코드는 서로 비슷해 보이게 만들어라
- 서로 연관된 코드는 하나의 블록으로 묶어라

#### 미학이 무슨 상관인가?
```java
class StatsKeeper {
public:
    void Add(double d);
	private: int count
    public : 
        double Average();
private: double minimum;
list<double>
    past_items
    ; double maximum;
}
```
다음 코드보다 위 코드를 이해하는 데 시간이 오래 걸릴 것이다.
```java
class StatsKeeper {
    public:
        void Add(double d);
        double Average();
		
    private: 
        list<double> past_items;
        int count;
        double minimum;
        double maximum;
}
```
미학적으로 보기 좋은 코드가 사용하기 더 편리하다는 사실은 명백하다.

#### 일관성과 간결성을 위해서 줄 바꿈을 재정렬하기
```java
public class PerformanceTester {
	public static final TcpConnectionSimulator wifi = new TcpConnectionSimulator(
		500, /* kbps */
        80, /* millisecs 대기시간 */
        200, /* 흔들림 */
        1 /* 패킷 손실 % */
	);
	
	public static final TcpConnectionSimulator t3_fiber = 
        new TcpConnectionSimulator(
		45000, /* kbps */
        10, /* millisecs 대기시간 */
        0, /* 흔들림 */
        0 /* 패킷 손실 % */
	);
	
	public static final TcpConnectionSimulator cell = new TcpConnectionSimulator(
		100, /* kbps */
        400, /* millisecs 대기시간 */
        250, /* 흔들림 */
        5 /* 패킷 손실 % */
	);
}
```
위 코드의 t3_fiber 실루엣이 이상하게 보여 불필요한 주목을 받는다. 이는 '비슷한 코드는 비스스하게 보여야 한다' 원리에 위배되는 사례다.
```java
public class PerformanceTester {
	public static final TcpConnectionSimulator wifi = 
        new TcpConnectionSimulator(
            500, /* kbps */
            80, /* millisecs 대기시간 */
            200, /* 흔들림 */
            1 /* 패킷 손실 % */
	);
	
	public static final TcpConnectionSimulator t3_fiber = 
        new TcpConnectionSimulator(
            45000, /* kbps */
            10, /* millisecs 대기시간 */
            0, /* 흔들림 */
            0 /* 패킷 손실 % */
	);
	
	public static final TcpConnectionSimulator cell = 
        new TcpConnectionSimulator(
            100, /* kbps */
            400, /* millisecs 대기시간 */
            250, /* 흔들림 */
            5 /* 패킷 손실 % */
	);
}
```
이 코드는 일광성 있는 패턴을 가지므로 훑어보기 용이하나, 수직 방향으로 너무 많은 빈칸을 사용하고, 똑같은 주석도 세번씩 반복되고 있다.
```java
public class PerformanceTester {
	// TcpConnectionSimulator ( 처리량, 지연속도, 흔들림, 패킷 손실)
    //                          [kbps]  [ms]  [ms]  [percent]
	public static final TcpConnectionSimulator wifi = 
        new TcpConnectionSimulator(500, 80, 200, 1);
	
	public static final TcpConnectionSimulator t3_fiber = 
        new TcpConnectionSimulator(45000, 10, 0, 0);
	
	public static final TcpConnectionSimulator cell = 
        new TcpConnectionSimulator(100, 400, 250, 5);
}
```
주석을 맨 위로 올리고 모든 파라미터를 한 줄에 놓았다.

#### 메소드를 활용하여 불규칙성을 정리하라
```java
// 'Doug Adams' 처럼 간단하게 쓰인 partial_name 을 'Mr. Douglas Adams' 로 바꾼다.
// 그게 가능하지 않으면, 이유와 함께 'error'가 채워진다.
string ExpandFullName(DatabaseConnection dc, string partial_name, string* error);
```
그리고 이 함수에 대해 아래와 같은 테스트 코드가 있다.
```java
DatabaseConnection database_connection;
string error;

assert(ExpandFullName(database_connection, "Doug Adams", &error) == "Mr. Douglas Adams");
assert(error == "");
assert(ExpandFullName(database_connection, " Jake Brown", &error) == "Mr. Jacob Brown III");
assert(error == "");
assert(ExpandFullName(database_connection, "No Such Guy", &error) == "");
assert(error == "no match found");
assert(ExpandFullName(database_connection, "John", &error) == "");
assert(error == "more than one result");
```
이 코드는 아름답지 않고 일관성 있는 패턴도 결여되었다.
```java
CheckFullName("Doug Adams", "Mr. Douglas Adams", "");
CheckFullName(" Jake Brown", "Mr. Jacob Brown III", "");
CheckFullName("No Such Guy", "", "no match found");
CheckFullName("John", "", "more than one result");
```
우리는 코드가 미학적으로 더 개선되는 걸 의도했지만, 이런한 수정에는 다음과 같이 원래 의도하지 않았던 장점도 있다.

- 중복된 코드를 없애서 코드를 더 간결하게 한다.
- 이름이나 에러 문자열 같은 테스트의 중요 부분들이 한 눈에 보이게 모아졌다.
- 새로운 테스트 추가가 훨씬 쉬워졌다.

#### 의미 있는 순서를 선택하고 일관성 있게 사용하라
```javascript
details     = request.POST.get('details')
location    = request.POST.get('location')
phone       = request.POST.get('phone')
email       = request.POST.get('email')
url         = request.POST.get('url')
```
예를 들어 위와 같은 변수 다섯개는 어떤 순서로 정의되어도 상관이 없는 경우, 의미 있는 순서로 나열하는 편이 낫다.

- 변수의 순서를 HTML 폼에 있는 `<input>` 필드의 순서대로 나열하라.
- '가장 중요한 것'에서 시작해서 '가장 덜 중요한 것'까지 순서대로 나열하라.
- 알파벳 순서대로 나열하라.

#### 선언문을 블록으로 구성하라
```javascript
class FrontendServer {
    public:
        FrontendServer();
        void ViewProfile(HttpRequest* request);
        void OpenDatabase(string location, string user);
        void SaveProfile(HttpRequest* request);
        string ExtractQueryParam(HttpRequest* request, string param);
        void ReplyOK(HttpRequest* request, string html);
        void FindFriends(HttpRequest* request);
        void ReplyNotFound(HttpRequest* request, string error);
        void CloseDatabase(string location);
        ~FrontendServer();
}
```
이 코드는 읽는 사람이 메소드 전체를 쉽게 파악하도록 전체적인 레이아웃이 구성되지는 않았다.
아래와 같이 논리적 영역에 따라서 여러 개의 그룹으로 나누면 더 좋을 것이다.
```javascript
class FrontendServer {
    public:
        FrontendServer();
        ~FrontendServer();
        // 핸들러들
        void ViewProfile(HttpRequest* request);
        void SaveProfile(HttpRequest* request);
        void FindFriends(HttpRequest* request);
        // 질의/응답 유틸리티
        string ExtractQueryParam(HttpRequest* request, string param);
        void ReplyOK(HttpRequest* request, string html);
        void ReplyNotFound(HttpRequest* request, string error);
        // 데이터베이스 헬퍼들
        void OpenDatabase(string location, string user);
        void CloseDatabase(string location);
}
```
#### 코드를 '문단'으로 쪼개라
일반 텍스트가 여러 개의 문단으로 나뉘어진 데에는 이유가 있다

- 비슷한 생각을 하나로 묶어서, 성격이 다른 생각과 구분한다.
- 문단은 '시각적 디딤돌' 역활을 수행한다. 문단이 없으면 하나의 페이지 안에서 읽던 부분을 놓치기 쉽다.
- 하나의 문단에서 다른 문단으로의 전진을 촉진시킨다.

```javascript
# 사용자의 이메일 주소를 읽어 들여 시스템에 존재하는 사용자와 매치시킨다.
# 그 다음 해당 사용자와 이미 친구인 사람들의 리스트를 나타낸다.    
def suggest_new_friends(user, email_password);
    friends = user.friends();
    friends_emails = set(f.email for f in friends)
    contacts = import_contacts(user.email, email_password)
    contact_emails = set(c.email for c in contacts)
    non_friend_emails = contact_emails - friends_emails
    suggested_friends = User.objects.select(email_in=non_friend_emails)
    display['user'] = user
    display['friends'] = friends
    display['suggested_friends'] = suggested_friends
    return render('suggested_friends.html', display)
```
뚜렷하게 드러나지는 않지만, 이 함수는 구별되는 여러 단계를 거친다. 따라서 그러한 줄을 여러 문단으로 나누면 확실히 도움이 된다.
```javascript
def suggest_new_friends(user, email_password);
    # 사용자 친구들의 이메일 주소를 읽는다.
    friends = user.friends();
    friends_emails = set(f.email for f in friends)
        
    # 이 사용자의 이메일 계정으로부터 모든 이메일 주소를 읽어들인다.    
    contacts = import_contacts(user.email, email_password)
    contact_emails = set(c.email for c in contacts)
        
    # 아직 친구가 아닌 사용자들을 찾는다    
    non_friend_emails = contact_emails - friends_emails
    suggested_friends = User.objects.select(email_in=non_friend_emails)

    # 사용자 리스트를 화면에 출력한다.
    display['user'] = user
    display['friends'] = friends
    display['suggested_friends'] = suggested_friends

    return render('suggested_friends.html', display)
```
#### 개인적인 스타일 대 일관성
```javascript
class logger {
    ...
}
```
혹은
```javascript
class logger 
{
    ...
}
```
두 가지 스타일 중에서 어느 하나를 선택한다고 가독성에 실질적인 영향을 주지는 않는다. 하지만 두 스타일이 뒤섞이면 가독성에 영향을 준다.
**일관성 있는 스타일은 '올바른' 스타일보다 더 중요하다.**

### 마무리

코드 작성간 들여쓰기, 줄바꿈 등을 이용해 읽기 편하게 만들고, 코드를 일관성 있게 의미있는 방식으로 유지할 수 있도록 신경을 써야 한다는 장인것 같습니다.

그럼 이만. 🥕👋🏼🖐🏼