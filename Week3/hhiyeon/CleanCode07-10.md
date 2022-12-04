### 📌 7장 오류처리
#### 오류 코드보다는 예외를 사용하라
- if else 문으로 오류 코드를 호출하지 않고, try catch 으로 오류 발생시 예외 던지기

#### 미확인(unchecked) 예외를 사용하라
- 확인된 예외는 OCP(Open Closed Principle)을 위반한다.
  - 하위 단계에서 코드를 변경하면 상위 단계 메서드 선언부를 전부 고쳐야 한다.

#### 예외에 의미를 제공하라
- 오류 메시지에 정보를 담아 예외와 함께 던진다.
- 실패한 연산 이름과 실패 유형도 언급한다.
- 애플리케이션이 로깅 기능을 사용하면, catch 블록에서 오류를 기록하도록 충분한 정보를 넘겨준다.

#### 호출자르 고려해 예외 클래스를 정의하라
- 오류가 발생한 컴포넌트로 분류한다.
- 유형으로도 분류가 가능하다. ex. 디바이스 실패, 네트워크 실패, 프로그래밍 오류
- 여러 catch 문을 사용하는 것 보다, 호출하는 라이브러리 API를 감싸서 예외 유형 하나를 반환해준다.
```java
LocalPort port = new LocalPort(12); 
try {
    port.open();
} catch (PortDeviceFailure e) {
    reportError(e);
    logger. log(e.getMessage(), e);
} finally {}

public class LocalPort {
    private ACMEPort innerPort;
    public LocalPort(int portNumber) { innerPort = new ACMEPort(portNumber);
}
    public void open () { 
        try {
            innerPort.open ();
        } catch (DeviceResponseException e) {
            throw new PortDeviceFailure(e);
        } catch (ATM1212UnlockedException e) {
            throw new PortDeviceFailure(e); } 
        catch (GMXError e) {
            throw new PortDeviceFailure(e);
        } 
    }
}
```
- 다른 라이브러리로 갈아탈 경우 비용이 적어진다.
- 프로그램 테스트가 쉬워진다.

#### 정상 흐름을 정의하라
- 외부 API를 감싸 독자적인 예외를 던지고, 코드 위에 처리기를 정의해 중단된 계산을 처리한다.
- 중단이 적합하지 않은 경우 : ex. 비용 청구 애플리케이션
  - Special Case Pattern 특수 사례 패턴
  - 클래스를 만들거나 객체를 조작해 특수 사례를 처리하는 방식 -> 클라이언트 코드가 예외적인 상황을 처리할 필요가 없어진다.
  - 예외가 논리를 어렵게 만드는 경우, 특수 상황을 처리할 필요가 없으면 더 개선될 것이다.
```java
try {
MealExpenses expenses = expenseReportDAO.getMeals(employee.getID()); 
    m_total += expenses.getTotaK);
} catch(MealExpensesNotFound e) { 
    m_total += getMealPerDiem();
}

// 개선 방법 : ExpenseReportDAO를 언제나 MealExpense 객체를 반환하도록 수정
MealExpenses expenses = expenseReportDAO.getMeals(employee.getID()); m_total += expenses.getTotaK);

public class PerDiemMealExpenses implements MealExpenses { 
    public int getTotal() {
// 기본값으로 일일 기본 식비를 반환한다.
} }
```

#### null 을 반환하지 마라
- 메서드로 null 을 전달하는 코드와 반환하는 코드는 피한다.


#### 결론
- 깨끗한 코드는 읽기 좋은 코드 + 높은 안정성
- 오류 처리를 프로그램 논리와 분리해 독자적인 사안으로 고려하자
---
### 📌 8장 경계
#### 외부 코드 사용하기
- Map은 제공하는 기능성과 유연성이 있지만, 다른곳으로 넘길 경우 clear 메소드로 Map 사용라면 누구나 Map 내용을 지울 권한이 있는 위험이 있다.
- Map의 인터페이스가 변하게 되면, 수정할 코드가 많아진다.
- Map을 깔끔하게 사용한 코드. 제네릭스의 사용 여부는 Sensor 안에서 결정한다.
```java
public class Sensors {
    private Map sensors = new HashMap();
    
    public Sensor getById(String id) { 
        return (Sensor) sensors.get(id);
}
// 이하 생략 }
```

#### 경계 살피고 익히기
- 타사 라이브러리를 사용할 경우
  - 하루나 이틀 이상의 시간동안 문서를 읽고 사용법 결정
  - 코드를 작성 후 라이브러리가 예상대로 동작하는지 확인
- 학습 테스트
  - 코드를 작성해 외부 코드를 호출하는 것보다 먼저 간단한 테스트 케이스 작성 후 외부 코드를 익히는 방법
  
#### log4j 익히기
- log4j를 사용해서 패키지를 내려받고, "hello"를 출력하는 테스트 케이스
```java
@Test
public void testLogCreate() {
    Logger logger = Logger.getLoggerC'MyLogger");
    logger, infoC'hello"); // Appender가 필요하다는 오류 발생
}

@Test
public void testLogAddAppender() {
    Logger logger = Logger.getLogger("MyLogger"); 
    logger.removeAllAppenders(); 
    logger.addAppender(new ConsoleAppender(
        new PatternLayout("%p %t %m%n"),
        ConsoleAppender.SYSTEM_OUT)) ;
    logger. infoC'hello");
}
```

#### 학습 테스트는 공짜 이상이다
- 이해도를 높여주는 정확한 실험이다.
- 패키지 새 버전이 나오면 학습 테스트를 돌려 차이를 확인한다.
- 패키지가 예상대로 도는지 검증한다.

#### 아직 존재하지 않는 코드를 사용하기
- 경계와 관련해 또 다른 유형은 아는 코드와 모르는 코드를 분리하는 경계
- 지식이 별로 없는 상태에서 시스템을 개발하려고 하는 경우, 하위 시스템과 먼 부분부터 작업을 진행

<img width="574" alt="스크린샷 2022-12-02 오후 2 20 22" src="https://user-images.githubusercontent.com/52193680/205220457-14041242-d7b1-48f9-b772-889421d525b2.png">
- 송신기 API에서 CommunicationsController 분리
- 다른 팀에서 송신기 API 정의 후에 TransmitterAdapter를 구현해 간극을 매운다.
- ADAPTER 패턴(어댑터 패턴) 으로 API 사용을 켑슐화 해서 API가 바뀔 때 수정할 코드를 한 곳으로 모았다.

#### 깨끗한 경계
- 경계에 위치하는 코드는 깔끔하게 분리한다.
- 기대치를 정의하는 테스트 케이스도 작성한다.
- 외부 패키지에 의존하는 것보다 우리 코드에 의존하는 편이 훨씬 좋다.
- 외부 패키지를 호출하는 코드를 가능한 줄인다.
  - 새로운 클래스로 경계를 감싸거나 어댑터 패턴을 사용해서 우리가 원하는 인터페이스를 패키지가 제공하는 인터페이스로 변환한다.
---
### 📌 9장 단위 테스트
#### TDD 법칙 세 가지
- 첫째 법칙 : 실패하는 단위 테스트를 작성할 때까지 실제 코드를 작성하지 않는다.
- 둘째 법칙 : 컴파일은 실패하지 않으면서 실행이 실패하는 정도로만 단위 테스트를 작성한다.
- 셋째 법칙 : 현재 실패하는 테스트를 통과할 정도로만 실제 코드를 작성한다.

#### 깨끗한 테스트 코드 유지하기
- 지저분한 테스트 코드가 있는 것보다 테스트가 없는 것이 낫다.
- 테스트는 유연성, 유지보수성, 재사용성을 제공한다.

#### 깨끗한 테스트 코드
- 중복되는 코드, 자질구레한 사항이 너무 많을 경우 표현력 저하
- BUILD-OPERATE_CHECK 패턴
  - BUILD : 테스트 자료를 만든다.
  - OPERATE : 테스트 자료를 조작한다.
  - CHECK : 조작한 결과가 올바른지 확인한다.

#### 도메인에 특화된 테스트 언어
- DSL(도메인 특화 언어) 으로 테스트 코드를 구현하는 기법
- 보통 시스템 조작 API를 사용하지만, 대신에 API 위에 함수와 유틸리티를 구현한 후 그 함수와 유틸리티를 사용한다.
  - 테스트 코드 작성과 읽기가 쉬워진다.
- 이중 표준
  - 실제 환경에서는 안되지만 테스트 환경에서는 문제 없는 방식
  - ex. StringBuffer는 효율적이지만 테스트 환경은 자원이 제한적일 가능성이 낮다.

#### 테스트 당 assert 하나
- given-when-then 이라는 관례를 사용해서, 테스트 코드를 읽기 쉽게 작성
- 테스트를 분리하게 되면 중복 코드가 생기게 된다.
- 방법1 : TEMPLATE METHOD(템플릿 메소드) 패턴을 사용하면 중복 제거 가능
  - given/when 부분을 부모 클래스에 두고, then 부분을 자식 클래스에 둔다.
- 방법2 : 독자적인 테스트 클래스를 만들어서, @Before 함수에 given/when 부분을 넣고 @Test 함수에 then 부분을 넣는 방법

#### 테스트 당 개념 하나

#### F.I.R.S.T
- 깨끗한 테스트를 위한 다섯가지 규칙
- Fast(빠르게) : 테스트는 빨리 돌아야 한다.
- Independent(독립적으로) : 각 테스트는 서로 의존하면 안된다.
- Repeatable(반복가능한) : 어떤 환경에서도 테스트는 반복이 가능해야 한다.
- Self-Validating(자가검증하는) : 테스트는 bool 값으로 결과를 내야 한다.
- Timely(적시에) : 단위 테스트는 테스트를 하려는 실제 코드를 구현하기 직전에 구현한다.

---
### 📌 10장 클래스
#### 클래스 체계
- 변수 목록
  - static public 변수
  - static private 변수
  - private instance 변수
  - public 변수가 필요한 경우는 거의 없다.
- 공개 함수
- 비공개 함수 : 자신을 호출하는 공개 함수 직후
- 캡슐화 
  - 변수와 유틸리티 함수는 가능한 공개하지 않는다.
  - 테스트 코드에 접근을 허용할 수 있도록 protected 으로 선언하는 경우가 있다.
  - 하지만 비공개 상태를 유지할 방법을 생각한 후, 캡슐화를 풀어주는 것은 마지막 수단으로 한다.

#### 클래스는 작아야 한다!
- 클래스 이름에 해당 클래스 책임을 기술한다.
- 클래스 설명은 if, and, or, but 등을 제외한다. 그리고 25단어 내외로 가능해야 한다.
- SRP 단일 책임 원칙(Single Responsibility Principle)
  - 클래스는 책임이 하나여야 한다.

### 응집도 Cohesion
- 클래스는 인스턴스 변수의 수가 작아야 한다.
- 각 클래스 메서드는 클래스 인스턴스 변수를 하나 이상 사용해야 한다.
- 변수를 많이 사용할수록 메소드와 클래스는 응집도가 더 높다.
- 응집도를 유지하면 작은 클래스 여럿이 나온다

#### 변경으로부터 격리
- 구체적인 클래스 : 상세한 코드를 포함
- 추상 클래스 : 개념만 포함
- 인터페이스와 추상 클래스를 사용해서 구현이 미치는 영향을 격리한다.
- 시스템의 결합도를 낮추면 유연성과 재사용성을 높일 수 있다.

---
### 📌 11장 시스템

---
### 📌 12장 창발성

---
### 📌 13장 동시성