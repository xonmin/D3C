## 다수의 Subscriber 에게 Flux를 멀티캐스팅하기 위한 Operator

구독한 Subscriber 모두에게 멀티캐스팅 처리

- 해당하는 Operator : Cold Sequence -> Hot Sequence 로 전환

### **publish()** 

![image](https://github.com/xonmin/D3C/assets/27190617/21eb6ed4-fe0a-4ee2-a9c9-94c64fb45676)

- publish() 는 구독을 하더라도, 구독 시점에 대한 데이터를 emit 하지 않고, connect()를 호출하는 시점에서야 비로소 데이터를 emit
- Hot Seqeunce 로 변환하므로, 구독 시점 이후에 emit 된 데이터만 전달받을  수 있다.

```java
/**
 * multicast 예제
 *  - publish() Operator
 *      - 다수의 Subscriber와 Flux를 공유한다.
 *      - 즉, Cold Sequence를 Hot Sequence로 변환한다.
 *      - connect()가 호출 되기 전까지는 데이터를 emit하지 않는다.
 */
@Slf4j
public class Example14_60 {
    public static void main(String[] args) throws InterruptedException {
        // 0.3s 별로 emit -> ConnnectableFlux 리턴 
				//. connect() 가 호출되지 않았기 때문에 이 시점에 emit 되는 데이터 x
				ConnectableFlux<Integer> flux =
                Flux
                    .range(1, 5)
                    .delayElements(Duration.ofMillis(300L))
                    .publish();

        Thread.sleep(500L);
				// 0.5s 뒤 구독 발생 
        flux.subscribe(data -> log.info("# subscriber1: {}", data));

        Thread.sleep(200L);
				// 0.2s 뒤 구독 발생 
        flux.subscribe(data -> log.info("# subscriber2: {}", data));
				// 해당 시점부터 0.3s 씩 emit 
        flux.connect();

        Thread.sleep(1000L);
				// 1s 뒤 구독 발생 (해당 시점에서는 이미 0.9s 까지 1~3은 이미 emit 되어 4부터)
        flux.subscribe(data -> log.info("# subscriber3: {}", data));

        Thread.sleep(2000L);
    }
}
```

publish 예제 2 

- 관객 입장마다 관객 수 체크 , 두 명 이상 입장시, 콘서트 시작하는 상황

```java
/**
 * multicast 예제
 *  - publish() Operator
 *      - 다수의 Subscriber와 Flux를 공유한다.
 *      - 즉, Cold Sequence를 Hot Sequence로 변환한다.
 *      - connect()가 호출 되기 전까지는 데이터를 emit하지 않는다.
 */
@Slf4j
public class Example14_61 {
    private static ConnectableFlux<String> publisher;
    private static int checkedAudience;
    static {
        publisher =
                Flux
                    .just("Concert part1", "Concert part2", "Concert part3")
                    .delayElements(Duration.ofMillis(300L))
                    .publish();
    }

    public static void main(String[] args) throws InterruptedException {
        checkAudience();
        Thread.sleep(500L);
        publisher.subscribe(data -> log.info("# audience 1 is watching {}", data));
        checkedAudience++;

        Thread.sleep(500L);
        publisher.subscribe(data -> log.info("# audience 2 is watching {}", data));
        checkedAudience++;
				// 콘서트 시작 
        checkAudience();
				
        Thread.sleep(500L);
				// 해당 관객 부터는 콘서트 2부부터 관람 가능 
        publisher.subscribe(data -> log.info("# audience 3 is watching {}", data));

        Thread.sleep(1000L);
    }

    public static void checkAudience() {
        if (checkedAudience >= 2) {
            publisher.connect();
        }
    }
}
```

### autoConnect()

![image](https://github.com/xonmin/D3C/assets/27190617/f5c64b29-15ce-4a5a-b9ff-4cc28f3527c2)

- publish() 와 달리 파라미터로 지정한 숫자 만큼 (minSubscribers) 구독 발생 시, Upstream 에 자동으로 connect() 연결된다.
    - 별도의 connect() 호출할 필요 x

```java
/**
 * multicast 예제
 *  - autoConnect() Operator
 *      - 다수의 Subscriber와 Flux를 공유한다.
 *      - 즉, Cold Sequence를 Hot Sequence로 변환한다.
 *      - 파라미터로 입력한 숫자만큼의 구독이 발생하는 시점에 connect()가 자동으로 호출된다.
 */
@Slf4j
public class Example14_62 {
    public static void main(String[] args) throws InterruptedException {
        Flux<String> publisher =
                Flux
                    .just("Concert part1", "Concert part2", "Concert part3")
                    .delayElements(Duration.ofMillis(300L))
                    .publish()
                    .autoConnect(2);

        Thread.sleep(500L);
        publisher.subscribe(data -> log.info("# audience 1 is watching {}", data));

        Thread.sleep(500L);
        publisher.subscribe(data -> log.info("# audience 2 is watching {}", data));

        Thread.sleep(500L);
        publisher.subscribe(data -> log.info("# audience 3 is watching {}", data));

        Thread.sleep(1000L);
    }
}
```

### refCount

![image](https://github.com/xonmin/D3C/assets/27190617/04171704-d6e6-4f72-9fdf-19109c244e20)

- 파라미터로 입력된 숫자(minSubscriber)만큼 구독이 발생하는 시점에 바로 연결되며, 모든 구독이 취소되거나 Upstream 데이터 emit 종료시, 연결이 해제된다.
- 주로 무한 스트림 상황에서 모든 구독이 취소될 경우 연결 해제하는데 사용

```java
/**
 * multicast 예제
 *  - refCount() Operator
 *      - 다수의 Subscriber와 Flux를 공유한다.
 *      - 즉, Cold Sequence를 Hot Sequence로 변환한다.
 *      - 파라미터로 입력한 숫자만큼의 구독이 발생하는 시점에 connect()가 자동으로 호출된다.
 *      - 모든 구독이 취소되면 Upstream 소스와의 연결을 해제한다.
 */
@Slf4j
public class Example14_63 {
    public static void main(String[] args) throws InterruptedException {
        Flux<Long> publisher =
                Flux
                    .interval(Duration.ofMillis(500))

                    .publish().autoConnect(1);
//                    .publish().refCount(1);
        Disposable disposable =
                publisher.subscribe(data -> log.info("# subscriber 1: {}", data));
				
				
        Thread.sleep(2100L);
				// disconnect -> Upstream 연결 해제 
        disposable.dispose();
				// refCount() 에 대해 다시 minSubscriber 개수가 만족되어 다시 0부터 interval() 시작 
				// 만약 autoConnect(1) 주석이 있다면 0~8로 콘솔 로그 
        publisher.subscribe(data -> log.info("# subscriber 2: {}", data));

        Thread.sleep(2500L);
    }
}

---
21:28:11.750 [parallel-1] INFO - # subscriber 1: 0
21:28:12.250 [parallel-1] INFO - # subscriber 1: 1
21:28:12.750 [parallel-1] INFO - # subscriber 1: 2
21:28:13.249 [parallel-1] INFO - # subscriber 1: 3
21:28:13.852 [parallel-2] INFO - # subscriber 2: 0
21:28:14.352 [parallel-2] INFO - # subscriber 2: 1
21:28:14.852 [parallel-2] INFO - # subscriber 2: 2
21:28:15.352 [parallel-2] INFO - # subscriber 2: 3

```
