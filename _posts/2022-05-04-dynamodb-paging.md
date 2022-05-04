---
title: "DynamoDB 페이징 처리 방법"
excerpt_separator: "<!--more-->"
categories:
  - development
tags:
  - AWS
  - dynamoDB
  - paging
  - Spring
---

오늘은 DynamoDB 페이징 처리하는 방법에 대해서 간단하게 작성을 해보려고 합니다.

각 DB 마다 페이징처리를 위한 다양한 방법을 제공하는데, DynamoDB 역시 별도의 페이징 처리 방법을 가지고 있었습니다.

<!--more-->
### 개발 환경
- Spring Boot 2.6.7
- Gradle
- IntelliJ

### DynamoDB 페이징 방법
DynamoDB 페이징 하는 방식은 다양하게 있는것 같지만 저는 queryPage 를 호출하여 구현 하였습니다.

동작 방식은 아래와 같습니다.
1. DynamoDBQueryExpression 에 페이지에 노출할 갯수를 limit 로 세팅
2. DynamoDBMapper.queryPage 로 쿼리 호출
3. QueryResultPage<T> 응답값에 LastEvaluatedKey 가 있는 경우 다음 페이지가 존재함 (없는 경우 마지막 페이지)
4. 다음 페이지 호출시 LastEvaluatedKey 를 withExclusiveStartKey 로 세팅하여 queryPage 호출하면, 이전에 조회한 이후 결과값이 조회됨

- 예제 코드
```java
	public QueryResultPage<DisplayCorner> getSiteListPaging(int limit, Map<String, AttributeValue> exclusiveStartKey) {
		Map<String, AttributeValue> eav = new HashMap<>();
	    eav.put(":site_dvs", new AttributeValue().withS("SITE_DVS$M"));

		DynamoDBQueryExpression<DisplayCorner> queryExpression = new DynamoDBQueryExpression<DisplayCorner>()
			.withKeyConditionExpression("site_dvs = :site_dvs")
			.withLimit(limit)   // 페이지당 노출 갯수
			.withExpressionAttributeValues(eav);

		if (exclusiveStartKey != null) {
			queryExpression.withExclusiveStartKey(exclusiveStartKey);   // QueryResultPage 의 LastEvaluatedKey 값
		}

		return dynamoDBMapper.queryPage(DisplayCorner.class, queryExpression);
	}
```
 
위와 같이 구현이 되어 있어 특정 페이지를 바로 조회하는 방법은 없어 보이며, 순차적으로 다음 페이지를 조회하는 방법만 존재하는 것 같습니다.

물론 방법이 존재하는데 제가 못 찾은것 일 수도 있겠지만요 🤣

### 마무리
검색을 하다보면 scan 을 이용하는 방법도 있는 것 같으나, 제가 구현하는 로직에는 queryPage 가 적합해 보여서, 해당 방법으로 구현하였습니다.

구현은 엄청 간단했지만 해당 내용을 찾느라 꽤나 많은 시간을 허비 했던것 같습니다. 다른 분은 저같이 시간 낭비 하지 않으시길 바라며.

그럼 이만. 🥕👋🏼🖐🏼

### 참고자료
[https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Query.Pagination.html](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Query.Pagination.html)