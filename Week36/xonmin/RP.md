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

# 06. 마블 다이어그램(Marble Diagram) 

## 6.1 마블 다이어그램이란?
- Reactor 에서 지원하는 Operator 를 이해하는 다이어그램

![image](https://github.com/xonmin/D3C/assets/27190617/d46fa62a-f330-4225-9f1e-79638df46d8a)

- 다이어그램에는 두 개의 타임라인이 존재, 상단의 타임라인은 Publisher가 데이터를 emit 하는 타임라인
  - 해당 Publisher는 데이터 소스를 최초로 emit 할 수도, 그렇지 않을 수도 있다.
- 우측으로 갈수록 타임라인이 흐르기 때문에, 좌측에 있는 상단 타임라인의 데이터가 가장 먼저 emit 되는 데이터
- Publisher의 타임라인 중 가장 마지막 수직으로 된 바는 데이터의 emit 이 정상적으로 끝났음을 의미한다.
  - Singal로 표현시, `onComplete` Singal 에 해당
- 가운데 `map` Operator 함수 쪽으로 들어가는 점선 화살표는 emit 된 데이터가 Operator 함수의 입력으로 전달되는 것을 의미
- 가운데 큰 네모는 데이터를 처리하는 Operator함수
- 하단의 타임라인은 Operator에서 가공되어 처리되어 출력으로 내보내지는 데이터의 타임라인(Flux의 경우 Output Flux 라 칭하기도함)
- 만약 하단 타임라인에서 x 표시가 있는 경우는 에러가 발생하여 데이터 처리가 종료됨을 의미하며 `onError()` Singnal 에 해당한다.


### Mono 마블 다이어그램 
![image](https://github.com/xonmin/D3C/assets/27190617/a2044bae-35ed-4c49-b65e-3fe3eccf727b)
- Mono : 단 하나의 데이터를 emit 하는 Publisher
- 이 떄 다이어그램 상에서는 DownStream 에서 error 가 발생하여 emit 되지 않는다. (onComplete 에만 전송가능)

예제 코드 
```java
package chapter6;


import reactor.core.publisher.Mono;

/**
 * Mono 기본 개념 예제
 *  - 1개의 데이터를 생성해서 emit한다.
 */
public class Example6_1 {
    public static void main(String[] args) {
        Mono.just("Hello Reactor")
                .subscribe(System.out::println);
    }
}
```
예제 코드 2 
```java
package chapter6;


import reactor.core.publisher.Mono;

/**
 * Mono 기본 개념 예제
 *  - 원본 데이터의 emit 없이 onComplete signal 만 emit 한다.
 */
public class Example6_2 {
    public static void main(String[] args) {
        Mono
            .empty()
            .subscribe(
                    none -> System.out.println("# emitted onNext signal"),
                    error -> {},
                    () -> System.out.println("# emitted onComplete signal")
            );
    }
}

결과 :# emitted onCompelete singal
```
- `empty()` operator 의 경우 데이터를 전달 받을 필요는 없지만, 작업이 끝났음을 알리고 이에 따른 후처리를 하고 싶을 때 사용
- `subscribte()`의 람다 표현식 파라미터
  - none 람다표현식은 onNext Singal 전송시 실행(Subscriber가 Publisher 로부터 데이터 전달받기 위해 사용)
  - error : onError 발생시 Exception 형태로 전달받기 위해 사용
  - () : Publisher 가 onComplete Signal 전송시 실행, 종료되었음을 알 수 있으며 이에 따른 후처리 가능

예제코드 3 
```java 
package chapter6;

import com.jayway.jsonpath.DocumentContext;
import com.jayway.jsonpath.JsonPath;
import org.springframework.http.HttpEntity;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpMethod;
import org.springframework.http.MediaType;
import org.springframework.web.client.RestTemplate;
import org.springframework.web.util.UriComponentsBuilder;
import reactor.core.publisher.Mono;

import java.net.URI;
import java.util.Collections;

/**
 * Mono 활용 예제
 *  - worldtimeapi.org Open API를 이용해서 서울의 현재 시간을 조회한다.
 */
public class Example6_3 {
    public static void main(String[] args) {
        URI worldTimeUri = UriComponentsBuilder.newInstance().scheme("http")
                .host("worldtimeapi.org")
                .port(80)
                .path("/api/timezone/Asia/Seoul")
                .build()
                .encode()
                .toUri();

        RestTemplate restTemplate = new RestTemplate();
        HttpHeaders headers = new HttpHeaders();
        headers.setAccept(Collections.singletonList(MediaType.APPLICATION_JSON));


        Mono.just(
                    restTemplate
                            .exchange(worldTimeUri,
                                    HttpMethod.GET,
                                    new HttpEntity<String>(headers),
                                    String.class)
                )
                .map(response -> {
                    DocumentContext jsonContext = JsonPath.parse(response.getBody());
                    String dateTime = jsonContext.read("$.datetime");
                    return dateTime;
                })
                .subscribe(
                        data -> System.out.println("# emitted data: " + data),
                        error -> {
                            System.out.println(error);
                        },
                        () -> System.out.println("# emitted onComplete signal")
                );
    }
}
```
- mono의 just() 를 통해 외부 시스템 API 호출 데이터 요청 처리
- map() 을 통해 데이터 파싱 및 변형
실행결과
```
emitted data: 2022-02-09T16:55:15.85956+09:00
emitted onComplete singal
```
- 위의 예시 코드는 Non-blockign I/O 방식 통신이 아니기 때문에 논블락의 이점을 얻을 수 없음 

### Flux 
- 0~n 개의 데이터를 emit 하는 Publisher 타입

```java
public class Example6_4 {
    public static void main(String[] args) {
        Flux.just(6, 9, 13)
                .map(num -> num % 2)
                .subscribe(System.out::println);
    }
}

// 결과 0, 1, 1

public class Example6_5 {
    public static void main(String[] args) {
        Flux.fromArray(new Integer[]{3, 6, 7, 9})
                .filter(num -> num > 6)
                .map(num -> num * 2)
                .subscribe(System.out::println);
    }
}

public class Example6_6 {
    public static void main(String[] args) {
        Flux<String> flux =
                Mono.justOrEmpty("Steve")
                        .concatWith(Mono.justOrEmpty("Jobs"));
        flux.subscribe(System.out::println);
    }
}
```
`justOrEmpty()` : `just()` 의 경우 null을 허용하지 않지만, 반대로 null을 허용 

`concatWith()`: Upstream에서 오는 데이터에 추가적인 데이터를 붙여서 emit 한다. 

![image](https://github.com/xonmin/D3C/assets/27190617/9fe090cd-7968-4f82-86b4-813ab91fc162)

```java 
public class Example6_7 {
    public static void main(String[] args) {
        Flux.concat(
                        Flux.just("Mercury", "Venus", "Earth"),
                        Flux.just("Mars", "Jupiter", "Saturn"),
                        Flux.just("Uranus", "Neptune", "Pluto"))
                .collectList()
                .subscribe(planets -> System.out.println(planets));
    }
}
```
- concat()에서 리턴하는 Publisher 는 Mono? Flux? : Flux - 3개의 Flux 를 9개의 Flux로 emit 한다.
- collectList() 에서 리턴하는 Publisher 는 Mono? Flux? : Mono - List 자체 하나만 반환하므로 mono

