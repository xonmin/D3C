# 05. Reactor 개요 

## 5.1 Reactor란? 
리액티브 스트림즈 구현체인 Reactor 는 리액티브 프로그래밍을 위한 라이브러리로, 
Reactor Core 라이브러리는 Spring Webflux 프레임워크 내에 포함되어있다. 

### Reactor 의 특징 
- Reactive Streams
- Non-blocking
- Java's Functional API
- Flux[N]
  - Publisher 타입 중 하나이며, 0~N개의 데이터를 emit 하는 Publisher
- Mono[0|1]
  - Publisher 타입 중 하나이며, 한 건도 emit 하지않거나, 단 한 건만 emit 하는 단발성 데이터 emit에 특화된 Publisher
- Well-suited for msa
- Backpressure-ready network
  - Publisher로부터 전달받은 데이터를 처리하는 데 있어 과부하가 걸리지 않도록 Backpressure 를 지원
  - Publiser로부터 전달되는 대량의 데이터를 Subscriber가 적절히 처리하기 위한 제어 방법

## 5.2 예제 코드로 보는 Reactor 구성요소 
```java
public class Example {
  psvm(String[] args) {
    Flux<String> sequence = Flux.just("he","re");
    sequence.map(data -> data.toUpperCase())
            .subscribe(data -> sout(data));
}
```
- 데이터 개수가 2개 이므로 Publisher 타입 중 Flux 사용
- `just(), `map()` : Reactor 에서 지원하는 Operator 메서드
