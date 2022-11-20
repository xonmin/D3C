### 📌 1장 깨끗한 코드
#### 깨끗한 코드란?
- 우아하고 효율적인 코드 : 의존성을 줄여야 유지보수가 쉬워지고, 오류를 처리하고, 성능을 최적으로 유지해야 한다. 
  - 오류 처리 : 메모리 누수, 경쟁 상태(race condition)


- 단순하고 직접적인, 가독성이 좋은 코드 


- 가독성이 좋고, 다른 사람이 고치기 쉬운 코드 : 각 의존성을 명확하게 정의한다.


- 주의 깊게 작성한 코드


- 켄트 백 코드 규칙 
  - 모든 테스트를 통과한다.
  - 중복이 없다.
  - 시스템 내 모든 설계 아이디어를 표현한다.
  - 클래스, 메소드, 함수 등을 최대한 줄인다.


- 읽으면서 짐작한 대로 돌아가는 코드
---
### 📌 2장 의미 있는 이름
#### 의도를 분명히 밝혀라
- 변수(혹은 함수나 클래스) 의 존재 이유, 수행 기능, 사용 방법을 주석 없이 이름으로 의도를 드러낸다.


- 아래의 코드는 하는 일을 짐작하기 어렵다.
  - theList에 무엇이 들었는지?
  - theList에서 0번째 값이 어째서 중요한지?
  - 값 4는 무엇을 의미하는지?
  - 함수가 반환하는 리스트 list1은 어떻게 사용하는지?
```java
public List<int[]> getThem() {
List<int[]> listl = new ArrayList<int[]>();
 for (int[] x : theList) if (x[0] = 4)
listl.add(x); return listl;
}
```
- 이름 변경후
```java
public List<Cell> getFlaggedCells() {
List<Cell> flaggedCells = new ArrayList<Cell>(); for (Cell cell : gameBoard)
if (cell.isFlagged()) flaggedCells.add(cell);
return flaggedCells; }
```
#### 그릇된 정보를 피하라
- 의미가 있는 단어를 다른 의미로 사용하지 않는다.
  - ex) hp, aix, sco


- 실제 List가 아니면, accountList라고 명명하지 않는다.
  - accountGroup, bunchOfAccounts, Accounts라 명명한다.

#### 의미 있게 구분하라
- 컴파일러를 통과해도 연속된 숫자를 덧붙이거나 불용어(noise word) 를 추가하는 방식은 적절하지 않다.
  - ex) a1, a2, a3, ... , aN -> source와 destination와 같이 정보를 제공할 수 있는 이름을 사용한다.
```java
public static void copyChars(char al[], char a2[]) { for (int i = 0; i < al.length; i++) {
a2[i] = al[i]; }
}
```

#### 발음하기 쉬운 이름을 사용하라

#### 검색하기 쉬운 이름을 사용하라
- 숫자 7을 사용하게 되면, 7이 들어가는 모든 이름들이 검색된다.
- MAX_CLASSES_PER_STUDENT 와 같이 검색하기 쉬운 이름을 사용한다.

#### 인코딩을 피하라

#### 자신의 기억력을 자랑하지 마라

#### 클래스 이름
- 클래스 이름과 객체 이름 : 명사, 명사구
- 적절한 예시 : Customer, WikiPage, Account, AddressParser
- 부적절한 예시 : Manager, Processor, Data, Info와 같은 이름, 동사

#### 메소드 이름
- 메소드 이름 : 동사, 동사구
- 적절한 예시 : postPayment, deletePage, save
- 접근자(Accessor), 변경자(Mutator), 조건자(Predicate)는 javabean 표준에 따라 값 앞에 get, set, is를 붙인다.

#### 기발한 이름은 피하라

#### 한 개념에 한 단어만 사용하라
- 추상적인 개념 하나에 단어 하나를 선택한다.
- 똑같은 기능의 메소드를 클래스마다 fetch, retrieve, get으로 제각각 부르면 혼란스럽다.

#### 말장난을 하지 마라
- 한 단어를 두 가지 목적으로 사용하지 마라
- ex) 기존에 add를 사용한 메소드가 두 개를 더하거나 이어서 새로운 값을 만드는 기능이라 가정했을 경우
  - 값 하나만 추가하는 메소드의 이름은 add보다 insert나 append를 사용하는 것이 적당하다.
  
#### 해법 영역에서 가져온 이름을 사용하라
- 익숙한 기술 개념에는 기술 이름이 적합한 선택

#### 문제 영역에서 가져온 이름을 사용하라
- 적절한 용어가 없으면 문제 영역에서 이름을 가져온다.

#### 의미 있는 맥락을 추가하라
- state -> addState

#### 불필요한 맥락을 없애라
- 고급 휘발유 충전소(Gas Station Deluxe) 라는 애플리케이션을 만들었을 때, 모든 클래스 이름을 GSD로 시작하는 것은 바람직하지 않다.

---
### 📌 3장 함수

#### 작게 만들어라
- 블록과 들여쓰기
  - 함수에서 들여쓰기 수준은 1단이나 2단을 넘으면 안된다.

#### 한 가지만 해라
- 함수는 한 가지를 해야 한다. 그 한 가지를 잘 해야 한다. 그 한 가지만을 해야 한다.

#### 함수 당 추상화 수준은 하나로
- 위에서 아래로 코드 읽기 : 내려가기 규칙
  - 코드는 위에서 아래로 이야기처럼 읽혀야 좋다.
- switch 문은 작게 만들기 어렵다.

#### 서술적인 이름을 사용하라
- ex) testableHtml -> SetupTeardownlncluder 함수가 하는 일을 좀 더 잘 표현할 수 있다.
- 길고 서술적인 이름이 짧고 어려운 이름보다 좋다.

#### 함수 인수
- 함수에서 이상적인 인수 개수 0개(무항)
- 3개 이상은 피하는 것이 좋다.
- 플래그 인수 
  - 함수로 부울 값을 넘기는 것은 좋지 않다.
  - ex) render(boolean isSuite) 보다는 renderForSuite(), renderForSingleTest()이라는 함수로 나눠야한다.
- 출력 인수 : 일반적으로 출력 인수는 피한다.


#### 명령과 조회를 분리하라

#### 오류 코드보다 예외를 사용하라
- try/catch 블록 뽑아내기
  - try/catch 블록은 정상 동작과 오류 처리 동작을 뒤섞기 때문에, 별도 함수로 뽑아낸다.
  - 정상 동작과 오류 처리 동작을 분리하면 코드를 이해하기 수정하기 쉬워진다.
  - 오류 처리도 한 가지 작업이다.

#### 반복하지 마라
- 알고리즘이 여러 함수에서 반복되지 않도록 작성한다.
- 객체지향 프로그래밍은 코드를 부모 클래스에 몰아 중복을 없앤다. 

#### 구조적 프로그래밍
- 루프 안에서 break, continue, goto를 사용하지 않는다.
- 함수를 작게 만드려면, return, break, continue를 사용해도 된다.