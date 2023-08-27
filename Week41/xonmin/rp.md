# 12. Debugging

## 12.1 Reactor에서의 디버깅 방법
- 동기식 또는 명령형 프로그래밍 방식은 Exception이 발생 했을 때 Stacktrace를 확인하거나 Breakpoint를 걸어서 문제가 된 원인을 단계적으로 찾아가므로 디버깅이 쉬움

- Reactor는 처리되는 작업들이 대부분 비동기적으로 실행, Reactor Sequence는 선언형 프로그래밍 방식으로 구성되므로 디버깅이 어려움


### 12.1.1 Debug Mode를 사용한 디버깅

```java
...
Hooks.onOperatorDebug();
...
```

- map Operator가 많아 성능상 오버헤드가 발생 할 수 있다는 경고메시지가 발생함. 실무에선 사용하지말고 디버그 용도로만 사용
- 애플리케이션 내에 있는 `모든 Operator`의 Stacktrace를 캡처, 이를 기반으로 에러가 발생하면 Assembly의 Stacktrace를 원본 Stacktrace에 삽입
    - Assembly: Operator에서 리턴하는 새로운 Mono 또는 Flux 가 선언된 지점
    - Traceback: 에러가 발생한 Operator의 stacktrace를 캡처한 Assembly 정보
- Production 환경에선 캡처비용이 들지 않은 다른 에이전트 디버거를 사용해야함. (eg. Spring Webflux: io.projectreactor:reactor-tool)

```java
java.lang.NullPointerException: The mapper returned a null value.
	at reactor.core.publisher.FluxMapFuseable$MapFuseableSubscriber.onNext(FluxMapFuseable.java:113)
	Suppressed: reactor.core.publisher.FluxOnAssembly$OnAssemblyException: 
Assembly trace from producer [reactor.core.publisher.FluxMapFuseable] :
	reactor.core.publisher.Flux.map(Flux.java:6274)
	chapter12.Example12_1.main(Example12_1.java:31)
Error has been observed at the following site(s):
	*__Flux.map ⇢ at chapter12.Example12_1.main(Example12_1.java:31)
	|_ Flux.map ⇢ at chapter12.Example12_1.main(Example12_1.java:32)
Original Stack Trace:
		at reactor.core.publisher.FluxMapFuseable$MapFuseableSubscriber.onNext(FluxMapFuseable.java:113)
		at reactor.core.publisher.FluxMapFuseable$MapFuseableSubscriber.onNext(FluxMapFuseable.java:127)
		at reactor.core.publisher.FluxMapFuseable$MapFuseableSubscriber.onNext(FluxMapFuseable.java:127)
		at reactor.core.publisher.FluxPublishOn$PublishOnSubscriber.runAsync(FluxPublishOn.java:440)
		at reactor.core.publisher.FluxPublishOn$PublishOnSubscriber.run(FluxPublishOn.java:527)
		at reactor.core.scheduler.WorkerTask.call(WorkerTask.java:84)
		at reactor.core.scheduler.WorkerTask.call(WorkerTask.java:37)
		at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264)
		...
```

### 12.1.2 checkpoint() Operator를 사용한 디버깅
- checkpoint() Operator를 사용하면 특정 Operator 체인 내의 stacktrace만 캡처

#### Traceback을 출력하는 방법
- checkpoint() 지점까지 오류가 전파되었는지 확인 가능

```java
@Slf4j
public class Example12_3 {
    public static void main(String[] args) {
        Flux
            .just(2, 4, 6, 8)
            .zipWith(Flux.just(1, 2, 3, 0), (x, y) -> x/y)
            .checkpoint()
            .map(num -> num + 2)
            .checkpoint()
            .subscribe(
                    data -> log.info("# onNext: {}", data),
                    error -> log.error("# onError:", error)
            );
    }
}
```

```java
java.lang.ArithmeticException: / by zero
	at chapter12.Example12_3.lambda$main$0(Example12_3.java:15)
	Suppressed: reactor.core.publisher.FluxOnAssembly$OnAssemblyException: 
Assembly trace from producer [reactor.core.publisher.FluxZip] :
	reactor.core.publisher.Flux.checkpoint(Flux.java:3352)
	chapter12.Example12_3.main(Example12_3.java:16)
Error has been observed at the following site(s):
	*__checkpoint() ⇢ at chapter12.Example12_3.main(Example12_3.java:16)
	|_ checkpoint() ⇢ at chapter12.Example12_3.main(Example12_3.java:18)
Original Stack Trace:
		at chapter12.Example12_3.lambda$main$0(Example12_3.java:15)
		at reactor.core.publisher.FluxZip$PairwiseZipper.apply(FluxZip.java:982)
		at reactor.core.publisher.FluxZip$PairwiseZipper.apply(FluxZip.java:971)
		at reactor.core.publisher.FluxZip$ZipCoordinator.drain(FluxZip.java:738)
		at reactor.core.publisher.FluxZip$ZipInner.onSubscribe(FluxZip.java:888)
		at reactor.core.publisher.FluxArray.subscribe(FluxArray.java:53)
		at reactor.core.publisher.FluxArray.subscribe(FluxArray.java:59)
		at reactor.core.publisher.Flux.subscribe(Flux.java:8466)
		at reactor.core.publisher.FluxZip$ZipCoordinator.subscribe(FluxZip.java:595)
		at reactor.core.publisher.FluxZip.handleBoth(FluxZip.java:332)
		at reactor.core.publisher.FluxZip.handleArrayMode(FluxZip.java:273)
		at reactor.core.publisher.FluxZip.subscribe(FluxZip.java:137)
		at reactor.core.publisher.Flux.subscribe(Flux.java:8466)
		at reactor.core.publisher.Flux.subscribeWith(Flux.java:8639)
		at reactor.core.publisher.Flux.subscribe(Flux.java:8436)
		at reactor.core.publisher.Flux.subscribe(Flux.java:8360)
		at reactor.core.publisher.Flux.subscribe(Flux.java:8330)
		at chapter12.Example12_3.main(Example12_3.java:19)
```

#### Trackback 출력 없이 식별자를 포함한 Description을 출력해서 에러 발생 지점을 예상하는 방법
- checkpoint(description)을 사용하면 Trackback를 생략하고 description을 통해 에러지점을 알 수 있다.

```java
@Slf4j
public class Example12_4 {
    public static void main(String[] args) {
        Flux
            .just(2, 4, 6, 8)
            .zipWith(Flux.just(1, 2, 3, 0), (x, y) -> x/y)
            .checkpoint("Example12_4.zipWith.checkpoint")
            .map(num -> num + 2)
            .checkpoint("Example12_4.map.checkpoint")
            .subscribe(
                    data -> log.info("# onNext: {}", data),
                    error -> log.error("# onError:", error)
            );
    }
}
```

```java
java.lang.ArithmeticException: / by zero
	at chapter12.Example12_4.lambda$main$0(Example12_4.java:16)
	Suppressed: reactor.core.publisher.FluxOnAssembly$OnAssemblyException: 
Error has been observed at the following site(s):
	*__checkpoint ⇢ Example12_4.zipWith.checkpoint
	|_ checkpoint ⇢ Example12_4.map.checkpoint
Original Stack Trace:
	...
```

#### Trackback과 Description을 모두 출력하는 방법
- checkpoint(description, forceStackTrace)를 사용하면 전부 찍을 수 있음 (forceStackTrace: true)

```java
@Slf4j
public class Example12_5 {
    public static void main(String[] args) {
        Flux
            .just(2, 4, 6, 8)
            .zipWith(Flux.just(1, 2, 3, 0), (x, y) -> x/y)
            .checkpoint("Example12_4.zipWith.checkpoint", true)
            .map(num -> num + 2)
            .checkpoint("Example12_4.map.checkpoint", true)
            .subscribe(
                    data -> log.info("# onNext: {}", data),
                    error -> log.error("# onError:", error)
            );
    }
}
```

```java
java.lang.ArithmeticException: / by zero
	at chapter12.Example12_5.lambda$main$0(Example12_5.java:16)
	Suppressed: reactor.core.publisher.FluxOnAssembly$OnAssemblyException: 
Assembly trace from producer [reactor.core.publisher.FluxZip], described as [Example12_4.zipWith.checkpoint] :
	reactor.core.publisher.Flux.checkpoint(Flux.java:3417)
	chapter12.Example12_5.main(Example12_5.java:17)
Error has been observed at the following site(s):
	*__checkpoint(Example12_4.zipWith.checkpoint) ⇢ at chapter12.Example12_5.main(Example12_5.java:17)
	|_     checkpoint(Example12_4.map.checkpoint) ⇢ at chapter12.Example12_5.main(Example12_5.java:19)
Original Stack Trace:
	...
```

### 12.1.3 log() Operator를 사용한 디버깅
- log() Operator는 Reactor Sequence의 동작을 로그로 출력
  - onSubscribe, request, onNext, ...
- log("logTag", Level.FINE) 처럼 로그 태그와 로그 레벨을 지정할 수 있다.

```java
@Slf4j
public class Example12_7 {
    public static Map<String, String> fruits = new HashMap<>();

    static {
        fruits.put("banana", "바나나");
        fruits.put("apple", "사과");
        fruits.put("pear", "배");
        fruits.put("grape", "포도");
    }

    public static void main(String[] args) {
        Flux.fromArray(new String[]{"BANANAS", "APPLES", "PEARS", "MELONS"})
                .map(String::toLowerCase)
                .map(fruit -> fruit.substring(0, fruit.length() - 1))
                .log()
                .log("Fruit.Substring", Level.FINE)
                .map(fruits::get)
                .subscribe(
                        log::info,
                        error -> log.error("# onError:", error));
    }
}
```

```java
[main] INFO - | onSubscribe([Fuseable] FluxMapFuseable.MapFuseableSubscriber)
[main] INFO - | request(unbounded)
[main] INFO - | onNext(banana)
[main] INFO - 바나나
[main] INFO - | onNext(apple)
[main] INFO - 사과
[main] INFO - | onNext(pear)
[main] INFO - 배
[main] INFO - | onNext(melon)
[main] INFO - | cancel()
[main] ERROR- # onError:
java.lang.NullPointerException: The mapper [chapter12.Example12_7$$Lambda$38/0x0000000800124840] returned a null value.
	at reactor.core.publisher.FluxMapFuseable$MapFuseableSubscriber.onNext(FluxMapFuseable.java:115)
	at reactor.core.publisher.FluxPeekFuseable$PeekFuseableSubscriber.onNext(FluxPeekFuseable.java:210)
	at reactor.core.publisher.FluxMapFuseable$MapFuseableSubscriber.onNext(FluxMapFuseable.java:129)
	at reactor.core.publisher.FluxMapFuseable$MapFuseableSubscriber.onNext(FluxMapFuseable.java:129)
	at reactor.core.publisher.FluxArray$ArraySubscription.fastPath(FluxArray.java:172)
	at reactor.core.publisher.FluxArray$ArraySubscription.request(FluxArray.java:97)
	at reactor.core.publisher.FluxMapFuseable$MapFuseableSubscriber.request(FluxMapFuseable.java:171)
	at reactor.core.publisher.FluxMapFuseable$MapFuseableSubscriber.request(FluxMapFuseable.java:171)
	at reactor.core.publisher.FluxPeekFuseable$PeekFuseableSubscriber.request(FluxPeekFuseable.java:144)
	at reactor.core.publisher.FluxMapFuseable$MapFuseableSubscriber.request(FluxMapFuseable.java:171)
	at reactor.core.publisher.LambdaSubscriber.onSubscribe(LambdaSubscriber.java:119)
	at reactor.core.publisher.FluxMapFuseable$MapFuseableSubscriber.onSubscribe(FluxMapFuseable.java:96)
	at reactor.core.publisher.FluxPeekFuseable$PeekFuseableSubscriber.onSubscribe(FluxPeekFuseable.java:178)
	at reactor.core.publisher.FluxMapFuseable$MapFuseableSubscriber.onSubscribe(FluxMapFuseable.java:96)
	at reactor.core.publisher.FluxMapFuseable$MapFuseableSubscriber.onSubscribe(FluxMapFuseable.java:96)
	at reactor.core.publisher.FluxArray.subscribe(FluxArray.java:53)
	at reactor.core.publisher.FluxArray.subscribe(FluxArray.java:59)
	at reactor.core.publisher.Flux.subscribe(Flux.java:8466)
	at reactor.core.publisher.Flux.subscribeWith(Flux.java:8639)
	at reactor.core.publisher.Flux.subscribe(Flux.java:8436)
	at reactor.core.publisher.Flux.subscribe(Flux.java:8360)
	at reactor.core.publisher.Flux.subscribe(Flux.java:8330)
	at chapter12.Example12_7.main(Example12_7.java:30)
```

---

# 13. Testing

- reactor-test 모듈의 기능을 사용하기 위해 아래 의존성을 추가해야 한다.
```java
dependency {
	testImplementation 'io.projectreactor:reactor-test'
}
```

## 13.1 StepVerifier를 사용한 테스팅
- Flux 또는 Mono를 Reactor Sequence로 정의한 후, 구독 시점에 해당 Operator 체인이 시나리오대로 동작하는지 테스트 하는 것

### Siginal 이벤트 테스트
- Reactor Sequence에서 발생하는 Signal 이벤트를 테스트 하는 것

```java
public class stepVerifierGeneralExample01 {
  @Test
  public void sayHelloReactorTest() {
    StepVerifier
      .create(Mono.just("Reactor"))  // 테스트 대상 Sequence 생성
      .expectNext("Functional")      // emit 된 데이터 기대값 평가
      .as("기대값: Functional")        // 이전 기대값 평가 단계에 대한 설명, 실패시 로그로 나옴
      .expectComplete()              // onComplete Signal 기대값 평가
      .verify();                     // 검증 실행
  }
}
```

```java
java.lang.AssertionError: expectation "기대값: Functional" failed (expected value: Reactor; actual value: Functional)
```

메서드|설명
-|-
expectSubscription()|구독이 이뤄짐을 기대
expectNext(T t)|onNext Signal을 통해 전달되는 값이 t와 같음을 기대
expectComplete()|OnComplete Signal 전송 기대
expectError()|onError Signal 전송 기대
expectNextCount(long count)|구독 시점 또는 이전 expectNext()를 통해 기대값이 평가된 데이터 이후부터 emit 된 데이터 수 count개 기대
expectNoEvent(Duration duration)|주어진 시간 동안 Signal 이벤트가 발생하지 않음 기대
expectAccessibleContext()|구독 시점 이후 Context 전파 기대
expectNextSequence(Iterable<? extends T> iterable)|emit 된 데이터들이 파라미터로 전달된 iterable의 요소와 매치 기대

메서드|설명
-|-
verify()|검증 트리거
verifyComplete()|검증 트리거, onComplete Signal 기대
verifyError()|검증 트리거, onError Signal 기대
verifyTimeout(Duration duration)|검증 트리거, duration 동아 publisher가 종료되지 않음 기대

