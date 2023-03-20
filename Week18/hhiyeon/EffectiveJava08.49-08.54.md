### 📌 8장 : 메서드
#### 아이템 49. 매개변수가 유효한지 검사하라
- 매개변수 검사를 제대로 하지 않았을 경우
  - 메서드가 수행되는 중간에 모호한 예외를 던지며 실패할 수 있다.
  - 메서드가 잘 수행되지만 잘못된 결과를 반환할 수 있다.
  - 메서드는 수행되지만 어떤 객체를 이상한 상태로 만들어 추후에 메서드와는 관련 없는 오류가 발생할 때 
  - 위와 같이 매개변수 검사에 실패하면 실패 원자성(failure atomicity)을 어기게 된다.


- public과 protected 메서드는 매개변수 값이 잘못되는 경우 던지는 예외들을 문서화 해야 한다.
  - @throws 자바독 태그 사용
  - 예외 종류 : IllegalArgumentException, IndexOutOfBoundsException, NullPointerException
  - 제약을 어겼을 경우의 예외도 추가해야 한다.  


```java
public Biginteger mod(Biginteger m) { 
    if (m.signum() <= 0)
        throw new ArithmeticException("계수(m)는 양수여야 합니다. " + m);
    ... // 계산 수행
}
```


- private 메서드에서 assert를 사용해 매개변수 유효성 검증


```java
private static void sort(long a[], int offset, int length){
    assert a != null;
    assert offset >= 0 && offset <= a.length;
    assert length >= 0 && length <= a.length - offset;
    ... // 계산 수행
}
```


- assert(단언문) 특징
  - 실패하면 AssertionError를 던진다.
  - 런타임에 아무런 효과도, 아무런 성능 저하도 없다.
---
#### 아이템 50. 적시에 방어적 복사본을 만들라
- 자바는 네이티브 메서드를 사용하지 않아 C, C++ 같은 언어에서 흔히 볼 수 있는 버퍼 오버런, 배열 오버런, 와일드 포인터 같은 메모리 충돌 오류에서 안전하다.
- 자바가 다른 클래스의 침범을 모두 막을 수 있는 것은 아니기 때문에, 방어적으로 프로그래밍해야 한다.
- 어떤 객체든 그 객체의 허락 없이 외부에서 내부를 수정하는 일은 불가능하다.
- 예외 상황 : 불변식을 지키지 못한 경우


```java
public final class Period {
    private final Date start;
    private final Date end;
    
    public Period(Date start, Date end) { 
        if (start.compareTo(end) > 0)
            throw new IllegalArgumentException( 
                start + "가 " + end + "보다 늦다.");
            this.start = start;
            this.end = end; 
    }
    
    public Date start() { 
        return start;
    }
    
    public Date end() { 
        return end;
    }
    
    ...// 나머지 코드 생략 
}

/* Period 인스턴스 내부 공격 */
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end); 
end.setYear(78); // p의 내부를 수정할 수 있다.
```


- Date는 오래된 API이기 때문에 새로운 코드 작성에서는 사용을 하지 않도록 한다.
  - Date 대신 불변인 Instant를 사용하면 된다.
  - 또는 LocalDateTime, ZonedDateTime 사용
- 외부 공격으로부터 인스턴스 내부를 보호하기 위해서는 생성자에서 받은 가변 매개변수 각각을 방어적 복사(defensive copy)해서 인스턴스 내부에서 원본이 아닌 복사본을 사용한다.


```java
public Period(Date start, Date end) { 
    this.start = new Date(start.getTime()); 
    this.end = new Date(end.getTime());
    
    if (this.start.compareTo(this.end) > 0) 
        throw new IllegalArgumentException(
            this.start + "가 " + this.end + "보다 늦다.");
}
```


- 접근자 메서드가 내부의 가변 정보를 직접 드러내는 문제점
  - 가변 필드의 방어적 복사본을 반환한다.


```java
public Date start() {
    return new Date(start.getTime());
}

public Date end() {
    return new Date(end.getTime());
}
```
---
#### 아이템 51. 메서드 시그니처를 신중히 설계하라
- API 설계 요령 리스트
  - 메서드 이름을 신중하게 짓자.
  - 편의 메서드를 너무 많이 만들지 말자. 
  - 매개변수 목록은 짧게 유지하자. (4개 이하)
    - 매개변수 목록을 짧게 줄이는 방법 
      - 여러 메서드로 쪼갠다. 
      - 매개변수 여러 개를 묶어주는 도우미 클래스를 만든다.(정적 멤버 클래스)
      - 객체 생성에 사용한 빌더 패턴을 메서드 호출에 응용한다. 먼저 모든 매개변수를 하나로 추상화한 객체를 정의하고, 클라이언트에서 이 객체의 setter를 호출해 필요한 값을 설정한다.


- 매개변수 타입으로는 클래스보다 인터페이스가 더 낫다.
  - 클래스를 사용하면 클라이언트에게 특정 구현체만 사용하도록 제한하게 된다.
- boolean보다 원소 2개짜리 열거 타입이 더 낫다. 
  - `public enum TemperatureScale { FAHRENHEIT, CELSIUS }`
---
#### 아이템 52. 다중정의는 신중히 사용하라
- 컬렉션 분류기 프로그램


```java
public class Collectionclassifier {
    public static String classify(Set<?> s) {
        return "집합"; 
    }
    
    public static String classify(List<?> 1st) { 
        return "리스트";
    }
    
    public static String classify(Collection<?> c) { 
        return "그 외";
    }
    
    public static void main(String[] args) { 
        Collection<?>[] collections = {
            new HashSet<String>(),
            new ArrayList<BigInteger>(),
            new HashMap<String, String>().values()
        };
        
        for (Collection<?> c : collections)
            System.out.println(classify(c));
    } 
}
```


- 예상 출력 결과는 "집합", "리스트", "그 외" 이지만
- 실제 출력 결과는 "그 외"를 3번 출력한다.
  - 다중정의(overloading)된 classify는 어느 메서드를 호출할지가 컴파일 타임에 정해지기 때문이다.
  - 컴파일 타임에 for문에 있는 c는 항상 Collection<?> 타입
  - 컴파일타임의 매개변수를 기준으로 classify(Collection<?>) 만 호출한다.


- 재정의 메서드 호출 메커니즘


```java
class Wine {
    String name() { return "포도주"; }
}

class SparklingWine extends Wine {
    @Override String name() { return "발포성 포도주"; }
}

class Champagne extends SparklingWine { (
    @Override String name() { return "샴페인"; }
}

public class Overriding {
    public static void main(String[] args) {
        List<Wine> wineList = List.of(
            new Wine(), new SparklingWine(), new Champagne());
            
        for (Wine wine : wineList) 
            System.out. printIn(wine.name());
    } 
}
```


- 출력 결과 : "포도주", "발포성 포도주", "샴페인"

- for문에 있는 타입과 무관하게 가장 하위에서 정의한 재정의 메서드가 실행된다.
- 다중정의 메서드에서는 객체의 런타임 타입이 중요하지 않다.
  - 선택은 컴파일 타임 타입에 의해 이뤄진다.
---
#### 아이템 53. 가변인수는 신중히 사용하라
- 가변인수(varargs) 메서드는 명시한 타입의 인수를 0개 이상 받을 수 있다.
  - 메서드를 호출하면, 먼저 인수의 개수와 길이가 같은 배열을 만들고 인수들을 배열에 저장하여 가변인수 메서드에 전해준다.


```java
static int sum(int... args) { 
    int sum = 0;
    for (int arg : args) 
        sum += arg;
    return sum;
}
```


- 인수가 1개 이상이어야 하는 가변인수 메서드 : 컴파일 타임이 아닌 런타임에 실패한다는 단점이 있다.


```java
static int min(int... args) { 
    if (args.length = 0)
        throw new IllegalArgumentException("인수가 !개 이상 필요합니다."); 
    int min = args[0];
    for (int i = 1; i < args.length; i++)
        if (args[i] < min) 
            min = args[i];
    return min; 
}
```


- 매개변수를 2개 받는 방법을 사용하면 문제를 해결할 수 있다.


```java
static int min(int firstArg, int... remainingArgs) { 
    int min = firstArg;
    for (int arg : remainingArgs)
        if (arg < min) 
            min = arg;
    return min; 
}
```
---
#### 아이템 54. null이 아닌, 빈 컬렉션이나 배열을 반환하라
- null 을 반환하게 되면 API 사용이 어려워지고 오류 처리 코드가 늘어난다.


- 빈 컬렉션 반환 코드 예시
```java
public List<Cheese> getCheeses() {
    return new ArrayList<>(cheesesInStock);
}

// 최적화 : 빈 컬렉션을 매번 할당하지 않는 코드
public List<Cheese> getCheeses() {
    return cheeseInStock.isEmpty() ? Collections.emptyList()
    : new ArrayList<>(cheesesInStock);
}
```
