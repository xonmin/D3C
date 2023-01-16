### 📌 4장 : 클래스와 인터페이스
#### 아이템 15. 클래스와 멤버의 접근 권한을 최소화하라
- 잘 설계된 컴포넌트 : 모든 내부 구현을 완벽히 숨겨, 구현과 API를 깔끔하게 분리한다.
- 클래스, 인터페이스, 멤버가 의도치 않게 API로 공개되는 일이 없도록 해야 한다.
- public 클래스에서 public static final 이외에 어떤 public 필드가 있어서는 안된다.
- public static final 필드가 참조하는 객체가 불변인지 확인한다.
- 정보 은닉의 장점 
  - 시스템을 구성하는 컴포넌트들을 서로 독립시켜서 개발, 테스트, 최적화, 적용, 분석, 수정을 개별적으로 할 수 있게 해주는 것과 연관되어 있다.
  - 시스템의 개발 속도를 높인다. (여러 컴포넌트를 병렬로 개발)
  - 시스템 관리 비용을 낮춘다. (각 컴포넌트 교체 부담이 적다.)
  - 성능 최적화에 도움을 준다. 
  - 소프트웨어 재사용성을 높인다. 외부에 거의 의존하지 않고, 독자적으로 동작 가능한 컴포넌트는 낯선 환경에서도 유용하게 쓰일 수 있다.
  - 큰 시스템을 제작하는 난이도를 낮춰준다.


- 접근 제어 메커니즘
  - 클래스, 인터페이스, 멤버의 접근성(접근 허용 범위)을 명시한다.
  - 각 요소의 접근성은 선언된 위치와 접근 제한자로 정해진다.
  - 접근 제한자를 제대로 활용하는 것이 정보 은닉의 핵심


- 기본 원칙
  - 모든 클래스와 멤버의 접근성을 가능한 좁혀야 한다.


- private : 멤버를 선언한 톱레벨 클래스에서만 접근 가능
- package-private : 멤버가 소속된 패키지 안의 모든 클래스에서 접근할 수 있다.
  - 클라이언트에 피해 없이 다음 릴리스에서 수정, 교체, 제거 가능
- protected : package-private의 접근 범위를 포함하며, 이 멤버를 선언한 클래스의 하위 클래스에서도 접근 가능
- public : 모든 곳에서 접근 가능. 공개 API, 하위 호환을 위해 계속 관리 필요
- 상위 클래스의 메서드를 재정의할 때는 접근 수준을 상위 클래스에서보다 좁게 설정할 수 없다.
  - 상위 클래스의 인스턴스는 하위 클래스의 인스턴스로 대체해 사용할 수 있어야 한다.(리스코프 치환 원칙)


- public 클래스의 인스턴스 필드는 되도록 public이 아니어야 한다.
  - 필드가 가변 객체를 참조하거나 final이 아닌 인스턴스 필드를 public으로 선언하면 그 필드에 담을 수 있는 값을 제한할 힘을 잃는다.
  - 꼭 필요한 구성요소의 상수일 경우 public static final 필드로 공개해도 된다.
- 클래스에서 public static final 배열 필드를 두거나 이 필드를 반환하는 접근자 메서드를 제공하면 안된다.
  - 길이가 0이 아닌 배열은 모두 변경이 가능하다.
  - 클라이언트에서 이런 접근자를 제공하면 그 배열의 내용을 수정할 수 있게 되기 떄문이다.


- 첫 번째 방법 : public 배열을 private으로 만들고 public 불변 리스트를 추가한다.
```java
private static final Thing[] PRIVATE_VALUES = { ... }; 
public static final List<Thing> VALUES =
    Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
```

- 두 번째 방법 : 배열을 private 으로 만들고 복사본을 반환하는 public 메서드를 추가(방어적 복사)
```java
private static final Thing[] PRIVATE_VALUES = { ... }; 
public static final Thing[] values() {
    return PRIVATE_VALUES.clone(); }
```
---
#### 아이템 16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라
- 데이터 필드에 직접 접근할 수 있지만, 캡슐화의 이점을 제공할 수 없는 상황


```java
class Point {
    public double x;
    public double y; 
}
```


 필드들을 모두 private으로 바꾸고 public 접근자(getter) 추가
- 접근자와 변경자(mutator) 메서드를 활용해 데이터를 캡슐화한다.


```java
class Point {
    private double x; 
    private double y;
    
    public Point(double xf double y) { 
        this.x = x;
        this.y = y; 
    }

    public double getX() { return x; } 
    public double getY() { return y; }
    
    public void setX(double x) { this.x = x; }
    public void setY(double y) { this.y = y; }
}
```

- package-private 클래스 혹은 private 중첩 클래스에서는 데이터 필드를 노출해도 문제가 없다.
- 패키지 바깥 코드는 변경하지 않아도 데이터 표현 방식을 바꿀 수 있다.
---
#### 아이템 17. 변경 가능성을 최소화하라


---
