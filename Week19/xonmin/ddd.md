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

---

# 8. 아키텍처 패턴
- 코드베이스가 처리해야 할 다양한 관심사로 인해 비즈니스 로직이 다양한 구성요소에 흩어지기 쉬움
  - 사용자 인터페이스 / DB에 의해 구현 혹은 다양한 구성요소에 중복 
- 관심사 구현 시, 엄격히 구성하지 않는다면 코드베이스의 변경에 대한 제약


## 계층형 아키텍처
고전적 아키텍처 :  PL(Presentation layer) + BLL(Business logic) + DAL(Data access)

### 프로젠테이션 계층
- 사용자와 상호작용 하기 위한 인터페이스 구현
- ex) GUI/CLI/연동 API/메시지 브로커 이벤트 구독/이벤트 발행
- 프로그램의 퍼블릭 인터페이스

### 비즈니스 로직 계층
- 소프트웨어의 중심
- 액티브 레코드 또는 도메인 모델과 같은 비즈니스 로직 패턴 적용
- 엔티티 / 규칙 / 프로세스

### 데이터 접근 계층
- 영속성 메커니즘 접근 가능케 함
- 프로그램 기능 구현 시 필요한 다양한 외부 정보 제공자와 연동 
- DB / Message Bus / 오브젝트 스토리지 

### 계층 간 커뮤니케이션
- 계층은 톱다운 커뮤니케이션 모델 따라연동
- 각 계층은 아래 계층에만 의존(결합도 낮추고, 공유할 지식 줄일 수 있음) 

### 서비스 계층
- 정의 : 가용한 오퍼레이션 구축, 각 오퍼레이션에서 애플리케이션의 응답 조정
- ex) 상황에 따른 적절한 액션 - 에러 발생시, 적절한 응답 생성
- 아키텍처 패턴의 컨텍스트에서 서비스 계층은 논리적 경계 
- 하부 계층을 조율하는 데 필요한 것들을 감싸, 퍼블릭 인터페이스의 메서드에 상응하는 인터페이스 노출
  - 각 모든 메서드는 시스템의 퍼블릭 인터페이스 
```C++
interface CampaignManageMentService
{
  OperationResult CreateCampaign(CampaignDetails details);
  OperationResult Publish(CampaignId id, PublishingSchedule schedule);
  ...
}
```
- 장점 : 
  - 동일한 서비스 계층을 여러 퍼블릭 인터페이스에 재사용 가능
  - 모든 관련 메서드를 한 곳에 모으면 모듈화 개선
  - PL 과 BLL의 결합도 낮춤
  - 테스트 용이
- 사용할 때:
  - 액티브 레코드 패턴을 사용하는 경우와 같이 비즈니스 로직 패턴에서 외부 조율이 필요할 경우 
  - 이와 반대로, 비즈니스 로직이 tx 스크립트로 구현된 경우 이미 일련의 메서드가 노출되기 때문에 이것이 바로 서비스 계층 역할을 하는 셈

### 용어 
- PL = 사용자 인터페이스 계층
- Service = 애플리케이션 계층
- BLL = 도메인 계층 = 모델 계층
- DAL = 인프라스트럭처 계층

### 계층형 아키텍처가 필요한 경우
- 비즈니스 로직이 트랜잭션 스크립트 / 액티브 레코드 패턴을 사용하여 구현된 시스템 
- 반면 도메인 모델을 구현하는데 계층형 적용은 어려움
  - 도메인 모델에서 비즈니스 앤티티(애그리게이트/VO)가 하부의 infra에 대해 의존성이 없어야함 

## 포트와 어댑터
- PL + DAL = 결국 외부 구성요소와 연동 (인프라스트럭처 계층 통합 가능) 
- 핵심 목적 : 인프라스트럭처 구성요소로부터 시스템 비즈니스 로직간의 분리 


### DIP 
상위 수준의 모듈은 하위 수준의 모듈에 의존해서는 안 된다.
- but 전통적인 계층형 아키텍처 : 비즈니스 로직 계층이 infra에 의존하는 꼴 
  - `비즈니스 로직 계층 <- 인프라스트럭처 계층`
- 해결 방법 : 둘 간의 의존성 반대로 / 중간에 애플리케이션 계층 추가 
  -  애플리케이션 계층 : 시스템이 노출하는 모든 오퍼레이션 설명 / 이를 실행할 떄 시스템의 비즈니스 로직 조율

### 인프라 구성요소 연동 
- 포트 :application 입장에서 consumer, 또는 application에서 나가거나/들어오는 인터페이스 
  - 비즈니스 로직 계층에서 정의
- 어댑터 : 클라이언트에 제공해야 할 인터페이스를 따르면서도 내부 구현은 서버의 인터페이스로 위임
  - 인프라스트럭처 계층에서 구현 

```C++
namespace App.BusinessLogicLayer
{
  public interface Imessaging
  {
     void Publish(Message payload);
     void Subscribe(MEssage type, Action callback);
   }
 }

namespace App.Infrastructure.Adapters
{
  public class SQSBus : IMessaging { ... } 
 }
 ```


- 포트와 어댑터 아키텍처 = 헥사고날 아키텍처 = 어니언 아키텍처 = 클린 아키텍처로 알려짐
- 계층형 아키텍처와의 용어 비교
  - 애플리케이션 계층 = 서비스 계층 = 유시케이스 계층
  - 비즈니스 로직 계층 = 도메인 계층 = 핵심 계층

### 포트와 어댑터를 사용하는 경우
- 도메인 모델 패턴을 사용하여 구현한 비즈니스 로직에 매우 적합


# CQRS (Command-query Responsibility segregation) 패턴
- 포트와 어댑터와 동일한 비즈니스 로직과 인프라스트럭처 관심사에 기반
- 이 패턴을 통해 여러 영속 모델에 시스템의 데이터 표현 가능

## 폴리글랏 모델링
- OLTP / OLAP 에서는 시스템 데이터의 다양한 표현이 필요
- 여러 모델로 작업하는 또 다른 이유 : 다양한 언어를 사용하는 영속성 개념과 관련이 있다.
- 완벽한 데이터 베이스는 없기 떄문에 확장성이나 일관성 또는 지원하는 질의 모델 간에 균형이 필요하다 
- 폴리글랏 영속 모델 = 완전한 데이터 베이스의 대안 : 다양한 데이터베이스를 사용 

CQRS 패턴 = 이벤트 소싱과 밀접한 관련
- 한 번에 하나의 애그리게이트 인스턴스에 대한 이벤트 질의 가능 
- 프로젝션된 모델을 물리적 데이터베이스에 머터리얼라이즈하여 유연한 질의가 가능하도록 함
 
> 머터리얼라이즈 뷰 기능을 활용하여 빈번한 질의 결과를 물리 테이블에 저장하여 성능 개선

### 구현
1. 커멘드 실행 모델
- 시스템의 상태 수정 전담
- 비즈니스 로직 구현 / 규칙 검사 / 불변성 강화 
- 일관적 상태를 읽을 수 있어야 하며 갱신시, 낙관적 동시성 지원 

2. 읽기 모델(프로젝션)
- 데이터를 보여주고, 정보 제공을 위한 다양한 모델 제공
- 읽기 전용 (수정 불가)
- 읽기 모델이 작동하기 위해서는 시스템은 커멘드 실행 모델의 변경 -> 모든 읽기 모델로 프로젝션 해야함 
- 원천 테이블 갱신시, 변경사항은 미리 작성된 뷰에 반영되어야 함


### 동기식 프로젝션 
- 격차 해소 구독 모델(catch-up subscription model)을 통해 OLTP 데이터 변경사항 가져옴
  - DB로부터 마지막 처리했던 체크포인트 이후 추가 및 갱신 레코드 조회
  - 조회된 데이터를 통해 읽기 모델을 재생성 또는 갱신
  - 마지막 체크포인트 저장 
- 해당 모든 DB는 레코드를 체크포인트로 관리해야 한다. 
- SQL의 `rowversion`, 혹은 카운터 칼럼 

### 비동기식 프로젝션
- 커맨드 실행 모델은 모든 커밋된 변경사항을 메시지 버스에 발행 
- 시스템 프로젝션 엔진은 구독 & 읽기 모델 갱신 
- 확장성과 성능의 장점이 있지만, 분산 컴퓨팅 문제가 자주 발생할 것 
- 따라서 동기식 프로젝션 방식 구현이 좋다. 

### 모델 분리 
CQRS의 일반적 오해 : ~~커맨드 실행 메서드는 데이터를 반환해서는 안된다.~~ 
- **대부분의 경우 커맨드는 데이터를 반환해야한다.**
  - 유효성 검사 혹은 기술적 문제 등의 실패 이유 파악이 가능 

### CQRS를 사용해야 하는 경우
- 다양한 종류의 DB에 저장된 동일한 데이터를 사용할 경우
- 이벤트 소싱 도메인 모델에 적합 
- 이벤트 소싱 모델에서는 애그리게이트의 상태에 기반한 레코드 조회가 불가능하지만, CQRS에서는 상태를 질의할 수 있는 DB에 상태를 프로젝션 (?) 무슨말이냐 

보면 좋은 것들 : 
- https://github.com/xonmin/TIL/blob/master/DesignData-Intensive/chap11.md
- https://www.youtube.com/watch?v=fg5xbs59Lro
