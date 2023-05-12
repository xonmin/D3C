# chap 05. 더 좋은 액션 만들기 


냄새 나는 코드 = 중복이 존재하는 코드 

- 비즈니스 요구사항 : 장바구니에 담긴 제품을 주문할 때 무료배송인지 확인하는 것 
- 기존 코드 : 
```js 
function gets_free_shipping(total, item_price) {
  return item_price + total >= 20;
}
```
해당 코드는 요구사항에 맞지 않게 제품의 합계 + 가격으로 확인 중이므로 이를 장바구니라는 엔티티로 변경
- 변경 코드 : 
```js 
function gets_free_shipping(cart) {
  return cart >= 20;
}
```

해당 코드를 사용하는 부분도 변경 
- 기존코드 : 
```js 
function update_shipping_icons() {
 ...
 for (var i = 0; i < buttons.length; i++) {
   var button = buttons[i];
   var item = button.item;
   
   if (gets_free_shipping(   // 기존 사용 코드 
         shopping_cart_total,
         item.price))
        button.show_free_shipping_icon();
    else 
        button.hide_free_shipping_icon(); 
   }
}
```

- 새 시그니처를 적용한 코드 
```js 
function update_shipping_icons() {
 ...
 for (var i = 0; i < buttons.length; i++) {
   var button = buttons[i];
   var item = button.item;
   var new_cart = add_item(shopping_cart, item.name, item.price);
   
   if (gets_free_shipping(new_cart))
        button.show_free_shipping_icon();
    else 
        button.hide_free_shipping_icon(); 
   }
}
```

### 원칙 : 암묵적 입력과 출력은 적을수록 좋다. 
해당 함수에서 암묵적 입력과 출력이 있다면 다른 컴포넌트와 강하게 연결된 컴포넌트 
- 다른 곳에서 사용할 수 없으므로 모듈이 아님 
- 압묵적 입력과 출력이 있는 함수는 아무 때나 실행할 수 없기 때문에 테스트하기 어려움 

### 원칙 : 설계는 엉켜있는 코드를 푸는 것 
- 함수를 통해 관심사를 자연스럽게 분리하도록 하자. 
- 재사용하기 쉽다. 
- 유지보수하기 쉽다
- 테스트하기 쉽다

장바구니에 아이템을 추가하는 함수를 예시로 보자 
```js 
function add_item(cart, name, price) {
  var new_cart = cart.slice(); // 복사 
  new_cart.push({
    name : name,  // item 객체 생성 , push 를 통해 복사본에 아이템 추가 
    price: price,
  }); 
  return new_cart; // 복사본 리턴 
}
```
item 객체 생성 코드 분리하기 
```js
function make_cart_item(name, price) {
  return {
    name: name,
    price: price
  }
}

function add_item(cart, item) {
  var new_cart = cart.slice();
  new_cart.push(item);
  return new_cart;
}

add_item(shopping_cart, make_cart_item("shoe",3.45));
}
```

- item 구조만 알고있는 함수(make_cart_iem)와 cart 구조만 알고 있는 함수(add_item)으로 나눠 고침 
- 복사하는 카피-온-라이트를 구현한 부분이므로 함께 두는 것이 좋다. 

### copy-on-write 패턴 빼내기 
앞선 add_item() 함수는 일반적인 함수이므로 다른 곳에서도 사용할 수 있다. 
- 어떤 배열이든 항목에도 쓸 수 있으므로 범용적으로 변경 
```js
function add_element_last(array, elem) {
 var new_array = array.slice();
 new_array.push(elem)
 return new_array;
}

function add_item(cart, item) {
  return add_element_last(cart, item);
}
```

# chap 06. 변경 가능한 데이터 구조를 가진 언어에서 불변성 유지하기 
nested data - 데이터 구조 안에 데이터 구조가 있는 경우 

### 동작을 읽기, 쓰기 또는 둘 다로 분류하자 
장바구니 동작에서 해당 동작들을 구분하면 다음과 같다. 

읽기 
- 제품 개수 가져오기 
- 제품이름으로 제품 가져오기 

쓰기 
- 제품 추가하기 
- 제품 이름으로 제품 빼기 
- 제품 이름으로 제품 구매 수량 바꾸기 

쓰기 동작은 불변성 원칙에 따라 구현되어야 한다. 
- 이러한 불변성 원칙은 copy-on-write 이라 칭함 

### copy-on-write 원칙 3단계 
1. 복사본 생성
2. 복사본 변경
3. 복사본 리턴 


앞선 `add_element_last(array, item)`은 읽기인가 쓰기인가 
- 기존 데이터를 바꾸지 않고(copy 후 새로운 데이터 리턴) 정보를 리턴하므로 읽기이다

장바구니 제품 빼는 함수를 읽기로 바꾸기 

```js
function remove_item_by_name(cart, name) {
  var idx = null;
  for(cart.length) {
    if(cart[i].name == name) {
      idx = i;
    }
  }
 if(idx !== null) {
  cart.splice(idx, 1); //해당 함수를 통해 장바구니 인덱스에 위치한 아이템 하나 삭제 
 }
}
```

하지만 우리는 장바구니가 바뀌지 않았으면 함. 장바구니를 카피-온-라이트를 통해 변경 불가능한 데이터로 사용해보자

```js
function remove_item_by_name(cart, name) {
  var new_cart = cart.slice();
  var idx = null;
  for(new_cart.length) {
    if(new_cart[i].name == name) {
      idx = i;
    }
  }
 if(idx !== null) {
  new_cart.splice(idx, 1); //해당 함수를 통해 장바구니 인덱스에 위치한 아이템 하나 삭제 
 }
 return new_cart;
}
```

### 불변 데이터 구조를 읽는 것 = 계산 
- 변경 가능한 데이터를 읽는 것 = 액션
- 쓰기는 데이터를 변경 가능한 구조로 만듦
- 어떤 데이터에 쓰기가 없다면 데이터는 변경 불가능한 데이터
- 불변 데이터 구조를 읽는 것은 계산
- 쓰기를 읽기로 바꾸면 코드에 계산이 많아지고 액션이 줄어든다. 

### 불변 데이터 구조는 충분히 빠르다. 
일반적으로 불변 데이터 구조는 변경 가능한 데이터 구조보다 메모리를 더 많이 쓰고 느리지만, 불변 데이터 구조를 통해 대용량의 고성능 시스템을 자주 구현함 
장점 
- 언제든 최적화 가능
- 가비지 콜렉터는 매우 빠르다. 
- 생각보다 많이 복사하지 않는다. 
  - 대부분 얕은 복사가 이루어져 구조적 공유만 하는 상황
- 함수형 프로그래밍 언어에는 빠른 구현체가 존재한다. 

보이러 플레이트 코드를 줄이기 위해 기본적인 배열과 객체 동작에 대한 카피-온-라이트 버전을 만들어주면 좋다. 
