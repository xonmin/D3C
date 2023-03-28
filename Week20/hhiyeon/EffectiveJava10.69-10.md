### 📌 10장 : 예외
#### 아이템 69. 예외는 진짜 예외 상황에만 사용하라
- 잘못된 예외상황 : 무한루프를 돌다가 배열에 끝에 도달해서 ArrayIndexOutOfBoundException 발생했을 때 종료


```java
try {
    int i = 0;
    while(true)
        range[i++].climb();
} catch(ArrayIndexOutOfBoundsException e){
}
```


- JVM은 배열에 접근할 때마다 경계가 넘는지 검사한다.
- 위의 코드에서 같은 일(경계가 넘는지 확인)이 반복된다.
- 예외를 사용 vs 표준 관용구 : 예외를 사용한 것이 속도가 더 느리다.
- 예외는 예외 상황에서만 사용하고, 일상적인 제어 흐름용에서는 쓰지 않는다.
---
