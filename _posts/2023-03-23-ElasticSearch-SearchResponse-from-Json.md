---
title: "[Test Code] Json 에서 ElasticSearch SearchResponse 객체 변환하기"
excerpt_separator: "<!--more-->"
categories:
- development

tags:
- 테스트 코드 
- ElasticSearch SearchResponse from Json
- ElasticSearch search mock
- Dummy SearchResponse
---

TDD는 아니지만 팀 자체 개선 프로젝트에 테스트 코드를 넣으면서 엄청 삽질한 내용을 포스팅 하고자 합니다.

현재 진행 중인 프로젝트에서 2가지 DB를 사용하는데요, 하나는 DynamoDB 와 나머지는 ElasticSearch 를 사용 중에 있습니다.

단위 테스트 코드를 작성 하다보니 ES 의 응답을 mocking 해야 하는 케이스가 발생하였고, 일반적인 방법으로 mocking 이 되지 않는 다는 것을 알게 되었습니다.
<!--more-->

mocking을 위해서 Mokito 라는 라이브러리를 사용하고 있었는데, RestHighLevelClient 클래스를 아래와 같이 mocking 하고,
```java
private final RestHighLevelClient mockClient = mock(RestHighLevelClient.class);
```
mockClient.search 를 when thenReturn 으로 감싸기 위해 응답값인 SearchResponse 를 Gson 을 이용하여 json 값에서 변환을 시도 하였습니다.

```java
String strJson = "json";
SearchResponse response = new Gson().fromJson(strJson, SearchResponse.class);
when(mockClient.search(searchRequest, RequestOptions.DEFAULT)).thenReturn(response);
```
하지만 respons 변환이 되지 않아, 테스트 실행시 NPE 를 뱉어 냈습니다.
```shell
java.lang.NullPointerException
	at org.elasticsearch.client.RestHighLevelClient.internalPerformRequest(RestHighLevelClient.java:1433)
	at org.elasticsearch.client.RestHighLevelClient.performRequest(RestHighLevelClient.java:1403)
	at org.elasticsearch.client.RestHighLevelClient.performRequestAndParseEntity(RestHighLevelClient.java:1373)
	at org.elasticsearch.client.RestHighLevelClient.search(RestHighLevelClient.java:915)
	at com.lotteon.bigbroapi.api.display.stat.service.StatServiceUnitTest1.getStatModules(StatServiceUnitTest1.java:52)
	...
```

RestHighLevelClient 의 search 메소드는 final 메소드로 mokito를 이용해서는 mocking 이 불가능하다는걸 알았습니다. (하단 참고자료 참고)
```java
public final SearchResponse search(SearchRequest searchRequest, RequestOptions options) throws IOException
```

위와 같은 이유 때문에 PowerMock 이라는 라이브러리를 사용하게 되었고, Json을 SearchResponse 로 변환하는 방법을 계속 찾게 되었습니다.

구글링을 열심히 한 결과 아래와 같은 2가지 메소드를 만들어서 Json을 SearchResponse 로 변환하는 방법을 찾아냈습니다.

```java
private static List<NamedXContentRegistry.Entry> getDefaultNamedXContents() {
    Map<String, ContextParser<Object, ? extends Aggregation>> map = new HashMap<>();
    map.put(TopHitsAggregationBuilder.NAME, (parser, content) -> ParsedTopHits.fromXContent(parser, (String) content));
    map.put(StringTerms.NAME, (parser, content) -> ParsedStringTerms.fromXContent(parser, (String) content));
    map.put(SumAggregationBuilder.NAME, (parser, content) -> ParsedSum.fromXContent(parser, (String) content));
    return map.entrySet()
        .stream()
        .map(entry -> new NamedXContentRegistry.Entry(Aggregation.class,
            new ParseField(entry.getKey()),
            entry.getValue()))
        .collect(Collectors.toList());
}

public static SearchResponse getSearchResponseFromJson(final String json) {
    try {
        NamedXContentRegistry registry = new NamedXContentRegistry(getDefaultNamedXContents());
        XContentParser parser = JsonXContent.jsonXContent.createParser(registry, DeprecationHandler.THROW_UNSUPPORTED_OPERATION, json);
        return SearchResponse.fromXContent(parser);
    } catch (IOException e) {
        throw new RuntimeException("Failed to get resource: " + jsonFileName, e);
    } catch (Exception e) {
        throw new RuntimeException(("exception " + e));
    }
}
```

최종 테스트 코드는 아래와 같습니다.

powerMock 을 사용한 mocking 과 작성한 메소드를 이용한 SearchResponse 변환하여 원하는 응답값을 리턴 받을 수 있게 만들어 단위 테스트를 완료할 수 있었습니다.  

```java
private final RestHighLevelClient mockClient = PowerMockito.mock(RestHighLevelClient.class);

String strJson = "{...}";
SearchResponse response = ElasticObjectHelper.getSearchResponseFromJson(strJson);
when(mockClient.search(searchRequest, RequestOptions.DEFAULT)).thenReturn(response);
```

### 마무리
ES 응답을 모킹 하기 위해서 몇일을 소비한거 같네요. 다른 분들은 저같은 시간 낭비 없이 빠르게 단위테스트 코드 작성할 수 있길 바랍니다.

그럼 이만. 🥕👋🏼🖐🏼

### 참고자료
[https://stackoverflow.com/questions/58914298/nullpointerexception-problem-when-trying-to-mock-elastic-searchs-resthighlevelc](https://stackoverflow.com/questions/58914298/nullpointerexception-problem-when-trying-to-mock-elastic-searchs-resthighlevelc)
[https://forl.tistory.com/142](https://forl.tistory.com/142)