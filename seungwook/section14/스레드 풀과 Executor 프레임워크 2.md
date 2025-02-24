# Section 14. 스레드 풀과 Executor 프레임워크 2

## ExecutorService 우아한 종료

우아한 종료란 현재 실행 중인 작업을 방해하지 않으면서도 안전하게 스레드 풀을 종료하는 방법을 의미한다.

문제 없이 우아하게 종료하는 방식을 **graceful shutdown**이라 한다.

### ExecutorService의 종료 메서드

- `void shutdown()`
    - 새로운 작업을 받지 않고, 이미 제출된 작업을 모두 완료한 후에 종료한다.
- `List<Runnable> shutdownNow()`
    - 실행 중인 작업을 중단하고, 대기 중인 작업을 반환하며 즉시 종료한다.
- `close()`
    - 자바 19부터 지원하는 서비즈 종료 메서드이다
    - `shutdown()` 을 호출하고, 하루를 기다려도 작업이 완료되지 않으면 `shutdownNow()` 를 호출한다

보통 graceful shutdown은 `shutdown()` 을 통해 종료를 시도하고, 10초간 종료되지 않으면 `shutdownNow()`통해 강제 종료하는 방식을 사용한다.

<br>

## Executor 스레드 풀 관리

`ExecutorService` 의 기본 구현체인 `ThreadPoolExecutor` 의 생성자는 다음 속성을 사용한다.

- `corePoolSize` : 스레드 풀에서 관리되는 기본 스레드의 수
- `maximumPoolSize` : 스레드 풀에서 관리되는 최대 스레드 수
- `keepAliveTime` , `TimeUnit unit` : 기본 스레드 수를 초과해서 만들어진 초과 스레드가 생존할 수 있는 대기 시간, 이 시간 동안 처리할 작업이 없다면 초과 스레드는 제거된다.
- `BlockingQueue workQueue` : 작업을 보관할 블로킹 큐

작업 요청 시 core 만큼 스레드를 생성하고, 이를 초과하면 큐에 저장되며, 큐가 꽉 차면 max까지 임시 스레드를 생성해 작업을 수행하고, 모든 자원이 꽉 차면 요청을 거절해 예외가 발생한다.

### 스레드 미리 생성하기

응답시간이 아주 중요한 서버라면, 요청받았을때 스레드를 생성하는게 아니라 미리 생성해둬서 스레드의 생성 시간을 줄일 수 있다.

`ThreadPoolExecutor.prestartAllCoreThreads()` 를 사용하면 기본 스레드를 미리 생성할 수 있다.

<br>

## Executor 전략

자바는 `Executors` 클래스를 통해 3가지 기본 전력을 제공한다.

- **newSingleThreadPool()**: 단일 스레드 풀 전략
- **newFixedThreadPool(nThreads)**: 고정 스레드 풀 전략
- **newCachedThreadPool()**: 캐시 스레드 풀 전략

### 고정 스레드 풀 전략

- **newFixedThreadPool(nThreads)**

이 방식의 가장 큰 장점은 스레드 수가 고정되어서 CPU, 메모리 리소스가 어느정도 예측 가능하다.

따라서 일반적인 상황에 가장 안정적으로 서비스를 운영할 수 있다.

그러나 스레드 수가 고정되어 있어, 트래픽이 급증해도 CPU가 확 늘어나지는 않지만 작업이 큐에서 대기해야 하므로 응답 속도가 느려질 수 있다.

### 캐시 스레드 풀 전략

- **newCachedThreadPool()**

이 전략은 기본 스레드도 없고, 대기 큐에 작업도 쌓이지 않는다. 대신에 작업 요청이 오면 초과 스레드로 작업을 바로바로 처리한다.

따라서 빠른 처리가 가능하다. 초과 스레드의 수도 제한이 없기 때문에 CPU, 메모리 자원만 허용한다면 시스템의 자원을 최대로 사용할 수 있다.

고정 스레드 풀 전략은 서버 자원은 여유가 있는데, 사용자만 점점 느려지는 문제가 발생할 수 있다. 

반면에 캐시 스레드풀 전략은 서버의 자원을 최대한 사용할 수 있다.

### 사용자 정의 스레드 풀 전략

#### 상황1 - 점진적인 사용자 확대

- 개발한 서비스가 잘 되어서 사용자가 점점 늘어난다.

#### 상황2 - 갑작스런 요청 증가

- 마케팅 팀의 이벤트가 대성공 하면서 갑자기 사용자가 폭증했다.

```java
ExecutorService es = new ThreadPoolExecutor(100, 200, 60, TimeUnit.SECONDS, new ArrayBlockingQueue<>(1000));
```

- 100개의 기본 스레드를 사용한다.
- 추가로 긴급 대응 가능한 긴급 스레드 100개를 사용한다. 긴급 스레드는 60초의 생존 주기를 가진다.
- 1000개의 작업이 큐에 대기할 수 있다.

이렇게 사용자 정의 스레드 풀을 사용하면 상황1, 상황2를 모두 어느정도 대응할 수 있다.
