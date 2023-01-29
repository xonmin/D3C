#### 아이템 17. 변경 가능성을 최소화하라
- 불변 클래스 : 인스턴스 내부 값을 수정할 수 없는 클래스
  - 가변 클래스보다 설계, 구현, 사용이 간단하고 오류가 생길 여지가 적다.
- 자바 플랫폼 라이브러리의 불변 클래스 : String, 기본 타입의 박싱된 클래스, BigInteger, BigDecimal


- 불변 클래스 만드는 5가지 규칙
  - 객체의 상태를 변경하는 메서드(변경자)를 제공하지 않는다.
  - 클래스 확장을 할 수 없게 한다. ex. final
  - 모든 필드를 final으로 선언한다.
  - 모든 필드를 private으로 선언한다.
  - 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다.
    - 클래스에 가변 객체를 참조하는 필드가 있으면, 클라이언트에서 객체를 참조할 수 없게 해야 한다.
    - 생성자, 접근자, readObject 메서드에서 방어적 복사를 수행해라


- 함수형 프로그래밍 
  - 피연산자에 함수를 적용해서 결과를 반환하지만, 피연산자 자체는 그대로인 프로그래밍 패턴
  - 절차적 혹은 명령형 프로그래밍은 메서드에서 피연산자인 자신을 수정해 자신의 상태가 변한다.

    
```java
public final class Complex { 
    private final double re; 
    private final double im;

    public Complex(double re, double im) { 
    this.re = re;
    this.im = im; 
    }

    public double realPartf) { return re; } 
    public double imaginaryPart() { return im; }

    public Complex plus(Complex c) {
        return new Complex(re + c.re, im + c.im);
    }
    
    ...
}
```

- 불변 객체는 근본적으로 스레드 안전하여 따로 동기화가 필요없다.
- 불변 클래스는 한 번 만든 인스턴스를 최대한 재활용한다.
- 가장 쉬운 재활용 방법 : 자주 쓰이는 값들을 상수로 제공한다. ex. public static final


- 불변 클래스는 자주 사용되는 인스턴스를 캐싱해서 같은 인스턴스를 중복 생성하지 않도록 정적 팩터리를 제공할 수 있다.
- 정적 패터리를 사용하면 여러 클라이언트가 인스턴스를 공유하여 메모리 사용량과 가비지 컬렉션 비용을 줄어든다.
- public 생성자 대신에 정적 팩터리를 만들어두면, 클라이언트를 수정하지 않아도 캐시 기능을 덧붙일 수 있다.
- 방어적 복사도 필요 없기 때문에 clone 메서드나 복사 생성자를 제공하지 않는 것이 좋다.


- 불변 객체는 자유롭게 공유할 수 있고, 불변 객채꼐리 내부 데이터를 공유할 수 있다.
- 객체를 만들 때 다른 불변 객체들을 구성요소로 사용하면 이점이 많다.
- 불변 객체는 그 자체로 실패 원자성을 제공한다.
- 단점으로는 값이 다르면 반드시 독립된 객체로 만들어야 한다는 점이다.


- 중간 단계에서 만들어진 객체들이 모두 버려지는 경우 성능 문제
  - 문제 대처 방법 : 다단계 연산(multistep operation) 들을 예측해서 기본 기능으로 제공하는 방법
     - 클라이언트가 원하는 복잡한 연산들을 정확하게 예측하면 package-private의 가변 동반 클래스만으로도 충분하다.


- 모든 클래스를 불변으로 만들 수 없다. 불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분을 최소한으로 줄인다.
- 다른 합당한 이유가 없으면 모든 필드는 private final 이어야 한다.
- 생성자는 불변식 설정이 모두 완료된, 초기화가 완벽히 끝난 상태의 객체를 생성해야 한다.
---
#### 아이템 18. 상속보다는 컴포지션을 사용하라
- 상속 : 클래스가 다른 클래스를 확장하는 구현 상속
- 메서드 호출과 다르게 상속은 캡슐화를 깨뜨린다.
  - 상위 클래스에 따라 하위 클래스의 동작에 이상이 생길 수 있다.


- 잘못된 상속 예시 : addAll 메서드로 원소 3개를 더했을 때


```
InstrumentedHashSet<String> s = new InstrumentedHashSeto();
s.addAlKList.of("틱", "탁탁", "펑,,));
```


- getAddCount 메서드를 호출하면 3을 반환해야 하지만 6을 반환한다.
- HashSet의 addAll 메서드가 add 메서드를 사용해 구현되어서 addCount에 값이 중복해서 더해져서 최종 값이 6으로 늘어났다.
- 하위 클래스에서 addAll 메서드를 재정의하지 않으면 문제를 고칠 수 없다.


```java
public class InstrumentedHashSet<E> extends HashSet<E> {
    private int addCount = 0;
    
    public InstrumentedHashSet() { }
    
    public InstrumentedHashSet(int initCap, float loadFactor) { 
        super(initCap, LoadFactor);
    }
    
    @Override public boolean add(E e) { 
        addCount++;
        return super.add(e); 
    }
    
    @Override public boolean addAll(Collection<? extends E> c) { 
        addCount += c.sizeO;
        return super.addAll(c);
    }
    
    public int getAddCount() { 
        return addCount;
    }
}
```


- 컴포지션(composition) : 기존 클래스를 확장하는 대신, 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하는 방법
- 새 클래스의 인스턴스 메서드들은 기존 클래스의 대응하는 메서드를 호출해서 결과를 반환한다.


- 래퍼 클래스
  - 다른 Set 인스턴스를 감싸고 있는 뜻에서 래퍼 클래스라고 한다.
  - 다른 계측 기능을 덧씌운다는 뜻에서 데코레이터 패턴(decorator pattern) 이라고도 부른다.
  - 단점이 거의 없지만, 래퍼 클래스가 콜백(callback) 프레임워크와는 어울리지 않는다는 점을 주의해야 한다.


```java
public class InstrumentedSet<E> extends Fon시ardingSet<E> {
    private int addCo나nt = 0;
    public InstrumentedSet(Set<E> s) { 
        super(s);
    }
    
    @Override public boolean add(E e) { 
        addCount++;
        return super.add(e); 
    }
    
    @Override public boolean addAll(Collection<? extends E> c) { 
        addCount += c.size();
        return super.addAll(c);
    }
    
    public int getAddCount() { 
    return addCount;
}
```


- 재사용할 수 있는 전달(forwarding) 클래스


```java
public class ForwardingSet<E> implements Set<E> { 
    private final Set<E> s;
    public ForwardingSet(Set<E> s) { this.s = s; }
    
    public void clear() { s.clearO; }
    public boolean contains(Object o) { return s.contains(o); }
    public boolean isEmpty() { return s.isEmpty(); }
    public int size() { return s.size(); }
    public Iterator<E> iterater() { return s.iterator(); }
    public boolean add(E e) { return s.add(e); }
    public boolean remove(Object o) { return s.remove(o); }
    public boolean containsAll(Collection<?> c) { return s.containsAll(c); } 
    public boolean addAll(Collection<? extends E> c){ return s.addAH(c); } 
    public boolean removeAll(Collection<?> c){ return s. removeAH(c); } 
    public boolean retainAlKCollection<?> c){ return s. retainAlKc); } 
    public Object[] toArrayO { return s.toArray(); } 
    public <T> T[] toArray(T[] a) { return s.toArray(a); } 
    @Override public boolean equals(Object o){ return s.equals(o); } 
    @Override public int hashCode() { return s.hashCodef); } 
    @Override public String toString() { return s.toString(); }
}
```

---

