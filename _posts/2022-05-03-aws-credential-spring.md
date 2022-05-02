---
title: "AWS Credentials Spring 설정 방법"
excerpt_separator: "<!--more-->"
categories:
  - development
tags:
  - AWS
  - Credentials
  - Spring
---

오늘은 새로운 프로젝트 코드 베이스 세팅을 하면서, DynamoDB 연동을 위해 AWS Credentials 설정 하면서 알게된 방법에 대해 간단히 포스팅 하고자 합니다.

<!--more-->
### 개발 환경
- Spring Boot 2.6.7
- Gradle
- IntelliJ

### DynamoDB 연동 방법
DynamoDB 와 Spring Boot 연동은 매우 간단하게 가능 하였습니다.

1. build.gradle 을 이용하여, AWS DynamoDB SDK Dependency 설정
```groovy
    implementation 'com.amazonaws:aws-java-sdk-dynamodb:1.11.161'
```
2. DynamoDB Configuration 빈 생성
```java
@Configuration
public class DynamoDBConfiguration {
    @Bean
    public DynamoDBMapper dynamoDBMapper(){
        return new DynamoDBMapper(buildAmazonDynamoDB());
    }
    private AmazonDynamoDB buildAmazonDynamoDB(){
        return AmazonDynamoDBClientBuilder
                .standard()
                .withEndpointConfiguration(
                        new AwsClientBuilder.EndpointConfiguration(
                                "dynamodb.us-east-1.amazonaws.com",//endpoint
                                "us-east-1"//Region
                        )
                )
                .withCredentials(
                        new AWSStaticCredentialsProvider(
                                new BasicAWSCredentials(
                                        "access key",//access key
                                        "secert key"//secret key
                                )
                        )
                )
                .build();
    }
}
```
3. 데이터를 조회할 Entity 객체 생성  
```java
@Getter
@Setter
@DynamoDBTable(tableName="DynamoDB 테이블 명")
public class Site {
	@ApiModelProperty(value = "PK (SITE$사이트 번호)")
	@DynamoDBHashKey(attributeName="hk")
	private String hk;

	@ApiModelProperty(value = "PK (SITE)")
	@DynamoDBRangeKey(attributeName="sk")
	private String sk;

	@ApiModelProperty(value = "사이트 번호 ")
	@DynamoDBAttribute(attributeName = "site_no")
	private Integer siteNo;

	@ApiModelProperty(value = "사이트 명")
	@DynamoDBAttribute(attributeName = "site_nm")
	private String siteNm;

	@ApiModelProperty(value = "사업 유형 코드")
	@DynamoDBAttribute(attributeName = "biz_typ_cd")
	private String bizTypCd;
}
```

4. DynamoDBMapper 를 이용한 데이터 조회  
```java
@RequiredArgsConstructor
@Repository
public class SiteRepository {

	private final DynamoDBMapper dynamoDBMapper;

	public Site getSiteBySiteNo(Integer siteNo) {
		return dynamoDBMapper.load(Site.class, /* HashKey */ "SITE$" + siteNo, /* SortKey */ "SITE");
	}
}
```
 
### 추가 작업
구글링을 통해 위와 같은 샘플 코드를 작성 했지만, Credential 관련하여, 코드에 하드코딩을 하지 않고, 환경(local, test, prod) 별로 access key 와 secret key 를 분리 해주고 싶었습니다.

먼저 머릿속으로 생각한 방법은 아래와 같습니다.

1. 환경별 yaml 파일을 분리하여, yaml 파일에 작성
2. ~/.aws/credentials 에 저장된 프로필 사용

위 2번 방법에 대해 구글링을 하다 좋은 방법을 찾았습니다. 바로 `DefaultAWSCredentialsProviderChain` 인스턴스를 사용하면, 아래와 같은 순서로 적합한 자격증명 클래스를 찾아 준다고 합니다.

| 클래스명 |	설명 |
| ---- | ---- |
|BasicAWSCredentials |	직접 accessKey, secretKey 설정 |
|EnvironmentVariableCredentialsProvider |	환경변수 |
|SystemPropertiesCredentialsProvider |	java 시스템 속성 |
|ProfileCredentialsProvider |	~/.aws/credentials 에 저장된 프로필 |
|ContainerCredentialsProvider |	ECS 컨테이너 자격 증명 |
|InstanceProfileCredentialsProvider |	EC2 인스턴스 자격 증명 |
|DefaultAWSCredentialsProviderChain |	위 클래스 순서대로 권한 검사 |

위 2번의 예제 소스를 아래와 같이 수정하였습니다.
```java
@Configuration
public class DynamoDBConfiguration {
	@Bean
	public DynamoDBMapper dynamoDBMapper(){
		return new DynamoDBMapper(buildAmazonDynamoDB());
	}

	@Bean
	public AWSCredentialsProvider awsCredentialsProvider() {
		return new DefaultAWSCredentialsProviderChain();
	}

	private AmazonDynamoDB buildAmazonDynamoDB() {
		return AmazonDynamoDBClientBuilder
			.standard()
			.withEndpointConfiguration(
				new AwsClientBuilder.EndpointConfiguration(
					"dynamodb.us-east-1.amazonaws.com",//endpoint
					"us-east-1"//Region
				)
			)
			.withCredentials(awsCredentialsProvider())  // Credentials
			.build();
	}
}
```

### 마무리
보안에 취약하지 않고, 코드에 access key 와 secret key 가 노출되지 않을 방법을 고민 했는데, 아주 간단히 해결이 되었네요.

그럼 이만. 🥕👋🏼🖐🏼

### 참고자료
[https://eun-dolphin.tistory.com/25](https://eun-dolphin.tistory.com/25)
[https://kouzie.github.io/spring/Spring-Boot-%EC%8A%A4%ED%94%84%EB%A7%81-%EB%B6%80%ED%8A%B8-aws-cloud/#awscredentialsprovider](https://kouzie.github.io/spring/Spring-Boot-%EC%8A%A4%ED%94%84%EB%A7%81-%EB%B6%80%ED%8A%B8-aws-cloud/#awscredentialsprovider)