## 10.4 publishOn() 과 subscribeOn()의 동작 이해 
- 원본 Publisher 와 나머지 동작을 역할에 맞게 분리하고자 subscribeOn()과 publishOn() 을 함께 사용하는 경우도 흔함
- publishOn() 이후 실행 스레드는 모두 지정한 스레드
- subscribeOn() 은 구독이 발생한 직후에, 실행될 스레드를 지정하는 Operator

  
책에서는 다음과 같은 Step이 있다고 가정합니다. 
1. `Flux.fromArray()`
2. `.filter()`
3. `.map()`
4. `.subscribe()`


publishOn() 과 SubscribeOn() 을 사용하지 않는 경우 
```java
package chapter10;

import lombok.extern.slf4j.Slf4j;
import reactor.core.publisher.Flux;


/**
 * subscribeOn()과 publishOn()의 동작 과정 예
 *  - subscribeOn()과 publishOn()을 사용하지 않은 경우
 *      - Sequence의 Operator 체인에서 최초의 쓰레드는 subscribe()가
 *        호출되는 scope에 있는 쓰레드이다.
 */
@Slf4j
public class Example10_5 {
    public static void main(String[] args) {
        Flux
            .fromArray(new Integer[] {1, 3, 5, 7})
            .doOnNext(data -> log.info("# doOnNext fromArray: {}", data))
            .filter(data -> data > 3)
            .doOnNext(data -> log.info("# doOnNext filter: {}", data))
            .map(data -> data * 10)
            .doOnNext(data -> log.info("# doOnNext map: {}", data))
            .subscribe(data -> log.info("# onNext: {}", data));
    }
}
```
결과 : 모든 doOnNext() Operator가 main 스레드에서 실행된다. 
```
11:52:34.219 [main] INFO chapter 10. Example10_5 - # doOnNext fromArray: 1
11:52:34.219 [main] INFO chapter 10. Example10_5 - # donNext fromArray: 3
11:52:34.219 [main] INFO chapter10. Example10_5 - # donNext fromArray: 5
11:52:34.219 [main] INFO chapter 10. Example10_5 - # doOnNext filter: 5
11:52:34.219 [main] INFO chapter10. Example10_5 - # donNext map: 50
11:52:34.219 [main] INFO chapter10.Example10_5 - # onNext: 50
11:52:34.219 [main] INFO chapter10. Example10_5 - # donNext fromArray: 7
11:52:34.219 [main] INFO chapter10. Example10_5 - # donNext filter: 7
11:52:34.219 [main] INFO chapter10. Example10_5 - # donNext map: 70
11:52:34.219 [main] INFO chapter10. Example10_5 - # onNext: 70
```
### publishOn() 하나만 사용하는 경우 
- Operator 체인에 publishOn() 을 추가하는 경우 publishOn()에서 지정한 Scheduler 유형의 스레드가 실행된다.
  
- publishOn() 이후 실행 스레드는 모두 지정한 스레드
- subscribeOn() 은 구독이 발생한 직후에, 실행될 스레드를 지정하는 Operator

```java
package chapter10;

import lombok.extern.slf4j.Slf4j;
import reactor.core.publisher.Flux;
import reactor.core.scheduler.Schedulers;


/**
 * subscribeOn()과 publishOn()의 동작 과정 예
 *  - 하나의 publishOn()만 사용한 경우
 *      - publishOn() 아래 쪽 Operator들의 실행 쓰레드를 변경한다.
 *
 */
@Slf4j
public class Example10_6 {
    public static void main(String[] args) throws InterruptedException {
        Flux
            .fromArray(new Integer[] {1, 3, 5, 7})
            .doOnNext(data -> log.info("# doOnNext fromArray: {}", data))
            .publishOn(Schedulers.parallel())
            .filter(data -> data > 3)
            .doOnNext(data -> log.info("# doOnNext filter: {}", data))
            .map(data -> data * 10)
            .doOnNext(data -> log.info("# doOnNext map: {}", data))
            .subscribe(data -> log.info("# onNext: {}", data));

        Thread.sleep(500L);
    }
}
```
결과 : publishOn 이후 모두 parallel-1 에서 실행 
```
14:42:26.140 [main] INFO chapter10. Example10_6 - # doOnNext fromArray: 1
14:42:26.141 [main] INFO chapter10. Example10_6 - # doOnNext fromArray: 3
14:42:26.141 [main] INFO chapter10. Example10_6 - # doOnNext fromArray: 5
14:42:26.141 [main] INFO chapter10. Example10_6 - # donNext fromArray: 7
14:42:26.141 [parallel-1] INFO chapter 10. Example10_6 - # donNext filter: 5
14:42:26.141 [parallel-1] INFO chapter10. Example10_6 - # donNext map: 50
14:42:26.141 parallel-1] INFO chapter10.Example10_6 - # onNext: 50
14:42:26.141 [parallel-1] INFO chapter10. Example10_6 - # doOnNext filter: 7
14:42:26.141 [parallel-1] INFO chapter10. Example10_6 - # doOnNext map: 70
14:42:26.141 [parallel-1] INFO chapter10. Example10_6 - # onNext: 70
```

### publishOn() 두 번 사용 

> 이 때 publishOn() 은 Operator 체인상에서 한 개 이상 사용할 수 없음
```java
package chapter10;

import lombok.extern.slf4j.Slf4j;
import reactor.core.publisher.Flux;
import reactor.core.scheduler.Schedulers;


/**
 * subscribeOn()과 publishOn()의 동작 과정 예
 *  - 두 개의 publishOn()을 사용한 경우
 *      - 다음 publishOn()을 만나기 전까지 publishOn() 아래 쪽 Operator들의 실행 쓰레드를 변경한다.
 *
 */
@Slf4j
public class Example10_7 {
    public static void main(String[] args) throws InterruptedException {
        Flux
            .fromArray(new Integer[] {1, 3, 5, 7})
            .doOnNext(data -> log.info("# doOnNext fromArray: {}", data))
            .publishOn(Schedulers.parallel())
            .filter(data -> data > 3)
            .doOnNext(data -> log.info("# doOnNext filter: {}", data))
            .publishOn(Schedulers.parallel())
            .map(data -> data * 10)
            .doOnNext(data -> log.info("# doOnNext map: {}", data))
            .subscribe(data -> log.info("# onNext: {}", data));

        Thread.sleep(500L);
    }
}
```
결과
```
16:22:17.697 [main] INFO chapter10. Example10_7 - # doOnNext tromArray: 1
16:22:17.698 [main] INFO chapter10. Example10_7 - # doOnNext fromArray: 3
16:22:17.698 [main] INFO chapter10. Example10_7 - # doOnNext fromArray: 5
16:22:17.698 [main] INFO chapter10. Example10_7 - # doOnNext TromArray: 7
16:22:17.698 [parallel-2] INFO chapter10. Example10_7 - # donNext filter: 5
16:22:17.698 [parallel-2] INFO chapter10. Example10_7 - # doOnNext filter: 7
16:22:17.698 parallel-1] INFO chapter10. Example10_7 - # donNext map: 50
16:22:17.698 Lparallel-1] INFO chapter10. Example10_/ - # onNext: 50
16:22:17.698 [parallel-1] INFO chapter10. Example10_/ - # doOnNext map: 70
16:22:17.698 Iparallel-1] INFO chapter1o.Example10_7 - # onNext: 70
```

### subscribeOn() 과 publishOn() 을 함께 사용할 경우 
- subscribeOn() 은 구독이 발생한 직후에, 실행될 스레드를 지정하는 Operator
 ![image](https://github.com/xonmin/D3C/assets/27190617/3132adb8-b23e-446e-9071-f7fe645633e1)

- fromArray() 는 subscribe()를 통해 구독이 되고나서 발행하므로 A스레드에서 실행
- 별도 publishOn() 을 추가하지 않았으므로 filter() 도 여전히 A스레드
- 마지막으로 publishOn()에서 지정한 스레드들을 통해 이후 체인들은 B스레드에서 실행

```java
package chapter10;

import lombok.extern.slf4j.Slf4j;
import reactor.core.publisher.Flux;
import reactor.core.scheduler.Schedulers;


/**
 * subscribeOn()과 publishOn()의 동작 과정 예
 *  - subscribeOn()과 publishOn()을 함께 사용한 경우
 *      - subscribeOn()은 구독 직후에 실행될 쓰레드를 지정하고, publishOn()을 만나기 전까지 쓰레드를 변경하지 않는다.
 *
 */
@Slf4j
public class Example10_8 {
    public static void main(String[] args) throws InterruptedException {
        Flux
            .fromArray(new Integer[] {1, 3, 5, 7})
            .subscribeOn(Schedulers.boundedElastic())
            .doOnNext(data -> log.info("# doOnNext fromArray: {}", data))
            .filter(data -> data > 3)
            .doOnNext(data -> log.info("# doOnNext filter: {}", data))
            .publishOn(Schedulers.parallel())
            .map(data -> data * 10)
            .doOnNext(data -> log.info("# doOnNext map: {}", data))
            .subscribe(data -> log.info("# onNext: {}", data));

        Thread.sleep(500L);
    }
}
```
결과 
```
18:07:51.465 [boundedElastic-1] INFO chapter10. Example10_8 - # doOnNext fromArray: 1
18:07:51.465 [boundedlastic-1] INFO chapter10. Example10_8 - # donNext fromArray: 3
18:07:51.465 [boundedElastic-1] INFO chapter10. Example10_8 - # doOnNext fromArray: 5
18:07:51.465 boundedElastic-1] INFO chapterio.Example10_8 - # donNext filter: 5
18:07:51.465 [boundedElastic-1] INFO chapter10. Example10_8 - # doOnNext fromArray: 7
18:07:51.465 [boundedElastic-1] INFO chapter10. Example10_8 - # doOnNext filter: 7
18:07:51.465 [parallel-1] INFO chapter10. Example10_8 - # donNext map: 50
18:07:51.465 Lparallel-1] INFO chapter10. Example10_8 - # onNext: 50
18:07:51.465 Lparallel-1] INFO chapter10. Example10_8 - # donNext map: 70
18:07:51.465 [parallel-1] INFO chapter1o. Example10_8 - # onNext: 70
```

> - publishOn() 과 subscribeOn() 의 특징
>  - publishOn() Operator 는 한 개 이상 사용할 수 없으며, 실행 스레드를 목적에 맞게 적절히 분리하는 용도
>  - subscribeOn() 과 publishOn 을 함께 사용하여 원본 Publisher에서 데이터를 emit 하는 스레드와 emit된 데이터를 가공처리하는 스레드로 분리 가능
>  - subscribeOn() 은 체인상 어떤 위치에 있든 간에 구독 시점 직후, 즉 Publisher가 데이터를 emit하기 전에 실행 스레드를 변경한다.

## 10.5 Scheduler 종류

### Schedulers.immediate() 
- 별도의 스레드를 추가적으로 생성하지 않고, 현재 스레드에서 작업으 ㄹ처리하고자 할 때 사용
```java

/**
 * Schedulers.immediate() 예
 *  - 별도의 쓰레드를 할당하지 않고, 현재 쓰레드에서 실행된다.
 *
 */
@Slf4j
public class Example10_9 {
    public static void main(String[] args) throws InterruptedException {
        Flux
            .fromArray(new Integer[] {1, 3, 5, 7})
            .publishOn(Schedulers.parallel())
            .filter(data -> data > 3)
            .doOnNext(data -> log.info("# doOnNext filter: {}", data))
            .publishOn(Schedulers.immediate()) // L6의 parallel 그대로 유지 
            .map(data -> data * 10)
            .doOnNext(data -> log.info("# doOnNext map: {}", data))
            .subscribe(data -> log.info("# onNext: {}", data));

        Thread.sleep(200L);
    }
}
```

> 그대로 스레드 유지하고자한다면, publishOn 을 빼도 되는데 왜 굳이 immediate() 를 선언하는가?
> - 위 예시 코드가 어떤 API 들 중 common 역할을 하는 API 이고, 해당 API 의 파라미터로 Scheduler 를 전달할 수 있을 때,
> - 이 경우 이 API 를 사용하는 입장에서는 map() 이후 Operator() 체인 작업은 원래 실행되던 스레드에서 실행하고 싶을 떄도 있을 것.
> - 즉, Scheduler가 필요한 API가 있긴 한데, 별도의 스레드를 추가 할당하고 싶지 않은 경우, Schedulers.immediate() 사용하면 된다.

###  Schedulers.single() 
스레드 하나만 생성하여 Scheduler가 제거되기 전까지 재사용 
```java
@Slf4j
public class Example10_10 {
    public static void main(String[] args) throws InterruptedException {
        doTask("task1")
                .subscribe(data -> log.info("# onNext: {}", data));

        doTask("task2")
                .subscribe(data -> log.info("# onNext: {}", data));

        Thread.sleep(200L);
    }

    private static Flux<Integer> doTask(String taskName) {
        return Flux.fromArray(new Integer[] {1, 3, 5, 7})
                .publishOn(Schedulers.single())
                .filter(data -> data > 3)
                .doOnNext(data -> log.info("# {} doOnNext filter: {}", taskName, data))
                .map(data -> data * 10)
                .doOnNext(data -> log.info("# {} doOnNext map: {}", taskName, data));
    }
}
```
실행 결과 : 
```
19:57:49.260 [single-1] INFO - # task1 doOnNext filter: 5
19:57:49.263 [single-1] INFO - # task1 doOnNext map: 50
19:57:49.264 [single-1] INFO - # onNext: 50
19:57:49.264 [single-1] INFO - # task1 doOnNext filter: 7
19:57:49.264 [single-1] INFO - # task1 doOnNext map: 70
19:57:49.264 [single-1] INFO - # onNext: 70
19:57:49.265 [single-1] INFO - # task2 doOnNext filter: 5
19:57:49.265 [single-1] INFO - # task2 doOnNext map: 50
19:57:49.265 [single-1] INFO - # onNext: 50
19:57:49.265 [single-1] INFO - # task2 doOnNext filter: 7
19:57:49.265 [single-1] INFO - # task2 doOnNext map: 70
19:57:49.265 [single-1] INFO - # onNext: 70
```
- doTask() 를 두 번 호출하더라도 첫번째 호출에서 이미 생성된 스레드를 재사용

### Schedulers.newSingle() 
호출할때마다 매번 새로운 스레드 하나를 생성한다.
```java
@Slf4j
public class Example10_11 {
    public static void main(String[] args) throws InterruptedException {
        doTask("task1")
                .subscribe(data -> log.info("# onNext: {}", data));

        doTask("task2")
                .subscribe(data -> log.info("# onNext: {}", data));

        Thread.sleep(200L);
    }

    private static Flux<Integer> doTask(String taskName) {
        return Flux.fromArray(new Integer[] {1, 3, 5, 7})
                .publishOn(Schedulers.newSingle("new-single", true)) // 데몬 스레드로 동작하게 할지 유무 
                .filter(data -> data > 3)
                .doOnNext(data -> log.info("# {} doOnNext filter: {}", taskName, data))
                .map(data -> data * 10)
                .doOnNext(data -> log.info("# {} doOnNext map: {}", taskName, data));
    }
}
```
결과
```
19:59:23.266 [new-single-2] INFO - # task2 doOnNext filter: 5
19:59:23.265 [new-single-1] INFO - # task1 doOnNext filter: 5
19:59:23.269 [new-single-2] INFO - # task2 doOnNext map: 50
19:59:23.269 [new-single-1] INFO - # task1 doOnNext map: 50
19:59:23.269 [new-single-2] INFO - # onNext: 50
19:59:23.269 [new-single-1] INFO - # onNext: 50
19:59:23.269 [new-single-2] INFO - # task2 doOnNext filter: 7
19:59:23.269 [new-single-1] INFO - # task1 doOnNext filter: 7
19:59:23.269 [new-single-2] INFO - # task2 doOnNext map: 70
19:59:23.269 [new-single-1] INFO - # task1 doOnNext map: 70
19:59:23.269 [new-single-2] INFO - # onNext: 70
19:59:23.269 [new-single-1] INFO - # onNext: 70
```
- doTask()를 호출할때마다 새로운 스레드 하나를 생성해서 각각의 작업을 처리하는 것을 볼 수 있다.

### 그외 Scheduler 


**Schedulers.boundedElastic()**
- ExecutorService 기반의 스레드 풀을 생성한 후, 그 안에서 정해진 수만큼의 스레드를 사용하여 작업을 처리하고 작업이 종료된 스레드는 반납하여 재사용한다.
- 기본적으로 CPI 코어 수 x 10 만큼의 스레드를 생성한다.
- Blocking I/O 작업을 효과적으로 처라히가 위한 방식이다.
  - 실행시간이 긴 Blocking I/O 작업이 포함된 경우, 다른 Non-Blocking 처리에 영향을 주지 않도록 전용 스레드를 할당해서 Blocking I/O 작업을 처리하기 때문에 처리 시간을 효율적으로 사용할 수 있다.

**Schedulers.parallel()**	
- Non-Blocking I.O에 최적화되어 있는 Scheduler로서 CPU 코어 수만큼의 스레드를 생성한다.

**Schedulers.fromExecutorService()**	
- 기존에 이미 사용하고 있는 ExecutorService가 있다면 이 ExecutorService로부터 Scheduler를 생성하는 방식이다.

**Schedulers.newXXX()**
- single(), boundedElastic(), parallel()은 Reactor에서 제공하는 디폴트 Scheduler 인스턴스를 사용하는데, 필요하다면 newSingle(), newBoundedElastic(), newParallel() 메서드를 이용해서 새로운 Scheduler 인스턴스를 생성할 수 있다.
