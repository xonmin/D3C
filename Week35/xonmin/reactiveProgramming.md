# Chapter 04. 리액티브 프로그래밍을 위한 사전 지식 

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
