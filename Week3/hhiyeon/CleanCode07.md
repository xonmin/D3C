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


---
### 📌 8장 경계

---
### 📌 9장 단위 테스트

---
### 📌 10장 클래스

---
### 📌 11장 시스템

---
### 📌 12장 창발성

---
### 📌 13장 동시성