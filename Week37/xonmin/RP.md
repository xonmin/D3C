# 07. Cold / Hot Sequence 
- Hot : 무엇인가 처음부터 다시 시작하지 않고, 같은 작업이 반복되지 않는 느낌 
- Cold : 처음부터 새로 시작해야하며, 새로 시작하기 때문에 같은 시작이 반복되는 

> Cold 는 새로 시작 / Hot 은 새로 시작하지 않음 

## 7.2 Cold Sequence 
대충 Sequence 가 다시 시작한다는 의미 정도 

![image](https://github.com/xonmin/D3C/assets/27190617/8eb68010-3867-45e7-831a-1ed81efc044b)


- 소비자가 구독할 떄마다 4개의 데이터를 emit 한다. 
- 구독 시점이 다르지만, 모두 동일한 데이터를 전달받음 

결론 : Cold Sequence - Subscriber의 구독 시점이 달라도 구독을 할 때마다 Publisher가 데이터를 emit 하는 과정을 처음부터 다시 시작

```java 
@Slf4j
public class Example7_1 {
    public static void main(String[] args) throws InterruptedException {

        Flux<String> coldFlux =
                Flux
                    .fromIterable(Arrays.asList("KOREA", "JAPAN", "CHINESE"))
                    .map(String::toLowerCase);

        coldFlux.subscribe(country -> log.info("# Subscriber1: {}", country));
        System.out.println("----------------------------------------------------------------------");
        Thread.sleep(2000L);
        coldFlux.subscribe(country -> log.info("# Subscriber2: {}", country));
    }
}
```

9번 라인에서 첫번째 구독이 발생하며, 12번 라인에서 두번째 구독이 발생할 수 있다. 

## 7.3 Hot Sequence 
- Cold Sqeunce의 경우 구독이 발생할 때마다 Sequence의 타임라인이 처음부터 새로 시작하기 때문에 Subscriber는 구독 시점과 상관없이 데이터를 처음부터 다시 전달받을 수 있음 
- 반면, Hot Sequence의 경우 구독이 발생한 시점 이전에 Publisher로부터 emit 된 데이터는 Subscriber가 전달받지 못하고, **구독이 발생된 시점 이후에 emit된 데이터만 전달받을 수 있다.**
![image](https://github.com/xonmin/D3C/assets/27190617/9237b168-cbbb-408c-834d-033c14b7a04c)

- Cold Sequence 의 경우 두 번의 구독이 발생하기 때문에 타임라인이 두 번 생성되지만, 
다음 그림과 같이 3번의 구독이 발생하여도 타임라인은 하나밖에 생성되지 않는다. 
- 즉, 구독이 아무리 많이 발생해도 Publisher가 데이터를 처음부터 emit하지 않는다. 
  - 첫번째 Subscriber는 1~7번까지의 데이터
  - 두번째 Subscriber 5~7, 세번째는 7만 받는다. 

ex> Hot Sequence는 현실 세계에서 잡지 구독을 했을 때 5월부터 구독시, 5월 간부터 받을 수 있음을 비유가능 
```java
@Slf4j
public class Example7_2 {
    public static void main(String[] args) throws InterruptedException {
        String[] singers = {"Singer A", "Singer B", "Singer C", "Singer D", "Singer E"};

        log.info("# Begin concert:");
        Flux<String> concertFlux =
                Flux
                    .fromArray(singers)
                    .delayElements(Duration.ofSeconds(1))
                    .share();

        concertFlux.subscribe(
                singer -> log.info("# Subscriber1 is watching {}'s song", singer)
        );

        Thread.sleep(2500);

        concertFlux.subscribe(
                singer -> log.info("# Subscriber2 is watching {}'s song", singer)
        );

        Thread.sleep(3000);
    }
}
```
- `delayElement()`는 각 데이터의 emit 을 일정시간 동안 지연시키는 Operator 
- `share()`: 원본 Flux를 멀티캐스트(공유)하는 새로운 Flux를 return 한다. 
- 실행시 로깅에는 main 스레드와, parallel-1, ... 스레드가 실행되는데,  이는 `delayElements()` 는 디폴트 스레드 스케줄러가 parallel 이기 때문이다. 


## 7.4 HTTP 요청/응답에서의 Cold/Hot Sequence 흐름 

### Cold Sequence 예제 
```java
@Slf4j
public class Example7_3 {
    public static void main(String[] args) throws InterruptedException {
        URI worldTimeUri = UriComponentsBuilder.newInstance().scheme("http")
                .host("worldtimeapi.org")
                .port(80)
                .path("/api/timezone/Asia/Seoul")
                .build()
                .encode()
                .toUri();

        Mono<String> mono = getWorldTime(worldTimeUri);
        mono.subscribe(dateTime -> log.info("# dateTime 1: {}", dateTime));
        Thread.sleep(2000);
        mono.subscribe(dateTime -> log.info("# dateTime 2: {}", dateTime));

        Thread.sleep(2000);
    }

    private static Mono<String> getWorldTime(URI worldTimeUri) {
        return WebClient.create()
                .get()
                .uri(worldTimeUri)
                .retrieve()
                .bodyToMono(String.class)
                .map(response -> {
                    DocumentContext jsonContext = JsonPath.parse(response);
                    String dateTime = jsonContext.read("$.datetime");
                    return dateTime;
                });
    }
}
```
- `getWorldTime()` 을 통해 Mono를 전달받고, Cold Sequence에서는 구독이 발생하지 않으면, emit을 하지 않기 때문에 subscribe() 이 일어났을 때 emit 된다.
- 이어서 2초의 지연 시간 이후에 두번째 구독에서 return 값으로 2초가 더 늦은 결과가 리턴된다. 
- 즉 두 번의 새로운 HTTP 요청이 발생하므로 두 결과의 시간은 2초 정도 차이가 난다. 

### Hot Sequence 예제 
```java
@Slf4j
public class Example7_4 {
    public static void main(String[] args) throws InterruptedException {
        URI worldTimeUri = UriComponentsBuilder.newInstance().scheme("http")
                .host("worldtimeapi.org")
                .port(80)
                .path("/api/timezone/Asia/Seoul")
                .build()
                .encode()
                .toUri();

        Mono<String> mono = getWorldTime(worldTimeUri).cache();
        mono.subscribe(dateTime -> log.info("# dateTime 1: {}", dateTime));
        Thread.sleep(2000);
        mono.subscribe(dateTime -> log.info("# dateTime 2: {}", dateTime));

        Thread.sleep(2000);
    }

    private static Mono<String> getWorldTime(URI worldTimeUri) {
        return WebClient.create()
                .get()
                .uri(worldTimeUri)
                .retrieve()
                .bodyToMono(String.class)
                .map(response -> {
                    DocumentContext jsonContext = JsonPath.parse(response);
                    String dateTime = jsonContext.read("$.datetime");
                    return dateTime;
                });
    }
}
```
- cache() 는 Cold Sequence로 동작하는 Mono 를 Hot Sequence 로 변경해주고 emit 된 데이터를 캐시한 뒤 구독이 발생할 때마다 캐시된 데이터를 전달한다. 
- 결과적으로는 캐시된 데이터를 전달하기 때문에 구독이 발생할 때마다 Subscriber 는 동일한 데이터를 전달받게 된다 
