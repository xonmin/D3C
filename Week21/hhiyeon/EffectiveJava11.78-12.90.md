### 📌 11장 : 동시성
#### 아이템 78. 공유 중인 가변 데이터는 동기화해 사용하라
- synchronized 
  - 해당 메서드나 블록을 한 번에 한 스레드씩 수행하도록 보장
  - 일반적인 동기화 
    - 한 객체가 일관된 상태를 가지고 생성, 객체에 접근하는 메서드는 그 객체에 lock 을 건다.
    - lock 을 건 메서드는 객체의 상태를 확인하고 필요하면 수정한다.
  - 동기화의 중요한 기능
    - 일관성이 깨진 상태를 볼 수 없게 해주고
    - 동기화된 메서드나 블록에 들어간 스레드가 같은 락의 보호하에 수행된 모든 이전 수정의 최종 결과를 보게 해준다.

- 동기화는 배타적 실행뿐만 아니라, 스레드 사이의 안정적인 통신에 필요하다.
- 다른 스레드를 멈추는 올바른 방법 
  - 첫 번째 스레드는 자신의 boolean 필드를 폴링하면서 값이 true가 되면 멈춘다.
  - 필드를 false로 초기화하고, 다른 스레드에서 이 스레드를 멈추려고 할 때 true 로 변경
- 적절한 스레드 종료 방법


```java
public class StopThread {
    private static boolean stopRequested;
    
    private static synchronized void requeststop() { 
        stopRequested = true;
    }
    
    private static synchronized boolean stopRequested() { 
        return stopRequested;
    }
    
    public static void main(String[] args) 
        throws InterruptedException {
            Thread backgroundThread = new Thread(() -> { 
                int i = 0;
                while (!stopRequested()) 
                    i++;
            }); 
            backgroundThread.start();
            
            TimeUnit.SECONDS.sleep(1);
            requestStop();
    } 
}
```


- 쓰기 메서드(requestStop), 읽기 메서드(stopRequested) 모두 동기화 해야 한다.
- 둘 다 동기화되지 않으면, 동작을 보장하지 않는다.


- volatile
  - 배타적 수행과는 상관없이 항상 가장 최근에 기록된 값을 읽게 해준다.
  - 반복문에서 동기화 하는 비용을 더 빠르게 하는 방법

```java
private static volatile boolean stopRequested;
```


- 주의해야 하는 상황
  - 증가 연산자는 코드상으로 하나지만, 실제로 nextSerialNumber 필드에 두 번 접근한다.
  - 먼저 값을 읽고, 증가한 새로운 값을 저장한다.
  - 두 번째 스레드가 이 두 접근 사이를 들어와 값을 읽게 되면 첫 번째 스레드와 같은 값을 받게 된다.
  - 안전 실패(safety failure) : 프로그램이 잘못된 결과를 계산하는 오류
  - 해결 방법 : generateSerialNumber 메서드에 synchronized 한정자를 붙인다.
    - 동시 호출을 해도 이전 호출이 변경한 값을 읽게 해준다.
    - synchronized을 사용하면 nextSeriaINumber 필드에서 volatile 를 제거해주어야 한다.


```java
private static volatile int nextSeriaINumber = 0;

public static int generateSerialNumber() { 
        return nextSerialNumber++;
}
```


- 위의 문제들을 피하는 가장 좋은 방법은 처음부터 가변 데이터를 공유하지 않도록 한다.
- 가변 데이터는 단일 스레드에서만 사용하도록 한다.
---
#### 아이템 79. 과도한 동기화는 피하라
- 과도한 동기화는 성능이 저하되고, 교착상태에 빠질 수 있다.
- 응답 불가와 안전 실패를 피하기 위해서 동기호 메서드나 동기화 블록 안에서 제어를 클라이언트에 양도하지 않기
  - 동기화된 영역 안에 재정의할 수 있는 메서드를 호출하면 안된다.
  - 클라이언트가 넘겨준 함수 객체를 호출하면 안된다.
- 외계인 메서드 호출을 동기화 블록 바깥으로 옮겨준다.


```java
private void notifyElementAdded(E element) { 
    List<SetObserver<E>> snapshot = null; 
    synchronized(observers) {
      snapshot = new ArrayList<>(observers);
    }
    for (SetObserver<E> observer : snapshot)
        observer.added(this, element);
}
```


- 열린 호출(open call)
  - 동기화 영역 바깥에서 호출되는 외계인 메서드
  - 외계인 메서드는 언제까지 실행될지 알 수 없는데, 동기화 영역 안에서 호출되면 다른 스레드는 사용 대기를 해야 한다.
  - 열린 호출은 실패 방지 효과 이외에 동시성 효율을 개선해준다.


- 더 좋은 방법은 자바의 동시성 컬렉션 라이브러리 CopyOnWriteArrayList가 있다.
  - ArrayList를 구현한 클래스
  - 내부의 배열이 수정되지 않아 순회할 때 락이 필요 없어 속도가 매우 빠르다.
  - 순회만 일어나는 관찰자 리스트 용도로 좋다.


```java
private final List<SetObserver<E>> observers = 
        new CopyOnWriteArrayList<>();
public void addObserver(SetObserver<E> observer) { 
    observers.add(observer);
}
public boolean removeObserver(SetObserver<E> observer) { 
    return observers.remove(observer);
}
private void notifyElementAdded(E element) { 
    for (SetObserver<E> observer : observers)
      observer.added(this, element);
}
```


- 기본 규칙
  - 동기화 영역에서 가능한 일을 적게 하기


- 멀티코어가 일반화 된 오늘날에, 과도한 동기화는 락을 얻는 데 드는 CPU 비용이 아니다.
- 경쟁하는 낭비 시간, 모든 코어가 메모리를 일관되게 보기 위한 지연 시간
---
#### 아이템 80. 스레드보다는 실행자, 태스크, 스트림을 애용하라
- java.util.concurrent
  - 실행자 프레임워크라고 하는 인터페이스 기반의 유연한 태스크 실행 기능을 담고 있다.
  - 간단한 작업 큐 사용 가능


```java
ExecutorService exec = Executors.newSingleThreadExecutor();
exec.execute(runnable); // 실행자에 실행할 태스크를 넘기는 법
exec.shutdown(); // 실행자를 종료시키는 법
```


- 큐를 2개 이상의 스레드가 처리하게 하려면, 다른 정적 팩토리로 다른 종류의 실행자 서비스(스레드 풀) 생성하기
- 스레드 풀 스레드 개수는 고정할 수 있고, 늘어나거나 줄어들게 할 수도 있다.
- 주로 사용하는 실행자는 `java.util.concurrent.Executors` 에서 사용
- `ThreadPoolExecutor` 클래스를 직접 사용해도 된다.


- 실행자 프레임워크에서는 작업 단위와 실행 메커니즘이 분리된다.
- 태스크 : 작업 단위를 나타내는 개념
- 태스크 종류 : Runnable, Callable
  - Callable은 Runnable과 비슷하지만 값을 반환하고 임의의 예외를 던질 수 있다.


- 실행자 서비스 : 태스크를 수행하는 일반적인 메커니즘
- 자바 7부터 실행자 프레임워크는 포크-조인(fork-join) 태스크를 지원한다.
- fork-join 태스크는 fork-join 풀이라는 특별한 실행자 서비스를 실행한다.
  - fork-join 태스크의 인스턴스는 작은 하위 태스크로 나뉠 수 있다.
  - fork-join 풀을 구성하는 스레드들이 태스크들을 처리한다.
  - 일을 먼저 끝낸 스레드는 다른 스레드의 남은 태스크를 가져와서 대신 처리할 수도 있다.
  - 모든 스레드가 움직여 CPU를 최대한 활용한다.
  - 높은 처리량, 낮은 지연시간
---
#### 아이템 81. wait와 notify보다는 동시성 유틸리티를 애용하라
- 자바 5부터 고수준 동시성 유틸리티가 wait, notify 대신 일을 처리해준다.
- java.util.concurrent 의 고수준 유틸리티
  - 실행자 프레임워크
  - 동시성 컬렉션(concurrent collection)
  - 동기화 장치(synchronizer)


- 동시성 컬렉션
  - List, Queue, Map과 같은 표준 컬렉션 인터페이스에 동시성을 추가한 고성능 컬렉션
  - 동기화를 각자 내부에서 수행해 높은 동시성을 갖춤
  - 동시성 컬렉션에서 동시성을 무력화할 수 없다. 외부에서 락을 추가로 사용하면 속도가 느려진다.

- ConcurrentMap 으로 구현한 동시성 정규화 맵



```java
public static String intern(String s) { 
    String result = map.get(s);
    if (result = null) {
      result = map.putIfAbsent(s, s); 
      if (result = null)
        result = s;
    }
    return result; 
}
```


- Collections.synchronizedMap 보다 ConcurrentHashMap 의 성능이 더 좋다.
  - 동기화된 맵 -> 동시성 맵


- wait, notify 보다 동시성 유틸리티를 사용하는 것이 더 좋지만
- 레거시 코드를 유지보수해야 하는 경우, wait은 while문 안에서 호출해야 한다.
- 일반적인 상황에서 notify보다 notifyAll을 사용해야 한다.
  - 응답 불가 상태에 빠질 수 있기 때문에
---
#### 아이템 82. 스레드 안전성 수준을 문서화하라
- 메서드 선언에서 synchronized 은 API에 속하지 않는다.
- 멀티 스레드 환경에서 API를 안전하게 사용하기 위해서 스레드 안전성 수준을 정확히 명시해야 한다.
- Collections.synchronizedMap API 문서


```java
// synchronizedMap이 반환한 맵의 컬렉션 뷰를순회하려면 반드시 그 맵을 락으로 사용해 수동으로 동기화하라.

Map<K, V> m = Collections. synchronizedMap(new HashMapo()); 
Set<K> s = m.keySet(); // 동기화 블록 밖에 있어도 된다.
        ...
synchronized(m) { // s가 아닌 m을 사용해 동기화해야 한다! 
  for (K key : s)
    key.f();
}
// 이대로 따르지 않으면 동작을 예측할 수 없다.
```
---
#### 아이템 83. 지연 초기화는 신중히 사용하라
- 지연 초기화(lazy initialization)
  - 필드의 초기화 시점을 그 값이 처음 필요할 때까지 늦추는 기법
  - 정적 필드, 인스턴스 필드에서 사용
  - 주로 최적화에 사용, 클래스와 인스턴스 초기화에 발생하는 위험 순환 문제 해결
  - 클래스, 인스턴스 생성의 초기화 비용은 줄지만, 필드 접근 비용이 커진다.
  - 지연 초기화가 성능을 느리게 할 수도 있다.
- 멀티 스레드 환경에서는 지연 초기화 필드를 2개 이상의 스레드가 공유하면 반드시 동기화 해야 하기 때문에 구현이 어렵다.
- 대부분의 상황에서 일반적인 초기화가 지연 초기보다 낫다.
- 인스턴스 필드 초기화하는 일반적인 방법

```java
private final FieldType field = computeFieldValue();
// 지연 초기화가 초기화 순환성(initialization circularity)을 깨뜨릴 것 같을 때 synchronized 접근자 사용하기

private FieldType field;
private synchronized FieldType getFieId() { 
    if (field = null)
        field = computeFieldValue(); 
    return field;
}
```


- 정적 필드를 지연 초기화해야 할 때, 지연 초기화 홀더 클래스(lazy initialization holder class) 관용구 사용


```java
private static class FieldHolder {
static final FieldType field = compiiteFieldValue();
}
private static FieldType getFieId() { return FieldHolder.field; }
```



- 인스턴스 필드를 지연 초기화해야 할 때, 이중검사(double-check) 관용구 사용
```java
private volatile FieldType field;

private FieldType getFieId() {
        FieldType result = field;
        if (result != null) { // 첫 번째 검사 (락 사용 안 함)
            return result;
            
        synchronized(this) {
          if (field = null) // 두 번째 검사 (락 사용)
            field = computeFieldValue(); 
          return field;
        }
}
```


---
#### 아이템 84. 프로그램의 동작을 스레드 스케줄러에 기대지 말라
- 여러 스레드가 실행 중일 때, 운영체제의 스레드 스케줄러는 스레드 실행 순서와 시간을 정한다.
- 구체적인 스케줄링 정책은 운영체제마다 다르다.
- 정확성이나 성능이 스레드 스케줄러에 따라 달라지는 프로그램은 다른 플랫폼에 이식하기 어렵다.
- 견고하고 빠르고 이식성 좋은 프로그램 작성 방법
  - 실행 가능 스레드의 평균 개수를 프로세서 수보다 크게 많아지지 않도록 해야한다.


- 실행 가능 스레드 수를 적게 유지하는 주요 기법
  - 각 스레드가 유용한 작업 후, 다음 일이 생기기 전까지 대기하기
  - 당장 처리할 작업이 없으면 실행 X
  - ex. 실행자 프레임워크 : 스레드 풀 크기를 적절히 설정하고 작업은 짧게 유지한다.
---
### 📌 12장 : 직렬화
#### 아이템 85. 자바 직렬화의 대안을 찾으라
- 1997년, 자바에 직렬화 도입
- 직렬화의 근본적인 문제
  - 공격 범위가 넓고, 지속적으로 넓어져 방어가 어렵다.
  - ObjectInputStream의 readObject 메서드를 호출하면 객체 그래프가 역직렬화 되기 떄문


- 가젯(gadget)
  - 역직렬화 과정에서 호출되어 잠재적으로 위험한 동작을 수행하는 메서드
- 역직렬화 폭탄(deserialization bomb)
  - 역직렬화에 시간이 오래 걸리는 짧은 스트림을 역직렬화하다 서비스 거부 공격에 쉽게 노출될 수 있다.


- 직렬화 문제 회피 방법
  - 아무것도 역직렬화하지 않는다.
  - 객체와 바이트 시퀀스를 변환해주는 다른 메커니즘을 사용한다.
  - 크로스-플랫폼 구조화된 데이터 표현(cross-platform structured-data representation)


- 크로스-플랫폼 구조화된 데이터 표현
  - 자바 직렬화보다 간단하다.
  - 임의 객체 그래프를 자동으로 직렬화/역직렬화 하지 않는다.
  - 속성-값 쌍의 집합으로 구성된 간단하고 구조화된 데이터 객체를 사용한다.
  - ex. JSON, 프로토콜 버퍼(protocol buffers)
    - JSON : 텍스트 기반이라 사람이 읽을 수 있다.
    - 프로토콜 버퍼 : 이진 표현이라 효율이 좋다.
    - JSON은 데이터를 표현하는 데 사용, 프로토콜 버퍼는 문서를 위한 스키마(타입)을 제공한다.
    - 프로토콜 버퍼는 사람이 읽을 수 있는 텍스트 표현(pbtxt)도 지원한다.
---
#### 아이템 86. Serializable 을 구현할지는 신중히 결정하라
- 클래스의 인스턴스를 직렬화 하기 위해서는 클래스 선언에 implements Serializable 붙이기
- 선언은 쉬워보이지만 Serializable 을 구현하면 릴리즈한 뒤에 수정이 어렵다.
  - Serializable 구현 후, 직렬화된 바이트 스트림 인코딩은 공개 API가 된다.
- Serializable 구현은 버그와 보안 구멍이 생길 위험이 높아진다.
  - 객체는 생성자를 사용해 만드는데, 직렬화는 기본 메커니즘을 우회하는 객체 생성 기법
  - 역직렬화를 사용하면 불변식 깨짐과 허가되지 않은 접근에 노출된다.
- Serializable 구현은 신버전 릴리즈할 때 테스트할 것이 늘어난다.
  - 직렬화 수정 후, 구버전으로 역직렬화 할 수 있는지 반대도 가능한지 확인해야 한다.


- 상속용으로 설계된 클래스는 대부분 Serializable을 구현하면 안된다. 인스턴스도 대부분 안된다.
- 내부 클래스는 직렬화를 구현하면 안된다.
---
#### 아이템 87. 커스텀 직렬화 형태를 고려해보라
- 수정이 어렵기 떄문에 고민한 후에 괜찮다고 판단될 때만 기본 직렬화 형태 사용하기
- 객체의 물리적 표현과 논리적 내용이 같으면 기본 직렬화 형태도 무방하다.
- 차이가 클 때 사용시 문제점
  - 공개 API가 현재의 내부 표현 방식에 영구히 묶인다.
  - 너무 많은 공간을 차지할 수 있다.
  - 시간이 너무 많이 걸릴 수 있다.
  - 스택 오버플로를 일으킬 수 있다.
---
#### 아이템 88. readObject 메서드는 방어적으로 작성하라
- readObject는 어떤 바이트 스트림이 넘어와도 유효한 인스턴스를 만들어야 한다.
- 안전한 readObject 메서드를 작성하는 방법
  - private 이어야 하는 객체 참조 필드는 각 필드가 가리키는 객체를 방어적으로 복사해야 한다.
  - 모든 불변식을 검사해서 어긋나는 게 발견되면, InvalidObjectException 을 던진다.
  - 역직렬화 후 객체 그래프 전체의 유효성을 검사해아 하면, ObjectInputValidation 인터페이스를 사용해라
  - 재정의할 수 있는 메서드를 호출하지 않는다.
---
#### 아이템 89. 인스턴스 수를 통제해야 한다면 readResolve 보다는 열거 타입을 사용하라
- readResolve 를 인스턴스 통제 목적으로 사용하려면 객체 참조 타입 인스턴스 필드는 모두 transient 으로 선언해야 한다.
  - 그렇게 하지 않으면, readResolve 메서드가 수행되기 전에 역직렬화된 객체의 참조를 공격할 수도 있다.

- 직렬화 가능한 인스턴스 통제 클래스를 열거 타입을 이용해 구현하면 선언한 상수 외의 다른 객체는 존재하지 않는 것을 보장해준다.
- 열거 타입 싱글턴 예시


```java
public enum Elvis { 
    INSTANCE;
    private String[] favoriteSongs =
        { "Hound Dog", "Heartbreak Hotel" };
    public void printFavorites() {
        System. out.println(Arrays.toString (favoriteSongs));
    } 
}
```
---
#### 아이템 90. 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라
- Serializable 으로 구현하면, 생성자 이외의 방법으로 인스턴스를 생성
  - 버그와 보안 문제가 일어날 수 있다.
  - 직렬화 프록시 패턴(serialization proxy pattern) 으로 해결


- 직렬화 프록시 패턴
  - 바깥 클래스의 논리적 상태를 정밀하게 표현하는 중첩 클래스를 설계해서 private static 으로 선언한다.
  - 중첩 클래스의 생성자는 1개
  - 바깥 클래스를 매개변수로 받아야 한다.
  - 생성자는 단순히 인수로 넘어온 인스턴스 데이터를 복사
  - 일관성 검사, 방어적 복사도 필요 없다.
  - 바깥 클래스, 직렬화 프록시 모두 Serializable 를 구현한다고 선언해야 한다.


- 직렬화한 Period 클래스 예시



```java
private static class SerializationProxy implements Serializable { 
    private final Date start;
    private final Date end;
    
    Serializationproxy(Period p) { 
        this.start = p.start; 
        this.end = p.end;
    }
    private static final long serialVersionUID = 
            234098243823485285L; // 아무 값이나 상관없다. (아이템 87)
}

// 바깥 클래스에 추가 : 직렬화 프록시 패턴용 writeReplace 메서드
// 직렬화가 이뤄지기 전에 바깥 클래스 인스턴스를 직렬화 프록시로 변환
private Object writeReplace() {
    return new SerializationProxy(this); 
}

// 직렬화 프록시 패턴용 readObject 메서드
private void readObject(ObjectlnputStream stream) 
        throws InvalidObjectException {
    throw new InvalidObjectException("프록시가 필요합니다.");
}

// Period.SerializationProxy용 readResolve 메서드
// 역직렬화 시에 직렬화 시스템이 직렬화 프록시를 다시 바깥 클래스의 인스턴스로 변환
private Object readResolve() {
  return new Period(start, end); // public 생성자를 사용한다.
}
```
---
