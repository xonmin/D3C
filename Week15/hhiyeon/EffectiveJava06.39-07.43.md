#### 아이템 39. 명명 패턴보다 애너테이션을 사용하라
- 명명 패턴의 단점
  - 실수로 이름을 tsetSafety Override로 지으면 JUnit 3이 메서드를 무시하고 지나치고 테스트가 통과한 것으로 오해할 수 있다.
  - 올바른 프로그램 요소에서만 사용된다는 보증이 없다.
  - 프로그램 요소를 매개변수로 전달할 방법이 없다.
- 애너테이션
  - JUnit 4에서 도입
  - marker 애너테이션 타입 선언 예시


```java
import j;*ava.lang.annotation.*;

/**
* 테스트 메서드임을 선언하는 애너테이션이다. 
* 매개변수 없는 정적 메서드 전용이다.
*/
@Retention (Retent ionPol.icy .RUNTIME) 
@Target(ElementType.METHOD)
public @interface Test {
}
```


- 메타애너테이션(meta-annotation) : 애터테이션 선언에 있는 애너테이션
  - @Test가 런타임에도 유지되어야 한다는 표시
---
#### 아이템 40. @Override 애너테이션을 일관되게 사용하라
- 상위 클래스의 메서드를 재정의하려는 모든 메서드에 @Override 애너테이션을 달아야 한다.
- 예외 : 구체 클래스에서 상위 클래스의 추상 메서드를 재정의 하는 경우
---
#### 아이템 41. 정의하려는 것이 타입이라면 마커 인터페이스를 사용하라
- 마커 인터페이스(marker interface) : 아무 메서드를 담고 있지 않고, 자신을 구현하는 클래스가 특정 속성을 가진 것을 표현해주는 인터페이스
  - ex) Serializable : 자신을 구현한 클래스의 인스턴스를 직렬화(serialization)할 수 있도록 알려준다.

- 마커 인터페이스와 마커 애너테이션
  - 마커 인터페이스는 구현한 클래스의 인스턴스들을 구분하는 타입으로 쓸 수 있다.
  - 마커 애너테이션은 그렇지 않다.
  - 마커 인터페이스는 적용 대상을 더 정밀하게 지정할 수 있다.
  - 마커 애너테이션은 거대한 애너테이션 시스템의 지원을 받는다.
  - 활용 
    - 클래스와 인터페이스 외의 프로그램 요소에 마킹하는 경우 -> 마커 애너테이션 사용
    - 새로 추가하는 메서드 없이 타입 정의가 목적인 경우 -> 마커 인터페이스 사용

- 자바의 직렬화
  - Serializable 마커 인터페이스를 보고 직렬화가 가능한 타입인지 확인
---
### 📌 7장 : 람다와 스트림
#### 아이템 42. 익명 클래스보다는 람다를 사용하라
- 함수 객체(function object)
  - 이전의 자바에서 함수 타입을 표현할 때 추상메서드를 하난만 담은 인터페이스를 사용했다.
  - 이런 인터페이스의 인스턴스를 함수 객체라고 한다.
  - JDK 1.1 부터 익명 클래스를 사용
  - 자바8에서 함수형 인터페이스라 불리는 인터페이스들의 인스턴스를 람다식을 사용해서 만들 수 있게 됬다.
- 익명 클래스 사용


```java
Collections.sort(words, new Comparator<String>() { 
  public int compare(String si, String s2) {
    return Integer.compare(si.length(), s2.length()); 
  }
});
```


- 람다식 사용


```java
Collections. sort(words,
  (si, s2) -> Integer.compare(si.length(), s2.length()));
  
// 비교자 생성 메서드를 사용
Collections.sort(words, comparinglnt(String::length));

// 자바 8 List 인터페이스에 추가된 sort 메서드 사용
words.sort(comparinglnt(String:: length));
```


- 람다와 익명 클래스의 인스턴스를 직렬화하는 일은 하지 않도록 한다.
- 직렬화해야 하는 경우, private 정적 중첩 클래스의 인스턴스를 사용한다.
---
#### 아이템 43. 람다보다는 메서드 참조를 사용하라
- 메서드 참조(method reference)
  - 람다는 익명 클래스보다 간결하다. 메서드 참조는 람다보다 더 간결하게 해준다.


<img width="540" alt="스크린샷 2023-02-26 오후 9 26 14" src="https://user-images.githubusercontent.com/52193680/221410386-533cedfd-2ee3-4336-8622-2b7495b62ce1.png">


---
