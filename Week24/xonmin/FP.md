
# chap 03. 액션과 계산, 데이터의 차이 

개발 과정 내 해당 세가지를 구분하는 기술 

### 1. 문제를 생각할 때 
- 문제를 액션과 계산, 데이터로 나눠보고 특별히 주의해야 할 부분(액션), 데이터로 처리해야 할 부분, 결정을 내려야 하는 부분(계산) 

### 2. 코딩할 때 
- 최대한 액션에서 계산을 빼내자
- 또 계산에서는 데이터를 분리할 수 있는 지 확인
- 더 나아가 액션이 계산이 될 수 있는지, 계산이 데이터가 될 수 있는 지 확인

### 3.코드를 읽을 때 
- 해당 코드가 액션과 계산, 데이터 중 어떤 것에 속하는 지 살펴보기 
- 특히 액션은 시간에 의존적이므로 조심 

## 유의점 
- 액션과 계산, 데이터는 어디에나 적용할 수 있음
- 액션 안에는 계산과 데이터 또 다른 액션이 숨어 있을 수 있음
- 계산은 더 작은 계산과 데이터로 나누고 연결할 수 있음
- 데이터는 데이터만 조합할 수 있음
- 계산은 때로 우리 머리속에서 저절로 일어나는 점 유의 

일반적인 구현 순서
- 데이터는 사용하는 데 제약이 많고, 액션은 가장 제약이 없음 
1. 데이터
2. 계산
3. 액션 


- 함수 내에서 액션을 사용하면 그 함수는 전체가 액션이다. 
- 결국 액션은 전파되므로 액션을 쓰는 순간 코드 전체로 퍼져나가기 때문에 주의 

## 액션의 다양한 형태 
- 함수 호출 -> `alter("Hello world")` 팝업 창 
- 메서드 호출 -> `console.log()` 콘솔 출력
- 생성자 -> `ex_ new Date()` : 부르는 시점에 시점 초기화 
- 표현식 
  - 변수참조 `y` (읽는 시점에 따라 달라질 수 있음) 
  - 속성참조 `user.first_name` (변경가능한 객체라면 속성 또한 값이 달라질 수 있음)
  - 배열참조 `stack[0]` 변경가능한 배열 
- 상태
  - 값 할당 `z=3;` 다른 곳에서 사용한다면 다른 코드에 영향을 줄 수 있다
  - 속성 삭제 : `delete user.first_name` : 다른 코드에 영향을 줌 

### 액션을 잘 사용하는 방법
- 가능한 액션을 적게 사용
- 액션은 가능한 작게 만듦
- 액션이 외부 셰게와 상호작용하는 것을 제한
- 액션이 호출 시점에 의존하는 것을 제한

--- 

# chapter 04. 액션에서 계산 빼내기 
입력과 출력은 명시적이거나 암묵적일 수 있음
- 명시적 입력 : 인자(param) 
- 암묵적 입력 : 인자 외 다른 입력(전역변수를 읽는 것) 
- 명시적 출력 : 리턴값 
- 암묵적 출력 : 리턴값 외 다른 출력 (전역변수를 바꾸는 것도 암묵적 출력)

### 함수에 암묵적 입력과 출력이 있다면 액션이 된다 
- 함수형 프로그래밍에서 암묵적 입력과 출력을 부수 효과라고 한다. 

```js
function cacl_cart_total() {
  shopping_cart_total = 0;
  for(var i = 0; i<shopping_car.length; i++){
    var item = shopping_cart[i];
    shopping_cart_total += item.price;
   }
  set_cart_total_dom();
  update_shipping_icons();
  update_tax_dom();
}
```

바꾼 코드 
```js 
function calc_cart_total() {
  calc_total();
  set_cart_total_dom();
  update_shipping_icons();
  update_tax_dom();
}

function calc_total() {
 shopping_cart_total = 0; //출력
 for(var i = 0; i < shopping_car.length; i++){ // shopping_car.length 입력
   var item = shopping_cart[i];
   shopping_cart_total += item.price; // 출력 
  }
}
```

- 서브루틴 추출 : 계산에 해당하는 코드를 function으로 변환하는 리팩토링 
- 하지만 여전히 새로 만든 함수도 액션 
  - 현재 전역변수(shopping_cart_total) 을 변경하므로 해당 변수를 지역변수로 하여 리턴하도록 처리 

```js
function calc_cart_total() {
  shopping_cart_total = calc_total();
  set_cart_total_dom();
  update_shipping_icons();
  update_tax_dom();
}

function calc_total() {
 var total = 0; 
 for(var i = 0; i < shopping_cart.length; i++){ // shopping_car.length 암묵적 입력
   var item = shopping_cart[i];
   total += item.price; 
  }
  return total; 
}
```

암묵적 입력 없애기 
```js 
function calc_total(cart) {
 var total = 0; 
 for(var i = 0; i < cart.length; i++){ 
   var item = cart[i];
   total += item.price; 
  }
  return total; 
}
```

정리 
- 계산코드를 찾아 빼내기
- 새 함수에 암묵적 입력과 출력을 찾는다. 
- 암묵적 입력은 이잔로 암묵적 출력은 리턴값으로 벼녁 
- 이를 통해 함수형 원칙을 적용시, 액션은 줄어들고 계싼은 늘어난다. 
