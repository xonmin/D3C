#### 아이템 19. 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라
- 상속용 클래스는 재정의할 수 있는 메소드들을 내부적으로 어떻게 이용하는지 문서로 남겨야 한다.
- 내부 메커니즘을 문서로 남기는 것이 상속을 위한 설계의 전부는 아니다.
- 클래스의 내부 동작 과정 중간에 끼어들 수 있는 훅(hook)을 잘 선별하여 protected 메서드 형태로 공개해야 할 수도 있따.
- 상속용 클래스를 시험하는 방법은 직접 하위 클래스를 만들어보는 것이 유일하다.


- clone과 readObject 모두 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안된다.
- clone이 잘못되면 복제본말고도 원본 객체에도 피해를 줄 수 있다.
- 상속용으로 설계하지 않은 클래스는 상속을 금지하는 것이 나을 수 있다.


- 상속을 금지하는 방법 2가지
  - 클래스를 final으로 선언하는 방법
  - 모든 생성자를 private이나 package-private으로 선언하고 public 정적 팩터리를 만들어주는 방법
    - 생성자 모두를 외부에서 접근할 수 없도록 만든다.
---
#### 아이템 20. 추상 클래스보다는 인터페이스를 우선하라
- 자바가 제공하는 다중 구현 메커니즘 : 인터페이스, 추상클래스
- 두 메커니즘 공통점 : 인스턴스 메서드를 구현 형태로 제공 가능
- 차이점 : 추상 클래스가 정의한 타입을 구현하는 클래스는 반드시 추상 클래스의 하위 클래스가 되어야 한다.
  - 추상 클래스 방식은 새로운 타입을 정의하는 데 커다란 제약이 생긴다.
  - 인터페이스 방식은 선언한 메서드를 모두 정의하고 일반 규약을 잘 지킨 클래스라면 어떤 클래스를 상속해도 같은 타입으로 취급된다.


- 기존 클래스에도 손쉽게 새로운 인터페이스를 구현해 넣을 수 있다.
- 인터페이스가 요구하는 메서드를 추가하거나 클래스 선언에 implements 구문만 추가하면 된다.
- 추상 클래스 방식은 클래스 계층 구조에 혼란을 일으켜 새로 추가된 추상 클래스의 모든 자손이 상속을 하게 되는 문제가 생길 수 있다.


- 인터페이스는 믹스인(mixin) 정의에 안성맞춤이다.
- 믹스인 : 클래스가 구현할 수 있는 타입. 
  - 믹스인을 구현한 클래스에 원래의 '주된 타입' 외에도 특정 선택적 행위를 제공한다고 선언하는 효과를 준다.
  - 대상 타입의 주된 기능에 선택적 기능을 혼합한다고 해서 믹스인이라고 부른다.


- 인터페이스로 계층 구조가 없는 타입 프레임워크를 만들 수 있다.
- 타입을 계층적으로 정의하면 개념을 구조적으로 잘 표현 가능하지만, 표현이 어려운 개념도 있다.


```java
public interface Singer { 
    AudioClip sing(Song s);
}

public interface Songwriter {
    Song compose(int chartPosition);
}
```

- 작곡도 하는 가수가 있기 때문에 가수 클래스가 Singer와 Songwriter 모두를 구현해도 문제가 되지 않는다.
- 가능한 조합 전부를 각각의 클래스로 정의한 복잡한 계층 구조가 만들어질 수 있다.
- 래퍼 클래스 관용구와 함께 사용하면 인터페이스는 기능을 향상시키는 안전하고 강력한 수단이 된다.


- 인터페이스와 추상 골격 구현(skeletal implementation) 클래스를 함께 제공하는 방법으로 인터페이스와 추상 클래스 장점을 모두 취하는 방법도 있다.
- 템플릿 메서드 패턴 : 인터페이스로 타입 정의, 필요하면 디폴트 메서드도 제공, 골격 구현 클래스는 나머지 메서드까지 구현한다.
- 인터페이스 이름이 Interface 이면, 골격 구현 클래스 이름은 AbstractInterface
- 골격 구현을 사용해 완성한 구체 클래스 예시


```java
static List<Integer> intArrayAsList(int[] a) {
    Objects.requireNonNull(a);
    
    // 다이아몬드 연산자를 이렇게 사용하는 건 자바 9부터 가능하다. 
    // 더 낮은 버전을 사용한다면 <Integer>로 수정하자.
    return new AbstractListo() {
        @Override public Integer get(int i) { 
            return a[i]; // 오토박싱(아이템 6)
        }

        @Override public Integer set(int i, Integer val) { 
            int oldVal = a[i];
            a[i] = val; // 오토언박싱
            return oldVal; // 오토박싱
        }
        
        @Override public int size() { 
            return a.length;
        } 
    };
}
```
---
#### 아이템 21. 인터페이스를 구현하는 쪽을 생각해 설계하라
- 자바 8 이전에는 추가된 메서드가 우연히 기존 구현체에 이미 존재할 가능성이 낮기 때문에, 기존 구현체를 깨뜨리지 않으면 인터페이스에 메소드를 추가할 방법이 없었다.
- 자바 8 이후부터는 기존 인터페이스에 메서드를 추가할 수 있도록 디폴트가 생겼지만, 위험이 아예 없어진 것은 아니다.
  - 디폴트 메서드를 선언하면, 인터페이스 구현 후 디폴트 메서드를 재정의하지 않은 모든 클래스에서 디폴트 구현이 쓰이게 된다.
  - 모든 기존 구현체들이 매끄럽게 연동되는 보장이 없다.
- 생각할 수 있는 모든 상황에서 불변식을 해치지 않는 디폴트 메서드 작성은 어렵다.
- 디폴트 메서드는 컴파일은 성공해도, 기존 구현체에 런타임 오류를 일으킬 수 있다.
- 기존 인터페이스에 디폴트 메서드로 새 메서드를 추가하는 일은 꼭 필요한 것이 아니면 피해야 한다.
- 새로운 인터페이스를 만드는 경우에는 표준적인 메서드 구현을 제공하는 데 유용한 수단이 된다.
---
#### 아이템 22. 인터페이스는 타입을 정의하는 용도로만 사용하라
- 인터페이스는 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할
- 자신의 인스턴스로 무엇을 할 수 있는지 클라이언트에게 알려줄 수 있다.
- 지침에 맞지 않는 예시 : static final 필드로만 가득찬 인터페이스


```java
public interface PhysicalConstants {
    // 아보가드로 수 (1/몰)
    static final double AVOGADROS_NUMBER = 6.022_140_857e23; 
    
    // 볼츠만 상수 (J/K)
    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
    
    // 전자 질량 (kg)
    static final double ELECTRON_MASS = 9.109_383_56e-31; 
}
```
---
#### 아이템 23. 태그 달린 클래스보다는 클래스 계층구조를 활용하라
- 태그가 달린 클래스 


```java
class Figure {
    enum Shape { RECTANGLE, CIRCLE };
    
    // 태그 필드 - 현재 모양을 나타낸다.
    final Shape shape;
    
    // 다음 필드들은 모양이 사각형(RECTANGLE)일 때만 쓰인다. 
    double length;
    double width;
    
    // 다음 필드는 모양이 원(CIRCLE)일 때만 쓰인다. 
    double radius;
    
    // 원용 생성자
    Figure(double radius) { 
        shape = Shape.CIRCLE; 
        this.radius = radius;
    }
    
    // 사각형용 생성자
    Figure(double length, double width) { 
        shape = Shape.RECTANGLE; 
        this.length = length;
        this.width = width;
    }
    
    double area() { 
        switch(shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
```


- 태그가 있는 클래스 단점
  - 열거 타입 선언, 태그 필드, switch 문 등등 쓸데없는 코드가 너무 많아 가독성이 나쁘다.
  - 많은 메모리 사용
  - 필드를 final로 선언하려면 쓰이지 않는 필드도 생성자에서 초기화해야 한다.
  - 태그 달린 클래스는 장황하고, 오류가 생기기 쉽고, 비효율적이다.


- 자바와 같은 객체지향 언어에서는 타입 하나로 다양한 의미의 객체를 표현하는 나은 수단을 제공한다.
  - 클래스 계층 구조를 활용하는 서브타이핑(subtyping)


- 태그 달린 클래스를 클래스 계층 구조로 변환


```java
abstract class Figure { 
    abstract double area();
}

class Circle extends Figure { 
    final double radius;
    
    Circle(double radius) { this.radius = radius; }
    
    @Override double area() { return Math.PI * (radius * radius); } 
}

class Rectangle extends Figure { 
    final double length;
    final double width;
    
    Rectangle(double length, double width) { 
        this.length = length;
        this.width = width;
    }

    @Override double area() { return length * width; } 
}
```
---
#### 아이템 24. 멤버 클래스는 되도록 static으로 만들라
- 중첩 클래스(nested class) 
  - 다른 클래스 안에 정의된 클래스
  - 자신을 감싼 바깥 클래스에서만 사용해야 한다.
  - 종류 : 정적 멤버 클래스. (비정적) 멤버 클래스, 익명 클래스, 지역 클래스
  - 정적 멤버 클래스를 제외한 나머지는 내부 클래스(inner class)


- 정적 멤버 클래스
  - 다른 클래스 안에 선언되고, 바깥 클래스의 private 멤버에 접근할 수 있다.
  - 바깥 클래스와 함께 쓰일 때 유용한 public 도우미 클래스로 쓰인다.
  - 중첩 클래스의 인스턴스가 바깥 인스턴스와 독립적으로 존재할 수 있으면 정적 멤버 클래스로 만들어야 한다.


- 비정적 멤버 클래스
  - 비정적 멤버 클래스의 인스턴스 메서드에서 정규화된 this를 사용해 바깥 인스턴스의 메서드를 호출하거나 바깥 인스턴스의 참조를 가져올 수 있다.
  - 정규화된 this : 클래스명.this
  - 바깥 인스턴스 없이 생성할 수 없다.
  - 비정적 멤버 클래스의 인스턴스 안에 관계 정보가 만들어져 메모리 공간을 차지하고, 생성 시간이 더 걸린다.
  - 어댑터 정의에 주로 사용된다.
  - 멤버 클래스에서 바깥 인스턴스에 접근할 일이 없으면 정적 멤버 클래스로 만들어준다.


- 익명 클래스
  - 쓰이는 시점에 선언과 동시에 인스턴스가 만들어진다.
  - 선언한 지점에서만 인스턴스를 만들 수 있고, 클래스 이름이 필요한 작업은 수행할 수 없다.
  - 여러 인터페이스를 구현할 수 없고, 인터페이스를 구현하는 동시에 다른 클래스를 상속할 수 없다.
  - 주로 사용되는 경우는 팩터리 메서드 구현


- 지역 클래스
  - 가장 드물게 사용되는 중첩 클래스
  - 지역변수를 선언할 수 있는 곳이면 어디서든 선언 가능
  - 유효 범위도 지역변수와 같다.
  - 멤버 클래스처럼 이름이 있고 반복해서 사용 가능
  - 익명 클래스처럼 비정적 문맥에서 사용될 때만 바깥 인스턴스를 참조할 수 있고, 정적 멤버는 가질 수 없고, 가독성을 위해 짧게 작성해야 한다.
---
#### 아이템 25. 톱레벨 클래스는 한 파일에 하나만 담으라
- 컴파일러가 한 클래스에 대한 정의를 여러 개 만들지 않도록, 소스 파일 하나에 톱레벨 클래스를 하나만 담는다.
---
