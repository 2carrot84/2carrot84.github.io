---
title: "Elasticsearch Query DSL"
excerpt_separator: "<!--more-->"
categories:
  - development
tags:
  - elasticsearch
  - DSL
---

이직 후 2달이 지난 지금 시점에서야 데이터 조회 방법을 학습 해보기로 마음 먹었습니다..😅

현재 직장에서는 3가지의 종류(MySQL, DynamoDB, Elasticsearch)의 데이터 저장소를 사용하는데, 저희 모듈 메인 DB인 DynamoDB 보다 먼저 ES(Elasticsearch) 의 데이터 조회 방법을 알아 보고자 합니다.

<!--more-->

### ES(Elasticsearch) 란?

우선 데이터 조회 방법을 포스팅 하기전에 ES가 무엇인지 간략하게 알아보는게 좋을 것 같습니다.

ES 공식 홈페이지에는 아래와 같이 설명이 되어 있습니다.

> Elasticsearch는 텍스트, 숫자, 위치 기반 정보, 정형 및 비정형 데이터 등 모든 유형의 데이터를 위한 무료 검색 및 분석 엔진으로 분산형 및 개방형을 특징으로 합니다.   
> 
> Elasticsearch는 Apache Lucene을 기반으로 구축되었으며, Elasticsearch N.V.(현재 명칭 Elastic)가 2010년에 최초로 출시했습니다.  
> 
> 간단한 REST API, 분산형 특징, 속도, 확장성으로 유명한 Elasticsearch는 데이터 수집, 보강, 저장, 분석, 시각화를 위한 무료 개방형 도구 모음인 Elastic Stack의 핵심 구성 요소입니다.  
> 
> 보통 ELK Stack(Elasticsearch, Logstash, Kibana의 머리글자)이라고 하는 Elastic Stack에는 이제 데이터를 Elasticsearch로 전송하기 위한 경량의 다양한 데이터 수집 에이전트인 Beats가 포함되어 있습니다.

간단하게 말하면, 검색 엔진 또는 데이터 수집/분석/모니터링 등 대량의 데이터를 다룰 수 있는 오픈 소스 시스템 이라고 이해하면 될 것 같습니다.

### Query Context 와 Filter Context

ES 데이터 조회 방식은 2가지 방식(search API, URI search)이 있지만, search API 에서 사용할 수 있도록 JSON 스타일의 도메인 전용 언어인 Query DSL에 대해 알아보겠습니다.

> Query Context : Query Context 에 사용되는 query 절이 해당 query 절과 얼마나 일치하는지 점수화

> Filter Context : Filter Context 에 사용되는 query 절이 해당 query 절과 일치 여부

#### 두 Context 의 차이점

| Query | Filter |
|---|---|
| Relevance | Yes or No |
| Full Text | Exact value |
| Not Cached | Cached |
| Scoring | No Scoring |

### Query DSL 설명
#### match_all, match_none
> match_all : 모든 document 조회
> match_none : 모든 document 를 가져오고 싶지 않을때 사용 (🙄🤔)

#### match
> match : text, 숫자, 날짜 허용  
> 표준 SQL 의 like '%{keyword}%' 와 유사하나, 검색 keyword 를 analyze 함  
> "operator":<operator> 를 사용하여, 결과를 다르게 만들 수 있음
>   
> address 에 `mill lane` 이 포함된 document 가 아니라, `mill` 또는 `lane` 이 포함된 document 가 결과로 반환   
> ```json
> GET /bank/_search
> {
> "query": { "match": { "address": "mill lane" } }
> } 
> ```

#### match_phrase
> match_phrase : token 과 일치하는 keyword 가 모두 존재하고, 순서도 순차적으로 동일한 document 만 검색  
> 표준 SQL 의 like '%{keyword}%' 과 일치
> slop 이라는 옵션을 이용해 slop에 지정된 값 만큼 단어 사이에 다른 "검색어"(=단어)가 끼어드는 것을 허용
> 
> address 에 `mill lane` 이 포함된 document 만 결과로 반환
> ```json
> GET /bank/_search
> {
> "query": { "match_phrase": { "address": "mill lane" } }
> } 
> ```

#### bool
> bool : bool 로직을 사용하는 쿼리

> must : bool must 절에 지정된 모든 query 가 일치하는 document 조회 
> 표준 SQL 의 where 절 and 와 동일
>
> address 에 `mill` 과 `lane` 이 모두 포함된 document 만 결과로 반환
> ```json
> GET /bank/_search
> {
>   "query": {
>     "bool": {
>       "must": [
>         { "match": { "address": "mill" } },
>         { "match": { "address": "lane" } }
>       ]
>     }
>   }
> }
> ```

> should : bool should 절에 지정된 모든 query 중 하나라도 일치하는 document 조회
> 표준 SQL 의 where 절 or 와 동일
>
> address 에 `mill` 또는 `lane` 이 포함된 document 를 결과로 반환
> ```json
> GET /bank/_search
> {
>   "query": {
>     "bool": {
>       "should": [
>         { "match": { "address": "mill" } },
>         { "match": { "address": "lane" } }
>       ]
>     }
>   }
> }
> ```

> must_not : bool must 절에 지정된 모든 query 가 일치하지 않는 document 조회
> 표준 SQL 의 where 절 not exists 와 동일
>
> address 에 `mill` 또는 `lane` 이 포함되지 않은 document 를 결과로 반환
> ```json
> GET /bank/_search
> {
>   "query": {
>     "bool": {
>       "must_not": [
>         { "match": { "address": "mill" } },
>         { "match": { "address": "lane" } }
>       ]
>     }
>   }
> }
> ```

#### filter
> filter : must 와 같이 filter 절에 지정된 모든 쿼리와 일치하는 document 를 조회하지만, score 를 무시

> range : 범위를 지정하여 범위에 해당하는 값을 갖는 document 를 조회
> - gte : <= , gt : < , lte : >= , lt : > , boost : 검색 가중치
> 
> balance 가 `2000` 이상, `3000` 이하인 document 를 반환
> ```json
> GET /bank/_search
> {
>   "query": {
>     "bool": {
>       "must": { "match_all": {} },
>       "filter": {
>         "range": {
>           "balance": {
>             "gte": 20000,
>             "lte": 30000
>           }
>         }
>       }
>     }
>   }
> }
> ```

#### term
> term : 역색인에 명시된 토큰 중 정확한 키워드가 포함된 document 를 조회
> - "Quick Foxes" 라는 문자열이 있을 때 text 타입은 [quick, foxes]으로 역색인 됩니다.
> - String 필드는 Text 타입(e-mail 본문 같은 전문(full-text)) 또는 keyword 타입(전화번호, 우편번호)
> - Text 타입(match 검색)은 ES 분석기를 통해 역색인이 되는 반면, keyword 타입(term 검색)은 역색인이 되지 않음
> ```json
> GET /bank/_search
> {
>   "query": {
>     "term": {
>       "message": "mill"
>     }
>   }
> }
> 
> -- 결과
> {
>   "took" : 63,
>   "timed_out" : false,
>   "_shards" : {
>     "total" : 5,
>     "successful" : 5,
>     "failed" : 0
>   },
>   "hits" : {
>     "total" : 1000,
>     "max_score" : null,
>     "hits" : [ {
>       "_index" : "bank",
>       "_type" : "account",
>       "_id" : "0",
>       "sort": [0],
>       "_score" : null,
>       "_source" : {"message":"the mill is ..."}
>     }, 
>     { ## 검색 안됨
>     "total" : 1000,
>     "max_score" : null,
>     "hits" : [ {
>       "_index" : "bank",
>       "_type" : "account",
>       "_id" : "0",
>       "sort": [0],
>       "_score" : null,
>       "_source" : {"message":"the mills is ..."}
>     } ...
>     ]
>   }
> }
> 
> 
> ```

#### terms
> terms : 배열에 나열된 역색인된 키워드 중 하나와 일치하는 document를 조회 
> ```json
> {
>   "query": {
>     "terms": {
>       "address": ["street", "place", "avenue"]
>     }
>   }
> }
> ```

#### regexp
> regexp : 정규표현식 term 쿼리  
> 상세 문법 : [링크](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/query-dsl-regexp-query.html#regexp-syntax)

### 집계 실행
> SQL 의 GROUP BY 와 유사  
> state 를 기준으로 모든 계정을 집계
```json
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      }
    }
  }
}
```

#### 응답의 일부
```json
{
  "took": 29,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits" : {
    "total" : 1000,
    "max_score" : 0.0,
    "hits" : [ ]
  },
  "aggregations" : {
    "group_by_state" : {
      "doc_count_error_upper_bound": 20,
      "sum_other_doc_count": 770,
      "buckets" : [ {
        "key" : "ID",
        "doc_count" : 27
      }, {
        "key" : "TX",
        "doc_count" : 27
      }, {
        "key" : "AL",
        "doc_count" : 25
      }, {
        "key" : "MD",
        "doc_count" : 25
      }, {
        "key" : "TN",
        "doc_count" : 23
      }, {
        "key" : "MA",
        "doc_count" : 21
      }, {
        "key" : "NC",
        "doc_count" : 21
      }, {
        "key" : "ND",
        "doc_count" : 21
      }, {
        "key" : "ME",
        "doc_count" : 20
      }, {
        "key" : "MO",
        "doc_count" : 20
      } ]
    }
  }
}
```

> state 의 평균 계정 잔액 집
```json
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
```
> 연령대를(20대, 30대, 40대)를 기준으로, 성별을 그룹화 하여, 연령대, 성별 별 평균 계정 잔액을 구하는 방법
#### 응답
```json
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_age": {
      "range": {
        "field": "age",
        "ranges": [
          {
            "from": 20,
            "to": 30
          },
          {
            "from": 30,
            "to": 40
          },
          {
            "from": 40,
            "to": 50
          }
        ]
      },
      "aggs": {
        "group_by_gender": {
          "terms": {
            "field": "gender.keyword"
          },
          "aggs": {
            "average_balance": {
              "avg": {
                "field": "balance"
              }
            }
          }
        }
      }
    }
  }
}
```
> [집계 참조 가이드](https://www.elastic.co/guide/en/elasticsearch/reference/5.4/search-aggregations.html)

### 마무리

ES Query DSL 의 예시에 대해서 두서 없이 알아 보았습니다.

실제 실무에서 많이 사용을 하면서 좀 더 다양한 문법과 사용 방법을 익혀야 할 것 같네요.

그럼 이만. 🥕👋🏼🖐🏼

### 참고자료
[https://victorydntmd.tistory.com/313](https://victorydntmd.tistory.com/313)  
[https://velog.io/@hanblueblue/Elastic-Search-2](https://velog.io/@hanblueblue/Elastic-Search-2)
