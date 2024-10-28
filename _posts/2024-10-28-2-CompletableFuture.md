---
title: "[Java] CompletableFuture"
excerpt_separator: "<!--more-->"
categories:
- development

tags:
- java
- async
- concurrent
- future
- completableFuture
---

지난 포스팅에 이어 CompletableFuture 에 대해 포스팅을 하려고 합니다.

이전 포스팅을 한번씩 읽고 오시면 CompletableFuture 를 왜 학습하게 되었는지 배경지식을 이해할 수 있을 것으로 보여집니다.

@Async 로 시작되었던 비동기 프로그래밍의 마지막 포스팅이 되지 않을까 싶습니다.

> [[Spring] @Async 사용 방법과 주의사항](https://2carrot84.github.io/development/async/)  
> [[Spring] ThreadPoolTaskExecutor 에 대한 짧은 지식](https://2carrot84.github.io/development/ThreadPoolTaskExecutor/)  
> [[Java] ExecutorService, Executors](https://2carrot84.github.io/development/1-ExecutorService-Executors/)  

<!--more-->

### CompletableFuture
> Java 5 부터 java.util.concurrent 패키지가 추가된 Future 의 한계점을 보완하기 위해 만들어 구현체이다.  
> java.util.concurrent.CompletionStage 를 구현하여, 다양한 콜백 메서드 및 조합 메서드를 지원한다.

#### Future 의 한계점
- 외부에서 완료시킬 수 없다.
- blocking 코드(get)를 통해서 결과 처리해야 한다.
- 여러 Future 나 비동기로 실행된 Task 조합이 불가
  - 예외 처리 할 수 없다.

#### 비동기 작업실행
- runAsync
  - `CompletableFuture<Void>` 를 리턴하여, get 함수를 호출하여도, null 이 리턴됨

> 반환값이 없는 작업 

```java
public CompletableFuture<Void> runAsync() {
    log.info(">>>>> runAsync Call!!");
    return CompletableFuture.runAsync(() -> {
        log.info(">>>>> CompletableFuture.runAsync Call!!");
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            log.error("error!");
        }
        log.info(">>>>> CompletableFuture.runAsync End!!");
    });
}
```
```java
@Test
void runAsync() throws ExecutionException, InterruptedException {
    CompletableFuture<Void> completableFuture = completableFutureStudy.runAsync();
    assertThat(completableFuture.isDone()).isFalse();
    Void unused = completableFuture.get();
    assertThat(unused).isNull();
    assertThat(completableFuture.isDone()).isTrue();
}
```
```shell
14:01:51.413 [Test worker] INFO com.example.demo.completableFuture.CompletableFutureStudy -- >>>>> runAsync Call!!
14:01:51.423 [ForkJoinPool.commonPool-worker-1] INFO com.example.demo.completableFuture.CompletableFutureStudy -- >>>>> CompletableFuture.runAsync Call!!
14:01:52.429 [ForkJoinPool.commonPool-worker-1] INFO com.example.demo.completableFuture.CompletableFutureStudy -- >>>>> CompletableFuture.runAsync End!!
```
> 테스트 결과 `ForkJoinPool.commonPool-worker-1` 와 같이 별도의 스레드를 생성하여, 수행되는 걸 확인할 수 있습니다.
> 
> get() 함수를 호출하지 않는다면, `CompletableFuture.runAsync End!!` 로그는 찍히지 않은 채 메인 스레드(`Test worker`)는 종료될 거라는 건 이제 다들 이해하시죠?

- supplyAsync

> 반환값이 있는 작업

```java
public CompletableFuture<String > supplyAsync() {
    log.info(">>>>> supplyAsync Call!!");
    return CompletableFuture.supplyAsync(() -> {
        log.info(">>>>> CompletableFuture.supplyAsync Call!!");
        return "Hello, world!";
    });
}
```
```java
@Test
void supplyAsync() throws ExecutionException, InterruptedException {
    CompletableFuture<String> completableFuture = completableFutureStudy.supplyAsync();
    assertThat(completableFuture.isDone()).isFalse();
    String s = completableFuture.get();
    assertThat(completableFuture.isDone()).isTrue();
    assertThat(s).isEqualTo("Hello, world!");
}
```
```shell
14:11:55.506 [Test worker] INFO com.example.demo.completableFuture.CompletableFutureStudy -- >>>>> supplyAsync Call!!
14:11:55.515 [ForkJoinPool.commonPool-worker-1] INFO com.example.demo.completableFuture.CompletableFutureStudy -- >>>>> CompletableFuture.supplyAsync Call!!
```

- 기본 적으로 ForkJoinPool 의 commonPool 을 이용하는 것도 확인 가능합니다.
  - [Guide to the Fork/Join Framework in Java](https://www.baeldung.com/java-fork-join)

```java
private static final Executor ASYNC_POOL = USE_COMMON_POOL ?
    ForkJoinPool.commonPool() : new ThreadPerTaskExecutor();
```

#### 작업 콜백
- thenAccept

> 반환 값을 처리하고 값을 반환하지 않음

```java
public CompletableFuture<Void> thenAccept(Consumer<? super T> action) {
    return uniAcceptStage(null, action);
}
```
```java
public CompletableFuture<Void> thenAccept() {
    log.info(">>>>> completableFutureThenAccept Call!!");
    return CompletableFuture.supplyAsync(() -> {
        log.info(">>>>> CompletableFuture.supplyAsync Call!!");
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            log.error(e.getMessage());
        }
        return "Hello,";
    }).thenAccept(log::info);
}
```
```shell
14:25:14.790 [Test worker] INFO com.example.demo.completableFuture.CompletableFutureStudy -- >>>>> completableFutureThenAccept Call!!
14:25:14.791 [ForkJoinPool.commonPool-worker-1] INFO com.example.demo.completableFuture.CompletableFutureStudy -- >>>>> CompletableFuture.supplyAsync Call!!
14:25:15.797 [ForkJoinPool.commonPool-worker-1] INFO com.example.demo.completableFuture.CompletableFutureStudy -- Hello,
```
> 위 결과 같이 `ForkJoinPool.commonPool-worker-1` 와 같이 새로운 스레드에서 작업 후 결과를 수신 후 처리한 것을 확인할 수 있습니다.

- thenApply

> 반환 값을 받아서 다른 값을 반환

```java
public <U> CompletableFuture<U> thenApply(
    Function<? super T,? extends U> fn) {
    return uniApplyStage(null, fn);
}
```
```java
public CompletableFuture<String> thenApply() {
    log.info(">>>>> completableFutureThenApply Call!!");
    return CompletableFuture.supplyAsync(() -> {
        log.info(">>>>> CompletableFuture.supplyAsync Call!!");
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            log.error(e.getMessage());
        }
        return "Hello,";
    }).thenApply((s) -> {
        log.info(s);
        return s + " world!";
    });
}
```
```java
@Test
void thenApply() throws ExecutionException, InterruptedException {
    CompletableFuture<String> completableFuture = completableFutureStudy.thenApply();
    assertThat(completableFuture.isDone()).isFalse();
    String s = completableFuture.get();
    assertThat(completableFuture.isDone()).isTrue();
    assertThat(s).isEqualTo("Hello, world!");
}
```
```shell
14:28:26.498 [Test worker] INFO com.example.demo.completableFuture.CompletableFutureStudy -- >>>>> completableFutureThenApply Call!!
14:28:26.512 [ForkJoinPool.commonPool-worker-1] INFO com.example.demo.completableFuture.CompletableFutureStudy -- >>>>> CompletableFuture.supplyAsync Call!!
14:28:27.516 [ForkJoinPool.commonPool-worker-1] INFO com.example.demo.completableFuture.CompletableFutureStudy -- Hello,
```

- thenRun

> 반환 값을 받지 않고 다른 작업을 실행

```java
public CompletableFuture<Void> thenRun(Runnable action) {
    return uniRunStage(null, action);
}
```
```java
public CompletableFuture<Void> thenRun() {
    log.info(">>>>> completableFutureThenRun Call!!");
    return CompletableFuture.supplyAsync(() -> {
            log.info(">>>>> CompletableFuture.supplyAsync Call!!");
            return "Hello, World!";
        }).thenRun(() -> log.info(">>>>> thenRun Call!!"))
        .thenRunAsync(() -> log.info(">>>>> thenRunAsync Call!!"));
}
```
```java
@Test
void thenRun() {
    CompletableFuture<Void> completableFuture = completableFutureStudy.thenRun();
    assertThat(completableFuture.isDone()).isFalse();
}
```
```shell
14:30:53.632 [Test worker] INFO com.example.demo.completableFuture.CompletableFutureStudy -- >>>>> completableFutureThenRun Call!!
14:30:53.646 [ForkJoinPool.commonPool-worker-1] INFO com.example.demo.completableFuture.CompletableFutureStudy -- >>>>> CompletableFuture.supplyAsync Call!!
14:30:53.647 [Test worker] INFO com.example.demo.completableFuture.CompletableFutureStudy -- >>>>> thenRun Call!!
14:30:53.647 [ForkJoinPool.commonPool-worker-1] INFO com.example.demo.completableFuture.CompletableFutureStudy -- >>>>> thenRunAsync Call!!
```
> thenRun 을 실행해본 결과 특이한 점은 `thenRun Call!!` 로그가 메인 스레드(`Test worker`) 에서 찍혔다는게 앞에 본 2가지 메서드와 차이점인 것 같습니다.
> 
> supplyAsync 의 결과를 수신하지 않기 때문에 이런 결과가 나온 것으로 생각됩니다.

#### 작업 조합
- thenCompose

> 두 작업이 `이어서 실행`하도록 조합, 앞선 작업의 결과를 받아서 사용가능

```java
public <U> CompletableFuture<U> thenCompose(
        Function<? super T, ? extends CompletionStage<U>> fn) {
  return uniComposeStage(null, fn);
}
```
```java
public CompletableFuture<String> thenCompose() {
    log.info(">>>>> thenCompose Call!!");
    return CompletableFuture.supplyAsync(() -> {
        log.info(">>>>> CompletableFuture.supplyAsync Call!!");
        return "Hello,";
    }).thenCompose((s) -> CompletableFuture.supplyAsync(() -> {
        log.info(">>>>> CompletableFuture.thenCompose Call!!");
        return s + " world!";
    }));
}
```
```java
@Test
void thenCompose() throws ExecutionException, InterruptedException {
    CompletableFuture<String> completableFuture = completableFutureStudy.thenCompose();
    assertThat(completableFuture.isDone()).isFalse();
    String s = completableFuture.get();
    assertThat(completableFuture.isDone()).isTrue();
    assertThat(s).isEqualTo("Hello, world!");
}
```
```shell
14:38:08.067 [Test worker] INFO com.example.demo.completableFuture.CompletableFutureStudy -- >>>>> thenCompose Call!!
14:38:08.082 [ForkJoinPool.commonPool-worker-1] INFO com.example.demo.completableFuture.CompletableFutureStudy -- >>>>> CompletableFuture.supplyAsync Call!!
14:38:08.084 [ForkJoinPool.commonPool-worker-2] INFO com.example.demo.completableFuture.CompletableFutureStudy -- >>>>> CompletableFuture.thenCompose Call!!
```
> 테스트 결과와 같이 `ForkJoinPool.commonPool-worker-1`, `ForkJoinPool.commonPool-worker-2` 각 스레드에서 실행되나 앞선 작업의 결과를 파라미터로 받아서 처리하는 것을 확인할 수 있습니다. 

- thenCombine

> 두 작업을 `독립적으로 실행`하고 두 작업 완료되었을 때 콜백을 실행   
> 어떤 스레드에서 콜백이 실행될지는 모른다.

```java
public <U,V> CompletableFuture<V> thenCombine(
    CompletionStage<? extends U> other,
    BiFunction<? super T,? super U,? extends V> fn) {
    return biApplyStage(null, other, fn);
}
```
```java
public CompletableFuture<String > thenCombine() {
    log.info(">>>>> thenCombine Call!!");
    return CompletableFuture.supplyAsync(() -> {
        log.info(">>>>> CompletableFuture.supplyAsync Call!!");
        return "Hello,";
    }).thenCombine(CompletableFuture.supplyAsync(() -> {
        log.info(">>>>> CompletableFuture.thenCombine Call!!");
        return " world!";
    }), (s1, s2) -> {
        log.info(">>>>> thenCombine Done!! result : " + s1 + s2);
        return s1 + s2;
    });
}
```
```java
@Test
void thenCombine() throws ExecutionException, InterruptedException {
    CompletableFuture<String> completableFuture = completableFutureStudy.thenCombine();
    String s = completableFuture.get();
    assertThat(completableFuture.isDone()).isTrue();
    assertThat(s).isEqualTo("Hello, world!");
}
```
```shell
14:45:12.583 [Test worker] INFO com.example.demo.completableFuture.CompletableFutureStudy -- >>>>> thenCombine Call!!
14:45:12.593 [ForkJoinPool.commonPool-worker-1] INFO com.example.demo.completableFuture.CompletableFutureStudy -- >>>>> CompletableFuture.supplyAsync Call!!
14:45:12.594 [ForkJoinPool.commonPool-worker-2] INFO com.example.demo.completableFuture.CompletableFutureStudy -- >>>>> CompletableFuture.thenCombine Call!!
14:45:12.596 [Test worker] INFO com.example.demo.completableFuture.CompletableFutureStudy -- >>>>> thenCombine Done!! result : Hello, world!

14:47:38.203 [Test worker] INFO com.example.demo.completableFuture.CompletableFutureStudy -- >>>>> thenCombine Call!!
14:47:38.215 [ForkJoinPool.commonPool-worker-1] INFO com.example.demo.completableFuture.CompletableFutureStudy -- >>>>> CompletableFuture.supplyAsync Call!!
14:47:38.215 [ForkJoinPool.commonPool-worker-2] INFO com.example.demo.completableFuture.CompletableFutureStudy -- >>>>> CompletableFuture.thenCombine Call!!
14:47:38.218 [ForkJoinPool.commonPool-worker-2] INFO com.example.demo.completableFuture.CompletableFutureStudy -- >>>>> thenCombine Done!! result : Hello, world!
```
> 위 테스트 결과와 같이 `ForkJoinPool.commonPool-worker-1`, `ForkJoinPool.commonPool-worker-2` 각 스레드에서 실행 후 메인 스레드(`Test worker`) 또는 작업 스레드(`ForkJoinPool.commonPool-worker-2`)에서 콜백이 실행되는걸 확인할 수 있습니다.

- allOf

> 여러 작업들을 동시에 실행, 모든 작업 결과에 콜백

```java
public static CompletableFuture<Void> allOf(CompletableFuture<?>... cfs) {
    return andTree(cfs, 0, cfs.length - 1);
}
```
```java
public CompletableFuture<Void> allOf(CompletableFuture<?>... cfs) {
    log.info(">>>>> allOf Call!!");
    return CompletableFuture.allOf(cfs);
}
```
```java
@Test
void allOf() throws ExecutionException, InterruptedException {
    CompletableFuture<String> c1 = CompletableFuture.supplyAsync(() -> {
        System.out.println(Thread.currentThread().getName());
        return "Hello,";
    });
    CompletableFuture<String> c2 = CompletableFuture.supplyAsync(() -> {
        System.out.println(Thread.currentThread().getName());
        return " world!";
    });

    CompletableFuture<Void> completableFuture = completableFutureStudy.allOf(c1, c2);
    Void unused = completableFuture.get();
    assertThat(unused).isNull();
    assertThat(completableFuture.isDone()).isTrue();

    String joined = Stream.of(c1, c2)
        .map(CompletableFuture::join)
        .collect(Collectors.joining(""));

    assertThat(joined).isEqualTo("Hello, world!");
}
```
```shell
ForkJoinPool.commonPool-worker-1
ForkJoinPool.commonPool-worker-2
14:51:30.928 [Test worker] INFO com.example.demo.completableFuture.CompletableFutureStudy -- >>>>> allOf Call!!
```
> allOf 의 결과는 `CompletableFuture<Void>` 이기 때문에 get 으로 결과 값을 조합은 불가합니다.
> 
> get() 과 유사한 join() 메서드로 조합을 할 수는 있으나, Future 가 정상적으로 완료되지 않을 경우 확인되지 않은 예외가 발생할 수 있는 단점이 있다는 점을 고려해야 한다고 합니다.

- anyOf

> 여러 작업 중 빨리 끝나는 하나의 결과에 콜백 

```java
public static CompletableFuture<Object> anyOf(CompletableFuture<?>... cfs) {
  ...
}
```
```java
public CompletableFuture<Object> anyOf(CompletableFuture<?>... cfs) {
    log.info(">>>>> anyOf Call!!");
    return CompletableFuture.anyOf(cfs);
}
```
```java
@Test
void anyOf() throws ExecutionException, InterruptedException {
    CompletableFuture<String> c1 = CompletableFuture.supplyAsync(() -> {
        System.out.println(Thread.currentThread().getName());
        return "Hello,";
    });
    CompletableFuture<String> c2 = CompletableFuture.supplyAsync(() -> {
        System.out.println(Thread.currentThread().getName());
        return " world!";
    });

    CompletableFuture<Object> completableFuture = completableFutureStudy.anyOf(c1, c2);
    String s = (String)completableFuture.get();

    assertThat(s).isNotEqualTo("Hello, world!");
    assertThat(Arrays.asList("Hello,", " world!")).contains(s);
}
```
```shell
ForkJoinPool.commonPool-worker-1
ForkJoinPool.commonPool-worker-2
14:58:36.498 [Test worker] INFO com.example.demo.completableFuture.CompletableFutureStudy -- >>>>> anyOf Call!!
```
> anyOf 는 여러 작업 중 하나의 결과만 반환 되는 것을 확인할 수 있었습니다.

### 예외처리
- exceptionally

> 발생한 에러를 받아서 예외 처리

```java
public CompletableFuture<T> exceptionally(
    Function<Throwable, ? extends T> fn) {
    return uniExceptionallyStage(null, fn);
}
```
```java
public CompletableFuture<String> exceptionally(String str) {
    log.info(">>>>> exceptionally Call!!");
    return CompletableFuture.supplyAsync(() -> {
        log.info(">>>>> CompletableFuture.supplyAsync Call!! : " + str);
        if (str == null) {
            throw new IllegalArgumentException("str is null!!");
        }
        return str;
    }).exceptionally((e) -> {
        log.info(">>>>> exceptionally call e : " + e);
        return e == null ? "success" : e.getMessage() ;
    });
}
```
```java
@Test
void exceptionally() throws ExecutionException, InterruptedException {
    CompletableFuture<String> completableFuture = completableFutureStudy.exceptionally("Hello, world!");
    assertThat(completableFuture.isDone()).isFalse();
    String s = completableFuture.get();
    assertThat(completableFuture.isDone()).isTrue();
    assertThat(s).isEqualTo("Hello, world!");

    CompletableFuture<String> completableFuture1 = completableFutureStudy.exceptionally(null);
    assertThat(completableFuture1.isDone()).isFalse();
    String s1 = completableFuture1.get();
    assertThat(s1).contains("str is null!!");
}
```
```shell
15:02:51.278 [Test worker] INFO com.example.demo.completableFuture.CompletableFutureStudy -- >>>>> exceptionally Call!!
15:02:51.290 [ForkJoinPool.commonPool-worker-1] INFO com.example.demo.completableFuture.CompletableFutureStudy -- >>>>> CompletableFuture.supplyAsync Call!! : Hello, world!
15:02:51.334 [Test worker] INFO com.example.demo.completableFuture.CompletableFutureStudy -- >>>>> exceptionally Call!!
15:02:51.335 [ForkJoinPool.commonPool-worker-1] INFO com.example.demo.completableFuture.CompletableFutureStudy -- >>>>> CompletableFuture.supplyAsync Call!! : null
15:02:51.335 [ForkJoinPool.commonPool-worker-1] INFO com.example.demo.completableFuture.CompletableFutureStudy -- >>>>> exceptionally call e : java.util.concurrent.CompletionException: java.lang.IllegalArgumentException: str is null!!
```
> 테스트 결과 예외가 발생한 경우만 `exceptionally call e :` 로그가 찍히믄 것으로 보아, exceptionally 이 호출되는 것을 확인할 수 있습니다.

- handle

> 에러 발생 유무와 관계없이, (결과값, 에러)를 항상 반환받아 처리

```java
public <U> CompletableFuture<U> handle(
    BiFunction<? super T, Throwable, ? extends U> fn) {
    return uniHandleStage(null, fn);
}
```
```java
public CompletableFuture<String> handle(String str) {
    log.info(">>>>> handle Call!!");
    return CompletableFuture.supplyAsync(() -> {
        log.info(">>>>> CompletableFuture.supplyAsync Call!! : " + str);
        if (str == null) {
            throw new IllegalArgumentException("str is null!!");
        }
        return str;
    }).handle((s, f) -> {
        log.info(">>>>> handle call s : " + s);
        log.info(">>>>> handle call f : " + f);
        return s != null ? s : f.getMessage();
    });
}
```
```java
@Test
void handle() throws ExecutionException, InterruptedException {
    CompletableFuture<String> completableFuture = completableFutureStudy.handle("Hello, world!");
    assertThat(completableFuture.isDone()).isFalse();
    String s = completableFuture.get();
    assertThat(completableFuture.isDone()).isTrue();
    assertThat(s).isEqualTo("Hello, world!");

    CompletableFuture<String> completableFuture1 = completableFutureStudy.handle(null);
    assertThat(completableFuture1.isDone()).isFalse();
    String s1 = completableFuture1.get();
    assertThat(s1).contains("str is null!!");
}
```
```shell
15:06:32.079 [Test worker] INFO com.example.demo.completableFuture.CompletableFutureStudy -- >>>>> handle Call!!
15:06:32.090 [ForkJoinPool.commonPool-worker-1] INFO com.example.demo.completableFuture.CompletableFutureStudy -- >>>>> CompletableFuture.supplyAsync Call!! : Hello, world!
15:06:32.091 [ForkJoinPool.commonPool-worker-1] INFO com.example.demo.completableFuture.CompletableFutureStudy -- >>>>> handle call s : Hello, world!
15:06:32.091 [ForkJoinPool.commonPool-worker-1] INFO com.example.demo.completableFuture.CompletableFutureStudy -- >>>>> handle call f : null
15:06:32.135 [Test worker] INFO com.example.demo.completableFuture.CompletableFutureStudy -- >>>>> handle Call!!
15:06:32.135 [ForkJoinPool.commonPool-worker-1] INFO com.example.demo.completableFuture.CompletableFutureStudy -- >>>>> CompletableFuture.supplyAsync Call!! : null
15:06:32.136 [ForkJoinPool.commonPool-worker-1] INFO com.example.demo.completableFuture.CompletableFutureStudy -- >>>>> handle call s : null
15:06:32.136 [ForkJoinPool.commonPool-worker-1] INFO com.example.demo.completableFuture.CompletableFutureStudy -- >>>>> handle call f : java.util.concurrent.CompletionException: java.lang.IllegalArgumentException: str is null!!
```
> 위와 같이 정상 처리시에는 handle 메서드의 첫번째 파라미터로 결과값이, 예외 발생시 두번째 파라미터로 예외 객체가 넘어오는 것을 확인할 수 있습니다. 

#### 비동기 메소드
- CompleteFuture 클래스가 제공하는 메서드들은 다른 스레드를 사용하여 비동기 연산을 수행할 수 있도록, 끝에 Async 가 붙는 메서드들도 제공하고 있습니다.

```java
public CompletableFuture<Void> thenRunAsync(Runnable action) {
    return uniRunStage(defaultExecutor(), action);
}
```
- 마지막 파리미터로 Executor 를 직접 제공할 수 있는 메소드도 제공하고 있습니다.

```java
public CompletableFuture<Void> thenRunAsync(Runnable action,
                                            Executor executor) {
    return uniRunStage(screenExecutor(executor), action);
}
```

### 마무리
이번 포스팅을 끝으로 @Async 로 시작한 비동기 프로그래밍에 대한 포스팅이 마무리 되었습니다.

팀에서 다양한 MSA 의 API 결과를 비동기로 어그리게이션 하는 방법을 스터디 해보자고 한적이 있었는데, 이번 학습을 통해 어렴풋이 방법을 떠올려 볼 수 있는 시간이었습니다.

다음은 또 어떤 포스팅으로 돌아올지 많은 기대 부탁드립니다.

그럼 이만. 🥕👋🏼🖐🏼

### 참고자료🤣
[Java CompletableFuture로 비동기 적용하기](https://11st-tech.github.io/2024/01/04/completablefuture/)  
[JAVA 비동기 프로그래밍: CompletableFuture](https://velog.io/@suyeon-jin/JAVA-CompletableFuture)  
[Java의 Future Interface 란?](https://javabom.tistory.com/96)