# 7. 시간 차원의 모델링

- 이벤트 소싱 도메인 모델 패턴 : 도메인 모델 패턴과 동일한 전술적 패턴(밸류 오브젝트, 애그리게이트, 도메인 이벤트)을 사용 
- 차이점 : 애그리게이트의 상태를 저장하는 방식이 다름 
  - 이벤트 소싱 도메인 모델 : 이벤트 소싱 패턴을 사용하여 애그리게이트 상태 관리
  - 애그리게이트의 상태를 유지하는 것이 아닌 각 모델에 대한 변경사항에 대한 도메인 이벤트 생성, 애그리게이트 데이터에 대한 원천 데이터로 사용 
 
 ## 이벤트 소싱
- 현재 상태만 저장되어 있다면, 그 이전에 어떤 일이 발생했는지 분석 불가 
- 이벤트 소싱 패턴 : 데이터 모델에 시간 차원을 도입 
- 이벤트 소싱 기반 시스템은 애그리게이트의 수명주기의 모든 변경사항 이벤트 유지 
- 상태에 대해 도메인 이벤트로부터 쉽게 프로젝션 가능

> 프로젝션 : 쓰기 모델을 통해 이벤트 소싱 시스템에 저장된 데이터를 다양한 읽기 모델을 적용해 원하는 시점의 데이터를 추출 

```C++
public class LeadSearchModelProjection {
  public long LeadId { get; private set; }
  public HashSet<string> FirstNames {}
  public HashSet<PhoneNumber> PhoneNumbers {get; private set; }
  public int Version { get; private set; }
  
  public void Apply(LeadInitialized @event){
    Version = 0;
  }
  public void Apply(ContactDetails @event){
    Version += 1;
  }
  ...
```

### 검색
- 검색 기능 구현해야 할 때
- 이벤트 소싱을 사용하여 과거 정보를 쉽게 프로젝션 가능 
  - `LeadId`를 통해 검색
  - `Version`만 업데이트된다. 
- 다른 이벤트는 특정 모델의 상태에 영향을 미치지 않는다. 

### 분석

- 분석하고자 하는 필드에 대해 버전 변경에 따른 분석이 가능하다. 
- 대부분의 시스템의 필요한 기능을 구현하려면 프로젝션된 모델을 DB에 유지해야 함 
  - CQRS(command-query responsibility segregation; 명령과 조회 책임 분리) 패턴을 통해 가능

### 원천 데이터 
- 이벤트 소싱 패턴이 작동하려면 객체 상태에 대한 모든 변경사항이 이벤트로 표현되고 저장되어야 함 
- 여기서 이벤트는 **시스템의 원천 데이터의 역할**
![image](https://user-images.githubusercontent.com/27190617/226333797-0de4b1f4-3fab-4893-b579-61eaa53a24b5.png)

### 이벤트 스토어 
- 시스템의 이벤트를 저장하는 DB 
- 이벤트 스토어는 추가만 가능한 저장소 (수정 및 삭제 불가)
- 이벤트 스토어의 최소한 기능 지원 
  1. 특정 비즈니스에 속한 모든 이벤트 호출
  2. 이벤트 추가 
  
```C++
interface IEEventStore {
  IEnumerable<Event> Fetch(Guid instanceId);
  void Append(Guid instanceId, Event[] newEvents, int expectedVerison);
}
```

- `expectedVerison` param의 경우 낙관적 동시성 제어 구현을 위해 필요 

### 이벤트 소싱 도메인 모델
- 도메인 모델 : 애그리게이트의 상태 표현 방식 유지/관리/선택된 도메인 이벤트 내보냄
- 이벤트 소싱 도메인 모델 : 애그리게이트의 수명주기를 모델링하기 위해 독점적으로 도메인 이벤트 사용 

이벤트 소싱 애그리게이트 task
1. 애그리게이트의 도메인 이벤트 로드
2. 이벤트를 비즈니스 의사결정을 내리는데 사용할 수 있게 프로젝션하여 상태 표현 재구성
3. 애그리게이트의 명령을 실행하여 비즈니스 로직 실행, 결과적으로 새로운 도메인 이벤트 생성
4. 새 도메인 이벤트를 이벤트 스토어에 커밋

```C++
public class TicketAPI {
  private ITicketRepository _ticketREpository;
  ...
  
  public void RequestEscalation(TicketId id, EscalationReason reason) {
    var events = _ticketsRepository.LoadEvents(id); // event 로드
    var ticket = new Ticket(events); // 리하이드레이션 & 재구성
    var originalVersion = ticket.Version;
    var cmd = new RequestEscalation(reason);
    // 비즈니스 로직 실행 및 새로운 도메인 이벤트 생성 
    ticket.Execute(cmd);
    // commit
    _ticketsRepository.CommitChanges(ticket, originalVersion);
   }
 }
 ```
 
### 이벤트 소싱 도메인 모델의 장점
- 모든 과거 상태 확인 및 복원 가능
  - 유스케이스 : 소급 디버깅(retoactive debugging)
- 시스템의 상태와 동작에 대한 통찰력 제공 / 새로운 프로젝션 방법 추가 가능
- 감사 로그 
- 고급 낙관적 동시성 제어 : 읽기 데이터 기록동안 다른 프로세스에 의해 덮어 쓰여지는 경우 예외 발생 

### 단점
- 학습 곡선 
- 모델의 발전이 어려움 
- 아키텍처 복잡성 (유동적인 부분 도입) 

    
### 성능
- 시스템에서 애그리게이트당 10000개 이상의 이벤트가 있을 경우, 성능 저하가 가시적
- 그러나 대다수의 시스템에서 애그리게이트의 평균 수명은 100개 이벤트를 초과하지 않음
- 상태를 프로젝션하는 것이 성능에 문제가 되는 경우는 많지 않지만, 스냅숏 패턴과 같은 다른 패턴 적용 가능

### 이벤트 소싱 모델의 확장
- 쉬운 확장성
- 모든 애그리게이트 관련 작업 : 단일 애그리게이트 컨텍스트에서 수행 
- 따라서 이벤트 스토어는 애그리게이트 ID로 분할 가능 
- 애그리게이트의 인스턴스에 속하는 모든 이벤트는 단일 샤드에 존재해야 함 

### 데이터 삭제 
이벤트 스토어에서 물리적으로 데이터를 삭제해야하는 경우 
- 페이로드 패턴을 통해 민감 민감 정보를 암호화하는 K-V 저장소에서 암호화 키를 삭제하여 해당 정보에 접근할 수 없도록 해결

### 해당 패턴의 용례
- 분석 및 최적화
- 법적으로 감사 로그를 요구할 경우 
