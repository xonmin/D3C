### 📌 10장 : 예외
#### 아이템 69. 예외는 진짜 예외 상황에만 사용하라
- 잘못된 예외상황 : 무한루프를 돌다가 배열에 끝에 도달해서 ArrayIndexOutOfBoundException 발생했을 때 종료


```java
try {
        int i = 0;
        while(true)
        range[i++].climb();
        } catch(ArrayIndexOutOfBoundsException e){
        }
```


- JVM은 배열에 접근할 때마다 경계가 넘는지 검사한다.
- 위의 코드에서 같은 일(경계가 넘는지 확인)이 반복된다.
- 예외를 사용 vs 표준 관용구 : 예외를 사용한 것이 속도가 더 느리다.
- 예외는 예외 상황에서만 사용하고, 일상적인 제어 흐름용에서는 쓰지 않는다.
---
#### 아이템 70. 복구할 수 있는 상황에는 검사 예외를, 프로그래밍 오류에는 런타임 예외를 사용하라
- 자바 문제 상황을 알리는 타입(throwable)
    - 검사 예외, 런타임 예외, 에러
    - 호출하는 쪽에서 복구해야 하는 상황이면 검사 예외를 사용
    - 검사 예외일 때 복구에 필요한 정보를 알려주는 메서드를 제공해야 한다.

- 비검사 throwable
    - 런타임 예외, 에러
    - 프로그래밍 오류를 나타낼 때 : 런타임 예외
    - JVM이 자원 부족, 불변식 깨짐 등 더 이상 수행을 할 수 없는 상황일 때 : 에러 사용
    - Error 클래스를 상속해서 하위 클래스를 만들지 않기
        - 비검사 throwable은 모두 RuntimeException 의 하위 클래스여야 한다.
---
#### 아이템 71. 필요 없는 검사 예외 사용은 피하라
- 검사 예외는 발생한 문제를 프로그래머가 처리해서 안전성을 높여준다.
- 과하게 사용하면 쓰기 불편한 API가 될 수 있다.
- 검사 예외를 회피하는 가장 쉬운 방법은 적절한 결과 타입을 담은 옵셔널을 반환하는 방법이다.
    - 검사 예외를 던지는 대신에 빈 옵셔널 반환하기
    - 단점 : 예외 발생 원인의 부가 정보를 담을 수 없다.
- 예외를 사용하면 구체적인 예외 타입, 그 타입이 제공하는 메서드를 활용해 부가 정보를 제공할 수 있다.
- 옵셔널만으로 상황을 처리해서 충분한 정보를 제공할 수 없을 때만 검사 예외를 사용한다.
---
#### 아이템 72. 표준 예외를 사용하라
- 표준 예외를 사용하면 사용이 쉽고, 예외 클래스가 적을수록 메모리 사용량이 줄고 클래스 적재 시간도 적게 걸린다.
- 가장 많이 사용하는 예외 : IllegalArgumentException
    - 호출자가 인수로 부적절한 값을 넘길 때 던지는 예외
    - 반복 횟수를 지정하는 매개변수에 음수를 건낼 때 사용할 수 있다.

- 자주 사용되는 예외들
  <img width="595" alt="스크린샷 2023-04-02 오후 10 27 57" src="https://user-images.githubusercontent.com/52193680/229355796-e6f46638-fbf3-4358-a907-fa319413e294.png">


- Exception, RuntimeException, Throwable, Error은 직접 재사용하지 말아야 한다.
- 이 예외들은 다른 예외들의 상위 클래스이기 때문에, 여러 성격의 예외들을 포괄해서 안정적으로 테스트할 수 없다.
---
#### 아이템 73. 추상화 수준에 맞는 예외를 던지라
- 예외 번역(exception translation)
    - 메서드가 저수준 예외를 처리하지 않았을 때, 관련 없는 예외가 나오는 문제가 생길 수 있다.
    - 문제를 해결하기 위해서, 상위 계층에서는 저수준 예외를 잡아 자신의 추상화 수준에 맞는 예외로 바꿔서 던져야 한다.


```java
try {
... // 저수준 추상화를 이용한다.
} catch (LowerLevelException e) {
    // 추상화 수준에 맞게 번역한다.
    throw new HigherLevelException(...);
}
```


- 예외 연쇄(exception chaining)
    - 예외를 번역할 때, 저수준 예외가 디버깅에 도움이 될 때 예외 연쇄를 사용하는 것이 좋다.
    - 문제의 근본 원인(cause)인 저수준 예외를 고수준 예외에 실어 보내는 방식
    - 별도의 접근자 메서드(Throwable의 getCause 메서드)를 통해 저수준 예외를 꺼내서 볼 수 있다.


```java
try {
... // 저수준 추상화를 이용한다.
} catch (LowerLevelException e) {
    // 저수준 예외를 고수준 예외에 실어 보낸다.
    throw new HigherLevelException(cause);
}
```


- 고수준 예외의 생성자는 상위 클래스의 생성자에 원인을 보내 최종적으로 Throwable 생성자까지 보낼 수 있다.


```java
class HigherLevelException extends Exception { 
    HigherLevelException(Throwable cause) {
        super(cause); 
    }
}
```
---
#### 아이템 74. 메서드가 던지는 모든 예외를 문서화하라
- 검사 예외는 항상 따로 선언하고, 각 예외가 발생하는 상황을 자바독 @throws 태그를 사용해서 문서화한다.
- 공통 상위 클래스 하나로 뭉뚱그려 선언하지 않기
    - ex. Exception, Throwable을 던진다고 선언하지 않기
    - 예외 상황 : main은 오직 JVM만 호출하기 떄문에 Exception을 던지도록 선언해도 된다.


- 메서드가 던질 수 있는 예외는 각각 @throws 태그로 문서화
- 비검사 예외는 메서드 선언의 throws 목록에 넣지 않기
---
#### 아이템 75. 예외의 상세 메시지에 실패 관련 정보를 담으라
- 예외를 잡지 못해 프로그램이 실패하면 자바 시스템은 그 예외의 스택 추적(stack trace) 정보를 자동으로 출력한다.
- 스택 추적 : 예외 객체의 toString 메서드를 호출해 얻는 문자열


```java
/**
* IndexOutOfBoundsException을 생성한다. *
* @param lowerBound 인덱스의 최솟값
* @param upperBound 인덱스의 최댓값 + 1 * @param index 인덱스의 실젯값
*/
public IndexOutOfBoundsException(int lowerBound, int upperBound, int index){
        // 실패를 포착하는 상세 메시지를 생성한다.
        super(String.format(
        "최솟값: %d, 최댓값: %d, 인덱스: %d",
        lowerBound, upperBound, index));

        // 프로그램에서 이용할 수 있도록 실패 정보를 저장해둔다.
        this.lowerBound=lowerBound;
        this.upperBound=upperBound;
        this.index=index;
}
```
---
#### 아이템 76. 가능한 한 실패 원자적으로 만드랄
- 실패 원자적(failure-atomic) 특성
    - 호출된 메서드가 실패해도, 해당 객체는 메서드 호출 전 상태를 유지해야 한다.

- 메서드를 실패 원자적으로 만드는 방법
    - 불변 객체로 설계 한다.
        - 불변 객체의 상태는 생성 시점에 고정되어 절대 변하지 않는다.
    - 가변 객체의 경우, 작업 수행에 앞서 매개변수 유효성을 검사한다.
        - 객체 내부 상태를 변경하기 전에 잠재적 예외 가능성을 걸러준다.
    - 객체의 임시 복사본에서 작업을 수행하고, 작업이 성공적으로 완료되면 원래 객체와 교체한다.
        - ex. 정렬 수행 전에 입력 리스트의 원소를 배열로 옮겨 담는다.
        - 배열을 사용하면 정렬 알고리즘의 반복문에서 원소들에 더 빠르게 접근 가능하고, 정렬이 실패해도 입력 리스트는 변하지 않는다.
    - 작업 도중 발생하는 실패를 가로채는 복구 코드를 작성해서 작업 전 상태로 되돌린다.
        - 주로 디스크 기반의 내구성(durability)을 보장해야 하는 자료구조에 쓰인다.
---
#### 아이템 77. 예외를 무시하지 말라
- 예외 상황을 무시하지 않도록 해야 한다.
- 예외를 무시해야 하는 경우도 있다.
    - ex. FileInputStream 을 닫을 때
- 예외를 무시하는 경우, catch 블록 안에 이유에 대해 주석으로 남기고, 예외 변수 이름을 ignored 으로 변경한다.


```java
// catch 블록을 비워두면 예외가 무시된다. 아주 의심스러운 코드다! 
try {
        ...
        } catch(SomeException e){
    
        }
```

```java
Future<Integer> f = exec.submit(planarMap::chromaticNumber);
int numColors = 4;
try {
    numColors = f.get(1L, TimeUnit.SECONDS);
} catch (TimeoutException | ExecutionException ignored) {
    // 기본값을 사용한다(색상 수를 최소화하면 좋지만, 필수는 아니다).
}
```
---
