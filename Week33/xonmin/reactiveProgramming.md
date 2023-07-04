
# 1. 리액티브 시스템과 리액티브 프로그래밍 

### 1.1 리액티브 시스템이란?  
- 반응을 잘하는 시스템 = 클라이언트의 요청에 대해 즉각적으로 응답하여 지연 시간을 최소화 한다. 

### 1.2 리액티브 선언문으로 리액티브 시스템 이해하기 
![image](https://github.com/xonmin/D3C/assets/27190617/10634e17-eb7a-4ec5-8bbb-668d34b99e6f)

- MEANS : 비동기 메시지 기반의 통신을 통해 구성요소 간의 loose coupling, 격리성, 위치 투명성 보장 
- FORM : 비동기 메시지 통신 기반하에 탄력성과 회복성을 가지는 시스템이여야 한다. 
  - 탄력성 : 시스템의 유입되는 입력량에 의존하지 않고 시스템에서 요구하는 응답성을 일정하게 유지한다. 
     - 일정한 응답성 유지를 위해, 시기 적절하게 자원 추가/감소 -> 작업량 변화에 대응 
  - 회복성 : 장애가 발생하더라도 응답성 유지 
     - 비동기 메시지 기반 통신을 통한 느슨한 결합과 격리성 보장 -> 회복성 확보 
     - 장애 발생시, 장애가 발생한 부분만 복구할 수 있도록 유지 
- Value : 회복성과 예측 가능한 규모 확장성을 통해 즉각적인 응답 가능 시스템의 가치 제공 

### 1.3 리액티브 프로그래밍이란? 
리액티브 시스템을 구축하는 데 필요한 프로그래밍 모델 
- 리액티브 시스템은 Non-Blocking I/O 

### 1.4 리액티브 프로그래밍의 특징 
- 선언형 프로그래밍 : 목표하고자 하는 동작만 선언 
- 데이터 스트림 & 변화의 전파(propagation of change) : 지속적인 데이터 발생과 데이터 발생으로 인한 이벤트 발생 및 전달 

### 1.5 명령형 프로그래밍 vs 선언형 프로그래밍 
- 명령형 프로그래밍 : 어떤 작업을 처리하기 위해 실행할 동작을 코드에 구체적 명시 
- 선언형 프로그래밍 : 동작을 구체적으로 명시하지 않고 목표만 선언 
  - 여러 동작을 별도의 코드로 분리하지 않고, 메서드 체인을 통해 한 문장으로 된 코드가 구성된다. (코드 간결성 & 가독성) 


### 1.6 리액티브 프로그래밍 코드 구성 
- Publisher : 데이터 제공자 역할 
- Subscriber : 데이터 전달 받는 주체 
- Data Source(Data Stream) : publisher 를 통해 들어오는 데이터 
- Operator : Publisher 와 Subscriber 사이의 가공 처리 


# 2. 리액티브 스트림즈(Reative Streams) 


### 2.1 리액티브 스트림즈란? 
데이터 스트림을 Non-Blocking 하면서 비동기적인 방식으로 처리하기 위한 리액티브 라이브러리의 표준 사양 
- ex_ RxJava, Reactor, Akka Streams, Java 9 Flow API 

### 2.2 리액티브 스트림즈 구성요소 

Publisher 와 Subscriber 의 상호작용 예시 
```
- subscribe() : 전달받을 데이터를 Subsciber 가 데이터 구독 
- onSubscribe() : Publisher 가 데이터를 발행할 준비가 되어있음을 알림 
- Subscription.request() : S가 전달받길 원하는 데이터 개수를 P에게 요청 
- onNext() : P가 S로부터 요청받은 만큼의 데이터를 통지 
- onComplete() / onError() : P가 모든 데이터 발행 완료/에러발생을 S에게 알림 
```
이 때 `Subscription.request()`로 개수를 지정하는 이유 
- 실제 P와 S는 각기 다른 스레드에서 비동기적으로 상호작용하는 경우가 대부분 
- P가 발행하는 속도 > S가 처리하는 속도 : 데이터가 쌓이고 부하 발생 

### 2.3 코드로 보는 리액티브 스트림즈 컴포넌트 
Publisher 

```java 
public interface Publisher<T> {
  public void subscribe(Subscriber<? super T> s);
}
```
Subsciber 가 아닌 Publisher 에 subscribe 메서드가 정의된 이유 
- 리액티브 스트림즈 내에서의 P/S 구조는 일반 메시지 기반 시스템의 P/S 와 다르다. 


Subscriber

```java
public interface Subscriber<T> {
  // 구독 시작 시점에 어떤 처리를 하는 역할 (개수 지정, 구독 해지 ...) 
  public void onSubscribe(Subscription s);
  // P가 발행한 데이터 처리 
  public void onNext(T t);
  // 데이터 발행을 위한 처리 과정중 에러 발생시 에러 처리 
  public void onError(Throwable t); 
  // P가 데이터 발행 완료를 알릴 때 호출되는 메서드 (완료 이후 후처리 코드 작성) 
  public void onComplete();
}
```

Subscription 

```java 
public interface Subscription {
  // n 개의 데이터 요청 
  public void request(long n);
  // 데이터 요청 취소(구독 해지) 
  public void cancel();
}
``` 

해당 코드들을 통해 앞선 상호작용 설명 
1. P가 S 인터페이스 구현체를 subscribe() 메서드 파라미터로 전달
2. P 내부에서 전달받은 S 인터페이스 구현체의 `onSubscribe()` 호출로 구독을 의미하는 Subscription 구현체를 S에게 전달 
3. 호출된 S 구현체의 onSubscribe() 메서드에서 전달받은 Subscription 객체를 통해 전달받을 데이터의 개수를 P에게 요청
4. P는 S에게 받은 요청 개수 만큼 S의 onNext() 만큼 호출하여 S에게 전달 
5. P가 전달할 데이터가 없는 경우 S의 onComplete() 호출 

Processer 

```java 
public interface Processer<T, R> extends Subscriber<T>, Publisher<R> {
}
```
- 다른 인터페이스와 다른 점 : Subscriber 인터페이스와 Publisher 인터페이스 상속 
- 리액티브 스트림즈 컴포넌트에서 Process 는 두가지 모두의 기능을 가져야하므로 
