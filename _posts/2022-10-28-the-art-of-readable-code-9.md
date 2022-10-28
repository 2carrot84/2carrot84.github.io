---
title: "[책] 읽기 좋은 코드가 좋은 코드다 - 10장"
excerpt_separator: "<!--more-->"
categories:
  - book
tags:
  - 클린코드, clean code, 읽기 좋은 코드, 좋은코드
---

지난번 포스팅에 이어, '읽기 좋은 코드가 좋은 코드다' PART 3 코드 재작성하기 의 10장 '상관없는 하위문제 추출하기' 에 대한 내용을 정리해 봅니다.

[책 링크](https://book.naver.com/bookdb/book_detail.nhn?bid=6871807)

<!--more-->

## PART TWO 코드 재작성하기

## 10 상관없는 하위문제 추출하기

엔지니어링은 커다란 문제를 작은 문제들로 쪼갠 다음, 각각의 문제에 대한 해결책을 구하고, 다시 하나의 해결책으로 맞추는 일련의 작업을 한다.

이 장에서는 큰 흐름과 관계가 적은 하위문제를 적극적으로 발견해서 추출하라는 것이다.

1. 상위수준에서 본 이 코드의 목적은 무엇인가?
2. 이 코드는 직접적으로 목적을 위해서 존재하는가? 목적 자체와 직접적으로 상관없는 하위문제를 해결하는가?
3. 목적과 직접적으로 관련되지 않은 하위문제를 해결하는 코드 분량이 많으면, 이를 추출해서 별도의 함수로 만든다.

핵심은 전체 목적에 직접 상관없는 하위문제를 다루는 코드를 적극적으로 찾으려고 노력하는 것이다.

### 소개를 위한 예: findClosestLocation()
```javascript
var findClosestLocation = function (lat, lng, array) {
    var closest;
    var closest_dist = Number.MAX_VALUE;
    
    for (var i = 0; i < array.length; i += 1) {
        var lat1_rad = radians(lat);
        var lng1_rad = radians(lng);
        var lat2_rad = radians(array[i].latitude);
        var lng2_rad = radians(array[i].longitude);
        
        var dist = Math.acos(Math.sin(lat1_rad) * Math.sin(lat2_rad) +
                             Math.cos(lat1_rad) * Math.cos((lat2_rad) *
                             Math.cos(lng2_rad - lng1_rad)));
        
        if (dist < closest_dist) {
            closest = array[i];
            closest_dist = dist;
        }
    }
    return closest;
}
```

루프의 내부에 있는 코드는 대부분 주요 목적과 상관없는 하위문제를 다룬다. 이 내용의 분량이 많으니 별도의 함수로 추출하는 편이 좋다.

```javascript
var spherical_distance = function (lat1, lng1, lat2, lng2) {
    var lat1_rad = radians(lat1);
    var lng1_rad = radians(lng1);
    var lat2_rad = radians(lat2);
    var lng2_rad = radians(lng2);

    return Math.acos(Math.sin(lat1_rad) * Math.sin(lat2_rad) +
        Math.cos(lat1_rad) * Math.cos((lat2_rad) *
            Math.cos(lng2_rad - lng1_rad)));
}
```
```javascript
var findClosestLocation = function (lat, lng, array) {
    var closest;
    var closest_dist = Number.MAX_VALUE;

    for (var i = 0; i < array.length; i += 1) {
        var dist = spherical_distance(lat, lng, array[i].latitude, array[i].longitude);
        if (dist < closest_dist) {
            closest = array[i];
            closest_dist = dist;
        }
    }
    return closest;
}
```
코드를 읽는 사람은 밀도 높은 기하 공식에 방해받지 않고 상위수준의 목적에 집중할 수 있으니 전반적으로 코드의 가독성이 높아졌다.

### 순수한 유틸리티 코드
문자열 변경, 해시테이블 사용, 파일 읽기/쓰기와 같이 프로그램이 수행하는 일에는 매우 기본적인 작업을 포괄하는 핵심적인 집합이 있다.

이러한 '기본적인 유틸리티'는 해당 프로그래밍 언어에 내장된 라이브러리에 있다. 예를 들어 파일의 전체 내용을 읽으려면, PHP에서는 file_get_contents("filename")을

파이썬에서는 open("filename").read()를 사용한다.

이는 별도의 함수로 추출되어야 하는 직접 상관없는 하위 문제를 다루는 함수의 전형적인 사례다. 

### 일반적인 목적의 코드
자바스크립트를 디버깅할 때 프로그래머는 보통 alert() 으로 일정한 정보를 출력하는 메시지 박스를 화면에 나타낸다. 예를 들어 다음 함수는 Ajax를 이용해서 데이터를 서버에 제출하고, 서버가 반환한 데이터 딕셔너리를 화면에 나타낸다.

```javascript
ajax_post({
    url: 'http://example.com/submit',
    data: data,
    on_success: function (response_data) {
        var str = "{\n";
        for (var key in response_data) {
            str += " " + key + " = " + response_data[key] + "\n";
        }
        alert(str + "}");
        ...
    },
})
```
이 코드의 상위수준 목적은 서버에 Ajax 호출을 하고 응답을 처리하는 것이다. 하지만 코드의 대부분이 이러한 목적과 상관없는 하위문제를 처리하는 일을 한다.

이러한 코드를 추출해서 어렵지 않게 format_pretty(obj) 함수로 만들어보자.
```javascript
var format_pretty = function (obj) {
    var str = "{\n";
    for (var key in response_data) {
        str += " " + key + " = " + response_data[key] + "\n";
    }
    return str + "}";
}
```

#### 뜻하지 않은 장점들
format_pretty() 를 추출하면 장점이 많다. 함수를 호출하는 코드를 간단하게 만들고, format_pretty()를 다룬 곳에서도 간편하게 사용할 수 있는 함수로 만들기 때문이다.

필요할 때 format_pretty() 함수를 훨씬 손쉽게 개선할 수 있다. 별도로 분리된 작은 함수를 다룰 때는 기능을 더하고, 가동성을 개선하고, 코너케이스를 다루는 일이 상대적으로 쉽게 느껴지기 때문이다.

format_pretty(obj) 가 다루지 않는 기능은 다음과 같다.
- obj를 객체라고 간주한다. 문자열(혹은 undefined)이면 예외가 발생한다.
- obj의 값이 간단한 타입이라고 생각한다. 중첩된 객체면 현재 코드는 [object Object] 라는 결과를 출력할 것이다

```javascript
var format_pretty = function (obj, indent) {
    if (obj === null) return "null";
    if (obj === undefined) return "undefined";
    if (typeof obj === "string") return '"' + obj + '"';
    if (typeof obj === "object") return String(obj);
    
    if (indent === undefined) indent = "";
    
    var str = "{\n";
    for (var key in obj) {
        str += indent + " " + key + " = ";
        str += format_pretty(obj[key], indent + " ") + "\n";
    }
    return str + indent + "}";
}
```

### 일반적인 목적을 가진 코드를 많이 만들어라
ReadFileToString() 과 format_pretty() 는 상관없는 하위문제를 다루는 대표적인 함수다. 이들은 매우 기본적이고 폭넓게 적용할 수 있는 일을 수행하므로 다른 프로젝트에서도 사용할 수 있다.

일반적인 목적을 가진 코드는 프로젝트의 나머지 부분에서 완전히 분리되므로 좋다.

SQL 데이터베이스, 자바스크립트 라이브러리, HTML 템플릿 시스템과 같이 여러분이 사용하는 강력한 라이브러리와 시스템을 생각해보라. 이들의 내부는 염려할 필요가 없다.

이들 코드베이스는 여러분 프로젝트에서 완전히 분리되어 있다. 때문에 여러분 코드베이스는 그만큼 작아질 수 있다.

프로젝트에서 사용하는 코드의 더 많은 부분이 이렇게 별도의 라이브러리로 만들어질 수록 더 좋다.

### 특정한 프로젝트를 위한 기능
이상적인 상황이라면 여러분이 추출한 하위문제는 사용하는 프로젝트를 전혀 몰라야 한다. 하위문제를 분리하는 것만으로도 큰 도움이 되기 때문이다.
```javascript
business = Business()
business.name = request.POST["name"]

url_path_name = business.name.lower()
url_path_name = re.sub(r"['\.", "", url_path_name)
url_path_name = re.sub(r"[^a-z0-9]+", "-", url_path_name)
url_path_name = url_path_name.strip("-")
business.url = "/biz/" + url_path_name

business.date_created = datetime.datetime.utcnow()
business.save_to_database()
```
이 코드에서 전체 목적과 직접 상관없는 하위문제는 name을 유효한 url로 변환하는 일을 한다.

```javascript
CHARS_TO_REMOVE = re.compile(r"['\.]+")
CHARS_TO_DASH = re.compile(r"[^a-z0-9]+")

def make_url_friendly(text):
    text = text.lower()
    text = CHARS_TO_REMOVE.sub('', text)
    text = CHARS_TO_DASH.sub('-', text)
    return text.strip("-")
```
```javascript
business = Business()
business.name = request.POST["name"]
business.url = "/biz/" + make_url_friendly(business.name)
business.date_created = datetime.datetime.utcnow()
business.save_to_database()
```
결과적으로 이 코드를 읽으면서 make_url_friendly()가 추출되어 정규표현식이나 복잡한 문자열 처리를 신경 쓰지 않아도 되므로 코드의 가독성이 더 좋아졌다. 

### 기존의 인터페이스를 단순화하기
적은 수의 인수를 받고, 별다른 설정을 요구하지 않으며, 사용하기 간편한 인터페이스가 좋다. 이러한 인터페이스 코드를 우아하게 만든다.
다음은 'max_results'라는 이름을 가진 쿠키의 값을 읽는 코드이다.
```javascript
var max_results;
var cookies = document.cookie.split(";");
for (var i = 0; i < cookies.length; i++) {
    var c = cookies[i];
    c = c.replace(/^[]+/, '');
    if (c.indexOf("max_results") === 0)
        max_results = Number(c.substring(12, c.length));
}
```
위 지저분한 코드를 다음과 같이 사용할 수 있는 get_cookie() 함수를 만들어야 할것처럼 보인다.
```javascript
var max_results = Number(get_cookie("max_results"));
```
쿠키의 값을 생성하거나 변경하는 작업은 더욱 이상하다.
```javascript
document.cookie = "max_results=50; expires=Wed, 1 Jan 2020 20:53:47 UTC; path=/";
```
쿠키를 설정하는 더 좋은 인터페이스는 다음과 같다.
```javascript
set_cookie(name, value, days_to_expire);
```
쿠키를 제거하는 작업도 직관에 어긋난다. 다음과 같은 인터페이스가 더 낫다.
```javascript
delete_cookie(name);
```
여기서 **이상적이지 않은 인터페이스를 그냥 받아들일 이유는 없다**는 교훈을 얻을 수 있다. 이런 인터페이스가 있으면 언제나 이를 둘러싸는 함수를 작성하여 지저분한 내부를 감출 수 있다.

### 자신의 필요에 맞춰서 인터페이스의 형태를 바꾸기
프로그램 안에 있는 많은 코드는 다른 코드를 지원하려고 존재한다. 예를 들어 함수에 주어지는 입력을 설정하거나 출력된 결과를 처리하는 일을 수행한다. 이와 같은 '접착'코드는 프로그램의 실제 논리와 별로 직접적인 관련이 없다. 이러한 코드는 따로 분리하여 독자적인 함수를 만들 만하다.
```javascript
user_info = {"username" : "...", "password": "..."}
user_str = json.dumps(user_info)
cipher = Cipher("aes_128_cbc", key=PRIVATE_KEY, init_vector=INIT_VECTOR, op=ENCODE)
encrypted_bytes = cipher.update(user_str)
encrypted_bytes += cipher.final()
url = "http://example.com/?user_info=" + base64.urlsafe_b64encode(encrypted_bytes)
```
우리는 사용자의 정보를 암호화해서 URL에 넣으면 되는데, 이 코드의 대부분은 해당 파이썬 객체를 암호화해서 URL 친화적인 문자열로 바꾸고 있다. 따라서 그러한 하위 문제를 쉽게 추출할 수 있다.

```javascript
def url_safe_encrypt(obj):
obj_str = json.dumps(obj)
cipher = Cipher("aes_128_cbc", key=PRIVATE_KEY, init_vector=INIT_VECTOR, op=ENCODE)
encrypted_bytes = cipher.update(user_str)
encrypted_bytes += cipher.final()
return base64.urlsafe_b64encode(encrypted_bytes)
```
이렇게 하니 프로그램이 수행하는 본래의 논리가 간단해졌다.
```javascript
user_info = {"username" : "...", "password": "..."}
url = "http://example.com/?user_info=" + url_safe_encrypt(user_info)
```

### 지나치게 추출하기
우리의 목적은 '상관없는 하위문제를 적극적으로 발견하고 추출하는'것이다. 하지만 지나친 수준으로 나아가는 일도 벌이질 수 있다.
```javascript
user_info = {"username" : "...", "password": "..."}
url = "http://example.com/?user_info=" + url_safe_encrypt_obj(user_info)

def url_safe_encrypt_obj(obj):
    obj_str = json.dumps(obj)
    return url_safe_encrypt_str(obj_str)

def url_safe_encrypt_str(data):
    encrypted_bytes = encrypt(data)
    return base64.urlsafe_b64encode(encrypted_bytes)

def encrypt(data):
    cipher = make_cipher()
    encrypted_bytes = cipher.update(user_str)
    encrypted_bytes += cipher.final()
    return encrypted_bytes

def make_cipher():
    return Cipher("aes_128_cbc", key=PRIVATE_KEY, init_vector=INIT_VECTOR, op=ENCODE)
```
이렇게 자잘한 함수를 사용하면 오히려 가독성을 해친다. 사용자가 신경 써야 하는 내용이 늘어나고, 실행 경로를 추적하려면 코드의 곳곳을 돌아다녀야 하기 때문이다.

코드에 새로운 함수를 더하는 일에는 약간의 (분명히) 가독성 비용이 든다. 앞의 예는 이러한 비용을 뛰어넘을 만한 이득이 전혀 없으므로 오히려 가독성이 나빠진 것이다.

### 요약
이 장에서 살펴본 내용을 한마디로 정리하면 **일반적인 목적의 코드를 프로젝트의 특정 코드에서 분리하라**는 것이다. 일반적인 문제를 해결하기 위한 라이브러리와 헬퍼 함수들로 이루어진 집합을 구성하면, 남아있는 코드는 여러분의 프로그램을 독특하게 만드는 작은 핵심에 불과할 것이다.

### 마무리

코드를 짜면서 함수로 분리하는 것은 늘 고민이 많은 작업인거 같습니다. 그 작업을 위한 기준을 알려주는 좋은 장이였다고 생각됩니다.

그럼 이만. 🥕👋🏼🖐🏼