### 시간 기반(Time-based) 테스트
- 가상의 시간(Virtual Time)을 이용해 미래에 실행 될 Sequence를 미리 테스트 할 수 있다.

``` java
// 대상 클래스
package chapter13;

import reactor.core.publisher.Flux;
import reactor.util.function.Tuple2;
import reactor.util.function.Tuples;

public class TimeBasedTestExample {
    public static Flux<Tuple2<String, Integer>> getCOVID19Count(Flux<Long> source) {
        return source
                .flatMap(notUse -> Flux.just(
                                Tuples.of("서울", 10),
                                Tuples.of("경기도", 5),
                                Tuples.of("강원도", 3),
                                Tuples.of("충청도", 6),
                                Tuples.of("경상도", 5),
                                Tuples.of("전라도", 8),
                                Tuples.of("인천", 2),
                                Tuples.of("대전", 1),
                                Tuples.of("대구", 2),
                                Tuples.of("부산", 3),
                                Tuples.of("제주도", 0)
                        )
                );
    }

    public static Flux<Tuple2<String, Integer>> getVoteCount(Flux<Long> source) {
        return source
                .zipWith(Flux.just(
                                Tuples.of("중구", 15400),
                                Tuples.of("서초구", 20020),
                                Tuples.of("강서구", 32040),
                                Tuples.of("강동구", 14506),
                                Tuples.of("서대문구", 35650)
                        )
                )
                .map(Tuple2::getT2);
    }
}
```

``` java
package chapter13;

import org.junit.jupiter.api.Test;
import reactor.core.publisher.Flux;
import reactor.test.StepVerifier;
import reactor.test.scheduler.VirtualTimeScheduler;

import java.time.Duration;

/**
 * StepVerifier 활용 예제
 * - 주어진 시간을 앞당겨서 테스트 한다.
 */
public class ExampleTest13_7 {
    @Test
    public void getCOVID19CountTest() {
        StepVerifier
                .withVirtualTime(() -> TimeBasedTestExample.getCOVID19Count( // VirtualTimeScheduler를 받기 위해
                                Flux.interval(Duration.ofHours(1)).take(1)
                        )
                )
                .expectSubscription()
                .then(() -> VirtualTimeScheduler  // 가상 스케쥴러
                                    .get()
                                    .advanceTimeBy(Duration.ofHours(1))) // 시간을 당겨줌
                .expectNextCount(11)
                .expectComplete()
                .verify();

    }
}
```

- VirtualTimeScheduler = 가상 스케쥴러
- withVirtualTime() 메서드를 이용하여 가상 스케쥴러의 제어를 받을 수 있도록 한다.
- advanceTimeBy()를 이요하여 1시간 빠르게 실행
- expectSubscription()으로 기대값 평가 후, then()으로 후속 작업. 이때, 가상 스케쥴러 사용.
- 위 결과는 passed

``` java
package chapter13;

import org.junit.jupiter.api.Test;
import reactor.core.publisher.Flux;
import reactor.test.StepVerifier;

import java.time.Duration;

/**
 * StepVerifier 활용 예제
 *  -검증에 소요되는 시간을 제한한다.
 */
public class ExampleTest13_8 {
    @Test
    public void getCOVID19CountTest() {
        StepVerifier
                .create(TimeBasedTestExample.getCOVID19Count(
                                Flux.interval(Duration.ofMinutes(1)).take(1) // 1분 뒤 emit
                        )
                )
                .expectSubscription()
                .expectNextCount(11)
                .expectComplete()
                .verify(Duration.ofSeconds(3)); // 소요시간
    }
}

```

- 3초 내에 완료 되는지 확인
- emit이 1분 후이기에 AssertionError

``` java
package chapter13;

import org.junit.jupiter.api.Test;
import reactor.core.publisher.Flux;
import reactor.test.StepVerifier;

import java.time.Duration;

/**
 * StepVerifier 활용 예제
 *  - 지정된 대기 시간동안 이벤트가 없을을 확인한다.
 */
public class ExampleTest13_9 {
    @Test
    public void getVoteCountTest() {
        StepVerifier
                .withVirtualTime(() -> TimeBasedTestExample.getVoteCount(
                                Flux.interval(Duration.ofMinutes(1))
                        )
                )
                .expectSubscription()
                .expectNoEvent(Duration.ofMinutes(1)) // Signal 이벤트 발생하지 않음을 기대
                .expectNoEvent(Duration.ofMinutes(1))
                .expectNoEvent(Duration.ofMinutes(1))
                .expectNoEvent(Duration.ofMinutes(1))
                .expectNoEvent(Duration.ofMinutes(1))
                .expectNextCount(5)
                .expectComplete()
                .verify();
    }
}
```
- withVirtualTime()을 이용해서, VirtualTimeScheduler 제어를 받도록
- expectNotEvent()를 이용하여 1분 동안 onNextSignal 발생하지 않도록 기대. 이때, **지정한 시간만큼 시간을 앞당김**
- 총 5분 동안 시간을 앞당김
- (이해한 바로는) expectNoEvent는 [0, 지정된 시간) 동안 발생 안하면 되도록 해놔서, emit인 정확히 지정된 시간에 발생하니 에러가 발생하지 않는 듯

### Backpressure 테스트

``` java
package chapter13;

import reactor.core.publisher.Flux;
import reactor.core.publisher.FluxSink;

public class BackpressureTestExample {
    public static Flux<Integer> generateNumber() {
        return Flux
                .create(emitter -> {
                    for (int i = 1; i <= 100; i++) {
                        emitter.next(i);
                    }
                    emitter.complete();
                }, FluxSink.OverflowStrategy.ERROR);
    }
}
```

- 숫자 100개 emit
- Backpressure로 ERROR 전략 사용 = 오버플로우 발생 시 OverflowException 발생

``` java
package chapter13;

import org.junit.jupiter.api.Test;
import reactor.test.StepVerifier;

/**
 * StepVerifier Backpressure 테스트 예제
 */
public class ExampleTest13_11 {
    @Test
    public void generateNumberTest() {
        StepVerifier
                .create(BackpressureTestExample.generateNumber(), 1L)
                .thenConsumeWhile(num -> num >= 1)
                .verifyComplete();
    }
}
```

- create로, 데이터 요청 개수를 1로 지정
- thenConsumeWhile()로 emit된 데이터를 소비하지만, 예상한 것보다 많이 수신해서 오버플로우
![image](https://media.oss.navercorp.com/user/21795/files/6d5df01f-6bf5-4825-bf04-3b7693c3f095)
- 오버플로 발생하여 에러 및 테스트 결과 fail

``` java
package chapter13;

import org.junit.jupiter.api.Test;
import reactor.test.StepVerifier;

/**
 * StepVerifier Backpressure 테스트 예제
 */
public class ExampleTest13_12 {
    @Test
    public void generateNumberTest() {
        StepVerifier
                .create(BackpressureTestExample.generateNumber(), 1L)
                .thenConsumeWhile(num -> num >= 1)
                .expectError()
                .verifyThenAssertThat()
                .hasDroppedElements();

    }
}
```

- 오버 플로우 기대 예제
- expectError() : 에러 기대
- verifyThenAssertThat() : 검증을 트리거 & 추가 Assertion
- hasDroppedElements() : drop 데이터가 있음을 assert
- 결과 passed

### context테스트

``` java
package chapter13;

import org.springframework.util.Base64Utils;
import reactor.core.publisher.Mono;

public class ContextTestExample {
    public static Mono<String> getSecretMessage(Mono<String> keySource) {
        return keySource
                .zipWith(Mono.deferContextual(ctx ->
                                               Mono.just((String)ctx.get("secretKey"))))
                .filter(tp ->
                            tp.getT1().equals(
                                   new String(Base64Utils.decodeFromString(tp.getT2())))
                )
                .transformDeferredContextual(
                        (mono, ctx) -> mono.map(notUse -> ctx.get("secretMessage"))
                );
    }
}
```

- Context에 두개의 데이터 저장
  - Base64 형식의 인코딩 된 값. key: secretKey
  - 리턴될 value 값. key: secretValue
- keySource의 값과 Context의 secretKey의 value 값을 비교하여, 같으면 해당 secreteMessage 리턴

``` java
package chapter13;

import org.junit.jupiter.api.Test;
import reactor.core.publisher.Mono;
import reactor.test.StepVerifier;

/**
 * StepVerifier Context 테스트 예제
 */
public class ExampleTest13_14 {
    @Test
    public void getSecretMessageTest() {
        Mono<String> source = Mono.just("hello");

        StepVerifier
                .create(
                    ContextTestExample
                        .getSecretMessage(source)
                        .contextWrite(context ->
                                        context.put("secretMessage", "Hello, Reactor"))
                        .contextWrite(context -> context.put("secretKey", "aGVsbG8="))
                )
                .expectSubscription()
                .expectAccessibleContext()
                .hasKey("secretKey")
                .hasKey("secretMessage")
                .then()
                .expectNext("Hello, Reactor")
                .expectComplete()
                .verify();
    }
}
```

- expectSubscription() : 구독 발생
- expectAccessibleContext() : Context 전파됨을 기대
- hasKey : 전파된 Context에 파라미터 값에 해당하는 값이 있음을 기대. 예제에선 secretKey, secretMessage
- aGVsbG8= : hello를 Base64로 인코딩한 값.
- hello와 context의 secretKey 값인 aGVsbG8=가 같으므로, secretMessage 값인 "Hello, Reactor" 반환
- 결과 pass

### Record 기반 테스트
- 단순 expectNext()로 emit되서 생성되는 기대값이 아닌, 더 구체적은 Assertion이 필요한 경우 사용
- recordWith()을 사용하여, emit된 데이터를 기록하는 세션을 시작.

``` java
package chapter13;

import reactor.core.publisher.Flux;

public class RecordTestExample {
    public static Flux<String> getCapitalizedCountry(Flux<String> source) {
        return source
                .map(country -> country.substring(0, 1).toUpperCase() +
                                country.substring(1));
    }
}
```

- 첫 글자만 대문자로 변환하여 Flux로 반환

``` java
package chapter13;

import org.junit.jupiter.api.Test;
import reactor.core.publisher.Flux;
import reactor.test.StepVerifier;

import java.util.ArrayList;

import static org.hamcrest.MatcherAssert.assertThat;
import static org.hamcrest.Matchers.is;

/**
 * StepVerifier Record 테스트 예제
 */
public class ExampleTest13_16 {
    @Test
    public void getCountryTest() {
        StepVerifier
                .create(RecordTestExample.getCapitalizedCountry(
                        Flux.just("korea", "england", "canada", "india")))
                .expectSubscription()
                .recordWith(ArrayList::new)
                .thenConsumeWhile(country -> !country.isEmpty())
                .consumeRecordedWith(countries -> {
                    assertThat(
                            countries
                                    .stream()
                                    .allMatch(country ->
                                            Character.isUpperCase(country.charAt(0))),
                            is(true)
                    );
                })
                .expectComplete()
                .verify();
    }
}
```

- recordWith() : emit된 데이터 기록 시작
- thenConsumeWhile() : 데이터 소비
- consumeRecordedWith() : 기록된 데이터를 소비 및 첫 글자 대문자인지 판별

``` java
package chapter13;

import org.junit.jupiter.api.Test;
import reactor.core.publisher.Flux;
import reactor.test.StepVerifier;

import java.util.ArrayList;

/**
 * StepVerifier Record 테스트 예제
 */
public class ExampleTest13_17 {
    @Test
    public void getCountryTest() {
        StepVerifier
                .create(RecordTestExample.getCapitalizedCountry(
                        Flux.just("korea", "england", "canada", "india")))
                .expectSubscription()
                .recordWith(ArrayList::new)
                .thenConsumeWhile(country -> !country.isEmpty())
                .expectRecordedMatches(countries ->
                        countries
                                .stream()
                                .allMatch(country ->
                                        Character.isUpperCase(country.charAt(0))))
                .expectComplete()
                .verify();
    }
}
```

- 13_16 코드와 동일하지만, assertThat대신 expectRecordedMatches로 Predicate을 사용하여 확인.

## 13.2 TestPublisher를 사용한 테스팅
- 테스트 전용 Publiser인 TestPublisher 이용 가능

### 정상 동작(Well-behaved) TestPublisher

``` java
package chapter13;

import reactor.core.publisher.Flux;

public class GeneralTestExample {
    public static Flux<String> sayHello() {
        return Flux
                .just("Hello", "Reactor");
    }

    public static Flux<Integer> divideByTwo(Flux<Integer> source) {
        return source
                .zipWith(Flux.just(2, 2, 2, 2, 0), (x, y) -> x/y);
    }

    public static Flux<Integer> takeNumber(Flux<Integer> source, long n) {
        return source
                .take(n);
    }
}
```

``` java
package chapter13;

import org.junit.jupiter.api.Test;
import reactor.test.StepVerifier;
import reactor.test.publisher.TestPublisher;

/**
 * 정상동작 하는 TestPublisher 예제
 */
public class ExampleTest13_18 {
    @Test
    public void divideByTwoTest() {
        TestPublisher<Integer> source = TestPublisher.create();

        StepVerifier
                .create(GeneralTestExample.divideByTwo(source.flux()))
                .expectSubscription()
                .then(() -> source.emit(2, 4, 6, 8, 10))
                .expectNext(1, 2, 3, 4)
                .expectError()
                .verify();
    }
}
```

- TestPublisher.create()로 TestPublisher 생성
- 테스트 대상의 파라미터가 Flux. 맞춰서 생성된 TestPublisher의 flux() 사용
- emit 메서드를 사용하여 데이터 emit
- 굳이 필요한가? 복잡한 로직이 포함된 대상 메서드를 테스트하거나, 조건에 따라 Signal 변경해야 되는 등의 특정 상황 테스트 용이

<details><summary>TestPubliser가 발생시키는 Signal 유형</summary>
- next(T) 또는 next(T, t...): 1개 이상의 onNext Signal
- emit(T...): 1개 이상의 onNext Singal 후, onComplete
- complete(): onComplete()
- error(Throwable) : onError Signal
</details>

### 오동작(Misbehaving) TestPublisher
- 리액티브 스트림즈 사양 위반 여부를 사전 체크하지 않는다.
- 사양에 위반되도 데이터 emit
- 위반 조건
  - ALLOW_NULL: 전송할 데이터가 null이어도 NullpointerException
  - CLEANUP_ON_TERMINATE: onComplete, onError, emit 같은 Terminal Signal을 연달아 여러번 보낼 수 있음
  - DEFER_CANCELLATION: cancel Signal 무시하고 계속 emit
  - REQUEST_OVERFLOW: 요청 개수보다 더 많은 Signal 발생해도 IllegalStateException 발생하지 않고 다음 호출 실행

``` java
package chapter13;

import org.junit.jupiter.api.Test;
import reactor.test.StepVerifier;
import reactor.test.publisher.TestPublisher;

import java.util.Arrays;
import java.util.List;

/**
 * 오동작 하는 TestPublisher 예제
 */
public class ExampleTest13_19 {
    @Test
    public void divideByTwoTest() {
        TestPublisher<Integer> source = TestPublisher.create();
//        TestPublisher<Integer> source =
//                TestPublisher.createNoncompliant(TestPublisher.Violation.ALLOW_NULL);

        StepVerifier
                .create(GeneralTestExample.divideByTwo(source.flux()))
                .expectSubscription()
                .then(() -> {
                    getDataSource().stream()
                            .forEach(data -> source.next(data));
                    source.complete();
                })
                .expectNext(1, 2, 3, 4, 5)
                .expectComplete()
                .verify();
    }

    private static List<Integer> getDataSource() {
        return Arrays.asList(2, 4, 6, 8, null);
    }
}
```
- `ALLOW_NULL` 위반 조건을 지정하여, null이라도 정상 동작하는 TestPublisher 생성
- **위 경우 onNext Signal 정송 전, Validation 과정을 거쳐 전송할 데이터가** null이라 NullPointerException 발생.

``` java
//        TestPublisher<Integer> source = TestPublisher.create();
        TestPublisher<Integer> source =
                TestPublisher.createNoncompliant(TestPublisher.Violation.ALLOW_NULL);
```

- **onNext Signal을 전송 하는 과정에서** NullPointerException 발생.

## 13.3 PubliserProbe를 사용한 테스팅
- Sequence의 실행 경로 테스트 가능
- 조건에 따라 Sequence 분기 시, 경로를 추적해서 정상적 실행 테스트 가능

``` java
package chapter13;

import reactor.core.publisher.Mono;

public class PublisherProbeTestExample {
    public static Mono<String> processTask(Mono<String> main, Mono<String> standby) {
        return main
                .flatMap(massage -> Mono.just(massage))
                .switchIfEmpty(standby);
    }

    public static Mono<String> supplyMainPower() {
        return Mono.empty();
    }

    public static Mono supplyStandbyPower() {
        return Mono.just("# supply Standby Power");
    }
}
```
- mainPower 끊기면, standByPower 사용해서 제공
- swithIfEmpty로

``` java
package chapter13;

import org.junit.jupiter.api.Test;
import reactor.test.StepVerifier;
import reactor.test.publisher.PublisherProbe;

/**
 * PublisherProbe 예제
 */
public class ExampleTest13_21 {
    @Test
    public void publisherProbeTest() {
        PublisherProbe<String> probe =
                PublisherProbe.of(PublisherProbeTestExample.supplyStandbyPower());

        StepVerifier
                .create(PublisherProbeTestExample
                        .processTask(
                                PublisherProbeTestExample.supplyMainPower(),
                                probe.mono())
                )
                .expectNextCount(1)
                .verifyComplete();

        probe.assertWasSubscribed(); // 구독 여부
        probe.assertWasRequested(); // 요청 여부
        probe.assertWasNotCancelled(); // 중간에 취소 여부
    }
}
```

- standByPower를 PublisherProbe.of() 메서드로 PublisherProbe로 래핑
- StepVerifer로 emit하고 정상 동작 확인
- probe에 저장된 정보로 Publisher 정상 동작 했는데 테스트 및 확인.
