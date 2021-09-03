---
title: "성능 개선 🐢 🔜 🐇"
excerpt_separator: "<!--more-->"
categories:
  - development
tags:
  - 성능 개선
date: 2021-09-03T23:40
---

전 직장에 입사 후 다양한 신규 서비스 개발과 운영 업무를 진행하였지만, 당시 가장 큰 목표는 성능 개선이었습니다.

입사 당시 카톡플친 발송만 되어도 제니퍼가 불바다가 되는 상황이었기에 다양한 방법을 통한 성능 개선이 필요했었죠. 

> 제니퍼의 x-view 가 아래와 같이 되는 상황을 겪어본 사람만이 그 공포감을 알 것입니다. 🤯🤬
> 
> ![x-view](/images/posts/2021/09/x-view.png)

그 과정에서 제가 경험한 간단한 성능 개선 방법들을 정리해보려고 합니다.
<!--more-->

### 스케일 Out & 스케일 Up

먼저 성능 저하가 발생할 때 가장 간단(?)하게 진행할 수 있는 방법은 스케일 Out 과 스케일 Up 이라고 생각합니다.  
비용이 허락한다면 가장 빠르고 단순하게 진행할 수 있는 방법이죠 💸💸💸

> **스케일 Out** - 서버(물리 or 논리)를 추가하여 처리량 분산을 통한 성능개선   
> **스케일 Up** - 서버의 스펙(Cpu, Memory)을 늘려 처리 능력 향상을 통한 성능개선

### 쿼리 튜닝

스케일 Out/Up 을 하여도 원래 응답속도가 느린 로직은 개선이 되지 않죠. 경험상 응답속도가 느린 가장 많은 이유는 쿼리의 수행 시간이 오래 걸리는 경우였습니다.
    
물론, 요즘은 복잡하게 설계된 테이블이 존재하는 RDB 사용 비중이 점점 낮아지고, ORM 이나 NoSql 의 사용이 늘어나는 상황에는 비중이 많이 줄었을것 같네요. (옛날사람..🤔)

> 개발자가 할 수 있는 쿼리 튜닝의 가장 기본은 작성한 쿼리의 인덱스 사용 여부이지 않을까 싶습니다.
> 
> 쿼리가 풀 스캔을 하는지 인덱스 스캔을 하는지에 따라 응답 속도와 자원 사용의 엄청난 차이를 보여줍니다.
> 
> 인덱스를 사용하기 위해서는 조건을 걸때 인덱스가 있는 컬럼을 사용해야 하며, **인덱스 컬럼을 변경하면 안됩니다.**
> 
> 단, 상황에 따라 풀 스캔이 인덱스 스캔 보다 효율적인 경우도 있으니 무조건 풀 스캔이 나쁜 것은 아닙니다.

개발자들이 자주 실수하는 경우를 예로 들어보겠습니다.

- 요건 : 2021년 9월 가입한 고객(CUSTOMER)의 수를 구하자! 
- 인덱스 컬럼 : JOIN_DATE 

잘못된 예
```sql
SELECT COUNT(ID) AS CNT
  FROM CUSTOMER C
 WHERE TO_CHAR(C.JOIN_DATE,'YYYYMM') = '202109'
```
잘된 예
```sql
SELECT COUNT(ID) AS CNT
  FROM CUSTOMER C
 WHERE C.JOIN_DATE BETWEEN TO_DATE('20210901','YYYYMMDD') AND TO_DATE('20210930235959','YYYYMMDDHH24MISS')
```
위와 같이 JOIN_DATE 컬럼을 **_TO_CHAR 함수로 변경하는 과정_**에서 기껏 만들어둔 인덱스를 사용하지 못하게 됩니다.

SQL 튜닝은 너무나 다양한 방법이 있어, 전문가인 튜너나 DBA 분들의 도움이 필요합니다.

대신 그 결과는 개발자인 저로썬 경이로울 정도의 효과적입니다. 👍🏻👍🏻

### 캐시(Cache) 도입

캐시가 도입되기 전에는 트래픽이 증가로 지연이 발생하면, 서버(Web, Was, DB) 스케일 Out & Up 또는 쿼리 튜닝 과정만 반복해 왔습니다.

하지만 **트래픽 분산과 빠른 응답속도**를 보여주는 다양한 캐시 서비스가 생기면서, 웹서비스 아케텍처가 좀 더 복잡해지게 되었죠. 

웹서버 앞에는 웹캐시(ex. Ston)가 DB서버 앞에는 메모리 캐시(ex. Redis)가 도입되며 안정화에 큰 도움을 주게 됩니다.

> 단, 캐싱된 데이터로 인해 고려해야할 것들과 관리 포인트도 많이 늘어나게 되었죠. 🤦🏻

간단한 예를 몇가지 들면
1. 오픈소스인 캐시 서버에 대한 관리는 누가 할것이며, 해당 시스템 Down 시 대책은? 
2. 변경된 데이터가 고객 화면에는 바로 반영이 되지 않는 것에 대한 현업의 불만은?
3. 캐싱된 데이터와 캐싱 되지 않은 데이터는 어떻게 구분하지?
4. 캐싱 정책, 만료 주기, 예외 케이스 등등등..🤢🤢

제가 언급한 캐시 시스템 외에 다양한 캐싱 방법이 존재하며, 필요에 따라 적합한 캐시를 적용하면 확실한 효과를 볼 수 있는 성능 개선 방안이라고 생각합니다.

### 급 마무리
벌써 2시기 다 되어 가는 관계로 성능 개선 관련 포스팅은 다음 편을 기약하며..🥕👋🏼🖐🏼

---

## 참고 자료
스케일 Out & Up 이란? [https://asfirstalways.tistory.com/66](https://asfirstalways.tistory.com/66)  
ORM 이란? [https://gmlwjd9405.github.io/2019/08/04/what-is-jpa.html](https://gmlwjd9405.github.io/2019/08/04/what-is-jpa.html)  
인덱스 란? [https://mangkyu.tistory.com/96](https://mangkyu.tistory.com/96)  
풀 스캔과 인덱스 스캔 이란?   
   - [https://hoon93.tistory.com/53](https://hoon93.tistory.com/53)  
   - [http://wiki.gurubee.net/pages/viewpage.action?pageId=26739956](http://wiki.gurubee.net/pages/viewpage.action?pageId=26739956)  

웹캐시(Ston) : [https://ston.readthedocs.io/ko/latest/admin/intro.html](https://ston.readthedocs.io/ko/latest/admin/intro.html)  
메모리캐시(Redis) : [https://aws.amazon.com/ko/redis/](https://aws.amazon.com/ko/redis/)
