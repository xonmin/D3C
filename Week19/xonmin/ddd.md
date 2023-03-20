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



