# 신뢰할 수 없는 코드를 쓰면서 불변성 지키기 
- 레거시 코드나 신뢰할 수 없는 코드로부터 내 코드 보호를 위한 방어적 복사
- 얕은 복사와 깊은 복사 비교
- 카피 온 라이트와 방어적 복사의 사용 시기 

## 방어적 복사 
방어적 복사 : 카피-온-라이트 규칙을 지키면서 안전하게 함수 사용할 수 있는 원칙 
- 신뢰할 수 없는 코드에 데이터 참조 및 데이터의 유입은 언제나 잠재적으로 바뀔 수 있다. 
- 기존의 카피-온-라이트 패턴은 데이터를 바꾸기 전에 복사한다. 
- 하지만, 레거시 코드에서는 무엇을 복사해야 할지 예상할 수 없으며 어떤 일이 일어날지 알 수 없음 
- 따라서 데이터가 바뀌는 것을 완벽히 막아주는 원칙이 필요하다 **(방어적 복사)**

방어적 복사는 원본이 바뀌는 것을 막아준다. 

레거시 코드에서 데이터 인입 상황 
1. 신뢰할 수 없는 코드의 데이터 인입
2. 들어온 데이터로 깊은 복사본을 만들고, 변경 가능한 원본 데이터는 버린다. 
3. 신뢰할 수 있는 코드만 복사본을 사용하므로 데이터는 바뀌지 않는다(데이터 보호) 

반대로 레거시 코드로 데이터를 보낼 때, 나가는 데이터도 바뀌면 인된다. 
1. 나가는 데이터에 대해서도 내보내기 전에 깊은 복사를 진행
2. 깊은 복사본의 데이터를 내보내 안전한 코드에서의 데이터가 변경되지 않도록 함 

> 방어적 복사를 통해 불변성을 지킬 수 있다. 

예시 코드 
```js
...
shopping_cart = add_item(shopping_cart, item);
...
black_friday_promotion(shopping_cart); // legacy code
```

방어적 복사 진행 
```js
...
shopping_cart = add_item(shopping_cart, item);
...
var cart_copy = deepCopy(shopping_cart);  // 넘기기 전에 복사 
black_friday_promotion(cart_copy); 
shopping_cart = deepCopy(cart_copy); // 들어오는 데이터를 위한 복사 
```

방어적 복사 규칙 정리 
- 규칙 1 : 데이터가 안전한 코드에서 나갈 때 복사하기 
  - 불변성 데이터를 위한 깊은 복사본을 생성
  - 신뢰할 수 없는 코드로 복사본을 전달합니다. 
- 규칙 2 : 안전한 코드로 데이터가 들어올 때 복사하기
  - 변경될 수도 있는 데이터가 들어오면 바로 깊은 복사본을 만들어 안전한 코드로 전달
  - 복사본을 안전한 코드에서 사용합니다. 
+ 방어적 복사와 관련된 코드를 wrapping 하면 더 좋은 코드가 될 것 


ex_ 공유 라이브러리를 만들거나 신뢰할 수 없는 라이브러리를 사용할 때 

### 신뢰할 수 없는 코드 wrapping 
앞선 코드에서 레거시 코드와 방어적 복사를 wrapping 
```js 
function add_item_to_cart(name, price) {
...
shopping_cart = black_friday_promotion_safe(shopping_cart);
}

function black_friday_promotion_safe(cart) { 
  var cart_copy = deepCopy(cart);
  black_friday_promotion(cart_copy);
  return deepCopy(cart_copy);
}
```

대부분의 Web API는 암묵적 방어적 복사 진행
- JSON 데이터는 깊은 복사본 
- return 에 대해서도 JSON 은 깊은 복사본 

### copy-on-write 과 방어적 복사 비교 
| |copy-on-write|defensive copy|
|:---:|:---:|:---:|
|when|통제할 수 있는 데이터 변경시 | 신뢰할 수 없는 코드와 데이터 상호작용|
|where|안전한 코드 내 어디서든 | 안전한 코드 경계에서 데이터 오고 갈 때 |
|copy| 얕은 방식 | 깊은 복사 |
|규칙|1. 얕은 복사 생성| 1. 들어온 데이터 깊은 복사 생성|
|   |2. 복사본 변경 | 2. 나갈 데이터 깊은 복사 생성 | 
|   |3. 복사본 리턴 | |

