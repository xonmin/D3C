  # Chapter 03. Blocking I/O & Non-Blocking I/O 

  ## 3.1 Blocking I/O

  - I/O :
    - 일반적으로 컴퓨터 시스템이 외부의 입출력 장치들과 데이터를 주고받는 것
    - 웹 어플리케이션 측면 : file I/O, DB I/O, Network I/O (다른 어플리케이션과의 네트워크 통신)
      -  ex_ : Tomcat 같은 서블릿 컨테이너에서 실행되는 어플리케이션의 경우 동적인 데이터 처리를 위해 I/O는 반드시 발생
     
![image](https://github.com/xonmin/D3C/assets/27190617/47fa6947-0f0d-405e-842a-45f1642690f1)

- 본사 API 서버에서는 결국 지점 API 서버에 요청을 보내는 시점에 요청 스레드는 차단되어 지점의 응답 전까지 대기(Blocking)
- 즉, **하나의 스레드가 I/O에 의해 차단되어 대기하는 것 = Blocking I/O**

해당 문제점을 위해 멀티스레딩 기법으로 추가 스레드를 할당하여 Blocking 된 시간에 효율적으로 사용할 수 있음
- 하지만 CPU 대비 많은 수의 스레드를 할당하는 멀티스레딩 기법의 문제점은 존재

### Context Switch
- 이하 CS 라 칭함
- 기존에 실행하는 프로세스의 정보를 PCB (Process Control Block) 에 save, 다른 PCB를 load 하는 과정 
- 프로세스의 컨텍스트 스위칭에 의해 두 개의 프로그램이 동시에 실행되는 것으로 보이지만, 컴퓨터 상에서의 두 개의 프로그램을 번갈아 가면서 실행
- PCB 에 save / load 시 일정시간 소요 및 CPU는 대기 상태
- 따라서 CS 가 많을 수록 CPU 전체의 idle 상태가 길어져 성능 저하 발생
- 마찬가지로 스레드간에는 CS가 존재, TCB(Thread Control Block)에 save, load
  - 프로세스 CS에 비해 오버헤드가 적지만, 대량의 멀티스레드가 지속적으로 생성된다면 마찬가지로 문제가 될 수 있음
 
### 메모리 과사용으로 인한 오버헤드 
- 일반적으로 새로운 스레드 실행시, JVM 에서는 스레드를 위한 Stack 할당
- 새로운 스레드 정보는 Stack 내 개별 Frame 형태 (Stack Frame) 으로 저장
- JVM Default Stack size : if 64bit -> 1024KB
  - if) 64000 요청 -> 1 request => 1 thread => then) 64GB 메모리가 필요
  - 일반적으로 서블릿 컨테이너 기반의 Java 어플리케이션 : 요청당 하나의 스레드 할당

 ### Thread Pool 응답 지연 발생 가능성 
 - Spring Boot 은 Tomcat 서블릿 컨테이너 내장
 - Tomcat : 요청 처리를 위해 스레드 풀 사용
 - 대량의 요청이 발생하여 스레드 풀에 사용 가능한 유휴 스레드가 없는 경우, 가용 스레드 확보전까지는 응답 지연 발생

## 3.2 Non-Blocking I/O 
![image](https://github.com/xonmin/D3C/assets/27190617/7951a77f-93f1-4911-9ff7-998e2aad707d)

- 만약 Blocking 방식이었다면, A지점 API 서버의 요청 처리 동안, B 지점 API 서버 요청 불가
- **Non-Blocking** : 작업 스레드 종료 여부와 관계 없이 요청 스레드가 차단되지 않음
  - 스레드가 차단되지 않기 떄문에, 하나의 스레드로 많은 수의 요청 처리 가능
- Blocking 방식에 비해 더 적은 수의 스레드를 사용하기 떄문에 Blocking 방식에서 멀티스레딩 기법 사용시의 문제점이 발생하지 않음
- CPU 대기 시간 및 사용량에 있어서 효율적이다.

### 단점 
- 스레드 내부에서 CPU-burst 가 높은 작업이 포함된 경우 성능에 악영향
- 사용자의 요청에서 응답까지의 전체 과정에 Blocking 요소가 포함된 경우, 진정한 Non-Blocking의 이점이 없음

### Fully Non-Blocking I/O 
만약 A 서버에서 DB I/O 처리 후 B/C 서버로 요청을 보내 해당 응답들에 대한 취합 결과를 최종 응답으로 보내는 경우, 
DB I/O, 다른 서버로 보내는 동안의 Network I/O 모두 Non-Blocking 이어야 Non-Blocking의 이점을 발휘할 수 있다. 
- 중간에 Blocking이 존재한다면, 스레드 차단으로 인한 병목 구간 발생

## 3.3 Spring Framework에서의 Blocking / Non-Blocking I/O 
- Spring MVC = Blocking I/O 방식 
- Spring Webflux = Non-Blocking I/O 방식 
  - 최근 기술 발전으로 인해, Blocking I/O 가 감당하지 못할 요청 트래픽이 발생

![image](https://github.com/xonmin/D3C/assets/27190617/bb8dfa53-f3d7-47a6-b779-228c59ad0078)

차이점 : 
- MVC : 서블릿 컨테이너 기반 - 1 request : 1 thread, 대량의 요청 처리를 위해서는 과도한 스레드 사용
- Webflux : Netty 기반, EventLoop 를 통해 적은 수의 스레드로 많은 수의 요청 처리
  - Netty : 자바 네트워크 프레임워크, 자바의 NIO 방식(Non-Blocking Input Ouput)
 
본사 Blocking API 서버 Controller 
```java
@Slf4j
@RestController
@RequiredArgsConstructor
@RequestMapping("/v1/books")
public class SpringMvcHeadOfficeController {
    private final RestTemplate restTemplate;

    URI baseUri = UriComponentsBuilder.newInstance()
            .scheme("http")
            ...
            .build();

    @ResponseStatus(HttpStatus.OK)
    @GetMapping("/{book-id}")
    public ResponseEntity<Book> getBook(@PathVariable("book-id") Long bookId) {
        URI getBookUri = UriComponentsBuilder.fromUri(baseUri)
                .path("/{book-id}")
                .build()
                .expand(bookId)
                .encode()
                .toUri(); 
        
        ResponseEntity<Book> response = restTemplate.getForEntity(getBookUri, Book.class);
        Book book = response.getBody();
        return ResponseEntity.ok(book);
    }
}
```

지점 Blocking API 서버 Controller 
```java
@Slf4j
@RestController
@RequiredArgsConstructor
@RequestMapping("/v1/books")
public class SpringMvcBranchOfficeController {
    private final Map<Long, Book> bookMap; 

    @ResponseStatus(HttpStatus.OK)
    @GetMapping("/{book-id}")
    public ResponseEntity<Book> getBook(@PathVariable("book-id") Long bookId) throws InterruptedException {
        Thread.sleep(5000); // 오래 걸리는 작업을 대체 
        return ResponseEntity.ok(bookMap.get(bookId));
    }
}
```
클라이언트 요청을 전달받은 MVC 애플리케이션의 현재 스레드 내에서 또 다른 요청 전송으로 인한 네트워크 I/O 발생시, 현재 스레드가 차단되느냐가 중요하다. 

클라이언트 코드 요약으로 대체 
- 본사 API 5회 호출 : 전체 응답시간 = 약 25초

본사 Non-Blocking API 서버 Controller 
```java
@Slf4j
@RestController
@RequiredArgsConstructor
@RequestMapping("/v1/books")
public class SpringMvcHeadOfficeController {
    private final RestTemplate restTemplate;

    URI baseUri = UriComponentsBuilder.newInstance()
            .scheme("http")
            ...
            .build();

    @ResponseStatus(HttpStatus.OK)
    @GetMapping("/{book-id}")
    public Mono<Book> getBook(@PathVariable("book-id") Long bookId) {
        URI getBookUri = UriComponentsBuilder.fromUri(baseUri)
                .path("/{book-id}")
                .build()
                .expand(bookId)
                .encode()
                .toUri();
        
        return WebClient.create()
                .get()
                .uri(getBookUri)
                .retrieve()
                .bodyToMono(Book.class);
    }
}
```
- WebClient 를 통해 요청 전송 후, Mono 타입으로 객체 반환

지점 Non-Blocking API 서버 Controller 
```
@Slf4j
@RestController
@RequiredArgsConstructor
@RequestMapping("/v1/books")
public class SpringMvcBranchOfficeController {

    @ResponseStatus(HttpStatus.OK)
    @GetMapping("/{book-id}")
    public Mono<Book> getBook(@PathVariable("book-id") Long bookId) throws InterruptedException {
        Thread.sleep(5000); // 오래 걸리는 작업을 대체
        return Mono.just(bookMap.get(bookId));
    }
}
```

클라이언트 코드 
- Publisher의 subscribe() 을 통해 응답 데이터 받은 후 처리
- Mono : Reactor에서 지원하는 Publisher 타입
- 결과적으로 약 5.5초 정도 소요

## 3.4 Non-Blocking I/O 방식의 통신이 적합한 시스템 

### Webflux 도입시 고려할 사항 
1. 학습난이도
  - DI, AOP, 서비스 추상화등을 이해하고 있다는 가정하에 학습 난도 : MVC < Webflux 
2. 개발 인력 확보
  - 숙련된 인력을 확보하는 것이 MVC 에 비해 어려움 (기술 부채로 이어질 가능성)

### 적합한 시스템 
**대량 요청 트래픽 발생**
- 일반적인 요청 트래픽이 감당할 수준이라면 서블릿 기반 Blocking I/O 방식의 어플리케이션이어도 충분
- 물론 서버 증설, VM 확장 등을 통해 트래픽 분산이 가능하지만, 그만큼의 비용 지불
- Webflux를 통해 적은 컴퓨팅 파워로 고수준의 성능을 끌어올릴 수 있음

**MSA 시스템**
- MSA 특성 간 서비스간 통신에서 Network I/O 다량 발생
- 각 서비스간 통신에서 Blocking 으로 인한 응답 지연 발생 시, 다른 서비스에 악영향

**스트리밍 및 실시간 시스템**
- 무한 데이터 스트림 처리를 위한 스트리밍 또는 실시간 시스템에 적합 
