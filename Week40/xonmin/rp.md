# Chapter 11. Context 

## 11.1 Context란? 
Context : 어떤 것을 이해하는 데 도움이 될 만한 관련 정보나 이벤트, 상황 (어떠한 상황에서 그 상황을 처리하기 위해 필요한 정보) 
- ex)
  - ServletContext : Servlet이 Servlet Container 와 통신하기 위해 필요한 정보를 제공하는 인터페이스
  - ApplicationContext : Spring 내 어플리케이션의 정보를 제공하는 인터페이스 (대표적인 정보 : Spring Bean 객체)
  - SecurityContext : Spring Security 에서 어플리케이션 사용자의 인증 정보를 제공하는 인터페이스

Reactor 내 Context : Operator 같은 Reactor 구성 요소간에 전파되는 K/V 형태의 저장소 
- 전파 : 다운스트림에서 업스트림으로 Context가 전파되어 Operator 체인상의 각 Operator가 해당 context의 정보를 동일하게 이용할 수 있음
- 이 때 Context 는 ThreadLocal과 유사하지만, 각각의 실행스레드와 매핑되는 ThreadLocal 과 달리 실행 스레드와 매핑되는 것이 아닌, **Subscriber와 매핑**된다.

**즉, 구독이 발생할 떄마다 해당 구독과 연결된 하나의 Context가 생긴다**

> ThreadLocal
ThreadLocal은 이름에서도 알 수 있듯이 한 Thread의 scope내의 local variable이다. 
하나의 Thread 안에서는 같은 값을 공유해서 사용한다.


```java
@Slf4j
public class Example11_1 {
    public static void main(String[] args) throws InterruptedException {
        Mono
            .deferContextual(ctx ->
                Mono
                    .just("Hello" + " " + ctx.get("firstName"))
                    .doOnNext(data -> log.info("# just doOnNext : {}", data))
            )
            .subscribeOn(Schedulers.boundedElastic())
            .publishOn(Schedulers.parallel())
            .transformDeferredContextual(
                    (mono, ctx) -> mono.map(data -> data + " " + ctx.get("lastName"))
            )
            .contextWrite(context -> context.put("lastName", "Jobs"))
            .contextWrite(context -> context.put("firstName", "Steve"))
            .subscribe(data -> log.info("# onNext: {}", data));

        Thread.sleep(100L);
    }
}
```
- Context 에 K/V 형태의 데이터를 저장할 수 있다는 의미 = Context의 데이터를 읽고 쓰기가 가능하다는 의미 


### `contextWriter()`를 통해 Context에 데이터 쓰기 

해당 Operator의 파라미터로 람다 표현식을 전달 
```java
.contextWrite(context -> context.put("lastName", "Jobs"))
.contextWrite(context -> context.put("firstName", "Steve"))
```

contextWriter 내부 구조 (Mono.java) 
```java
...
public final Mono<T> contextWrite(Function<Context, Context> contextModifier) {
    return onAssembly(new MonoContextWrite<>(this, contextModifier));
}
...
```
- contextWrite() Operator의 파라미터를 Function 타입의 함수형 인터페이스인데, 람다 표현식으로 표현할 경우 람다 파라미터 타입이 Context이고, 리턴 값도 Context이다.
- 11-1 코드에서  구독이 발생하면 Context에 Steve, Jobs 두 데이터가 저장이 된다.

### Context에 쓰인 데이터 읽기 
방법은 두가지 
1. 원본 데이터 소스레벨에서 읽는 방식
2. Operator 체인 중간에서 읽는 방식


### 원본 데이터 소스 레벨에서 읽는 방식 
```java
.deferContextual(ctx ->
    Mono
        .just("Hello" + " " + ctx.get("firstName"))
        .doOnNext(data -> log.info("# just doOnNext : {}", data))
)
```
`deferContextual()` 사용
  - `defer()` Operator 와 같은 원리로 동작
  - Context에 저장딘 데이터와 원본 데이터 소스의 처리를 지연시키는 역할을 수행 (자세한 건 14장에서)

deferContextual() 내부 
- Mono.java 
```java
...
public static <T> Mono<T> deferContextual(Function<ContextView, ? extends Mono<? extends T>> contextualMonoFactory) {
    return onAssembly(new MonoDeferContextual<>(contextualMonoFactory));
}
...
```
- deferContextual() 의 파라미터로 정의된 람다 표현식의 람다 파라미터(ctx) 는 Context 타입 객체가 아닌, **ContextView** 타입 객체라는 것이 중요
  - 데이터를 쓸 때는 Context 타입 객체를 사용하며, 읽을 때는 ContextView 타입 객체를 사용한다.

### Operator 체인 중간에 읽는 방식 
```java
.transformDeferredContextual(
        (mono, ctx) -> mono.map(data -> data + " " + ctx.get("lastName"))
)
```
- transfromDeferredContextual() 을 통해 Operator 중간에서 Context 데이터를 읽음 
- ContextView 의 get() 을 통해 Context 에 저장된 lastName 을 읽어온다.

11-1 전체 실행 결과 
```
09:32:30.009 [boundedElastic-1] INFO - # just doOnNext : Hello Steve
09:32:30.017 [parallel-1] INFO - # onNext: Hello Steve Jobs
```
- 정상적으로 두번의 데이터를 읽어오며
- subscribeOn() 과 publishOn() 을 통해 emit 스레드와 데이터 처리하는 스레드를 분리하였으므로 데이터를 읽어오는 스레드가 각각 다르다.
- **이처럼 Reactor 에서 Operator 체인상 서로 다른 스레드가 Context에 저장된 데이터를 손쉽게 접근 가능**
- context.put() 은 데이터를 쓴 후 매번 불변 객체를 다음 contextWrite() 오퍼레이터에 전달하여 스레드 안전성을 보장한다.

## 11.2 Context 관련 API 
![image](https://github.com/xonmin/D3C/assets/27190617/46954b7c-9153-433d-bde9-9cb4bad6b3e0)

예제코드 
```java 
@Slf4j
public class Example11_3 {
    public static void main(String[] args) throws InterruptedException {
        final String key1 = "company";
        final String key2 = "firstName";
        final String key3 = "lastName";

        Mono
            .deferContextual(ctx ->
                    Mono.just(ctx.get(key1) + ", " + ctx.get(key2) + " " + ctx.get(key3))
            )
            .publishOn(Schedulers.parallel())
            .contextWrite(context ->
                    context.putAll(Context.of(key2, "Steve", key3, "Jobs").readOnly()) 
            )
            .contextWrite(context -> context.put(key1, "Apple"))
            .subscribe(data -> log.info("# onNext: {}" , data));

        Thread.sleep(100L);
    }
}
```
-  putAll()의 파라미터는 contextView 객체여야 하는데, Context.of()의 리턴 값은 새로운 Context 객체다. 
- 따라서 이 Context 객체를 ContextView 객체로 변환해 주어야 하는데, 이 작업은 readOnly() API를 통해 이루어진다. readOnly()는 Context를 읽기 작업만 가능한 ContextView로 변환해주는 API다.

실행 결과 
```
09:50:28.113 [parallel-1] INFO - # onNext: Apple, Steve Jobs
```

### ContextView 관련 API 
![image](https://github.com/xonmin/D3C/assets/27190617/910f71c4-67f7-4e8f-9eb1-851288b9a6b5)

```java
@Slf4j
public class Example11_4 {
    public static void main(String[] args) throws InterruptedException {
        final String key1 = "company";
        final String key2 = "firstName";
        final String key3 = "lastName";

        Mono
            .deferContextual(ctx ->
                    Mono.just(ctx.get(key1) + ", " +
                            ctx.getOrEmpty(key2).orElse("no firstName") + " " +
                            ctx.getOrDefault(key3, "no lastName"))
            )
            .publishOn(Schedulers.parallel())
            .contextWrite(context -> context.put(key1, "Apple"))
            .subscribe(data -> log.info("# onNext: {}" , data));

        Thread.sleep(100L);
    }
}
```
```
10:40:25.574 [parallel-1] INFO - # onNext: Apple, no firstName no lastName
```
- 3개의 키에 해당하는 데이터를 읽는데, 'company' 키에 해당하는 데이터는 Context에 저장되었지만, 그 외는 모두 Context에 저장되지 않았다. 따라서 값이 있는 경우를 제외하고 모두 디폴트 값을 전달받는다.

## 11.3 Context 의 특징 
Context는 구독이 발생할 때마다 하나의 Context가 해당 구독에 연결된다.
```java
@Slf4j
public class Example11_5 {
    public static void main(String[] args) throws InterruptedException {
        final String key1 = "company";

        Mono<String> mono = Mono.deferContextual(ctx ->
                        Mono.just("Company: " + " " + ctx.get(key1))
                )
                .publishOn(Schedulers.parallel());


        mono.contextWrite(context -> context.put(key1, "Apple"))
                .subscribe(data -> log.info("# subscribe1 onNext: {}", data));

        mono.contextWrite(context -> context.put(key1, "Microsoft"))
                .subscribe(data -> log.info("# subscribe2 onNext: {}", data));

        Thread.sleep(100L);
    }
}

```
- 첫 번째 구독이 발생하면서 Context에 'Apple'이란 회사명 저장
- 두 번째 구독 발생 -> 'microsoft' 저장
- 위 두개의 데이터가 하나의 Context에 저장될 것 같지만, Context는 구독별로 연결되는 특징이 있기 때문에 구독이 발생할 때마다 해당하는 하나의 Context가 하나의 구독에 연결

 ```
10:44:17.536 [parallel-2] INFO - # subscribe2 onNext: Company:  Microsoft
10:44:17.536 [parallel-1] INFO - # subscribe1 onNext: Company:  Apple
```

### Context는 Operator 체인의 아래에서 위로 전파된다. 
- 만약 동일한 키에 대해 값을 중복 저장하면 가장 위쪽에 위치한 contextWrite() 가 저장한 값으로 덮어씌여진다.

```java
@Slf4j
public class Example11_6 {
    public static void main(String[] args) throws InterruptedException {
        String key1 = "company";
        String key2 = "name";

        Mono
            .deferContextual(ctx ->
                Mono.just(ctx.get(key1))
            )
            .publishOn(Schedulers.parallel())
            .contextWrite(context -> context.put(key2, "Bill")) // Bill 이란 이름 저장 
            .transformDeferredContextual((mono, ctx) ->
                    mono.map(data -> data + ", " + ctx.getOrDefault(key2, "Steve")) // ContextView 객체를 통해 key2에 해당하는 값을 DownStream으로 emit 
            )
            .contextWrite(context -> context.put(key1, "Apple")) // 1) Apple 이란 회사명 저장 
            .subscribe(data -> log.info("# onNext: {}", data));

        Thread.sleep(100L);
    }
}
```

- context에는 분명 Bill 을 저장했으나, Steve 가 출력되는데 이 이유는 Operator 체인 상 아래에서 위로 전파되는 특징이 있어 그렇다.

코드의 흐름

1) Applte 이란 회사 명 저장
```java
.contextWrite(context -> context.put(key1, "Apple"))
```

2) 그 다음은 아래 코드 실행되지만, 해당 시점에는 `name`(key2) 키에 대한 해당하는 컨텍스트가 없음
```java
.transformDeferredContextual((mono, ctx) ->
        mono.map(data -> data + ", " + ctx.getOrDefault(key2, "Steve"))
)
```

  - 이 때 getOrDefault 말고 get() 사용 -> `NoSuchElementException` 발생
  - 모든 Operator 에서 Context에 저장된 데이터를 읽을 수 있도록 contextWrite() 은 Operator 맨 마지막에 위치 시켜야 한다.

### Inner Squenece 내부에서는 외부 Context에 저장되는 데이터를 읽을 수 있다.
- Inner Squenece 내부에서는 외부 Context에 저장되는 데이터를 읽을 수 있다.
- Inner Sqeuence 외부에서는 Inner Sequence 내부 Context 에 저장된 데이터를 읽을 수 없다. 

```java
@Slf4j
public class Example11_7 {
    public static void main(String[] args) throws InterruptedException {
        String key1 = "company";
        Mono
            .just("Steve")
//            .transformDeferredContextual((stringMono, ctx) ->
//                    ctx.get("role"))
            .flatMap(name -> // flatMap Operator 내부에 존재하는 Operator 체인에서 값 사용 
                Mono.deferContextual(ctx ->
                    Mono    // 내부 Sequence 
                        .just(ctx.get(key1) + ", " + name) // 외부 Context 읽기 
                        .transformDeferredContextual((mono, innerCtx) -> 
                                mono.map(data -> data + ", " + innerCtx.get("role")) 
                        )
                        .contextWrite(context -> context.put("role", "CEO"))
                )
            )
            .publishOn(Schedulers.parallel())
            .contextWrite(context -> context.put(key1, "Apple")) // 체인 맨마지막에 context write 
            .subscribe(data -> log.info("# onNext: {}", data));

        Thread.sleep(100L);
    }
}

// 결과 
11:12:56.321 [parallel-1] INFO - # onNext: Apple, Steve, CEO
```

7~8번 주석 라인 해제시,
```
Caused by: java.util.NoSuchElementException: Context does not contain key: role
```
- Context에 'role' 이란 키가 없기 때문에 에러 발생
- 이를 통해 Inner Sequence 외부에서는 Inner Sequence 내부 Context 내 저장된 데이터를 읽을 수 없음

### Context 활용 예제
인증된 도서 관리자가 신규 도서를 등록하기 위해 도서 정보와 인증 토큰을 서버로 전송하는 예제

```java
@Slf4j
public class Example11_8 {
    public static final String HEADER_AUTH_TOKEN = "authToken";
    public static void main(String[] args) {
    // 1) 도서정보 Book 전송
    // Mono<Book> 객체를 postBook() 메서드의 파라미터로 전달한다.
    // Mono가 어떤 과정을 거치든 상관없이 가장 마지막에 리턴된 Mono를 구독하기 직전에 contextWrite()으로 데이터를 저장하기 때문에 Operator 체인의 위쪽으로 전파되고, Operator 체인 어느 위치에서든 Context에 접근할 수 있다.
    // 결론적으로, Context는 인증 정보 같은 직교성(독립성)을 가지는 정보를 전송하는데 적합하다.
        Mono<String> mono =
                postBook(Mono.just(
                        new Book("abcd-1111-3533-2809"
                                , "Reactor's Bible"
                                ,"Kevin"))
                )
                .contextWrite(Context.of(HEADER_AUTH_TOKEN, "eyJhbGciOi"));

        mono.subscribe(data -> log.info("# onNext: {}", data));

    }

    private static Mono<String> postBook(Mono<Book> book) {
        // 2) zip() Operator
        // Mono<Book> 객체와 인증 토큰 정보를 의미하는 Mono<String> 객체를 하나의 Mono로 합친다.
        // Context에 저장한 인증 토큰을 2개의 Mono를 합치는 과정에서 다시 Context로부터 읽어와서 사용한다.
        // 이때 합쳐진 Mono는 Mono<Tuple2>의 객체가 된다.
        return Mono
                .zip(book,
                        Mono
                            .deferContextual(ctx ->
                                    Mono.just(ctx.get(HEADER_AUTH_TOKEN)))
                )  // 도서 정보를 전송한다.
                .flatMap(tuple -> {
                    String response = "POST the book(" + tuple.getT1().getBookName() +
                            "," + tuple.getT1().getAuthor() + ") with token: " +
                            tuple.getT2();
                    return Mono.just(response); // HTTP POST 전송을 했다고 가정
                });
    }
}

@AllArgsConstructor
@Data
class Book {
    private String isbn;
    private String bookName;
    private String author;
}
```

