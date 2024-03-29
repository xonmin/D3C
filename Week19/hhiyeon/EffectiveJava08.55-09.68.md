#### 아이템 55. 옵셔널 반환은 신중히 하라
- 자바 8 이전에 값을 반환할 수 없는 경우
  - 예외 던지기
  - null 반환

- 예외 생성할 경우의 스택 추적 전체를 캡처하는 비용 발생
- null 값을 갖고있으면 추후에 NullPointerException 발생 위험이 있다.


- 자바 8 에서 Optional<T>으로 null이 아닌 T 타입 참조를 하나 담거나 아무것도 담지 않을 수 있다.
- Optional(옵셔널)은 원소를 최대 1개 가질 수 있는 불변 컬렉션


- 컬렉션에서 최댓값을 구하는 메소드


```java
public static <E extends Comparable<E>> E max(Collection<E> c) { 
    if (c.isEmpty())
        throw new IllegalArgumentException("빈 컬렉션");
    
    E result = null; 
    for (E e : c)
        if (result = null || e.compareTo(resuLt) > 0)
            result = Objects.requireNonNull(e);
    
    return result;
}
```


- Optional<E>로 반환하는 경우


```java
public static <E extends Comparable<E>> 
        Optional<E> max(Collection<E> c) { 
    if (c.isEmpty())
        return Optional.empty();
    
    E result = null; 
    for (E e : c)
        if (result = null || e.compareTo(resuLt) > 0)
            result = Objects.requireNonNull(e);
    
    return result;
}
```


- 옵셔널을 반환하는 메서드에서는 절대 null 반환하지 않기
---
#### 아이템 56. 공개된 API 요소에는 항상 문서화 주석을 작성하라
- API를 올바르게 문서화하기 위해서는 공개된 모든 클래스, 인터페이스, 메서드, 필드 선언에 문서화 주석을 달아야 한다.
- 메서드용 문서화 주석에는 해당 메서드와 클라이언트 사이의 규약을 명료하게 기술해야 한다.
- 표준 규약을 지켜서 작성한다.
---
### 📌 9장 : 일반적인 프로그래밍 원칙
#### 아이템 57. 지역변수의 범위를 최소화하라
- 지역변수의 유효 범위를 최소로 줄이면
  - 코드 가독성 증가
  - 유지보수성 증가
  - 오류 가능성 감소

- 지역변수 범위를 줄이는 방법
  - 가장 처음 쓰일 때 선언하기
  - 대부분 지역변수는 선언과 동시에 초기화한다.
  - 메서드를 작게 유지하고 한 가지 기능에 집중한다.
---
#### 아이템 58. 전통적인 for 문보다는 for-each 문을 사용하라
- for-each(enhanced for statement 향상된 for문)
  - 반복자와 인덱스 변수를 사용하지 않는다.
  - 코드가 깔끔하고 오류가 날 일이 없다.
  - 하나의 관용구로 컬렉션과 배열을 모두 처리할 수 있다.


- for-each를 사용할 수 없는 상황
  - 파괴적인 필터링(destructive filtering)
    - 컬렉션을 순회하면서 선택된 원소를 제거해야 하는 경우, 반복자의 remove 메서드를 호출해야 한다.
    - 자바 8에서 Collection의 removeIf 메서드를 사용해 컬렉션을 명시적 순회하는 일을 피할 수 있다.
  - 변형(transforming) 
    - 리스트나 배열을 순회하면서 일부 값이나 전체를 교체해야 하는 경우 리스트의 반복자나 배열의 인덱스를 사용해야 한다.
  - 병렬 반복(parallel iteration)
    - 여러 컬렉션을 병렬로 순회해야 하는 경우 각각의 반복자와 인덱스 변수를 사용해야 한다.
---
#### 아이템 59. 라이브러리를 익히고 사용하라
- 표준 라이브러리를 사용해서 크게 관련 없는 문제에 시간을 허비하지 않고, 기능 개발에 집중할 수 있다.
- 자바 7 부터는 Random 사용을 하지 않는게 좋다. `ThreadLocalRandom` 으로 대체
  - 포크-조인 풀이나 병렬 스트림에서는 `SplittableRandom` 사용
- 알아두면 좋은 API
  - java.lang
  - java.util
  - java.io
  - 이외 하위 패키지
  - java.util.concurrent 동시성 관련
---
#### 아이템 60. 정확한 답이 필요하다면 float와 double은 피하라
- flaot, dobule 타입은 과학과 공학 계산용으로 설계
- 부동 소수점연산에 쓰이기 때문에 정확한 결과가 필요한 경우에는 사용하지 않는다.
- `System.out.printIn(1.03 - 0.42);` 코드의 출력 결과
  - `0.6100000000000001` 출력


- 금융 계산과 같이 정확한 계산이 필요한 경우에는 BigDecimal, int, long을 사용해야 한다.
  - BigDecimal의 단점
    - 기본 타입보다 쓰기 불편하고 느리다.
    - int 혹은 long 타입을 사용할 수 있다.
- 숫자를 9 자리 십진수로 표현할 수 있으면 `int`
- 18 자리 십진수로 표현할 수 있으면 `long`
- 18 자리가 넘어가면 `BigDecimal`
---
#### 아이템 61. 박싱된 기본 타입보다는 기본 타입을 사용하라
- 자바의 데이터 타입 
  - 기본 타입 : int, double, boolean
  - 참조 타입 : String, List


- 박싱된 기본 타입
  - 각각의 기본 타입에 대응하는 참조 타입
  - int : Integer
  - double : Double
  - boolean : Boolean


- 기본타입과 박싱된 기본 타입의 차이
  - 기본 타입은 값만 가지고 있고, 박싱된 기본 타입은 값 + 식별성(identity)을 갖는다.
  - 기본 타입의 값은 언제나 유효하지만, 박싱된 기본 타입은 null을 가질 수 있다.
  - 기본 타입이 박싱된 기본 타입보다 시간과 메모리 사용면에서 더 효율적이다.



```java
Comparator<Integer> naturalOrder =
    (i, j) -> (i < j) ? -1 : (i == j ? 0 : 1);
```


- 같은 값 비교시, 0을 출력해야 하지만 1을 출력하는 이유는?
  - 같은 객체를 비교하는 게 아니면 박싱된 기본 타입에 == 연산자를 사용하면 오류 발생
  

- 실무에서는 기본 타입을 다루는 비교자가 필요할 때
  - Comparator, naturalOrder() 사용


```java
Comparator<Integer> naturalOrder = (iBoxed, jBoxed) -> { 
    int i = iBoxed, j = jBoxed; // 오토박싱
    return i < j ? -1 : (i == j ? 0 : 1);
};
```


- 기본 타입과 박싱된 기본 타입을 혼용한 연산에서 박싱된 기본 타입의 박싱은 자동으로 해제된다.
- null 참조를 다시 언박싱 하게 되면 NullPointerException 이 발생한다.
---
#### 아이템 62. 다른 타입이 적절하다면 문자열 사용을 피하라
- 문자열이 다른 값 타입을 대신하기 적합하지 않는 경우
  - 열거 타입 : 상수를 열거할 때 문자열보다 열거 타입이 낫다.
  - 혼합 타입 : 각 요소를 개별로 접근하려면 문자열을 파싱해야 하기 때문에 속도가 느리고 오류가 생길 수 있다.
    - 이런 경우, private 정적 멤버 클래스로 새로 선언하는 것이 낫다.
  - 권한을 표현하는 경우
---
#### 아이템 63. 문자열 연결은 느리니 주의하라
- 문자열 연결 연산자 (+)
  - 문자열 연결 연산자를 사용한 문자열 n개를 잇는 시간은 `n^2` 에 비례한다.
  - 성능 개선을 위해 String 보다 StringBuilder 를 사용한다.
---
#### 아이템 64. 객체는 인터페이스를 사용해 참조하라
- 매개변수 타입으로 클래스보다 인터페이스가 더 적합하다.
- 적합한 인터페이스가 있는 경우, 매개변수 말고도 반환 값, 변수, 필드를 전부 인터페이스 타입으로 선언해야 한다.
- 객체의 실제 클래스를 사용해야 하는 경우는 생성자로 생성할 경우이다.
- 예시 : Set 인터페이스를 구현한 LinkedHashSet 변수를 선언


```java
// 좋은 예. 인터페이스를 타입으로 사용했다.
Set<Son> sonSet = new LinkedHashSet();

// 나쁜 예. 클래스를 타입으로 사용했다.
LinkedHashSet<Son> sonSet = new LinkedHashSet();
```
---
#### 아이템 65. 리플렉션보다는 인터페이스를 사용하라
- 리플렉션 기능(java.lang.refelct)을 사용하면 프로그램에서 임의의 클래스에 접근할 수 있다.
- Class 객체가 주어지는 경우, 클래스의 생성자, 메서드, 필드에 해당하는 Constructor, Method, Field 인스턴스를 가져올 수 있다.
- 인스턴스들로 클래스의 멤버 이름, 필드 타입, 메서드 시그니처를 가져올 수 있다.
- 리플렉션 단점
  - 컴파일타임 타입 검사가 주는 이점을 누릴 수 없다.
  - 리플렉션을 이용하면 코드가 지저분해진다.
  - 성능이 저하된다.

- 리플렉션을 써야 하는 복잡한 애플리케이션이 있지만, 단점 때문에 사용을 줄이고 있다.
- 리플렉션은 아주 제한된 형태로 사용해야 단점을 피하고 이점을 취할 수 있다.
- 컴파일 타임에는 알 수 없는 클래스를 사용해야하는 프로그램을 작성해야 할 때
  - 리플렉션은 인스턴스 생성에만 사용하고, 만든 인스턴스는 인터페이스나 상위 클래스로 참조해서 사용한다.
---
#### 아이템 66. 네이티브 메서드는 신중히 사용하라
- 자바 네이티브 인터페이스(Java Native Interface, JNI)
  - 자바 프로그램이 네이티브 메서드를 호출하는 기술
  - 네이티브 메서드 : 네이티브 프로그래밍 언어로 작성한 메서드

- 네이티브 메서드의 쓰임새
  - 레지스트리 같은 플랫폼 특화 기능을 사용한다.
  - 네이티브 코드로 작성된 기존 라이브러리를 사용한다.
  - 성능 개선을 목적으로 성능에 결정적인 영향을 주는 영역만 따로 네이티브 언어로 작성한다.

- 자바는 점점 하부 플랫폼의 기능을 흡수하고 있기 떄문에 네이티브 메서드 사용이 줄어들고 있다.
  - ex. 자바9에서 process API를 추가해 OS 프로세스에 접근하는 길을 열어줌

- 네이티브 메서드로 성능 개선이 되는 일은 적기 때문에 되도록 사용하지 않는다.
---
#### 아이템 67. 최적화는 신중히 하라
- 빠른 프로그램보다 좋은 프로그램을 작성해야 한다.
- 성능을 제한하는 설계는 피해야한다.
- API를 설계할 때 성능에 주는 영향을 고려해라
- 모든 변경 후에 성능을 측정하라
---
#### 아이템 68. 일반적으로 통용되는 명명 규칙을 따르라

<img width="593" alt="스크린샷 2023-03-26 오후 10 32 22" src="https://user-images.githubusercontent.com/52193680/227779156-f3b7bbb9-601a-4449-b2b3-bc2c2b807e04.png">

- 임의의 타입 T
- 컬렉션 원소의 타입 E
- 맵의 키와 값에 K, V
- 예외 X
- 메서드 반환 타입 R

---
