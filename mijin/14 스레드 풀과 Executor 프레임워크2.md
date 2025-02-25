# Section 14. 스레드 풀과 Executor 프레임워크 2


`Executors` 클래스를 통한 3가지 기본 전략 
- **newSingleThreadPool()**: 단일 스레드 풀 전략 
- **newFixedThreadPool(nThreads)**: 고정 스레드 풀 전략 
- **newCachedThreadPool()**: 캐시 스레드 풀 전략

## Executor 전략 - 고정 풀 전략

```java
new ThreadPoolExecutor(1, 1,0L, TimeUnit.MILLISECONDS,
                       new LinkedBlockingQueue<Runnable>())
```
`newFixedThreadPool(nThreads)`
- 스레드 풀에 `nThreads` 만큼 기본 스레드를 생성한다. 초과 스레드는 생성 X
- 큐 사이즈에 제한 없음
- 스레드 수가 고정되어 있기 때문에 CPU, 메모리 리소스가 어느정도 예측 가능한 안정적인 방식

## Executor 전략 - 캐시 풀 전략

```java
new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60L, TimeUnit.SECONDS,
                       new SynchronousQueue<Runnable>());
```
`newCachedThreadPool()`

- 기존 스레드를 사용하지 않고, 60초 생존 주기를 가진 초과 스레드만 사용
- 초과 스레드의 수는 제한 X
- 큐에 작업을 저장하지 않고(`SynchronousQueue`), 생산자의 요청을 스레드 풀의 소비자가 바로 처리
- 모든 요청이 대기하지 않고 스레드가 바로 처리하기 때문에 빠른 처리가 가능
- 결과적으로 이 전략의 모든 작업은 초과 스레드가 처리

## Executor 전략 - 사용자 정의 풀 전략

- **일반**: 일반적인 상황에는 CPU, 메모리 자원을 예측할 수 있도록 고정 크기의 스레드를 사용
- **긴급**: 사용자의 요청이 갑자기 증가하면 긴급하게 스레드를 추가로 투입해서 작업을 빠르게 처리
- **거절**: 사용자의 요청이 폭증해서 긴급 대응도 어렵다면 사용자의 요청을 거절
  
시스템이 감당할 수 없을 정도로 사용자의 요청이 폭증하면, 처리 가능한 수준의 사용자 요청만 처리하고 나머지 요청은 거절해야 한다. 
어떤 경우에도 시스템이 다운되는 최악의 상황은 피해야 한다.

## Executor 예외 정책

`ThreadPoolExecutor` 의 작업을 거절하는 다양한 정책
- **AbortPolicy**: 새로운 작업을 제출할 때 `RejectedExecutionException` 을 발생. (기본 정책) 
- **DiscardPolicy**: 새로운 작업을 조용히 버림 (어쩔 때 사용하는거지?)
- **CallerRunsPolicy**: 새로운 작업을 제출한 스레드가 대신해서 직접 실행
- **사용자 정의**( `RejectedExecutionHandler` ): 개발자가 직접 정의한 거절 정책을 사용할 수 있음

## 정리

< 실무 전략 선택 >
- **고정 스레드 풀 전략**: 트래픽이 일정하고, 시스템 안전성이 가장 중요
- **캐시 스레드 풀 전략**: 일반적인 성장하는 서비스
- **사용자 정의 풀 전략**: 다양한 상황에 대응

**가장 좋은 최적화는 최적화하지 않는 것이다!!**

일반적인 상황이라면 고정 스레드 풀 전략이나, 캐시 스레드 풀 전략을 사용하면 충분하다.
- 한번에 처리할 수 있는 수를 제안하고 안정적으로 처리하고 싶다면 -> 고정 풀 전략
- 사용자의 요청을 빠르게 대응하고 싶다면 -> 캐시 스레드 풀 전략 

**최악의 경우 적절한 거절을 통해 시스템이 다운되지 않도록 해야한다. (가장 중요)**