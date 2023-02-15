---
title: "Elastic Search Aggregation 사용시 default size"
excerpt_separator: "<!--more-->"
categories:
- development

tags:
- ES
- Aggregation
- default
- size
---

오늘은 Elastic Search Aggregation 기능 사용간 겪었던 이슈와 그를 해결한 내용을 포스팅 하고자 합니다. 

팀내 자체 개선 취지로 작은 프로젝트를 진행하는 과정에 겪은 트러블 슈팅 내용에 대해 자세히 적도록 하겠습니다.

<!--more-->

저희 팀은 전시 모듈을 관리하는 전시개발팀으로 여러가지 전시 모듈을 만들어서 별도의 추가 개발 없이 모듈 조합으로 매장이나 페이지를 꾸밀 수 있도록 제공하고 있습니다.

이런 모듈들이 얼마나 효과적인지 효과 수치(모듈 노출 수, 모듈 클릭 수)를 ES(Elastic Search) 에 raw 데이터를 쌓아두고, 모듈 ID로 aggregation 기능을 구현하게 되었습니다. 

### 1차 기능 구현 
- 요건 : 모듈 리스트 API에 효과 수치 2가지 항목을 추가하라.
- 구현 방법 : 모듈 갯수 만큼 for 문을 돌며, 노출 수와 클릭 수를 각각 구해와 모듈 리스트 객체에 담아 주었습니다.

위와 같은 방향으로 1차 기능 구현을 마치고, 배포를 하고 났더니 아래와 같은 이슈가 발생 하였습니다.

> 이슈1 : 1초 미만이던 모듈 리스트 API 성능이 7~8초로 느려짐

모듈 갯수 만큼 ES를 조회하니 성능이 나오지 않는게 당연하다고 생각하여, 아래와 같이 2차 기능 구현을 진행 하였습니다.

### 2차 기능 구현
- 요건 : 모듈 리스트 API 의 성능을 개선하라.
- 구현 방법 : 모듈 리스트 갯수 만큼 조회하던 ES 쿼리를 모듈 ID 리스트로 한번에 조회 (ES should 문 사용) 

위와 같은 방법으로 다시 1초 미만의 응답속도를 보장하게 되었지만 아래와 같이 생각하지 못한 이슈가 발생 하였습니다.

> 이슈2 : 모듈 리스트 중 일부 모듈의 효과 수치가 조회되지 않음

처음에는 matchPhrase 를 사용한게 문제인가 해서 Term 으로 변경을 해보았으나, 동일한 현상이 발생되었습니다.

디버깅을 통해 ES 쿼리를 복사하여, kibana 를 통해 결과를 확인 했더니 쿼리 중에 아래와 같은 구문이 있었습니다.

`aggregations > terms > size 가 10으로 고정이 되어, aggregation 결과가 10개만 리턴`되고 있었습니다.

API에 조회하는 모듈의 수가 10개를 넘어가자 10개 이후의 모듈은 aggregation 결과가 조회되지 않는 것이었습니다.

```json
"aggregations":{
      "module_id_aggs":{
         "terms":{
            "field":"module_id",
            "size":10,
            "min_doc_count":1,
            "shard_min_doc_count":0,
            "show_term_doc_count_error":false,
            "order":[
               {
                  "_count":"desc"
               },
               {
                  "_key":"asc"
               }
            ]
         },
         "aggregations":{
            "click_cnt_sum":{
               "sum":{
                  "field":"click_cnt"
               }
            },
            "unique_click_cnt_sum":{
               "sum":{
                  "field":"unique_click_cnt"
               }
            }
         }
      }
   }
```

aggregation 기본값이 10으로 고정되어 있는 부분을 class 를 분석해보니 아래 클래스들에 설정이 되어 있는걸 확인할 수 있었습니다.

```java
TermsAggregationBuilder.class

static final TermsAggregator.BucketCountThresholds DEFAULT_BUCKET_COUNT_THRESHOLDS = new TermsAggregator.BucketCountThresholds(1, 0, 10,
	-1);
```
```java
TermsAggregator.class

public abstract class TermsAggregator extends DeferableBucketAggregator {

	public static class BucketCountThresholds implements Writeable, ToXContentFragment {
		private long minDocCount;
		private long shardMinDocCount;
		private int requiredSize;
		private int shardSize;

		public BucketCountThresholds(long minDocCount, long shardMinDocCount, int requiredSize, int shardSize) {
			this.minDocCount = minDocCount;
			this.shardMinDocCount = shardMinDocCount;
			this.requiredSize = requiredSize;
			this.shardSize = shardSize;
		}
        ...
	}
}
```

### 3차 기능 구현
- 요건 : 모듈 리스트 모든 모듈의 효과 수치를 나타내라.
- 구현 방법 : 모듈 리스트 갯수를 TermsAggregationBuilder size 에 셋팅

```java
TermsAggregationBuilder aggregationBuilder = AggregationBuilders.terms("module_id_aggs")
			.field(ModuleImpressionStatField.MODULE_ID.getFieldName());
		aggregationBuilder.size(moduleIds.size());
```

위와 같이 3번의 수정을 통해 기존 성능을 보장하는 모듈 리스트 API를 상용에 배포할 수 있게 되었습니다.

처음 aggregation 결과가 전체 모듈 갯수 만큼 나오지 않을때 원인을 찾느라 많이 해맸는데, 다른 분들은 저와 같은 삽질을 하지 않았으면 하는 마음에 해당 포스팅을 남깁니다.

### 마무리
어떤 기능을 사용할때 default 값이나 기능 제한이 있는지 한번쯤 확인 해보고 구현하는 것도 좋은 습관일꺼 같습니다.

그럼 이만. 🥕👋🏼🖐🏼