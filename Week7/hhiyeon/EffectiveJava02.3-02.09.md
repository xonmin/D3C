### 📌 2장 : 객체 생성과 파괴
#### 아이템 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라
- 싱글톤(singleton) : 인스턴스를 오직 하나만 생성할 수 있는 클래스
  - 전형적인 예시 : 무상태(stateless) 객체, 설계상 유일해야 하는 시스템 컴포넌트
  - 클래스를 싱글톤으로 만들면 클라이언트가 테스트하기 어려울 수 있다.

- 싱글톤을 만드는 방식 : 생성자는 private 으로 감추고, 유일하게 인스턴스에 접근할 수 있는 수단으로 public static 멤버를 하나 만든다.
- 방식1 : public static 멤버가 final 필드
- 방식2 : 정적 팩토리 메소드를 public static 멤버로 제공
  - 장점 : API를 바꾸지 않아도 싱글톤이 아니게 변경 가능하다. 정적 팩토리를 제너릭 싱글톤 팩토리로 만들 수 있다. 정적 팩토리 메소드 참조를 공급자(supplier)로 사용할 수 있다.
  - 장점이 필요하지 않으면 방법1이 더 좋다.

- 싱글톤 클래스 직렬화
  - Serializable 을 구현하는 것 만으로는 부족하다.
  - 모든 인스턴스 필드를 일시적(transient)으로 선언하고, readResolve 메소드를 제공해야 한다.
  - 이렇게 하지 않으면, 직렬화된 인스턴스를 역직렬화할 때마다 새로운 인스턴스가 만들어진다.

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public void leaveTheBuilding() { ... } 
}
```

```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis(); private Elvis() { ... }
    public static Elvis getlnstance() { return INSTANCE; }
    public void leaveTheBuilding() { ... } 
}
```
---
#### 아이템 4. 인스턴스화를 막으려거든 private 생성자를 사용하라
- 생성자를 명시하지 않으면 컴파일러가 자동으로 기본 생성자를 만든다.
- 추상 클래스로 만드는 것으로는 인스턴스화를 막을 수 없다.
- 컴파일러가 기본 생성자를 만드는 경우는 명시된 생성자가 없을 때 뿐이기 때문에
- private 생성자를 추가하면 클래스의 인스턴스화를 막을 수 있다.
---
#### 아이템 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라
- 정적 유틸리티를 잘못 사용한 예시
```java
public class Spellchecker {
  private static final Lexicon dictionary = ...;
  private SpellChecker() {} // 객체 생성 방지
  public static boolean isValid(String word) { ... }
  public static List<String> suggestions(String typo) { ... } 
}
```


- 싱글톤을 잘못 사용한 예시
```java
public class Spellchecker {
  private final Lexicon dictionary = ...;
  private Spellchecker(...) {}
  public static Spellchecker INSTANCE = new SpellChecker(••.);
  public boolean isValid(String word) { ... }
  public List<String> suggestions(String typo) { ... } 
}
```

- 사용하는 자원에 따라 동작이 달라지는 클래스에는 유틸리티 클래스나 싱글톤 방식이 적합하지 않다.
- 인스턴스를 생성할 때 생성자가 필요한 자원 넘겨주는 방식
- 의존 객체 주입의 한 형태 : 유연성과 테스트 용이성을 높여준다.
```java
public class Spellchecker {
  private final Lexicon dictionary;
  public Spellchecker(Lexicon dictionary) {
    this.dictionary = Objects.requireNonNull(dictionary); 
  }
  public boolean isValid(String word) { ... }
  public List<String> suggestions(String typo) { ... } 
}
```
---
#### 아이템 6. 불필요한 객체 생성을 피하라
- 생성자 대신 정적 팩토리 메소드를 제공하는 불변 클래스는 정적 패곹리 메소드를 사용해 불필요한 객체 생성을 피할 수 있다. 
- ex. Boolean(String) 생성자 대신에 Boolean.valueOf(String) 팩토리 메소드를 사용하는 것이 좋다.
```java
String s = new StringC'bikini"); // 따라 하지 말 것!
String s = "bikini";
```


- 생성 비용이 아주 비싼 객체도 있다. 자신이 만드는 객체가 비싼 객체인지 알 방법이 명확하게 없기 때문에
- 정규 표현식을 사용해 성능을 끌어올릴 수 있다.
- 하지만, String.matches 메소드를 사용하기 때문에 성능이 중요한 상황에서 반복해 사용하기 적합하지 않다.
- Pattern 인스턴스는 한 번 쓰고 버려지기 때문에 가비지 컬렉션 대상이 된다.
- Pattern 은 입력받은 정규표현식에 해당하는 유한 상태 머신(finite state machine)을 만들기 때문에 인스턴스 생성 비용이 높다.
```java
static boolean isRomanNumeraI(String s) { 
  return s.matchesC'^?#. ()*CM[MD] |D?C{0,3})"
    + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}
```


- 성능을 개선하기 위해서, 필요한 정규 표현식을 표현하는 Pattern 인스턴스를 클래스 초기화 과정에서 직접 생성해 캐싱하고
- 나중에 메소드가 호출될 때마다 이 인스턴스를 재사용한다.

```java
public class RomanNumerals {
  private static final Pattern ROMAN = Pattern.compile(
    u("*C(?[=M.)DM]|D?C{0,3})"
      + "(X[C니 |L?X{0,3})(I[XV]|V?I{0,3})$");
  static boolean isRomanNumeral(String s) { 
   return ROMAN.matcher(s).matches();
  } 
}
```

```java
private static long s니m() { 
  Long sum = 0L; // Long으로 선언해서 불필요한 Long 인스턴스가 2^31개 만들어진다.
  for (long i = 0; i <= Integer.MAX_VALUE; i++) 
    sum += i;
  return sum; 
}
```


- 불필요한 객체를 만드는 또 다른 예시 : 오토박싱(auto boxing)
- 박싱된 기본 타입보다 기본 타입을 사용하고, 의도치 않은 오토박싱이 숨어들지 않도록 주의하기
---
#### 아이템 7. 다 쓴 객체 참조를 해제하라
- 메모리 누수가 일어나는 위치
- stack은 element 배열로 저장소 풀을 만들고 원소들을 관리하기 때문에
- 가비지 컬렉션은 비활성 영역에 대해 알 수 없다.
- 비활성 영역이 되는 순간 null 처리 해준다.
```java
public Object pop() { 
  if (size = 0)
  throw new EmptyStackException(); 
  return elements[—size];
}

// 메모리 누수 개선
public Object pop() { 
  if (size = 0)
  throw new EmptyStackException(); 
  Object result = elements[—size]; 
  elements[size] = null; // 다 쓴 참조 해제 
  return result;
}
```

- 객체 참조를 null 처리하는 일은 예외적이어야 한다.
- 다 쓴 참조를 해제하는 가장 좋은 방법은 그 참조를 담은 변수를 유효 범위(scope) 밖으로 밀어내는 것이다.
- 자기 메모리를 직접 관리하는 클래스는 항시 메모리 누수에 주의한다.
- 캐시도 메모리 누수를 일으키기 쉽다. 객체 참조를 캐시에 넣고 놔두는 경우가 있다.
  - weakHashMap 사용 : 다 쓴 entry는 즉시 자동 제거
  - Scheduled ThreadPoolExecutor 같은 백그라운드 스레드를 사용하거나 LinkedHashMap은 removeEldestEnry 메소드를 사용
---
#### 아이템 8. finalizer 와 cleaner 사용을 피하라
- 자바의 두 가지 객체 소멸자
  - finalizer : 예측 불가능, 상황에 따라 위험할 수 있어 일반적으로 불필요
  - cleaner : finalizer보다 덜 위험하지만, 예측 불가능, 느리고 일반적으로 불필요하다.
- finalizer, cleaner 로는 제때 실행되어야 하는 작업은 절대 할 수 
- 상태를 영구적으로 수정하는 작업에서는 절대 finalizer, cleaner 에 의존해서는 안된다.
  - ex. 데이터베이스 같은 공유 자원의 lock 해제를 맡기면 분산 시스템 전체가 서서히 멈추게 된다.
- finalizer 는 가비지 컬렉터의 효율을 약 50배 떨어뜨리기 때문에 성능 저하 문제가 있다.
- finalizer, cleaner 대신에 사용할 방법 : AutoCloseable
- AutoCloseable
  - 클라이언트에서 인스턴스를 다 쓰고, close 메소드를 호출하면 된다.
---
#### 아이템 9. try-finally 보다는 try-with-resources 를 사용하라
- 자바에서는 close 메소드를 호출해서 직접 닫아줘야하는 경우가 있다.
- try-finally 를 사용해서 자원을 안전하게 닫을 수 있지만, 자원이 둘 이상이면 지저분해진다.
- 자바 7에서 try-with-resources 덕에 문제 해결
  - 해당 자원이 AutoCloseable 인터페이스를 구현해야 한다.
```java
static void copy(String src, String dst) throws lOException { 
  try (Inputstream in = new FilelnputStream(src);
    Outputstream out = new FileOutputStream(dst)) { 
      byte[] buf = new byte[BUFFER_SIZE];
      int n;
      while ((n = in.read(buf)) >= 0)
        out.write(buf, 0, n);
  } 
}
```
