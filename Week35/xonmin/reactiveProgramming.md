# Chapter 04. 리액티브 프로그래밍을 위한 사전 지식 

책에서의 해당 챕터에서는 리액티브 프로그래밍을 학습하는 데 최소한의 함수형 프로그래밍 기법을 다룬다.

## 4.1 Functional Interface 
함수형 인터페이스 : body가 없는 단 하나의 추상 메서드로만 구성되어있음 
- 이 때 default method 는 예외
- fp에서는 함수를 값으로 취급한다. 따라서 어떤 함수를 호출 시, 함수 자체를 파라미터로 전달이 가능

```java
public class Example4_1 {

  List<CryptoCurrency> cryptoCurrencies = SampleData.cryptoCurrencies;

  Collections.sort(cryptoCurrencies, new Comparator<CryptoCurrency>() {
      @Override
      public int compare(CryptoCurrency cc1, CryptoCurrency cc2) {
          return cc1.getUnit().name().compareTo(cc2.getUnit().name());
      }
   });
}
```
- 여기서 Comparator 인터페이스는 `compareTo()`가 단 하나의 추상 메서드를 가지고, 결국 함수형 인터페이스
- Comparator 인터페이스를 익명 구현 객체의 형태로 `Collections.sort()` 파라미터로 전달, 코드가 지저분
  - 좀 더 FP스럽게 변경하기 위해서는 Lambda Expression(람다 표현식)을 사용한다

```java
@FunctionalInterface
public interface Comparator<T> {
    /**
     * Compares its two arguments for order.  Returns a negative integer,
     * zero, or a positive integer as the first argument is less than, equal
     * to, or greater than the second.<p>
     *
     * The implementor must ensure that {@code sgn(compare(x, y)) ==
     * -sgn(compare(y, x))} for all {@code x} and {@code y}.  (This
     * implies that {@code compare(x, y)} must throw an exception if and only
     * if {@code compare(y, x)} throws an exception.)<p>
     *
     * The implementor must also ensure that the relation is transitive:
     * {@code ((compare(x, y)>0) && (compare(y, z)>0))} implies
     * {@code compare(x, z)>0}.<p>
     *
     * Finally, the implementor must ensure that {@code compare(x, y)==0}
     * implies that {@code sgn(compare(x, z))==sgn(compare(y, z))} for all
     * {@code z}.<p>
     *
     * It is generally the case, but <i>not</i> strictly required that
     * {@code (compare(x, y)==0) == (x.equals(y))}.  Generally speaking,
     * any comparator that violates this condition should clearly indicate
     * this fact.  The recommended language is "Note: this comparator
     * imposes orderings that are inconsistent with equals."<p>
     *
     * In the foregoing description, the notation
     * {@code sgn(}<i>expression</i>{@code )} designates the mathematical
     * <i>signum</i> function, which is defined to return one of {@code -1},
     * {@code 0}, or {@code 1} according to whether the value of
     * <i>expression</i> is negative, zero, or positive, respectively.
     *
     * @param o1 the first object to be compared.
     * @param o2 the second object to be compared.
     * @return a negative integer, zero, or a positive integer as the
     *         first argument is less than, equal to, or greater than the
     *         second.
     * @throws NullPointerException if an argument is null and this
     *         comparator does not permit null arguments
     * @throws ClassCastException if the arguments' types prevent them from
     *         being compared by this comparator.
     */
    int compare(T o1, T o2);

  
    default Comparator<T> reversed() {
        return Collections.reverseOrder(this);
    }

    default Comparator<T> thenComparing(Comparator<? super T> other) {
        Objects.requireNonNull(other);
        return (Comparator<T> & Serializable) (c1, c2) -> {
            int res = compare(c1, c2);
            return (res != 0) ? res : other.compare(c1, c2);
        };
    }
```
Comparator 내부에 보면 `compare()` 만 추상 메서드로 정의, 외에 모두 default Method 

## 4.2 Lambda Expression 
- js 의 Arrow Function(`unit => unit.length`)과 유사
- Lambda Expression : Java 에서도 인터페이스의 익명 구현 클래스를 화살표 함수처럼 간결하게 표현하는 방법
  - 단 하나의 추상 메서드를 가지는 인터페이스(함수형 인터페이스)를 구현한 클래스의 메서드 구현을 단순화한 표현식 
```kotlin
(a: String, b: String) => a.equals(b)
```
```java
psvm {
  Collections.sort(cryptoCurrencies,
      (cc1, cc2) -> cc1.getUnit().name().compareTo(cc2.getUnit().name()));
```
함수형 인터페이스의 추상 메서드를 람다 표현식으로 메서드의 파라미터로 전달하는 의미 : 
- 함수형 인터페이스의 추상 메서드 자체를 파라미터로 전달하는 것이 아닌, 
- **함수형 인터페이스를 구현한 클래스의 인스턴스를 람다 표현식으로 작성해서 전달한다**

Lambda Capturing : 람다 표현식 외부에서 정의된 변수를 표현식 내에서 사용하는 것 
- 이 때 외부 변수는 final / final과 같은 효력이 있어야 한다.

## 4.3 Method Reference 
```java 
car -> car.getCarName() = Car::getCarName
```
### 메서드 레퍼런스를 사용할 수 있는 유형
1. ClassName::static method

```java
.map(StringUtils.upperCase)
```
2. ClassName:: instance method
```java
.map(StringUtils.toUpperCase)
```
3. object::instance method
```java
Calculator cal = new PaymentCalculator(); 

...
.map(cc -> new ImmutablePair(cc.getPrice(), amount)
.map(cal::getTotalPayment)
.forEach(System.out::println)
```
4. ClassName::new

## 4.4 함수 디스크립터(Function Desriptor) 
함수 디스크립터 : 일반화된 람다 표현식을 통해, 이 함수형 인터페이스가 어떤 파라미터를 가지고, 어떤 값을 리턴하는 지 설명해주는 역할 
- 예시> functional Interface 와 함수 디스크립터 관계
  - Predicate<T> : T -> boolean
  - Function<T, R> : T -> R
  - Supplier<T> : () -> T

### Predicate 
함수형 인터페이스 중 구현해야 되는 추상 메서드가 하나의 파라미터를 가지고, 리턴 값이 boolean 일 때 함수형 인터페이스를 `Predicate`로 정의해놓음 
- ex> `.filter()` : 파라미터로 Predicate 를 가지며 내부적으로 test() 메서드를 사용하여 return True 인 애들만 내보낸다. 
```java
@FunctionalInterface
public interface Predicate<T> {
  boolean test(T t);
  ..
}
```

### Consumer 
Consumer 함수형 인터페이스는 단어의 의미와 같이 데이터를 소비하기 때문에 리턴 값이 없다. 
- 일정 주기별로 특정 작업 수행 후, 결과 값을 리턴할 필요가 없는 경우가 대부분인 배치 처리에 사용되는 예시 
`T -> void`
```java
@FunctionalInterface
public interface Consumer<T>  {
  void accpet(T t);
}
```

### Function 
구현해야하는 되는 추상 메서드가 하나의 파라미터(T) 를 가지며 리턴 값을 R로 반환 
- 대표적인 예시 함수 `.map()`
```java
@FunctionalInterface
public interface Function<T, R>  {
  R apply(T t);
}
```
```java 
// 용례
private static int calculatePayment(List<CryptoCurrency> cryptoCurrencies, Function<CryptoCurrency, Integer> f) {
  int totalPayment = 0;
  for (CryptoCurrency cc : cryptoCurrencies) {
     totalPayment += f.appply(cc);
  }

  return totalPayment;
} 

// 사용처
...
int totalPayment = calculatePayment(cryptoCurrencies, cc -> cc.getPrice() * 2); 
```

### Supplier 
추상 메서드가 파라미터를 갖지 않으며, 리턴 T를 하는 함수형 인터페이스
```java
@FunctionalInterface
public interface Supplier<T> {
  T get();
}
```
```java
// 예시
private static String createMneomonic() {
  return Stream.generate(() -> getMnemonic())
               .limit(10)
...
```

### Bixxxxx 
- BiPredicate, BiConsumer, BiFunction 과 같이 Bi로 시작하는 함수형 인터페이스
- 함수형 인터페이스에서 구현해야 하는 추상 메서드에 전달하는 파라미터가 하나 더 추가되어 두 개의 파라미터를 가지는 함수형 인터페이스
- Java에서 지원하는 기본 함수형 인터페이스의 확장형

> 정리 
> - 함수형 인터페이스 : Java에서 FP 지원을 위해 Java 8부터 지원하는 인터페이스
> - 함수형 인터페이스는 단 하나의 추상 메서드를 가짐
> - 람다 표현식 : 함수형 인터페이스의 정의된 추상 메서드를 표현식으로 구현한 것
> - method Reference : 람다 표현식을 간결하게 표현하는 것
> - 함수 디스크립터를 통해 함수형 인터페이스의 파라미터 형식과 리턴 값의 형태를 알 수 있음 

