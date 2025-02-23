### ExecutorService 우아한 종료 


- void shutdown()
   - 새로운 작업을 받지 않고, 이미 제출된 작업을 모두 완료한 후에 종료한다.

 - List<Runnable> shutdownNow()
  - 실행 중인 작업을 중단하고, 대기 중인 작업을 반환하며 즉시 종료한다. 
  - 실행 중인 작업을 중단하기 위해 인터럽트를 발생시킨다.

```java
static void shutdownAndAwaitTermination(ExecutorService es) {
    es.shutdown(); // non-blocking, 새로운 작업을 받지 않는다. 처리 중이거나, 큐에 이미 대기중인 작업은 처리한다. 이후에 풀의 스레드를 종료한다.
    try {
        log("서비스 정상 종료 시도");
       // 이미 대기중인 작업들을 모두 완료할 때 까지 10초 기다린다.
        if (!es.awaitTermination(10, TimeUnit.SECONDS)) {
            // 정상 종료가 너무 오래 걸리면...
            log("서비스 정상 종료 실패 -> 강제 종료 시도");
            es.shutdownNow();
            // 작업이 취소될 때 까지 대기한다.
            if (!es.awaitTermination(10, TimeUnit.SECONDS)) {
                log("서비스가 종료되지 않았습니다.");
            }
        }
    } catch (InterruptedException ex) {
        // awaitTermination()으로 대기중인 현재 스레드가 인터럽트 될 수 있다.
        es.shutdownNow();
    }
}

```


### Executor 스레드 풀 관리 

- ExecutorService` 의 기본 구현체인 `ThreadPoolExecutor` 의 생성자는 다음 속성을 사용한다. 

  - `corePoolSize` : 스레드 풀에서 관리되는 기본 스레드의 수
  - `maximumPoolSize` : 스레드 풀에서 관리되는 최대 스레드 수
  - `keepAliveTime` , `TimeUnit unit` : 기본 스레드 수를 초과해서 만들어진 초과 스레드가 생존할 수 있는 대기 시간, 이 시간 동안 처리할 작업이 없다면 초과 스레드는 제거된다. 
  - `BlockingQueue workQueue` : 작업을 보관할 블로킹 큐 


### Executor 스레드 풀 관리 - 고정 풀 전략 

**newFixedThreadPool(nThreads)** 

- 스레드 풀에 `nThreads` 만큼의 기본 스레드를 생성한다. 초과 스레드는 생성하지 않는다.
- 큐 사이즈에 제한이 없다. ( `LinkedBlockingQueue` )
- 스레드 수가 고정되어 있기 때문에 CPU, 메모리 리소스가 어느정도 예측 가능한 안정적인 방식이다. 
