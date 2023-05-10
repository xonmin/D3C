# 5. 간단한 비즈니스 로직 구현 

## 트랜잭션 스크립트 패턴

> 프레젠테이션으로부터 단일 요청을 처리하는 여러 프로시저를 모아, 비즈니스 로직을 구현 (시스템의 퍼블릭 인터페이스 - 마틴 파울러) 

- 트랜잭션 스크립트 패턴 : 프로시저를 기반으로 시스템의 비즈니스 로직을 구성, 각 프로시저는 퍼블릭 인터페이스를 통해 시스템 사용자가 실행하는 작업 구현 
- 각 프로시저는 간단하고 쉬운 절차지향 스크립트로 구현 
- 각 프로시저는 트랜잭션 동작을 통해 각 작업이 성공/실패할 수 있고, 유효하지 않은 상태를 만들면 안된다. 


```c++
DB.StartTransaction();

var job = DB.LoadNextJob();
var json = LoadFile(job.Source);
var xml = ConvertJsonToXml(json);
WriteFIle(job.Destination, xml.toString());
DB.MarkJobAsCompleted(job);

DB.Commit();
```

### 트랜잭션 동작 구현 실패 
- case1) 전체를 아우르는 tx 없이, 여러 업데이트 처리 
  - 해결 : try-catch 를 통해 두 데이터 변경을 모두 포함하는 tx 생성 
  - 하지만, 다중 레코드 tx를 지원하지 않거나, 분산 tx에서 통합할수 없는 여러 저장 장치로 작업하는 경우에는 위 해결방법으로 해결 안된다. 
  
### 분산 트랜잭션
ex) db의 데이터 변경 후, 메시지 버스에 메시지를 발행(알림) 
- 만약 변경은 처리되었으나, 발행 전 오류가 발생할 경우 메시지 발행하지 못해 알림을 받지 못함 
- 여러 저장 장치에 걸친 분산 트랜잭션의 경우 확장 어려움 / 오류 발생 가능성 높아 일반적으로 피하는 방법
  - 해결 방안 : 1. CQRS 아키텍처 패턴(8장) / 2.아웃박스 패턴 (변경사항 커밋 후 안정적 메시지 발행) 

### 암시적 분산 트랜잭션 
```c++
public void Execute(Guid userId)
{
  _db.execute("UPDATE Users SET visits=visits+1 Where user_id=@p1, userID);
 }
 ```
 - 여전히 잠재적으로 일관성 없는 상태로 이어질 수 있는 분산 트랜잭션
 
- (클라이언트) -> 1. Execute -> (LogVisit) -> 2.Update -> DB 
- (클라이언트) <- 3. Return Result - Logvisit

메서드는 성공했으나 호출자에게 전달 실패 시나리오 
- logvisit이 rest 의 일부이고 네트워크 중단 발생
- logvisit이 호출자가 동일한 프로세스에서 실행되고 있지만, 호출자가 logvisit 작업의 성공적 실행을 추적 전에 프로세스가 실패하는 경우 

결과적으로 모두 사용자는 실패 가정후, logvisit 재호출하여 카운트 2증가 

**해결방법**
1. 트랜잭션 동작을 보장하는 법 : 멱등성 만들기 (같은 요청을 여러번 반복하더라도 그 결과는 매번 동일해야함) 

```c++
public void Execute(Guid userId, long visits)
{
  _db.execute("UPDATE Users SET visits=@p1 Where user_id=@p2, visits, userID);
 }
 ```
 
 2. 낙관적 동시성 제어 
 - 작업 호출 전 카운터의 현재 값을 읽고 매개변수로 logVisit에 전달 이 떄, 호출자가 처음 읽은 값과 동일한 경우에만 카운터 값 업데이트 처리 
```c++
public void Execute(Guid userId, long expectedVisits)
{
  _db.execute("UPDATE Users SET visits=visits+1 Where user_id=@p1 and visits = @p2, userID, visits);
 }
 ```
  
### 트랜잭션 스크립트를 사용하는 경우 
- 비즈니스 로직이 단순한 절차적 작업처럼 매우 간단한 문제 도메인에 효과적 (ex_ ETL) 
- 정의상 비즈니스 로직이 단순한 지원 하위 도메인에 적합
- 일반 하위 도메인과 같은 외부 시스템과 연동하기 위한 어댑터로 사용 / 충돌 방지 계층 일부로 사용 
- 트랜잭션 스크립트의 장점 : 단순함 
  - 최소한의 추상화 -> 런타임 성능 최적화 
- 비즈니스 로직이 복잡 -> tx 간 비즈니스 로직이 중복되기 쉽고, 중복된 코드가 동기화되지 않을 떄 일관성없는 동작 발생 
  - 결국 핵심 하위 도메인에는 트랜잭션 스크립트 사용 X 
- 이러한 단순함으로 인해 때로는 안티 패턴 취급 


## 액티브 레코드 패턴
> DB 테이블 / 뷰의 행을 감싸고 DB 접근을 캡슐화하고 해당 데이터에 도메인 로직 추가하는 오브젝트 - 마틴 파울러 

- 액티브 레코드 : 좀 더 복잡한 자료구조에서 비즈니스 로직 작동 가능 (물론 비즈니스 로직 단순한 경우에도 사용) 
- **액티브 레코드**라고 하는 전용 객체를 통해 복잡한 자료구조 구현 
- 자료구조 외에도 CRUD 구현 
- ORM 또는 데이터 접근 프레임워크와도 관련 
- 액티브 레코드는 데이터 접근 로직 구현 

### 트랜잭션 스크립트 패턴과의 차이 
- 우선, 공통적으로 액티브 레코드 또한 트랝잭션 스크립트로 시스템의 비즈니스 로직 생성
- 차이점 :
  - 액티브 레코드 :  DB 직접 접근하는 것이 아닌 트랜잭션 스크립트가 액티브 레코드 객체 조작 
  - 작업이 완료되면 트랜잭션의 원자성으로 인해 작업 성공 or 실패 
```c++
public void Execute(userDetails) {
  try {
    _db.StartTx();
    
    var user = new User();
    user.Name = userDetails.Name();
    user.Email = userDetails.Email;
    user.Save();
    _db.Commit();
  } catch {
    _db.Rollback();
    throw;
  }
}
```
- 목적 : 메모리 상의 객체를 DB 스키마에 매핑하는 복잡성 숨기기 
- 영속성 담당 외, 비즈니스 로직 포함 가능 (유효성 검사 및 데이터 조작 관련 비즈니스 절차) 
- 즉, **액티브 레코드 객체의 고유한 기능은 자료구조와 동작(비즈니스 로직) 분리**

### 액티브 레코드를 사용하는 경우 
- 지원 하위 도메인, 일반 하위 도메인과 외부 솔루션의 연동, 모델 변환 작업에 적합 
- 빈약한 도메인 모델 안티패턴으로도 불림 

# 6. 복잡한 비즈니스 로직 다루기 

## 도메인 모델
- 에릭 에반스의 비즈니스 도메인 도메인의 하위 모델과 코드를 연결하기 위해(복잡한 비즈니스 로직) 사용
- 애그리게이트, VO, repository 
- 흔히 전술적 도메인 주도 설계라고 얘기하지만, 저자는 그냥 해당 패턴을 도메인 모델로 칭하고 애그리게이트와 VO는 구성요소라고 설명한다. 

### 구현
- 도메인 모델은 행동과 데이터를 모두 포함하는 객체 모델 

### 복잡성
- 도메인 비즈니스 로직이 이미 본질적으로 복잡하기에 모델링에 사용되는 객체가 모델에 우발적 복잡서을 추가해선 X
- 이러한 모델 객체는 Plain Old Object라고 한다.
  - 인프라 구성 요소 및 프레임워크에 의지/협업 X, 비즈니스 로직 구현하는 객체 

### VO(Value Object)
복합적인 값에 의해 식별되는 객체
```C++
class Color
{
  int _red;
  int _green;
  int _blue;
}
```
- 해당 객체를 식별하기 위해 명시적인 식별 필드 필요 없음(Id) 
- 불변의 객체로 구현되므로 밸류 오브젝트에 있는 필드가 바뀌면 다른 인스턴스가 생성된다.
  - ex) Java / .Net - String  
- 원시 집착 코드 징후(Primitve obsession code smell) 원시 데이터 타입에 의존하여 비즈니스 도메인 개념을 표현하는 것 
  - ex) 특정 모델의 필드들을 String, 및 값에 대해서는 관례에 따라 할당될 경우의 단점
  - **유효성 검사 로직이 중복되기 쉬움**
  - **값이 사용되기 전, 유효성 검사 로직 호출이 어려움**
  - **유지보수 어려움**

개선된 버전
```C++
class Person {
private PersonId _id;
private Name _name;
private PhoneNumber _landline;
private PhoneNumber _mobile;
...
}

var dave = new Person(
   id: new PersonId(30217),
   name: new Name("Dave", "Luece"),
   landline: PhoneNumber.Parse("0231232"),
   mobile: PhoneNumber.Parse("01023123123"),
   ...
   
 )
 
```
장점 
- 명료성 향상
- 유효성 검사 로직이 밸류 오브젝트 내에 있기 떄문에 값 할당 전 유효성 검사 필요 없음 
- 밸류 오브젝트 값 조작 시, 비즈니스 로직을 한 곳에 모아 구현 및 테스트 용이 
- 코드의 표현력을 높이고 분산되기 쉬운 비즈니스 로직을 묶음, 안전성 향상 
- 불변에 의한 내포된 동작의 부작용 및 동시성 문제 없음 

**VO를 사용하는 경우**
- 가능한 모든 경우에 사용하라 

### 엔티티 
- VO와 정반대
- 다른 엔티티 인스턴스와 구별하기 위한 명시적 식별 필드 필요 (각 인스턴스 마다 생명주기내 고유/불변해야함)
- 이외 다른 상태는 변할 수 있음 
- 밸류 오브젝트는 엔티티의 속서을 설명하는 역할 

### 애그리게이트 
- 애그리게이트는 엔티티의 일종
- 목적 : 데이터의 일관성 보호 
- 일관성 강화 경계의 역할: 
  -  로직은 모든 들어오는 변경 요청에 대한 검사를 통해 비즈니스 규칙을 위배되지 않도록 해야함 
  -  즉, 외부의 프로세스 및 객체는 애그리게이트 상태를 read만 가능 
  -  **애그리게이트의 퍼블릭 인터페이스에 포함된 관련 메서드(커멘드)**를 실행해야만 상태 변형 가능 

> 커맨드 구현 방식
> 1. 애그리게이트 객체 내 퍼블릭 메서드 구현
> 2. 커맨드 실행에 필요한 모든 Input을 포함하는 파라미터 객체로 표현 

- 동시성 문제에 대해서도 애그리게이트 상태의 일관성을 유지할 수 있도록 해야함 (ex_ tx) 
  -ex) 매 갱신 떄마다 증가하는 버전 필드를 애그리게이트에서 관리하도록

트랜잭션 경계 
- 애그리게이트 상태 : 자신의 비즈니스 로직을 통해서만 수정될 수 있으므로 본인이 트랜잭션 경계의 역할 수행 
- 모든 애그리게이트 상태 변경 : 원자적 단일 오퍼레이션으로 tx 처리 
- 다중 애그리게이트 tx를 지원하는 시스템 오퍼레이션은 없다고 가정

> 만약 동일한 tx에서 여러 객체를 수정해야 한다면?

### 엔티티 계층 
- 엔티티는 독립적 패턴이 아닌 애그리게이트의 일부로서만 사용
- 여러 객체의 변경을 원자적 단일 tX 지원을 위해 애그리게이트 패턴은 엔티티 계층 구조와 유사하게 모든 tx 공유 -> 일관성 유지 
<img width="403" alt="image" src="https://user-images.githubusercontent.com/27190617/225534016-7557bc12-24a7-4210-b41e-1b02d6e051c6.png">

- 엔티티 계층은 엔티티와 배류 오브젝트를 모두 담고 있음
- 만약 해당 요소들이 도메인의 비즈니스 로직 경계 내에 있다면 동일한 애그리게이트 

### 다른 애그리게이트 참조하기 
- 애그리게이트 내의 모든 객체는 같은 tx 경계를 공유하므로 애그리게이트가 너무 커지면 성능과 확장 문제 가능성 존재
- 궁극적으로 일관돼도 좋은 모든 정보는 애그리게이트 경계 밖에 다른 애그리게이트의 일부로 둔다. 
![image](https://user-images.githubusercontent.com/27190617/225536737-e3a11f70-8272-43fd-a1be-4471c7afb6fe.png)

```C++
public class Ticket
{
  private UserId _customer;
  private List<ProductId> _products;
  private UserId _assginedAgent;
  private List<Message> _messages;
}
```
- 내부 애그리게이트는 직접 참조, 외부 애그리게이트는 ID로 참조한다. 
- 내부 애그리게이트에 속하는 지 판단 방법
  - 비즈니스 로직 내 궁극적 일관된 데이터를 다루는 상황에서 시스템의 상태를 손상시키는 지 판단한다
  - 그리고 그 비즈니스 로직이 애그리게이트에 있는 지 여부 확인

### 애그리게이트 루트 
애그리게이트 상태는 커맨드 중 하나를 실행해서만 수정 가능 
- 애그리게이트 루트(애그리게이트 퍼블릭 인터페이스) : 애그리게이트가 엔티티의 계층 구조를 대표하는 경우 

### 도메인 이벤트
- 애그리게이트 퍼블릭 인터페이스 외 외부에서 애그리게이트와 커뮤니케이션 할 수 있는 방법 
- 정의 : 비즈니스 도메인에서 일어나는 중요한 이벤트를 설명하는 메시지 
- 도메인 이벤트는 이미 발생한 것을 나타내므로 과거형으로 명명 
- 목적 : 비즈니스 도메인에서 일어난 일 설명, 이벤트와 관련된 모든 필요 데이터 제공 
- 애그리게이트의 퍼블릭 인터페이스 중 일부 
- 다른 프로세스 및 애그리게이트 심지어 외부 시스템도 해당 도메인 이벤트 구독 가능 

### 도메인 서비스 
- 애그리게이트에도 밸류 오브젝트에도 속하지 않거나, 복수의 애그리게이트에 관련된 비즈니스 로직을 다룰 때 구현 
- 도메인 서비스는 비즈니스 로직을 구현한 상태가 없는 객체(Stateless object) 
- 대부분 로직은 어떤 계산 혹은 분석을 수행하기 위해 다양한 시스템 구성요소 호출 조율 (여러 애그리게이트 작업 조율 가능) 
- 그러나 한 개의 DB tx에서 한 개의 애그리게이트 인스턴스만 수정 가능한 제약은 도메인 서비스 또한 동일한 제약을 가짐
- 따라서 **도메인 서비스는 애그리게이트를 읽는 것에 대한 로직을 구현하는 데 사용**

