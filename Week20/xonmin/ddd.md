# 09. 커뮤니케이션 패턴 
- 바운디드 컨텍스트 간 커뮤니케이션을 용이하게 
- 애그리게이트 설계 원칙의 제한 사항 해결, 여러 시스템 컴포넌트에 걸쳐 비즈니스 프로세스 조율 역할

## 모델 변환
- 바운디드 컨텍스트 : 유비쿼터스 언어 모델의 경계 
- 만약 다른 바운디드 컨텍스트가 협력할 의향이 있다면?
  - 바운디드 컨텍스트를 파트너십을 통해 통합 가능 : 프로토콜 조정 및 팀간 커뮤니케이션으로 해결 가능
  - 협력 기반 통합 방법 : 공유 커널 방법 사용 (제한된 부분을 공동으로 함께 발전시킴) 

- 업스트림(제공자) / 다운스트림(사용자)
- 이 때 다운스트림이 업스트림의 바운디드 컨텍스트 모델을 따를 수 없을 때 처리 방법은 2가지

1. 다운스트림 측에서 ACL(충돌방지계층) 사용하여 업스트림 모델을 조정하여 사용 
2. 업스트림 측에서 OHS(오픈호스트서비스) 역할을 통해 구현 모델에 대한 병경으로부터 사용자 보호 

그렇다면 모델 변환 로직은? 

### Stateless 모델 변환 
스테이트리스 모델 변환을 소유하는 바운디드 컨텍스트는 **Proxy 디자인 패턴**을 통해 모델 매핑처리
- 업스트림의 경우 : OHS(수신)
- 다운스트림의 경우 : ACL(발신) 

만약 업스트림에서 힘이 없어서 업스트림에서 변환 처리 해줘야 하는 상황
- 다운스트림 -> (요청) -> Proxy(OHS) -> 업스트림 

다운스트림에서 힘이 없어서 본인이 직접 변환 처리 해야해
- 다운스트림 <- (응답) <- Proxy(ACL) <- 업스트림

**StateLess 방식의 동기/비동기 방식**

동기
- OHS : 공용 언어로의 변환은 유입되는 요청을 처리할 때 발생 
- ACL : 업스트림 바운디드 컨텍스트를 호출할 때 발생

- 경우에 따라 변환 로직을 **API 게이트웨이 패턴**과 같은 외부 컴포넌트로 넘기는 것이 효율적일 수 있음 
- OHS 를 구현하는 바운디드 컨텍스트의 경우(업스트림) API게이트웨이는 내부 모델을 통합에 최적화된 공표 언어로 각각 다운스트림 별로 변환하는 역할 수행
- ACL 를 API게이트웨이 패턴을 사용하는 경우 어차피 다운스트림에서 순응하기 떄문에 ACL은 통합 관련 바운디드 컨텍스트 역할을 한다. 
- 이처럼 바운디드 컨텍스트를 편리하게 모델 변환하는 역할을 하므로, **교환 컨텍스트**라고도 명함 


비동기
- OHS 구현시 비동기식 모델 변환은 반드시 필요
- 비동기 변환 사용시 도메인 이벤트를 가로채서 공표된 언어 변환할 수 있으므로 바운디드 컨텍스트의 구현상세를 더 잘 캡슐화할 수 있다(?)
- 메시지를 공표된 언어로 변환시 private event와 public event(다른 바운디드 컨텍스트 연동을 위한) 구분 가능 

**StateFull 모델 변환**
- 데이터 집계(일괄 처리 / 이벤트 통합)시 사용
- 데이터 집계를 통한 모델 변환은 API 게이트 웨이를 사용하여 구현할 수 없음 
- 따라서 들어오는 데이터 추적 및 처리에 대한 변환 로직에 대해 자체 영구 저장소가 필요함 (Proxy 에 별도 storage 필요) 
- 요청 통합에 대한 방법
  - BackEnd-for-frontEnd Pattern 사용 : 사용자 인터페이스가 여러 서비스에서 발생하는 데이터 결합
  - ACL 을 바운디드 컨텍스트 전면에 배치하여 바운디드 컨텍스트 연동과 비즈니스 로직의 복잡성 분리 (ex_ 여러 다른 데이터 처리/복잡한 비즈니스 로직 구현해야하는 바운디드 컨텍스트) 
    
## 애그리게이트 연동
- 애그리게이트에서 바로 도메인 이벤트 발행은 좋지 않음
  - 1. 애그리게이트의 새로운 상태가 DB에 커밋 전에 이벤트가 전달된다. 
  - 2. 경합 조건 : 후속 애그리게이트 로직에 인해 작어비 무효화되거나 단순 DB 문제로 인해 Tx가 커밋되지 않을 수 있음 

- 그렇다면 도메인 이벤트 발행 책임을 어플리케이션 계층으로 이전한다면 (인스턴스 로드 -> 갱신된 상태 DB 커밋 -> 이벤트 발행) 이라면 괜찮을까?
  - 메시지 버스 다운 혹은 이벤트 발행 전 로직 다운 등의 이유로 일관성없을 수 있음 
- **아웃박스 패턴**을 사용해보자 

## 아웃박스 패턴 
도메인 이벤트의 안정적인 발행 보장 
- 업데이트된 애그리게이트의 상태와 도메인 이벤트 모두 동일한 원자성 트랜잭션으로 커밋
- 메시지 릴레이는 DB에서 새로 커밋된 도메인 이벤트를 가져옴 
- 릴레이는 도메인 이벤트를 메시지 버스에 발행
- 성공적으로 발행 시, 릴레이는 이벤트를 DB에 발행한 것으로 표시 / 삭제 
![image](https://user-images.githubusercontent.com/27190617/227963735-77ef7856-707c-4ce9-86cd-d9e9a2752698.png)

- 다중 문서를 지원하지 않는 NoSQL은 전달된 도메인 이벤트는 애그리게이트 레코드에 포함되어야 함 

### 발행되지 않은 이벤트 가져오기 
Pull 방식 (발행자 폴링) 
- 릴레이는 발행되지 않은 이벤트에 대해 DB를 지속해서 질의 가능
- 지속적인 폴링으로 인한 DB 부하를 최소화하기 위해서는 적절한 인덱스 필요 

Push 방식(트랜잭션 로그 추적) 
- DB 기능을 이용하여 새 이벤트가 추가될 떄마다 발행 릴레이를 호출 가능 
- ex) DB 트랜잭션 로그를 추적하여 업데이트/삽입된 레코드에 대한 알림을 받을 수 있음 

아웃박스 패턴은 적어도 한 번은 메시지 배달을 보장한다는 점 유의 
- 메시지 발행한 후 릴레이가 실패했지만 DB에 발행한 것으로 표시하기 전에, 릴레이 실패한다면 다음 이터레이션에서 같은 메시지가 다시 발행 

## SAGA 패턴 
- 핵심 애그리게이트 설계 원칙 중 하나 : 각 트랜잭션을 애그리게이트의 단일 인스턴스로 제한 
- 애그리게이트의 경계를 신중하게 고려하고 응집된 비즈니스 기능 집합을 캡슐화 할 수 있음 
- 하지만, 여러 애그리게이트에 걸쳐있는 비즈니스 프로세스를 구현해야 하는 경우가 있음

책임이 분리되어있는 바운디드 컨텍스트에 속할 수 있는 부념ㅇ히 다른 비즈니스 엔티티에 대해 다른 바운디드 컨텍스트에 속할 수 있는 분명히 다른 비즈니스 엔티티이기 때문이다. 
- 이러한 흐름은 사가로 구현할 수 있다.
- 사가는 트랜잭션 측면에서 보는 것 
- 즉, 여러 트랜잭션에 걸쳐 있는 비즈니스 프로세스 
- 사가는 관련 컴포넌트에서 발생하는 이벤트를 수신, 다른 컴포넌트에 후속 커맨드를 실행 
- 실행 단계 중 하나가 실패하면 사가는 시스템 상태를 일관되게 유지하도록 적절한 보상 조치를 내림 

![image](https://user-images.githubusercontent.com/27190617/227967584-87dd82cf-9ab6-41b6-99df-143aeafab6ad.png)
- 발행 프로세스를 구현하기 위해 사가는 Campaign 애그리게이트로부터 CampaignActivated Event, AdPublishing 바운디드 컨텍스트로부터 Publishing 이벤트를 기다린다. 
- 이벤트를 전달하고 관련 커멘드를 실행하여 이벤트에 반응 
- 사가는 상태 관리 필요의 유무에 따라 달라진다. 
  - 상태 관리가 필요한 경우 : 실행된 작업을 추적하여 실패 시 적절한 보상 조치를 실행할 수 있음 
  - 이러한 경우, 사가는 이벤트 소싱 애그리게이트로 구현하여 수신된 이벤트와 실행된 커멘드의 전체 기록 유지 가능 
  - 그러나 커맨드 실행 로직은 도메인 이벤트가 아웃박스 패턴으로 전달하는 방식과 유사하게 사가 패턴 자체에서 벗어나 비동기적으로 실행한다. 

### 일관성
- 사가 패턴이 다중 컴포넌트의 트랜잭션을 조율하기는 하지만, 관련된 컴포넌트의 상태는 궁극적으로 일관성을 가짐 
- 사가가 결국 관련 커맨드를 실행한다고 해도 두 개의 트랜잭션은 원자적으로 간주되지 않기 떄문에 모두 성공하거나 실패할 수 없다. 
> 애그리게이트 경계 내의 데이터만 강한 일관성을 가진다. 외부의 모든 것은 궁극적으로 일관성을 갖는다. 
- 이러한 부적절한 애그리게이트 경계를 보상하기 위해 사가를 남용하지 않는 지침을 따르는 것을 원칙으로 하자 
- 동일한 애그리게이트에 속해야 하는 비즈니스 작업에는 강한 일관성을 갖는 데이터가 필요

## 프로세스 관리자 패턴 
- 사가 패턴은 단순하고 선형적인 흐름을 관리 (사가는 이벤트를 해당 커멘드와 일치시킴) 
- 구현 관점에서는 상태 기반 혹은 이벤트 소싱 애그리게이트로 구현 
- 프로세스 관리자는 시퀀스의 상태를 유지하고 다음 처리 단계를 결정하는 중앙 처리 장치로 정의한다.
![image](https://user-images.githubusercontent.com/27190617/227971320-00acb946-7bed-4a0a-adc4-de0e2821a4a5.png)

- 경험에 비추어 볼 때 사가에 `if-else` 문이 포함되어 있다면 아마도 프로세스 관리자일 것 
- 프로세스 관리자와 사가의 차이점 
  - 사가는 특정 이벤트가 관찰될 떄 암시적으로 인스턴스화 
  - 프로세스 관리자 : 단일 소스 이벤트에 바인딩될 수 없음 (여러 단계로 구성된 응집된 비즈니스 프로세스) : 따라서 명시적으로 인스턴스화 
![image](https://user-images.githubusercontent.com/27190617/227972328-471a822e-5f67-403e-ad86-2eaf2187666a.png)

- 출장 예약 프로세스를 트리거하는 중앙 엔티티가 없으며, 출장 예약은 프로세스 이기 때문에 프로세스 관리자로 구현 


## 결론 
- ACL, OHS 를 구현하는데 사용할 수 있는 모델 변환 패턴 (즉석 변환 혹은 상태 추적이 필요한 경우 좀 더 복잡한 로직으로 구현) 
- 아웃박스 패턴 : 애그리게이트의 도메인 이벤트 발행하는 안정적인 방법 (다른 프로세스가 실패해도 도메인 이벤트 항상 발행) 
- 사가 패턴 : 간단한 교차 컴포넌트 비즈니스 프로세스 구현에 사용 
- 프로세스 관리자 패턴 : 좀 더 복잡한 비즈니스 프로세스 구현 가능
  - 두 패턴 모두 도메인 이벤트에 대한 비동기식 반응과 커맨드 실행에 의존 

---

# 10장. 휴리스틱 

휴리스틱 : 다양한 소프트웨어 설계 의사결정, 도메인 주도 설계를 돕는 분석 도구로 사용 

## 휴리스틱
- 모든 상황에 맞게 보자오디고 수학적으로 검증된 규칙이 아님
- 당면한 목적에 충분할 만큼만의 경험에 기반한 규칙 
- 수많은 단서에 대해서 내재된 노이즈 무시, 가장 중요한 단서에서 느껴지는 것에 집중하여 효과적으로 문제를 해결하는 방법 

### 바운디드 컨텍스트 
- 최적의 바운디드 컨텍스트 크기는 무엇인가 

> 저자 친구 닉튠 왈 : 한 서비스의 경계를 정의하는데 유용한 휴리스틱은 많지만, 그중 크기로 경계를 구분하는 것은 가장 도움이 되지 않는다. 

- 작은 바운디드 컨텍스트로 만들려고 기능을 원하는 크기에 최적화해서 모델링하지마라
- 모델의 어떤 기능이 포함하는 크기 그대로 바운디드 컨텍스트를 정의할 것 
- 여러 바운디드 컨텍스트에 영향을 미치는 변경이 단일 바운디드 컨텍스트 범위 내에 존재하지 않는 다면, 이는 컨텍스트 경계의 설계가 효과적이지 않다는 신호 
- 바운디드 컨텍스트의 경계를 무효화하는 변경(즉 여러 바운디드에 영향)은 일반적으로 비즈니스 도메인이 잘 알려지지 않거나, 비즈닛그 요구사항이 빈번하게 바뀔 때 발생 
- 바운디드 컨텍스트의 경계를 설계할 떄 이런 특성에 **휴리스틱** 사용 가능 


- 넓은 바운디드 컨텍스트의 경계는 그 경계/ 하위 도메인을 포함하는 모델이 잘못돼도 안전하게 해줌
- 논리적 경계를 리팩터링하는 것이 물리적 경계를 리팩터링하는 것보다 적은 비용이 든다. 
- 그러므로 **바운디드 컨텍스트 설계 : 경계를 넓게 해서 시작**하고, 필요에 따라 추후에 작은 여러 경계로 쪼개자 
- 정형화되지 않고, 변동성이 많은 핵심 하위 도메인을 포함하는 바운디드 컨텍스트에 휴리스틱이 보통 적용된다. 
  - 여러 하위 도메인과의 상호작용이 많기 때문에 휴리스틱을 통해 스스로를 보호할 수 있음(왜냐 넓자나) 

### 비즈니스 로직 구현 패턴 
이전 장에서 배운 비즈니스 로직 모델링 4가지 패턴 
- 간단한 비즈니스 로직 포함하는 하위 도메인에 적합 
  - 트랜잭션 스크립트 : 자료구조 단순
  - 액티브 레코드 패턴 : 복잡한 자료구조를 하부 DB에 매핑에 도움(캡슐화) 
- 복잡한 비즈니스 로직 핵심 하위 도메인에 적합
  - 도메인 모델
  - 이벤트 소싱 도메인 모델 

적절한 구현 패턴 선택에 대해 효과적인 휴리스틱 의사결정
- 비즈니스 로직과 그 자료구조의 복잡성에 따라 비즈니스 로직 구현 패턴을 결정하는 것은 하위 도메인의 유형에 대한 가정을 검증하는 방법 

### 아키텍처 패턴 
![image](https://user-images.githubusercontent.com/27190617/228573985-b4aa6b0a-3dc3-4e33-b868-71e0b805b2ae.png)
- 이벤트 소싱 도메인 모델은 CQRS가 필요 
  - else) 데이터 질의 옵션이 제한되어 id만으로 단일 인스턴스르 가져와야 한다 (다른 옵션을 쓸수 없다는 얘기겠지)
- 도메인 모델은 포트/어댑터 아키텍처 필요 
  - 계층형에서는 영속성 고려 없이 애그리게이트와 밸류 오브젝트를 만들기 어려움 
- 액티브 레코드 패턴 : 서비스 계층을 추가한 계층형 아키텍처와 잘 어울림(액티브레코드 제어 로직) 
- tx 스크립트 패턴 : 최소한의 계층형으로 구현 가능 

## 테스트 전략 
![image](https://user-images.githubusercontent.com/27190617/228574043-86fe3e90-f3b5-44b4-b287-672d9add201b.png)
### 피라미드형 테스트 
- 단위 테스트 강조 
- 애그리게이트와 밸류 오브젝트 도메인 모델 패턴에 모두 잘 지원 ((이벤트소싱)도메인모델 - 비즈니스 로직 테스트에 완벽) 

### 다이아몬드형 테스트
- 통합에 집중
- 액티브 레코드 패턴 사용시 : 애플리케이션 계층 테스트를 통해 아주 효과적

### 역전된 피라미드형 테스트
- 앤드 투 앤드 테스트에 집중 (처음부터 끝까지 애플리케이션 워크플로를 검증) 
- tx 스크립트 패턴에 잘 어울림(계층 수가 적고 비즈니스 로직이 간단) 

## 아키텍처 패턴 의사결정 트리 
![image](https://user-images.githubusercontent.com/27190617/228574214-476c608d-7ba5-4f22-a503-e4c8ac17f35b.png)

## 전술적 설계 의사결정 트리 (최종본) 
![image](https://user-images.githubusercontent.com/27190617/228574120-aa7f79e0-58c1-4fcd-bea3-4cbd195e3bc2.png)


