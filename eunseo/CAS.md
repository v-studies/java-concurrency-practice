- CAS(Compare-And-Swap)와 락(Lock) 방식의 비교


자바는 `Atomic` 의 `compareAndSet()` 메서드를 통해 CAS 연산을 지원한다.


- 락(Lock) 방식 
  - 비관적(pessimistic) 접근법
  - 데이터에 접근하기 전에 항상 락을 획득
  - 다른 스레드의 접근을 막음
  - 다른 스레드가 방해할 것이다"라고 가정
  - 단점
     -  **락 획득 대기 시간**: 스레드가 락을 획득하기 위해 대기해야 하므로, 대기 시간이 길어질 수 있다.
     - **컨텍스트 스위칭 오버헤드**: 락을 사용하면, 락 획득을 대기하는 시점과 또 락을 획득하는 시점에 스레드의 상태가 변경된다. 이때 컨텍스트 스위칭이 발생할 수 있으며, 이로 인해 오버헤드가 증가할 수 있다.


- CAS(Compare-And-Swap) 방식
  - 낙관적(optimistic) 접근법
  - 락을 사용하지 않고 데이터에 바로 접근
  - 충돌이 발생하면 그때 재시도 "대부분의 경우 충돌이 없을 것이다"라고 가정
  - 락 프리 : CAS는 락을 사용하지 않기 때문에, 락을 획득하기 위해 대기하는 시간이 없다. 따라서 스레드가 블로킹 되지 않으며, 병렬 처리가 더 효율적일 수 있다. 
  - 단점
    - **충돌이 빈번한 경우**: 여러 스레드가 동시에 동일한 변수에 접근하여 업데이트를 시도할 때 충돌이 발생할 수 있다. 충돌이 발생하면 CAS는 루프를 돌며 재시도해야 하며, 이에 따라 CPU 자원을 계속 소모할 수 있다. 반복적인 재
시도로 인해 오버헤드가 발생할 수 있다.
    - **스핀락과 유사한 오버헤드**: CAS는 충돌 시 반복적인 재시도를 하므로, 이 과정이 계속 반복되면 스핀락과 유사한
성능 저하가 발생할 수 있다. 특히 충돌 빈도가 높을수록 이런 현상이 두드러진다.


CAS 락 구현

```java
package thread.cas.spinlock;

import java.util.concurrent.atomic.AtomicBoolean;
import static util.MyLogger.log;

public class SpinLock {
    private final AtomicBoolean lock = new AtomicBoolean(false);

    public void lock() {
        log("락 획득 시도");
        while (!lock.compareAndSet(false, true)) {
            // 락을 획득할 때까지 스핀 대기(바쁜 대기) 한다.
            log("락 획득 실패 - 스핀 대기");
        }
        log("락 획득 완료");
    }

    public void unlock() {
        lock.set(false);
        log("락 반납 완료");
    }
}

```
