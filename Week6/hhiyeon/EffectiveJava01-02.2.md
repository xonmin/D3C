### 📌 1장 : 들어가기
- 컴포넌트 : 개별 메서드부터 여러 패키지로 이뤄진 복잡한 프레임워크까지 재사용 가능한 모든 소프트웨어 요소
- 자바 자료형(type) 
  - 참조 타입(reference type) : 인터페이스, 클래스, 배열
  - 기본 타입(primitive type)
- annotation : 인터페이스의 일종
- enum : 클래스의 일종
---
### 📌 2장 : 객체 생성과 파괴
#### 아이템 1. 생성자 대신 정적 팩터리 메서드를 고려하라
- 생성자 : 클라이언트가 클래스의 인스턴스를 얻는 수단
- 클래스는 생성자 대신에 정적 팩토리 메소드(static factory method)를 제공할 수 있다.
- 정적 팩토리 메소드의 장점
  1. 이름을 가질 수 있다.
     - 생성자보다 객체의 특성을 이름으로 묘사할 수 있다.
  2. 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.
     - 인스턴스 통제 클래스 : 반복되는 요청에 같은 객체를 반환하는 식으로 정적 팩토리 방식의 클래스는 언제 어느 인스턴스를 살아 있게 할지 통제할 수 있다.
     - 인스턴스를 통제하면, 클래스를 싱글톤 패턴 or 인스턴스화 불가로 만들 수 있다.
     - 또한, 불변 값 클래스에서 동치 인스턴스가 단 하나임을 보장할 수 있다.
     - ex. a == b 일 때, a.equals(b)가 성립
  3. 반환 타입의 하위 타입 객체를 반화할 수 있는 능력이 있다.
     - 구현 클래스를 공개하지 않아도, 그 객체를 반환할 수 있어 API 생성을 할 때 작게 유지 가능하다.
  4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
  5. 정적 팩토리 메소드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.
     - 서비스 제공자 프레임워크 3개의 핵심 컴포넌트
       - 서비스 인터페이스(service interface) : 구현체의 동작을 정의
       - 제공자 등록 API(provider registration API) : 제공자가 구현체를 등록할 때 사용
       - 서비스 접근 API(service access API) : 클라이언트가 서비스의 인스턴스를 얻을 때 사용
     - 종종 서비스 제공자 인터페이스(service provider interface)라는 네 번째 컴포넌트도 쓰인다.
       - 서비스 인터페이스의 인스턴스를 생성하는 팩토리 객체를 설명해준다.
     - ex. JDBC 
       - 서비스 인터페이스 : Connection
       - 제공자 등록 API : DriverManager.registerDriver
       - 서비스 접근 API : DriverManager.getConnection
       - 서비스 제공자 인터페이스 : Driver
- 정적 팩토리 메소드 패턴의 단점
  1. 상속을 하려면 public, protected 생성자가 필요하다. 정적 팩토리 메소드만 제공하면 하위 클래스를 만들 수 없다.
  2. 정적 팩토리 메소드는 프로그래머가 찾기 어렵다.
     - 알려진 규약을 따라 이름을 짓는 것으로 문제점을 완화한다.
     - from : 매개 변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메소드<br> `Date d = Date.from(instant);`
     - of : 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메소드<br> `Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);`
     - valueOf : from과 of의 자세한 버전<br> `BigInteger prime = BingInteger.valueOf(Integer.MAX_VALUE);`
     - instance 혹은 getInstance : (매개 변수를 받으면) 매개 변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지 않는다.<br> `StackWalker luke = StackWalker.getInstance(options);`
     - create 혹은 newInstance : instance, getInstance 와 같지만 매번 새로운 인스턴스를 생성해 반환한다.<br> `Object newArray = Array.newInstance(classObject, arrayLen);`
     - getType : getInstance 와 같지만, 생성할 클래스가 아닌 다른 클래스에 팩토리 메소드를 정의할 때 쓴다.<br> `FileStore fs = Files.getFilesStore(path);`
     - newType : newInstance 와 같지만, 생성할 클래스가 아닌 다른 클래스에 팩토리 메소드를 정의할 때 쓴다.<br> `BufferReader br = Files.newBufferReader(path);`
     - type : getType 과 newType 의 간결한 버전<br> `List<Complaint> litany = Collections.list(legacyLitany);`
---
#### 아이템 2. 생성자에 매개변수가 많다면 빌더를 고려하라
- 정적 팩토리, 생성자는 둘 다 선택적 매개변수가 많으면 적절히 대응하기 힘들다.
- 점층적 생성자 패턴(telescoping constructor pattern)
  - 필수 매개변수만 받는 생성자, 필수 매개변수와 선택 매개변수 1개만 받는 생성자, ..., 형태들로 선택 매개변수를 전부 다 받는 생성자 까지 늘려가는 방식
- 점층적 패턴을 사용할 수 있지만, 매개변수의 개수가 많아지면 클라이언트 코드를 작성하거나 읽기 어려워진다.


- 선택 매개 변수가 많은 경우, 두 번째 대안 -> 자바빈즈 패턴(JavaBeans pattern)
  - 매개변수가 없는 생성자로 객체를 만들고, setter 으로 원하는 매개변수 값을 설정하는 방식
  - 하지만, 객체 하나를 만들기 위한 여러 메소드 호출의 필요성과 객체 생성 전까지 일관성이 무너진다.


- 세번째 대안, 빌더 패턴(Builder pattern)
  - 점층적 생성자 패턴의 안전성 + 자바빈즈 패턴의 가독성
  - 클라이언트는 필요한 객체를 직접 만드는 대신, 필수 매개변수 만으로 생성자(혹은 정적 팩토리)를 호출해 빌더 객체를 얻는다.
  - 그 후에 빌더 객체가 제공하는 setter 으로 원하는 선택 매개변수를 설정
  - 마지막으로, 매개 변수가 없는 build 를 호출해서 필요한 객체를 얻는다.
  - 플루언트(fluent) API 혹은 메소드 연쇄(method chaining)
    - 빌더의 setter 들은 빌더 자신을 반환하기 때문에 연쇄적으로 호출할 수 있다.
    - 메소드 호출이 흐르듯 연결된다는 의미
```java
public class NutritionFacts { 
    private final int servingSize; 
    private final int servings; 
    private final int calories;
    private final int fat;
    private final int sodium; 
    private final int carbohydrate;
    
    public static class Builder {
    // 필수 매개변수
        private final int servingSize; 
        private final int servings;

        // 선택 매개변수 - 기본값으로 초기화한다.
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;
    
        public Builder(int servingSize, int servings) { 
            this.servingSize = servingSize; 
            this.servings = servings;
        }
        
        public Builder calories(int val)
            { calories = val; return this; }
            
        public Builder fatdnt val)
            { fat = val; return this; }
        public Builder sodium(int val)
            { sodium = val; return this; }
        public Builder carbohydrate(int val)
            { carbohydrate = val; return this; }
            
        public NutritionFacts build() { 
        return new NutritionFacts(this);
        } 
   }
   
   private NutritionFacts(Builder builder) { 
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
   }
}        
```


- 빌더 패턴 사용한 클래스를 사용하는 클라이언트 코드
`NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8) .calories(100).sodium(35).carbohydrate(27).build ();`
- 잘못된 매개변수는 빌더의 생성자, 메소드에서 검사하는 것이 좋다.
- 불변성을 위해서는 빌더로부터 매개변수를 복사하고 해당 객체 필드들을 검사해야 한다.
- 어떤 매개 변수가 어떻게 잘못되었는지는 `IllegalArgumentException`를 사용한다.


- 빌더 패턴은 계층적으로 설계된 클래스와 함께 사용하기 좋다.
```java
public abstract class Pizza {
    public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE } 
    final Set<Topping> toppings;

    abstract static class Builder<T extends Builder<T» { 
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class); 
        public T addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return self();
        }
        
        abstract Pizza build();
        // 하위 클래스는 이 메서드를 재정의(overriding)하여 
        // "this"를 반환하도록 해야 한다.
        protected abstract T self();
    }
 
    Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone(); // 아이템 50 참조
    }   
}
```