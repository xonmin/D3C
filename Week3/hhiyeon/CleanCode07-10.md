### ๐ 7์ฅ ์ค๋ฅ์ฒ๋ฆฌ
#### ์ค๋ฅ ์ฝ๋๋ณด๋ค๋ ์์ธ๋ฅผ ์ฌ์ฉํ๋ผ
- if else ๋ฌธ์ผ๋ก ์ค๋ฅ ์ฝ๋๋ฅผ ํธ์ถํ์ง ์๊ณ , try catch ์ผ๋ก ์ค๋ฅ ๋ฐ์์ ์์ธ ๋์ง๊ธฐ

#### ๋ฏธํ์ธ(unchecked) ์์ธ๋ฅผ ์ฌ์ฉํ๋ผ
- ํ์ธ๋ ์์ธ๋ OCP(Open Closed Principle)์ ์๋ฐํ๋ค.
  - ํ์ ๋จ๊ณ์์ ์ฝ๋๋ฅผ ๋ณ๊ฒฝํ๋ฉด ์์ ๋จ๊ณ ๋ฉ์๋ ์ ์ธ๋ถ๋ฅผ ์ ๋ถ ๊ณ ์ณ์ผ ํ๋ค.

#### ์์ธ์ ์๋ฏธ๋ฅผ ์ ๊ณตํ๋ผ
- ์ค๋ฅ ๋ฉ์์ง์ ์ ๋ณด๋ฅผ ๋ด์ ์์ธ์ ํจ๊ป ๋์ง๋ค.
- ์คํจํ ์ฐ์ฐ ์ด๋ฆ๊ณผ ์คํจ ์ ํ๋ ์ธ๊ธํ๋ค.
- ์ ํ๋ฆฌ์ผ์ด์์ด ๋ก๊น ๊ธฐ๋ฅ์ ์ฌ์ฉํ๋ฉด, catch ๋ธ๋ก์์ ์ค๋ฅ๋ฅผ ๊ธฐ๋กํ๋๋ก ์ถฉ๋ถํ ์ ๋ณด๋ฅผ ๋๊ฒจ์ค๋ค.

#### ํธ์ถ์๋ฅด ๊ณ ๋ คํด ์์ธ ํด๋์ค๋ฅผ ์ ์ํ๋ผ
- ์ค๋ฅ๊ฐ ๋ฐ์ํ ์ปดํฌ๋ํธ๋ก ๋ถ๋ฅํ๋ค.
- ์ ํ์ผ๋ก๋ ๋ถ๋ฅ๊ฐ ๊ฐ๋ฅํ๋ค. ex. ๋๋ฐ์ด์ค ์คํจ, ๋คํธ์ํฌ ์คํจ, ํ๋ก๊ทธ๋๋ฐ ์ค๋ฅ
- ์ฌ๋ฌ catch ๋ฌธ์ ์ฌ์ฉํ๋ ๊ฒ ๋ณด๋ค, ํธ์ถํ๋ ๋ผ์ด๋ธ๋ฌ๋ฆฌ API๋ฅผ ๊ฐ์ธ์ ์์ธ ์ ํ ํ๋๋ฅผ ๋ฐํํด์ค๋ค.
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
- ๋ค๋ฅธ ๋ผ์ด๋ธ๋ฌ๋ฆฌ๋ก ๊ฐ์ํ ๊ฒฝ์ฐ ๋น์ฉ์ด ์ ์ด์ง๋ค.
- ํ๋ก๊ทธ๋จ ํ์คํธ๊ฐ ์ฌ์์ง๋ค.

#### ์ ์ ํ๋ฆ์ ์ ์ํ๋ผ
- ์ธ๋ถ API๋ฅผ ๊ฐ์ธ ๋์์ ์ธ ์์ธ๋ฅผ ๋์ง๊ณ , ์ฝ๋ ์์ ์ฒ๋ฆฌ๊ธฐ๋ฅผ ์ ์ํด ์ค๋จ๋ ๊ณ์ฐ์ ์ฒ๋ฆฌํ๋ค.
- ์ค๋จ์ด ์ ํฉํ์ง ์์ ๊ฒฝ์ฐ : ex. ๋น์ฉ ์ฒญ๊ตฌ ์ ํ๋ฆฌ์ผ์ด์
  - Special Case Pattern ํน์ ์ฌ๋ก ํจํด
  - ํด๋์ค๋ฅผ ๋ง๋ค๊ฑฐ๋ ๊ฐ์ฒด๋ฅผ ์กฐ์ํด ํน์ ์ฌ๋ก๋ฅผ ์ฒ๋ฆฌํ๋ ๋ฐฉ์ -> ํด๋ผ์ด์ธํธ ์ฝ๋๊ฐ ์์ธ์ ์ธ ์ํฉ์ ์ฒ๋ฆฌํ  ํ์๊ฐ ์์ด์ง๋ค.
  - ์์ธ๊ฐ ๋ผ๋ฆฌ๋ฅผ ์ด๋ ต๊ฒ ๋ง๋๋ ๊ฒฝ์ฐ, ํน์ ์ํฉ์ ์ฒ๋ฆฌํ  ํ์๊ฐ ์์ผ๋ฉด ๋ ๊ฐ์ ๋  ๊ฒ์ด๋ค.
```java
try {
MealExpenses expenses = expenseReportDAO.getMeals(employee.getID()); 
    m_total += expenses.getTotaK);
} catch(MealExpensesNotFound e) { 
    m_total += getMealPerDiem();
}

// ๊ฐ์  ๋ฐฉ๋ฒ : ExpenseReportDAO๋ฅผ ์ธ์ ๋ MealExpense ๊ฐ์ฒด๋ฅผ ๋ฐํํ๋๋ก ์์ 
MealExpenses expenses = expenseReportDAO.getMeals(employee.getID()); m_total += expenses.getTotaK);

public class PerDiemMealExpenses implements MealExpenses { 
    public int getTotal() {
// ๊ธฐ๋ณธ๊ฐ์ผ๋ก ์ผ์ผ ๊ธฐ๋ณธ ์๋น๋ฅผ ๋ฐํํ๋ค.
} }
```

#### null ์ ๋ฐํํ์ง ๋ง๋ผ
- ๋ฉ์๋๋ก null ์ ์ ๋ฌํ๋ ์ฝ๋์ ๋ฐํํ๋ ์ฝ๋๋ ํผํ๋ค.


#### ๊ฒฐ๋ก 
- ๊นจ๋ํ ์ฝ๋๋ ์ฝ๊ธฐ ์ข์ ์ฝ๋ + ๋์ ์์ ์ฑ
- ์ค๋ฅ ์ฒ๋ฆฌ๋ฅผ ํ๋ก๊ทธ๋จ ๋ผ๋ฆฌ์ ๋ถ๋ฆฌํด ๋์์ ์ธ ์ฌ์์ผ๋ก ๊ณ ๋ คํ์
---
### ๐ 8์ฅ ๊ฒฝ๊ณ
#### ์ธ๋ถ ์ฝ๋ ์ฌ์ฉํ๊ธฐ
- Map์ ์ ๊ณตํ๋ ๊ธฐ๋ฅ์ฑ๊ณผ ์ ์ฐ์ฑ์ด ์์ง๋ง, ๋ค๋ฅธ๊ณณ์ผ๋ก ๋๊ธธ ๊ฒฝ์ฐ clear ๋ฉ์๋๋ก Map ์ฌ์ฉ๋ผ๋ฉด ๋๊ตฌ๋ Map ๋ด์ฉ์ ์ง์ธ ๊ถํ์ด ์๋ ์ํ์ด ์๋ค.
- Map์ ์ธํฐํ์ด์ค๊ฐ ๋ณํ๊ฒ ๋๋ฉด, ์์ ํ  ์ฝ๋๊ฐ ๋ง์์ง๋ค.
- Map์ ๊น๋ํ๊ฒ ์ฌ์ฉํ ์ฝ๋. ์ ๋ค๋ฆญ์ค์ ์ฌ์ฉ ์ฌ๋ถ๋ Sensor ์์์ ๊ฒฐ์ ํ๋ค.
```java
public class Sensors {
    private Map sensors = new HashMap();
    
    public Sensor getById(String id) { 
        return (Sensor) sensors.get(id);
}
// ์ดํ ์๋ต }
```

#### ๊ฒฝ๊ณ ์ดํผ๊ณ  ์ตํ๊ธฐ
- ํ์ฌ ๋ผ์ด๋ธ๋ฌ๋ฆฌ๋ฅผ ์ฌ์ฉํ  ๊ฒฝ์ฐ
  - ํ๋ฃจ๋ ์ดํ ์ด์์ ์๊ฐ๋์ ๋ฌธ์๋ฅผ ์ฝ๊ณ  ์ฌ์ฉ๋ฒ ๊ฒฐ์ 
  - ์ฝ๋๋ฅผ ์์ฑ ํ ๋ผ์ด๋ธ๋ฌ๋ฆฌ๊ฐ ์์๋๋ก ๋์ํ๋์ง ํ์ธ
- ํ์ต ํ์คํธ
  - ์ฝ๋๋ฅผ ์์ฑํด ์ธ๋ถ ์ฝ๋๋ฅผ ํธ์ถํ๋ ๊ฒ๋ณด๋ค ๋จผ์  ๊ฐ๋จํ ํ์คํธ ์ผ์ด์ค ์์ฑ ํ ์ธ๋ถ ์ฝ๋๋ฅผ ์ตํ๋ ๋ฐฉ๋ฒ
  
#### log4j ์ตํ๊ธฐ
- log4j๋ฅผ ์ฌ์ฉํด์ ํจํค์ง๋ฅผ ๋ด๋ ค๋ฐ๊ณ , "hello"๋ฅผ ์ถ๋ ฅํ๋ ํ์คํธ ์ผ์ด์ค
```java
@Test
public void testLogCreate() {
    Logger logger = Logger.getLoggerC'MyLogger");
    logger, infoC'hello"); // Appender๊ฐ ํ์ํ๋ค๋ ์ค๋ฅ ๋ฐ์
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

#### ํ์ต ํ์คํธ๋ ๊ณต์ง ์ด์์ด๋ค
- ์ดํด๋๋ฅผ ๋์ฌ์ฃผ๋ ์ ํํ ์คํ์ด๋ค.
- ํจํค์ง ์ ๋ฒ์ ์ด ๋์ค๋ฉด ํ์ต ํ์คํธ๋ฅผ ๋๋ ค ์ฐจ์ด๋ฅผ ํ์ธํ๋ค.
- ํจํค์ง๊ฐ ์์๋๋ก ๋๋์ง ๊ฒ์ฆํ๋ค.

#### ์์ง ์กด์ฌํ์ง ์๋ ์ฝ๋๋ฅผ ์ฌ์ฉํ๊ธฐ
- ๊ฒฝ๊ณ์ ๊ด๋ จํด ๋ ๋ค๋ฅธ ์ ํ์ ์๋ ์ฝ๋์ ๋ชจ๋ฅด๋ ์ฝ๋๋ฅผ ๋ถ๋ฆฌํ๋ ๊ฒฝ๊ณ
- ์ง์์ด ๋ณ๋ก ์๋ ์ํ์์ ์์คํ์ ๊ฐ๋ฐํ๋ ค๊ณ  ํ๋ ๊ฒฝ์ฐ, ํ์ ์์คํ๊ณผ ๋จผ ๋ถ๋ถ๋ถํฐ ์์์ ์งํ

<img width="574" alt="แแณแแณแแตแซแแฃแบ 2022-12-02 แแฉแแฎ 2 20 22" src="https://user-images.githubusercontent.com/52193680/205220457-14041242-d7b1-48f9-b772-889421d525b2.png">


- ์ก์ ๊ธฐ API์์ CommunicationsController ๋ถ๋ฆฌ
- ๋ค๋ฅธ ํ์์ ์ก์ ๊ธฐ API ์ ์ ํ์ TransmitterAdapter๋ฅผ ๊ตฌํํด ๊ฐ๊ทน์ ๋งค์ด๋ค.
- ADAPTER ํจํด(์ด๋ํฐ ํจํด) ์ผ๋ก API ์ฌ์ฉ์ ์ผ์ํ ํด์ API๊ฐ ๋ฐ๋ ๋ ์์ ํ  ์ฝ๋๋ฅผ ํ ๊ณณ์ผ๋ก ๋ชจ์๋ค.

#### ๊นจ๋ํ ๊ฒฝ๊ณ
- ๊ฒฝ๊ณ์ ์์นํ๋ ์ฝ๋๋ ๊น๋ํ๊ฒ ๋ถ๋ฆฌํ๋ค.
- ๊ธฐ๋์น๋ฅผ ์ ์ํ๋ ํ์คํธ ์ผ์ด์ค๋ ์์ฑํ๋ค.
- ์ธ๋ถ ํจํค์ง์ ์์กดํ๋ ๊ฒ๋ณด๋ค ์ฐ๋ฆฌ ์ฝ๋์ ์์กดํ๋ ํธ์ด ํจ์ฌ ์ข๋ค.
- ์ธ๋ถ ํจํค์ง๋ฅผ ํธ์ถํ๋ ์ฝ๋๋ฅผ ๊ฐ๋ฅํ ์ค์ธ๋ค.
  - ์๋ก์ด ํด๋์ค๋ก ๊ฒฝ๊ณ๋ฅผ ๊ฐ์ธ๊ฑฐ๋ ์ด๋ํฐ ํจํด์ ์ฌ์ฉํด์ ์ฐ๋ฆฌ๊ฐ ์ํ๋ ์ธํฐํ์ด์ค๋ฅผ ํจํค์ง๊ฐ ์ ๊ณตํ๋ ์ธํฐํ์ด์ค๋ก ๋ณํํ๋ค.
---
### ๐ 9์ฅ ๋จ์ ํ์คํธ
#### TDD ๋ฒ์น ์ธ ๊ฐ์ง
- ์ฒซ์งธ ๋ฒ์น : ์คํจํ๋ ๋จ์ ํ์คํธ๋ฅผ ์์ฑํ  ๋๊น์ง ์ค์  ์ฝ๋๋ฅผ ์์ฑํ์ง ์๋๋ค.
- ๋์งธ ๋ฒ์น : ์ปดํ์ผ์ ์คํจํ์ง ์์ผ๋ฉด์ ์คํ์ด ์คํจํ๋ ์ ๋๋ก๋ง ๋จ์ ํ์คํธ๋ฅผ ์์ฑํ๋ค.
- ์์งธ ๋ฒ์น : ํ์ฌ ์คํจํ๋ ํ์คํธ๋ฅผ ํต๊ณผํ  ์ ๋๋ก๋ง ์ค์  ์ฝ๋๋ฅผ ์์ฑํ๋ค.

#### ๊นจ๋ํ ํ์คํธ ์ฝ๋ ์ ์งํ๊ธฐ
- ์ง์ ๋ถํ ํ์คํธ ์ฝ๋๊ฐ ์๋ ๊ฒ๋ณด๋ค ํ์คํธ๊ฐ ์๋ ๊ฒ์ด ๋ซ๋ค.
- ํ์คํธ๋ ์ ์ฐ์ฑ, ์ ์ง๋ณด์์ฑ, ์ฌ์ฌ์ฉ์ฑ์ ์ ๊ณตํ๋ค.

#### ๊นจ๋ํ ํ์คํธ ์ฝ๋
- ์ค๋ณต๋๋ ์ฝ๋, ์์ง๊ตฌ๋ ํ ์ฌํญ์ด ๋๋ฌด ๋ง์ ๊ฒฝ์ฐ ํํ๋ ฅ ์ ํ
- BUILD-OPERATE_CHECK ํจํด
  - BUILD : ํ์คํธ ์๋ฃ๋ฅผ ๋ง๋ ๋ค.
  - OPERATE : ํ์คํธ ์๋ฃ๋ฅผ ์กฐ์ํ๋ค.
  - CHECK : ์กฐ์ํ ๊ฒฐ๊ณผ๊ฐ ์ฌ๋ฐ๋ฅธ์ง ํ์ธํ๋ค.

#### ๋๋ฉ์ธ์ ํนํ๋ ํ์คํธ ์ธ์ด
- DSL(๋๋ฉ์ธ ํนํ ์ธ์ด) ์ผ๋ก ํ์คํธ ์ฝ๋๋ฅผ ๊ตฌํํ๋ ๊ธฐ๋ฒ
- ๋ณดํต ์์คํ ์กฐ์ API๋ฅผ ์ฌ์ฉํ์ง๋ง, ๋์ ์ API ์์ ํจ์์ ์ ํธ๋ฆฌํฐ๋ฅผ ๊ตฌํํ ํ ๊ทธ ํจ์์ ์ ํธ๋ฆฌํฐ๋ฅผ ์ฌ์ฉํ๋ค.
  - ํ์คํธ ์ฝ๋ ์์ฑ๊ณผ ์ฝ๊ธฐ๊ฐ ์ฌ์์ง๋ค.
- ์ด์ค ํ์ค
  - ์ค์  ํ๊ฒฝ์์๋ ์๋์ง๋ง ํ์คํธ ํ๊ฒฝ์์๋ ๋ฌธ์  ์๋ ๋ฐฉ์
  - ex. StringBuffer๋ ํจ์จ์ ์ด์ง๋ง ํ์คํธ ํ๊ฒฝ์ ์์์ด ์ ํ์ ์ผ ๊ฐ๋ฅ์ฑ์ด ๋ฎ๋ค.

#### ํ์คํธ ๋น assert ํ๋
- given-when-then ์ด๋ผ๋ ๊ด๋ก๋ฅผ ์ฌ์ฉํด์, ํ์คํธ ์ฝ๋๋ฅผ ์ฝ๊ธฐ ์ฝ๊ฒ ์์ฑ
- ํ์คํธ๋ฅผ ๋ถ๋ฆฌํ๊ฒ ๋๋ฉด ์ค๋ณต ์ฝ๋๊ฐ ์๊ธฐ๊ฒ ๋๋ค.
- ๋ฐฉ๋ฒ1 : TEMPLATE METHOD(ํํ๋ฆฟ ๋ฉ์๋) ํจํด์ ์ฌ์ฉํ๋ฉด ์ค๋ณต ์ ๊ฑฐ ๊ฐ๋ฅ
  - given/when ๋ถ๋ถ์ ๋ถ๋ชจ ํด๋์ค์ ๋๊ณ , then ๋ถ๋ถ์ ์์ ํด๋์ค์ ๋๋ค.
- ๋ฐฉ๋ฒ2 : ๋์์ ์ธ ํ์คํธ ํด๋์ค๋ฅผ ๋ง๋ค์ด์, @Before ํจ์์ given/when ๋ถ๋ถ์ ๋ฃ๊ณ  @Test ํจ์์ then ๋ถ๋ถ์ ๋ฃ๋ ๋ฐฉ๋ฒ

#### ํ์คํธ ๋น ๊ฐ๋ ํ๋

#### F.I.R.S.T
- ๊นจ๋ํ ํ์คํธ๋ฅผ ์ํ ๋ค์ฏ๊ฐ์ง ๊ท์น
- Fast(๋น ๋ฅด๊ฒ) : ํ์คํธ๋ ๋นจ๋ฆฌ ๋์์ผ ํ๋ค.
- Independent(๋๋ฆฝ์ ์ผ๋ก) : ๊ฐ ํ์คํธ๋ ์๋ก ์์กดํ๋ฉด ์๋๋ค.
- Repeatable(๋ฐ๋ณต๊ฐ๋ฅํ) : ์ด๋ค ํ๊ฒฝ์์๋ ํ์คํธ๋ ๋ฐ๋ณต์ด ๊ฐ๋ฅํด์ผ ํ๋ค.
- Self-Validating(์๊ฐ๊ฒ์ฆํ๋) : ํ์คํธ๋ bool ๊ฐ์ผ๋ก ๊ฒฐ๊ณผ๋ฅผ ๋ด์ผ ํ๋ค.
- Timely(์ ์์) : ๋จ์ ํ์คํธ๋ ํ์คํธ๋ฅผ ํ๋ ค๋ ์ค์  ์ฝ๋๋ฅผ ๊ตฌํํ๊ธฐ ์ง์ ์ ๊ตฌํํ๋ค.

---
### ๐ 10์ฅ ํด๋์ค
#### ํด๋์ค ์ฒด๊ณ
- ๋ณ์ ๋ชฉ๋ก
  - static public ๋ณ์
  - static private ๋ณ์
  - private instance ๋ณ์
  - public ๋ณ์๊ฐ ํ์ํ ๊ฒฝ์ฐ๋ ๊ฑฐ์ ์๋ค.
- ๊ณต๊ฐ ํจ์
- ๋น๊ณต๊ฐ ํจ์ : ์์ ์ ํธ์ถํ๋ ๊ณต๊ฐ ํจ์ ์งํ
- ์บก์ํ 
  - ๋ณ์์ ์ ํธ๋ฆฌํฐ ํจ์๋ ๊ฐ๋ฅํ ๊ณต๊ฐํ์ง ์๋๋ค.
  - ํ์คํธ ์ฝ๋์ ์ ๊ทผ์ ํ์ฉํ  ์ ์๋๋ก protected ์ผ๋ก ์ ์ธํ๋ ๊ฒฝ์ฐ๊ฐ ์๋ค.
  - ํ์ง๋ง ๋น๊ณต๊ฐ ์ํ๋ฅผ ์ ์งํ  ๋ฐฉ๋ฒ์ ์๊ฐํ ํ, ์บก์ํ๋ฅผ ํ์ด์ฃผ๋ ๊ฒ์ ๋ง์ง๋ง ์๋จ์ผ๋ก ํ๋ค.

#### ํด๋์ค๋ ์์์ผ ํ๋ค!
- ํด๋์ค ์ด๋ฆ์ ํด๋น ํด๋์ค ์ฑ์์ ๊ธฐ์ ํ๋ค.
- ํด๋์ค ์ค๋ช์ if, and, or, but ๋ฑ์ ์ ์ธํ๋ค. ๊ทธ๋ฆฌ๊ณ  25๋จ์ด ๋ด์ธ๋ก ๊ฐ๋ฅํด์ผ ํ๋ค.
- SRP ๋จ์ผ ์ฑ์ ์์น(Single Responsibility Principle)
  - ํด๋์ค๋ ์ฑ์์ด ํ๋์ฌ์ผ ํ๋ค.

### ์์ง๋ Cohesion
- ํด๋์ค๋ ์ธ์คํด์ค ๋ณ์์ ์๊ฐ ์์์ผ ํ๋ค.
- ๊ฐ ํด๋์ค ๋ฉ์๋๋ ํด๋์ค ์ธ์คํด์ค ๋ณ์๋ฅผ ํ๋ ์ด์ ์ฌ์ฉํด์ผ ํ๋ค.
- ๋ณ์๋ฅผ ๋ง์ด ์ฌ์ฉํ ์๋ก ๋ฉ์๋์ ํด๋์ค๋ ์์ง๋๊ฐ ๋ ๋๋ค.
- ์์ง๋๋ฅผ ์ ์งํ๋ฉด ์์ ํด๋์ค ์ฌ๋ฟ์ด ๋์จ๋ค

#### ๋ณ๊ฒฝ์ผ๋ก๋ถํฐ ๊ฒฉ๋ฆฌ
- ๊ตฌ์ฒด์ ์ธ ํด๋์ค : ์์ธํ ์ฝ๋๋ฅผ ํฌํจ
- ์ถ์ ํด๋์ค : ๊ฐ๋๋ง ํฌํจ
- ์ธํฐํ์ด์ค์ ์ถ์ ํด๋์ค๋ฅผ ์ฌ์ฉํด์ ๊ตฌํ์ด ๋ฏธ์น๋ ์ํฅ์ ๊ฒฉ๋ฆฌํ๋ค.
- ์์คํ์ ๊ฒฐํฉ๋๋ฅผ ๋ฎ์ถ๋ฉด ์ ์ฐ์ฑ๊ณผ ์ฌ์ฌ์ฉ์ฑ์ ๋์ผ ์ ์๋ค.

---