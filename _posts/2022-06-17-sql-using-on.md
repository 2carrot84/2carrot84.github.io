---
title: "JOIN 시 USING 과 ON의 차이"
excerpt_separator: "<!--more-->"
categories:
- development

tags:
- SQL
- JOIN
- USING
---

오늘은 두 테이블간 JOIN 시 사용되는 두가지 키워드 (USING, ON) 의 차이에 대해 간단히 포스팅 해보도록 하겠습니다.

<!--more-->

우선 10년 넘게 쿼리를 만져 왔지만, USING 이 있다는걸 이제야 알게 된 사실에 놀랐습니다..🤔🤣

저처럼 USING 키워드에 모르는 분들이 있지 않을까 싶어 제가 알게된 내용을 포스팅 하려고 합니다.

### 결론
USING 은 `두 테이블간 조인 조건 컬럼의 컬럼명이 동일한 경우` 사용할 수 있는 키워드 입니다. 즉, 기능은 같다고 할 수 있습니다.

간단한 예제를 보며 두 키워드 사용 방법을 확인해 보겠습니다.

### 예제
```sql
/* ON 을 이용한 JOIN */
SELECT * 
  FROM employees e 
 INNER JOIN salaries s 
    ON e.emp_no = s.emp_no 
 ORDER BY e.emp_no;

/* USING 을 이용한 JOIN */
SELECT *
  FROM employees e
 INNER JOIN salaries s
 USING (emp_no)
 ORDER BY e.emp_no;
```
위와 같이 USING을 사용하면 조금 더 간략하게 쿼리를 완성할 수 있습니다.

### 마무리
컬럼명이 같을 경우 ON, USING 모두 사용 가능하나, `컬럼명이 다른 경우는 ON 만 사용 가능`한 점 유의 하시기 바라며..

그럼 이만. 🥕👋🏼🖐🏼

### 참고자료
[https://linux.systemv.pe.kr/mysql-using-vs-%EC%B0%A8%EC%9D%B4/](https://linux.systemv.pe.kr/mysql-using-vs-%EC%B0%A8%EC%9D%B4/)