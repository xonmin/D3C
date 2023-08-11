# chapter 10. 일급함수 I 
코드의 냄새와 중복을 없애 추상화를 잘할 수 있는 리팩토링 두 개를 알아보자 

### 코드의 냄새 : 함수 이름에 있는 암묵적 인자 
- 해당 냄새는 일급 값으로 바꾸면 표현력이 좋아진다.
- 함수 본문에서 사용하는 어떤 값이 함수 이름에 나타난다면 **함수 이름에 있는 암묵적 인자**는 코드의 냄새가 된다.

**특징**
1. 거의 똑같이 구현된 함수가 있다.
2. 함수 이름이 구현에 있는 다른 부분을 가리킨다.

### 리팩터링: 암묵적 인자를 드러내기 
해당 리팩토링은 암묵적 인자가 일급 값이 되도록 함수에 인자를 추가한다. 
- 이를 통해 잠재적 중복을 없애고 코드의 목적을 더 잘 표현이 가능하다.

step
1. 함수 이름에 있는 암묵적 인자를 확인
2. 명시적 인자 추가
3. 함수 본문에 하드코딩된 값을 새로운 인자로 바꿈
4. 함수를 호출하는 곳 수정

### 함수 본문을 콜백으로 바꾸기 
비슷한 함수에 있는 서로 다른 부분을 콜백으로 바꾼다. 
- 이를 통해 일급 함수로 동작을 전달이 가능

step
1. 함수 본문에서 바꿀 부분의 앞/뒷 부분을 확인
2. 리팩터링할 코드를 함수로 extract
3. 빼낸 함수의 인자로 넘길 부분을 또 다른 함수로 추출

```js
function setPriceByName(cart, name, price) {
    var item = cart[name];
    var newItem = objectSet(item, 'price', price);
    var newCart = objectSet(cart, name, newItem);
    return newCart;
}
 
function setQuantityByName(cart, name, quant) {
    var item = cart[name];
    var newItem = objectSet(item, 'quantity', quant);
    var newCart = objectSet(cart, name, newItem);
    return newCart;
}
 
function setShippingByName(cart, name, ship) {
    var item = cart[name];
    var newItem = objectSet(item, 'shipping', ship);
    var newCart = objectSet(cart, name, newItem);
    return newCart;
}
 
function setTaxByName(cart, name, tax) {
    var item = cart[name];
    var newItem = objectSet(item, 'tax', tax);
    var newCart = objectSet(cart, name, newItem);
    return newCart;
}
 
function objectSet(object, key, value) {
    var copy = Object.assign({}, object);
    copy[key] = value;
    return copy;
}
```
`set[   ]ByName` 의 이름을 갖는 함수들의 로직은 동일합니다. 대상만 다를 뿐입니다. 이렇게 구체적인 함수들은 일반적인 함수를 선언하여 리팩토링할 수 있습니다.

### 리팩토링된 코드 
```js
function setFieldByName(cart, name, field, value) {
    var item = cart[name];
    var newItem = objectSet(item, field, value);
    var newCart = objectSet(cart, name, newItem);
    return newCart;
}
 
cart = setFieldByName(cart, "shoe", 'price', 13);
cart = setFieldByName(cart, "shoe", 'quantity', 3);
cart = setFieldByName(cart, "shoe", 'shipping', 0);
cart = setFieldByName(cart, "shoe", 'tax', 2.34);
```
비슷한 함수를 모두 일반적인 함수 하나로 바꾸었다. 
- 리팩토링으로 필드명을 일급 값으로 만들었고, 이제 암묵적인 이름은 인자로 넘길 수 있는 값이 되었다.
- 값은 변수나 배열에 담을 수 있어 그래서 일급이라고 부른다.
- 일급 값은 언어 전ㄴ체에 어디서나 쓸 수 있음
- 하지만, 필드명을 문자열로 넘기는 것은 안전하지 않다고 느낄지도 모른다.

## 일급인 것과 일급이 아닌 것을 구별하기 
- 함수에 인자로 넘길 수 있고, 함수의 리턴값으로 받을 수 있는 것 -> 이런 식으로 쓸 수 있는 값을 일급이라 칭함
- 일급이 아닌 것
  - 언어마다 상의하지만, JS 기준
  - 수식 연산자 / 반복문 / 조건문 / try-catch ...
 
필드명을 문자열로 사용하면 위험도가 있지 않을까? 
- Enum 타입과 같은 타입 시스템을 통해 오류를 줄이도록 한다.

일급 필드를 사용하면 API를 더욱 바꾸기 어렵지 않을까? 
- 앤타타 필드명을 일급으로 만들어 사용한다면 세부 구현을 밖으로 노출하는 겪일 수 있지 않을까란 고민이 있을 수 있음
- 이는 추상화 벽의 원칙을 위반하는 것이 아닌지?

A. 필드명을 계속 유지해야하지만, 구현이 외부에 노출된 것은 아니다. 
- 만약 내부에 정의한 필드명이 바뀐다고해도, 사용하는 쪽에서는 원래 필드명을 그대로 사용하고 내부에서 그냥 바꿔주면 된다.

### 객체와 배열을 너무 많이 쓰게 됩니다. 
- 데이터를 사용할 때 임의의 인터페이스로 감싸지 않고 그대로 사용하고 있다는 점
- 데이터를 데이터 그대로 사용하는 것의 중요한 장점은 여러가지 방법으로 해석할 수 있다.
- 제한된 API로 정의하면 데이터를 제대로 활용할 수 없음
- 데이터는 미래에 어떤 방법으로 해석될지 모르기 때문에, 필요할 떄 알맞은 방법으로 해석할 수 있어야 한다

> 이것이 데이터 지향이라고 하는 중요한 원칙

### 고차 함수 
- 인자로 함수를 받거나 리턴 값으로 함수를 리턴할 수 있는 함수
