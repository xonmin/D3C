#### 아이템 32. 제네릭과 가변인수를 함께 쓸 때는 신중하라
- 가변 인수는 메서드를 호출하면 가변 인수를 담기 위한 배열이 자동으로 생성된다.
- 배열이 클라이언트에 노출이 되어 매개변수에 제네릭이나 매개변수화 타입이 포함되면 컴파일 경고가 발생하게 된다.
- 매개변수 인수 메서드를 호출할 경우, 매개변수가 실체화 불가 타입으로 추론될 때 경고 형태
  - 매개변수화 타입의 변수가 타입이 다른 객체를 참조하면 힙 오염이 발생한다.

```java
warning: [unchecked] Possible heap pollution from 
    parameterized vararg type List<String>
```


- 자바 7에서 @SafeVarargs 애너테이션 추가 : 메서드 작성자가 타입 안전함을 보장하는 장치
- 제네릭 varargs 매개변수를 안전하게 사용하는 메서드


```java
@SafeVarargs
static <T> List<T> flatten(List<? extends T>... lists) {
    List<T> result = new ArrayListo(); 
    for (List<? extends T> list : lists)
        result.addAlKlist); 
    return result;
}
```

- 제네릭 varargs 메서드가 안전한 기준
  - varargs 매개변수 배열에 아무것도 저장하지 않는다.
  - 그 배열(혹은 복제본)을 신뢰할 수 없는 코드에 노출하지 않는다.
---
#### 아이템 33. 타입 안전 이종 컨테이너를 고려하라
- 


---
### 📌 6장 : 

---