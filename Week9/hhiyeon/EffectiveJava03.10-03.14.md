### 📌 3장 : 모든 객체의 공통 메서드
#### 아이템 10. equals는 일반 규약을 지켜 재정의하라
- 아래의 상황 중 하나에 해당하면 재정의하지 않는 것이 좋다.
  - 각 인스턴스가 본질적으로 고유하다.
  - 인스턴스의 '논리적 동치성(logical equality)'을 검사할 일이 없다.
  - 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다.
  - 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다.

  
- 값 클래스 = Integer와 String 처럼 값을 표현하는 클래스
- equals를 재정의해야 하는 경우?
  - 주로 값 클래스가 해당된다.
  - 값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장하는 인스턴스 통제 클래스면 equals 재정의를 하지 않아도 된다.


- Object 명세 규약
  - equals 메서드가 쓸모 있으려면 모든 원소가 같은 동치류에 속한 어떤 원소와도 서로 교환할 수 있어야 한다.
    - 반사성(reflexivity) : 자기 자신과 같아야 한다.
    - 대칭성(symmetry) : 두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다.
    - 추이성(transitivity) : 첫 번째 객체와 두 번째 객체가 같고, 두 번째 객체와 세 번째 객체가 같으면, 첫 번째 객체와 세 번째 객체도 같아야 한다.
    - 일관성(consistency) : 두 객체가 같으면 앞으로도 영원히 같아야 한다.
    - null-아님
---
#### 아이템 11. equals를 재정의하려거든 hashCode도 재정의하라
- hashCode도 재정의하지 않으면, hashCode 일반 규약을 어기게 되어 해당 클래스의 인스턴스를 컬렉션의 원소로 사용할 때 문제가 생긴다.
- Object 명세 규약
  - equals 비교에 사용되는 정보가 변경되지 않았으면, 애플리케이션이 실행되는 동안 그 객체의 hashCode 메서드는 몇 번 호출해도 일관되게 항상 같은 값을 반환해야 한다.
  - equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다.
  - equals(Object)가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다.
- hashCode 재정의를 잘못하게 되면, 2번째 조항에 문제가 생긴다.
- 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.
- AutoValue 프레임워크를 사용하면 equals, hashCode를 자동으로 만들어준다.
---
#### 아이템 12. toString을 항상 재정의하라
- toString 일반 규약 
  - 간결하면서 사람이 읽기 쉬운 형태의 유익한 정보를 반환해야 한다.
  - 모든 하위 클래스에서 이 메서드를 재정의하라고 한다.


- 실전에서 toString은 그 객체의 모든 주요 정보를 반환하는 게 좋다. ex. 전화번호
- toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공하자.
- 모든 구체 클래스에서 Object의 toString을 재정의하자. (상위 클래스에서 이미 알맞게 재정의한 경우는 예외)
---
#### 아이템 13. clone 재정의는 주의해서 진행하라
- Cloneable : 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스(mixin interface)
- 문제점 : clone 메서드가 선언된 곳이 Cloneable이 아닌 Object + protected
  - Cloneable을 구현한다고 외부 객체에서 clone 메서드를 호출할 수 없다.
- Cloneable 인터페이스는 Object의 protected 메서드인 clone 동작 방식을 결정한다.
- 인스턴스에서 clone을 호출하면 그 객체의 필드들을 복사한 객체를 반환하고, 그렇지 않은 클래스의 인스턴스에서 호출하면 CloneNotSupportedException을 던진다.
- 실무에서 Cloneable을 구현한 클래스는 clone 메서드를 public으로 제공하고, 사용자는 당연히 복제가 제대로 이뤄지 않아야 한다.
- clone 메서드는 사실상 생성자와 같은 효과를 낸다.
- clone은 원본 객체에 아무런 해를 끼치지 않고 복제된 객체의 불변식을 보장해야 한다.
```java
@Override public Stack cloneO { 
  try {
    Stack result = (Stack) super.clone(); 
    result.elements = elements.clone(); 
    return result;
  } catch (CloneNotSupportedException e) { 
    throw new AssertionError();
  } 
}
```


```java
Entry deepCopy() {
  return new Entry(key, value,
    next = null ? null : next.deepCopy());
}
```


- 연결리스트를 복제하는 방법으로, deepCopy를 사용하는 것은 좋지 않다.
- 재귀 호출 때문에 리스트의 원소 수만큼 스택 프레임을 소비하기 때문에 리스트가 길면 스택 오버플로를 일으킬 위험이 있다.
- 이 문제를 피하려면, deepCopy를 재귀 호출 대신 반복자를 써서 순회하는 방법으로 수정한다.


```java
Entry deepCopy() {
  Entry result = new Entry(key, value, next);
  for (Entry p = result; p.next != null; p = p.next)
    p.next = new Entry(p.next.key, p.next.value, p.next.next); 
  return res나It;
}
```


- 생성자에서는 재정의 될 수 있는 메서드를 호출하지 않아야 한다.
  - clone 메소드에서도 하위 클래스에서 재정의한 메소드를 호출하면, 하위 클래스는 복제 과정에서 자신의 상태를 수정할 기회가 없어지기 때문에 원본과 복제본의 상태가 달라질 수 있다.
  - put(key, value) 메서드는 final 혹은 private 이어야 한다.


- 상속용 클래스는 Cloneable을 구현하면 안된다.
```java
@Override
protected final Object cloneO throws CloneNotSupportedException {
   throw new CloneNotSupportedException(); 
}
```

- 기본 원칙은 '복제 기능은 생성자와 팩터리를 이용하는게 좋다.'
- 배열은, clone 메서드 방식이 가장 깔끔하다.
---
#### 아이템 14. Comparable을 구현할지 고려하라
- Comparable 인터페이스의 메서드 compareTo
- equals와의 차이점은?
  - compareTo는 단순 동치성 비교에 더해 순서 비교도 가능하고, 제너릭하다.


- compareTo는 equals와 다르게, 타입이 다른 객체를 신경쓰지 않아도 된다.
- 타입이 다르면 ClassCastException을 던져도 된다.
- 비교를 활용하는 클래스의 예시
  - 정렬된 컬렉션 : TreeSet, TreeMap
  - 검색과 정렬 알고리즘을 활용하는 유틸리티 클래스 : Collections, Arrays


- compareTo 메서드 작성 요령
  - Comparable은 타입을 인수로 받는 제네릭 인터페이스 이기 때문에, compareTo 메서드의 인수 타입은 컴파일 타임에 정해진다.
  - null을 인수로 넣어 호출하면 NullPointerException을 던져야 한다.
  - 각 필드가 동치인지 비교하는 게 아니라 순서를 비교한다.
  - compareTo 메서드에서 필드 값을 비교할 때 <와 > 연산자는 쓰지 말아야 한다.
  - 그 대신 박싱된 기본 타입 클래스가 제공하는 정적 compareTo 메서드나 Comparator 인터페이스가 제공하는 비교자 생성 메서드를 사용한다.
  - 핵심 필드가 여러개인 경우, 중요 필드부터 비교한다.


```java
public int compareTo(PhoneNumber pn) {
  int result = Short.compare(areaCode, pn.areaCode); // 가장 중요
  if (result = 0) {
    result = Short.compare(prefix, pn.prefix); // 두 번째로 중요
    if (result = 0)
      result = Short.compare(lineNumf pn.lineNum); // 세 번째로 중요
  }
  return result; 
}
```


- 해시코드 값의 차를 기준으로 하는 비교 : 추이성 위배


```java
static Comparator<Object> hashCodeOrder = new Comparatero() { 
  public int compare(Object ol, Object o2) {
    return ol.hashCode() - o2.hashCode(); 
    }
};
```


- 정적 compare 메서드를 활용한 비교자


```java
static Comparator<0bject> hashCodeOrder = new Comparatoro() { 
  public int compare(Object ol, Object o2) {
    return Integer.compare(ol.hashCode(), o2.hashCode()); 
    }
};
```


- 비교자 생성 메서드를 활용한 비교자


```java

static Comparator<Object> hashCodeOrder = 
  Comparator.comparinglnt(o -> o.hashCode());
```
