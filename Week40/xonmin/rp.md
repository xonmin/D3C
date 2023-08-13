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
- 
