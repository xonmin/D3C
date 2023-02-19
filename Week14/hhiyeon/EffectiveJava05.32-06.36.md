#### 아이템 32. 제네릭과 가변인수를 함께 쓸 때는 신중하라
- 가변 인수는 메서드를 호출하면 가변 인수를 담기 위한 배열이 자동으로 생성된다.
- 배열이 클라이언트에 노출이 되어 매개변수에 제네릭이나 매개변수화 타입이 포함되면 컴파일 경고가 발생하게 된다.
- 매개변수 인수 메서드를 호출할 경우, 매개변수가 실체화 불가 타입으로 추론될 때 경고 형태
  - 매개변수화 타입의 변수가 타입이 다른 객체를 참조하면 힙 오염이 발생한다.

```java
warning: [unchecked] Possible heap pollution from 
    parameterized vararg type List<String>
```


- 자바 7에서 @SafeVarargs 애너테이션 추가 : 메서드 작성자가 타입 안전함을 보장하는 장치
- 제네릭 varargs 매개변수를 안전하게 사용하는 메서드


```java
@SafeVarargs
static <T> List<T> flatten(List<? extends T>... lists) {
    List<T> result = new ArrayListo(); 
    for (List<? extends T> list : lists)
        result.addAlKlist); 
    return result;
}
```

- 제네릭 varargs 메서드가 안전한 기준
  - varargs 매개변수 배열에 아무것도 저장하지 않는다.
  - 그 배열(혹은 복제본)을 신뢰할 수 없는 코드에 노출하지 않는다.
---
#### 아이템 33. 타입 안전 이종 컨테이너를 고려하라
- 제네릭 형태에서는 한 컨테이너가 다룰 수 있는 타입 매개변수의 수가 고정되어 있다.
- 컨테이너 자체가 아니라 키를 타입 매개변수로 바꾸면 제약이 없는 타입 안전 이종 컨테이너를 만들 수 있다.
- 타입 토큰 : 타입 안전 이종 컨테이너는 Class를 키로 사용한다.
---
### 📌 6장 : 열거 타입과 애너테이션
#### 아이템 34. int 상수 대신 열거 타입을 사용하라
- 정수 열거 패턴 단점
  - 타입 안전을 보장할 방법이 없고, 표현력이 좋지 않다.
  - 문자열 출력이 까다롭다.
- 열거 타입(enum type)
  - 열거 타입 자체는 클래스이다.
  - 상수 하나당 자신의 인스턴스를 만들어 public static final 필드로 공개한다.
  - 밖에서 접근할 수 있는 생성자를 제공하지 않기 때문에 final
  - 열거 타입 선언으로 만들어진 인스턴스는 딱 하나씩 존재하는 것을 보장해준다.
---
#### 아이템 35. ordinal 메서드 대신 인스턴스 필드를 사용하라
- 열거 타입 상수는 하나의 정수값과 대응된다.
- 모든 열거 타입은 해당 상수가 열거 타입에서 몇 번째 위치인지 ordinal이라는 메서드를 제공해준다.
- 잘못된 사용 예시 
  - 상수 선언 순서가 바뀌면 numberOfMusicians 오동작
  - 해결방법 : 열거 타입 상수에 연결된 값은 ordinal 메서드로 얻지 말고 인스턴스 필드에 저장하기

```java
public enum Ensemble {
    SOLO, DUET, TRIO, QUARTET, QUINTET, 
    SEXTET, SEPTET, OCTET, NONET, DECTET;
    
    public int numberOfMusicians() { return ordinal() + 1; } 
}
```

```java
public enum Ensemble {
    SOLO(l), DUET(2), TRI0(3), QUARTET(4), QUINTET(5), 
    SEXTET(6), SEPTET(7), 0CTET(8), D0UBLE_QUARTET(8), 
    N0NET(9), DECTET(10), TRIPLE_QUARTET(12);
    
    private final int numberOfMusicians;
    Ensemble(int size) { this.numberOfMusicians = size; }
    public int numberOfMusicians() { return numberOfMusicians; }
}
```
---
#### 아이템 36. 비트 필드 대신 EnumSet을 사용하라
- 비트 필드(bit field)
  - 비트별 OR을 사용해 여러 상수를 하나의 집합으로 모을 수 있는 집합
  - 비트별 연산을 사용해 합집합과 교집합 같은 집합 연산을 효율적으로 수행
  - 정수 열거 상수의 단점을 갖고있다.
- java.util 패키지의 EnumSet 클래스는 열거 타입의 상수 값으로 구성되 집합을 효과적으로 표현한다.


```java
public class Text {
    public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }
    
    // 어떤 Set을 넘겨도 되나, EnumSetoi 가장 좋다.
    public void applyStyles(Set<Style> styles) { ... } 
}
```
---