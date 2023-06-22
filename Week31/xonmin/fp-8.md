# chap.08 계층형 설계 

계층형 설계 : 소프트웨어를 계층으로 구성하는 기술 
계층형 설계 패턴 : 
 - 패턴 1 : 직접 구현
   - 직접 구현된 함수를 읽을 때, 함수 시그니처가 나타내고 있는 문제를 함수 본문에서 적절한 구체화 수준에서 해결해야 한다. 
   - 너무 구체적이라면 냄새나는 코드
 - 패턴 2 : 추상화 벽
   - 중요한 세부 구현을 감추고 인터페이스를 통해 제공한다. 
   - 인터페이스를 통해 코드를 만들면 더 높은 차원으로 생각할 수 있음 
   - 고수준의 추상화 단계만 생각하면 되기 때문에 좋다 
 - 패턴 3 : 작은 인터페이스
   - 시스템이 커질수록 비즈니스 개념을 나타내는 중요한 인터페이스는 작고 강력한 동작으로 구성하는 것이 좋다. 
   - 다른 동작도 직간접적으로 최소한의 인터페이스를 유지하면서 정의해야 한다. 
 - 패턴 4 : 편리한 계층 
   - 계층형 설계는 개발자의 요구 만족과 비즈니스 문제를 모두 풀 수 있어야 함 
   - 계층 추가는 코드와 그 코드가 속한 추상화 게층이 작업할 때 편리할 수 있을 때 추가하는 것 

### 패턴 1 : 직접 구현 
- 같은 추상화 단계에 있는 function 을 쓰는 것이 직접 구현 패턴 
- 같은 계층에 있는 함수는 같은 목적을 가져야 한다. (추상화 정도가 같아야 함) 
- 만약 중간의 여러 계층을 한 번에 뛰어넘는 호출이 있다면 그것은 추상화 정도가 잘못된 가까운 추상화 function을 부르는 것이 아니다.

as-is
```js 
function remove_item_by_name(cart, name) {
 var idx = null; 
 for (var i =0; i < cart.length; i ++) {
   if(cart[i].name === name)
     idx = i;
  }
  if(idx !== null) 
    return removeItems(cart, idx, 1);
    
  return cart;
}
```
to-be
```js
function remove_item_by_name(cart, name) {
  var idx = indexOfItem(cart, name);
  
 if(idx !== null) 
    return removeItems(cart, idx, 1);
    
  return cart;
 }
 
function indexOfItem(cart, name) {
 for (var i =0; i < cart.length; i ++) {
   if(cart[i].name === name)
     return i;
  }
  return null;
}
```

- 직접 구현한 코드는 한 단계의 구체화 수준에 관한 문제만 해결 
- 계층형 설계는 특정 구체화 단계에 집중할 수 있게 도와준다. 
- 함수를 추출하면 더 일반적인 함수로 만들 수 있다. 
- 일반적인 함수가 많을수록 재사용에 용이하다. 
- 복잡성을 감추지 않는다. 
  
