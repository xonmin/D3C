#### 아이템 44. 표준 함수형 인터페이스를 사용하라
- 필요한 용도에 맞는 게 있으면, 직접 구현하지 않고 표준 함수형 인터페이스를 활용하는 것이 좋다.
- 표준 함수형 인터페이스는 다른 코드와의 상호운용성이 크게 좋아질 것이다.
- 기본 함수형 인터페이스


<img width="583" alt="스크린샷 2023-03-12 오후 9 53 41" src="https://user-images.githubusercontent.com/52193680/224545955-4a77e3eb-4b38-4865-8189-1c79042dcff6.png">


<img width="586" alt="스크린샷 2023-03-12 오후 9 53 48" src="https://user-images.githubusercontent.com/52193680/224545963-3141f9b4-8542-48e6-99e8-a80bbf9c914c.png">


- 직접 만든 함수형 인터페이스는 항상 @FunctionInterface 애너테이션을 사용한다.
- 애너테이션을 사용하는 이유
  - 해당 클래스의 코드나 설명 문서를 읽는 사람에게 인터페이스가 람다용으로 설계된 것을 알려주기 위해
  - 해당 인터페이스가 추상 메서드 하나만 가지고 있어야 컴파일을 해준다.
  - 유지보수 과정에서 실수로 메서드를 추가하지 못하게 막아준다.
---
#### 아이템 45. 스트림은 주의해서 사용하라
- 스트림 API
  - 다량의 데이터 처리 작업을 돕기위해 자바8에서 추가되었다.
  - 스트림(stream) : 데이터 원소의 유한 혹은 무한 시퀀스(sequence)
    - 기본 타입 값 : int, long, double
  - 스트림 파이프라인(stream pipeline) : 원소들로 수행하는 연산 단계를 표현하는 개념
    - 소스 스트림에서 시작해 종단 연산(terminal operation)으로 끝난다.
    - 사이에 하나 이상의 중간 연산(intermediate operation)이 있을 수 있다.
    - 중간 연산은 스트림을 어떠한 방식으로 변환(transform)한다.
    - 스트림 파이프라인은 종단 연산이 호출될 때 지연 평가(lazy evaluation)된다.

- 스트림을 과하게 사용한 경우 -> 유지보수하기 어려워진다.


```java
public class Anagrams {
    public static void main(String[] args) throws lOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parselnt(args[1]);
        
        try (Stream<String> words = Files.lines(dietionary)) { 
            words.collect (
                groupingBy(word -> word.chars().sorted() 
                    .collect(StringBuilder::new,
                        (sb, c) -> sb.append((char) c), 
                        StringBuilder::append).toString()))
            .values().stream()
            .filter(group -> group.size() >= minGroupSize)
            .map(group -> group.size() + ": " + group)
            .forEach(System.out::println);
        }
    }
}
```
---
#### 아이템 46. 스트림에서는 부작용 없는 함수를 사용하라
- 스트림뿐만 아니라 스트림 관련 객체에 건네지는 모든 함수 객체가 부작용이 없도록 사용해야 한다.
- 종단 연산 중에 forEach는 스트림이 수행한 계산 결과를 보고할 때만 사용해야 한다. 계산할 때는 사용하지 않는다.
- 수집기(collector)를 사용하면 스트림의 원소를 손쉽게 컬렉션에 모을 수 있다.
  - 수집기 팩터리 종류 : toList, toSet, toMap, groupingBy, joining



```java
List<String> topTen = freq.keyset().stream() 
    .sorted(comparing(freq::get).reversedO) 
    .limit(10)
    .collect(toList());

```



- 맵 수집기 이용 : toMap(keyMapper, valueMapper)
  - 스트림 원소를 키에 매핑하는 함수와 값에 매핑하는 함수를 인수로 받는다.


```java
private static final Map<String, Operation> stringToEnum =
    Stream.of(values()).collect(
        toMap(Object::toString, e -> e));
```


- groupingBy 
  - 입력으로 분류 함수(classifier)를 받고 
  - 출력으로는 원소들을 카테고리별로 모아 놓은 맵을 담은 수집기를 반환한다.
  - 분류 함수는 입력받은 원소가 속하는 카테고리를 반환한다.
  - 이 카테고리는 해당 원소의 맵 키(Map key)로 쓰인다


- joining
  - 문자열 등의 CharSequence 인스턴스의 스트림에만 적용할 수 있다.
  - 매개변수가 없는 joining은 단순히 원소들을 연결하는 수집기를 반환한다.
  - 인수 하나짜리 joining은 CharSequence 타입의 구분문자(delimiter)를 매개변수로 받는다.
  - 연결 부위에 구분문자를 삽입하고, 구분문자로 쉼표를 입력하면 CSV 형태의 문자열을 만들어준다.
---
#### 아이템 47. 반환 타입으로는 스트림보다 컬렉션이 낫다
- 스트림은 반복(iteration)을 지원하지 않는다.
- 스트림과 반복을 알맞게 사용하는 것이 좋은 코드를 만들 수 있다.


- Stream 인터페이스는 Iterable 인터페이스가 정의한 추상 메서드를 전부 포함하고, Iterable 인터페이스가 정의한 방식대로 동작한다.
- 하지만 for-each으로 스트림을 반복할 수 없는 이유는 Stream이 Iterable을 확장(extend)하지 않기 때문이다.


- 원소 시퀀스를 반환하는 메서드를 작성할 때는, 스트림으로 처리하기 원하는 사용자와 반복으로 처리하기 원하는 사용자 양쪽 모두 만족시킬 수 있도록 한다.
- 반환 전부터 원소들을 컬렉션에 관리하고 있거나 컬렉션에 하나 더 만들어도 될 정도의 원소 개수가 적으면 ArrayList 같은 표준 컬렉션에 담아서 반환한다.
- 컬렉션 반환이 힘든 경우, Stream이나 Iterable 중 자연스러운 것을 사용한다.
---
#### 아이템 48. 스트림 병렬화는 주의해서 적용하라
- 스트림을 잘못 병렬화하면 프로그램을 오동작하거나 성능이 급격히 저하된다.
- 수정 후에 코드가 정확한지 확인하고 운영 환경과 유사한 조건에서 수행해보며 성능 지표를 관찰해야 한다.
- 결과적으로 계산이 정확하고 성능이 좋아진 것을 확인하고나서 병렬화 버전을 코드에 반영해야 한다.
---
