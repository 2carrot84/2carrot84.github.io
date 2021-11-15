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

#### ES(Elasticsearch) 란?

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

#### Query Context 와 Filter Context

ES 데이터 조회 방식은 몇가지 방식(search API 호출, URI search 등)이 있지만, search API 의 Query DSL에 대해 알아보겠습니다.

### 마무리

그럼 이만. 🥕👋🏼🖐🏼

### 참고자료
[https://victorydntmd.tistory.com/313](https://victorydntmd.tistory.com/313)