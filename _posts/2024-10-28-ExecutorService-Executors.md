---
title: "[Java] ExecutorService, Executors"
excerpt_separator: "<!--more-->"
categories:
- development

tags:
- java
- threadPool
- executor
- executorService
---

요즘 비동기관련 포스팅을 자주하게 되는 것 같습니다. 그동안 실무적으로 비동기 로직을 적용해 본적이 없었는데, 보다 보니 볼게 참 많은 것 같네요.

스프링에서 제공하는 @Async 를 이용한 비동기 방식과 ThreadPoolTaskExecutor 에 대한 짧은 포스팅에 이어, 오늘은 Java 에서 제공하는 ExecutorService 와 Executors 에 대해 알아보겠습니다.  

> [[Spring] @Async 사용 방법과 주의사항](https://2carrot84.github.io/development/async/)  
> [[Spring] ThreadPoolTaskExecutor 에 대한 짧은 지식](https://2carrot84.github.io/development/ThreadPoolTaskExecutor/)

<!--more-->

### ExecutorService
> 병렬 작업 시 여러 개의 작업을 효율적으로 처리하기 위해 제공하는 Java 라이브러리
> 
> Runnable 인스턴스를 실행하는 Executor(작업 실행 책임만 가짐) 를 상속한 인터페이스로 작업(Runnable, Callable) 등록 및 Executor 의 실행을 위한 인터페이스

Runnable 은 응답값 없이 실행만 가능

```java
package java.lang;

@FunctionalInterface
public interface Runnable {
	public abstract void run();
}
```

Callable 은 응답값이 있음

```java
package java.util.concurrent;

@FunctionalInterface
public interface Callable<V> {
	V call() throws Exception;
}
```

- 비동기 작업을 지원하는 submit 메소드를 제공
  - Future 를 반환
    - Future 의 결과를 받기 위해 get 함수를 호출할 수 있으나, Blocking 으로 처리되어 비동기의 이점을 얻기 어렵다.

```java
public interface Future<V> {
    ...
	/**
	 * Waits if necessary for the computation to complete, and then
	 * retrieves its result.
	 ...
	 */
	V get() throws InterruptedException, ExecutionException;
    ...
}
```

- 대표적인 구현체로 java.util.concurrent.ThreadPoolExecutor 가 존재한다.

간단한 예재 코드는 아래와 같이 만들어 보았습니다.

```java
@Slf4j
public class ExecutorStudy {
	private final ExecutorService executorService;

	public ExecutorStudy(ExecutorService executorService) {
		this.executorService = executorService;
	}

	public void callExecutorService() {
		executorService.submit(() -> {
			log.info(">>>>> callExecutorService.submit 시작");
			try {
				Thread.sleep(1000);
			} catch (InterruptedException e) {
				throw new RuntimeException(e);
			}
			log.info(">>>>> callExecutorService.submit 종료");
		});
		// executorService.shutdown();
	}

	public void callMethod() {
		log.info(">>>>> callMethod 시작");
		try {
			Thread.sleep(500);
		} catch (InterruptedException e) {
			throw new RuntimeException(e);
		}
		log.info(">>>>> callMethod 종료");
	}
}
```

위 코드를 Executors 를 통해 다양한 테스트를 진행해 보려고 합니다.

### Executors

> 스레드풀을 쉽게 생성할 수 있는 팩토리 클래스

#### newFixedThreadPool
> 고정된 스레드 개수를 갖는 스레드풀 생성

```java
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```
```java
	@Test
	void newFixedThreadPool() {
		ExecutorService executorService = Executors.newFixedThreadPool(2);
		ExecutorStudy executorStudy = new ExecutorStudy(executorService);
		ThreadPoolExecutor executor = (ThreadPoolExecutor)executorService;

		executorStudy.callExecutorService();
		executorStudy.callExecutorService();
		executorStudy.callExecutorService();
		executorStudy.callMethod();

		assertThat(executor.getPoolSize()).isEqualTo(2);
		assertThat(executor.getQueue().size()).isEqualTo(1);
	}
```
```shell
12:10:30.898 [pool-1-thread-1] INFO com.example.demo.executor.ExecutorStudy -- >>>>> callExecutorService.submit 시작
12:10:30.898 [pool-1-thread-2] INFO com.example.demo.executor.ExecutorStudy -- >>>>> callExecutorService.submit 시작
12:10:30.898 [Test worker] INFO com.example.demo.executor.ExecutorStudy -- >>>>> callMethod 시작
12:10:31.407 [Test worker] INFO com.example.demo.executor.ExecutorStudy -- >>>>> callMethod 종료
```
주어진 스레드풀을 이용하여, 위에서 보았던 ExecutorService 의 submit 메소드를 실행하는 예제 코드입니다. 

> 2개의 스레드를 갖는 스레드풀을 생성 후 3개의 요청을 보낸 결과입니다.  
> 
> `pool-1-thread-1`, `pool-1-thread-2` 과 같이 스레드풀에서 2개의 스레드를 가져와 각 스레드에서 실행을 하는걸 확인할 수 있습니다.
> 
> ThreadPoolExecutor 생성 시 설정된 corePoolSize(2) 를 넘어서는 요청은 LinkedBlockingQueue 에 담기게 되는 것도 확인할 수 있습니다.
> 
> ExecutorService 의 submit 결과인 Future 를 리턴 받아 get 을 호출하지 않았기 때문에, non-blocking 방식으로 실행되어, ">>>>> callExecutorService.submit 종료" 로그가 남지 않고, 메인 스레드(Test worker)가 종료되는 것을 볼 수 있습니다.

#### newCachedThreadPool
> 필요할 때 필요한 만큼의 스레드풀을 생성  
> 이미 생성된 스레드가 있다면 재활용

```java
    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```
```java
	@Test
	void newCachedThreadPool() throws InterruptedException {
		ExecutorService executorService = Executors.newCachedThreadPool();
		ExecutorStudy executorStudy = new ExecutorStudy(executorService);
		ThreadPoolExecutor executor = (ThreadPoolExecutor)executorService;

		executorStudy.callExecutorService();
		executorStudy.callExecutorService();
		executorStudy.callExecutorService();
		executorStudy.callMethod();

		assertThat(executor.getPoolSize()).isEqualTo(3);
		assertThat(executor.getQueue().size()).isEqualTo(0);

		Thread.sleep(5000);

		executorStudy.callExecutorService();

		assertThat(executor.getPoolSize()).isEqualTo(3);
		assertThat(executor.getQueue().size()).isEqualTo(0);
	}
```
```shell
13:10:22.687 [pool-1-thread-3] INFO com.example.demo.executor.ExecutorStudy -- >>>>> callExecutorService.submit 시작
13:10:22.687 [Test worker] INFO com.example.demo.executor.ExecutorStudy -- >>>>> callMethod 시작
13:10:22.687 [pool-1-thread-2] INFO com.example.demo.executor.ExecutorStudy -- >>>>> callExecutorService.submit 시작
13:10:22.687 [pool-1-thread-1] INFO com.example.demo.executor.ExecutorStudy -- >>>>> callExecutorService.submit 시작
13:10:23.196 [Test worker] INFO com.example.demo.executor.ExecutorStudy -- >>>>> callMethod 종료
13:10:23.696 [pool-1-thread-1] INFO com.example.demo.executor.ExecutorStudy -- >>>>> callExecutorService.submit 종료
13:10:23.696 [pool-1-thread-2] INFO com.example.demo.executor.ExecutorStudy -- >>>>> callExecutorService.submit 종료
13:10:23.696 [pool-1-thread-3] INFO com.example.demo.executor.ExecutorStudy -- >>>>> callExecutorService.submit 종료
13:10:28.239 [pool-1-thread-2] INFO com.example.demo.executor.ExecutorStudy -- >>>>> callExecutorService.submit 시작
```
> newCachedThreadPool 로 생성 후 3개의 요청을 보내고 5초 후 다시 요청을 보낸 결과입니다.
> 
> `pool-1-thread-1``pool-1-thread-2``pool-1-thread-3` 과 같이 3개의 스레드를 갖는 스레드 풀이 생성되고 3개를 사용 후 재요청 시 기존 스레드(`pool-1-thread-2`)가 재활용 되는 것을 확인할 수 있었습니다.
> 
> 만약 3개 이상의 요청을 순간적으로 한다면 Integer.MAX_VALUE 까지 추가로 스레드를 늘리게 될 것입니다.

#### newSingleThreadExecutor
> 1개의 스레드만을 가지는 스레드 풀을 생성

```java
public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
```
```java
	@Test
	void newSingleThreadExecutor() throws InterruptedException {
		ExecutorService executorService = Executors.newSingleThreadExecutor();
		ExecutorStudy executorStudy = new ExecutorStudy(executorService);

		executorStudy.callExecutorService();
		executorStudy.callExecutorService();
		executorStudy.callExecutorService();
		executorStudy.callMethod();

		Thread.sleep(5000);
	}
```
```shell
13:19:51.581 [Test worker] INFO com.example.demo.executor.ExecutorStudy -- >>>>> callMethod 시작
13:19:51.581 [pool-1-thread-1] INFO com.example.demo.executor.ExecutorStudy -- >>>>> callExecutorService.submit 시작
13:19:52.090 [Test worker] INFO com.example.demo.executor.ExecutorStudy -- >>>>> callMethod 종료
13:19:52.590 [pool-1-thread-1] INFO com.example.demo.executor.ExecutorStudy -- >>>>> callExecutorService.submit 종료
13:19:52.591 [pool-1-thread-1] INFO com.example.demo.executor.ExecutorStudy -- >>>>> callExecutorService.submit 시작
13:19:53.597 [pool-1-thread-1] INFO com.example.demo.executor.ExecutorStudy -- >>>>> callExecutorService.submit 종료
13:19:53.598 [pool-1-thread-1] INFO com.example.demo.executor.ExecutorStudy -- >>>>> callExecutorService.submit 시작
13:19:54.603 [pool-1-thread-1] INFO com.example.demo.executor.ExecutorStudy -- >>>>> callExecutorService.submit 종료
```
> 싱글 스레드풀을 생성 후 3개의 요청을 보내고 5초간 기다려본 결과입니다.
> 
> 3개의 요청 모두 하나의 스레드(`pool-1-thread-1`) 를 순차적으로 사용하는 걸 확인할 수 있습니다.  

### 마무리
이번에는 ExecutorService 와 Executors 를 통해 간단히 몇가지 스레드풀을 만들어서 동작을 확인해 볼 수 있었습니다.

결국 비동기는 `별도의 스레드로 요청을 보내고 결과 수신과 상관없이, 메인 스레드(호출한 스레드)를 종료하는게 핵심`인거 같습니다.

다음에는 CompleteFuture 에 관해 학습한 내용을 포스팅해 보도록 하겠습니다.

그럼 이만. 🥕👋🏼🖐🏼

### 참고자료🤣
[[Java] ExecutorService란?](https://simyeju.tistory.com/119)  
[[Java] Callable, Executors, ExecutorService 의 이해 및 사용법](https://velog.io/@ssssujini99/Java-Callable-Executors-ExecutorService-%EC%9D%98-%EC%9D%B4%ED%95%B4-%EB%B0%8F-%EC%82%AC%EC%9A%A9%EB%B2%95)