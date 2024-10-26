---
title: "ThreadPoolTaskExecutor 에 대한 짧은 지식"
excerpt_separator: "<!--more-->"
categories:
- development

tags:
- spring
- async
- threadPool
- annotation
---

지난번 @Async 어노테이션에 대해 알아보다 나온 ThreadPoolTaskExecutor 에 대해 간략히 알아보도록 하겠습니다.

> [@Async 사용 방법과 주의사항](https://2carrot84.github.io/development/async/)

<!--more-->

### ThreadPoolTaskExecutor
> 스프링에서 제공하는 스레드풀 관련 클래스 입니다.

@Async 에서 SimpleAsyncTaskExecutor 를 기본값으로 사용하고 있어, ThreadPoolTaskExecutor 얘기가 나오게 되었는데요.

간단한 사용방법은 아래와 같습니다.

```java
	@Bean(name = "taskExecutor")
	public ThreadPoolTaskExecutor executor(){
		ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
		executor.setCorePoolSize(5);
		executor.setQueueCapacity(20);
		executor.setMaxPoolSize(10);
		executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
		return executor;
	}
```

- java.util.concurrent.Executor 을 구현한 클래스
- CorePoolSize : 스레드풀에 속할 기본 스레드 갯수 default. 1
- QueueCapacity : 이벤트 대기 큐 크기 default. Integer.MAX_VALUE
- MaxPoolSize : 최대 스레드 갯수 default. Integer.MAX_VALUE
- RejectedExecutionHandler : QueueCapacity, MaxPoolSize 가 모두 초과한 경우 발생
    - AbortPolicy (default)
        - TaskRejectedException(RejectedExecutionException) 발생
      
```java
	@Async("taskExecutor")
	public void asyncWait() throws InterruptedException {
		log.info(">>>>> asyncWait()");
		Thread.sleep(1000);
	}
```
```java
	@Test
	void rejectedExecutionException() throws InterruptedException {
		for (int i = 0; i < 10; i++) {
			asyncService.asyncWait();
		}
	}
```
```shell
2024-10-26T21:40:14.990+09:00  INFO 8648 --- [demo] [ taskExecutor-1] com.example.demo.async.AsyncService      : >>>>> asyncWait()
org.springframework.core.task.TaskRejectedException: ExecutorService in active state did not accept task: org.springframework.aop.interceptor.AsyncExecutionInterceptor$$Lambda$1776/0x0000007001a8d618@59881424
Caused by: java.util.concurrent.RejectedExecutionException: Task java.util.concurrent.FutureTask@394e0104[Not completed, task = org.springframework.aop.interceptor.AsyncExecutionInterceptor$$Lambda$1776/0x000000e801a90c38@2aea7775] rejected from org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor$1@5502f74c[Running, pool size = 1, active threads = 1, queued tasks = 1, completed tasks = 0]
	at java.base/java.util.concurrent.ThreadPoolExecutor$AbortPolicy.rejectedExecution(ThreadPoolExecutor.java:2065)
	at java.base/java.util.concurrent.ThreadPoolExecutor.reject(ThreadPoolExecutor.java:833)
	at java.base/java.util.concurrent.ThreadPoolExecutor.execute(ThreadPoolExecutor.java:1365)
```
> CorePoolSize QueueCapacity MaxPoolSize 를 모두 1로 변경한 뒤 AbortPolicy 로 테스트 시 위와 같은 에러를 만나게 된다.

   - DiscardOldestPolicy
     - 오래된 작업을 skip (모든 task 가 반드시 처리될 필요 없을때 사용)
   - DiscardPolicy 
     - 처리하려는 작업을 skip (모든 task 가 반드시 처리될 필요 없을때 사용)
   - CallerRunsPolicy
     - shutdown 상태가 아니라면 요청한 caller Thread 에서 직접 처리
     - `태스크 유실 최소화`
```shell
2024-10-26T21:40:49.427+09:00  INFO 8726 --- [demo] [    Test worker] com.example.demo.async.AsyncService      : >>>>> asyncWait()
2024-10-26T21:40:49.427+09:00  INFO 8726 --- [demo] [ taskExecutor-1] com.example.demo.async.AsyncService      : >>>>> asyncWait()
2024-10-26T21:40:50.433+09:00  INFO 8726 --- [demo] [    Test worker] com.example.demo.async.AsyncService      : >>>>> asyncWait()
2024-10-26T21:40:50.433+09:00  INFO 8726 --- [demo] [ taskExecutor-1] com.example.demo.async.AsyncService      : >>>>> asyncWait()
2024-10-26T21:40:51.435+09:00  INFO 8726 --- [demo] [    Test worker] com.example.demo.async.AsyncService      : >>>>> asyncWait()
2024-10-26T21:40:51.435+09:00  INFO 8726 --- [demo] [ taskExecutor-1] com.example.demo.async.AsyncService      : >>>>> asyncWait()
2024-10-26T21:40:52.441+09:00  INFO 8726 --- [demo] [ taskExecutor-1] com.example.demo.async.AsyncService      : >>>>> asyncWait()
2024-10-26T21:40:52.441+09:00  INFO 8726 --- [demo] [    Test worker] com.example.demo.async.AsyncService      : >>>>> asyncWait()
2024-10-26T21:40:53.443+09:00  INFO 8726 --- [demo] [    Test worker] com.example.demo.async.AsyncService      : >>>>> asyncWait()
2024-10-26T21:40:53.447+09:00  INFO 8726 --- [demo] [ taskExecutor-1] com.example.demo.async.AsyncService      : >>>>> asyncWait()
```
> CallerRunsPolicy 적용 시 위와 같이 Test worker (caller thread) 와 taskExecutor-1 스레드가 번갈아가면서 method 를 수행하는 걸 확인할 수 있습니다.

### 처리순서
1. CorePoolSize 만큼 스레드를 생성
2. 스레드가 CorePoolSize 초과 시 QueueCapacity 크기의 LinkedBlockingQueue(ReentrantLock 을 통한 Blocking Queue) 를 생성하여 대기
3. Queue 도 가득 차면 MaxPoolSize 만큼 스레드를 추가

### 마무리
@Async 어노테이션에 이어 ThreadPoolTaskExecutor 에 대해 학습해 볼 수 있는 시간이었습니다.

서비스 유형에 따라 적잘한 RejectedExecutionHandler 정책을 설정하여 운영하면 좋을 것 같습니다.

다음에는 Future, CompletableFuture 등에 대해서도 학습 후 포스팅을 해보도록 하겠습니다.

그럼 이만. 🥕👋🏼🖐🏼

### 참고자료🤣
[[Spring] 쓰레드 풀 설정하기 - ThreadPoolTaskExecutor](https://velog.io/@think2wice/Spring-Async-Thread-Pool%EC%97%90-%EB%8C%80%ED%95%98%EC%97%AC)  
[Guide to RejectedExecutionHandler](https://www.baeldung.com/java-rejectedexecutionhandler)