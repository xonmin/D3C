# 03. 도메인 복잡성 관리 

- 도메인에 대한 모든 이해관계자가 의사소통에 사용할 수 있는 유비쿼터스 언어를 개발하는 것이 중요
- 내부 동작과 기본원칙에 대한 도메인 전문가의 멘탈 모델을 반영해야 함 
- 그러나 조직 규모에 따라 멘탈 모델은 일관성이 없을 수 있음

### 일관성 없는 모델 

본 책에서는 `lead(리드)`라는 용어가 마케팅 부서와 영업 부서에서 다른 의미로 사용되는 예시를 보여준다. 

마케팅 부서 
- 리드 : 누군가가 제품 중 하나에 관심이 있다는 이벤트(알림)

영업 부서 
- 리드 : 영업 프로세스의 전체 수명주기 (장기적 진행 과정)

따라서 두 부서의 도메인 전문가 사이에는 '리드'의 멘탈 모델은 차이가 존재
- 이는 실제로 다른 부서의 사람들 사이에서 의사소통이 어려울 수 있지만, 
- 사람들이 서로 상호작용하면서 정확한 의미를 추론하는 것은 쉽다(?) - p36 
   - 문맥적으로 글이 매끄럽지 않음 이해 못하겠다.

- 다양한 비즈니스 모델은 사람들간의 상호작용에서 이해하기 쉽지만, SW로 표현하는 것은 더 어려움
  - **소스 코드는 모호성에 잘 대처하지 못한다.**

### 이러한 유비쿼터스 언어 공식화에 대한 전통적인 솔루션 방법들 

1. 모든 종류의 문제에 대해 단일 모델을 설계하는 방식 

- 모든 종류에 대해 단일 모델을 생성하므로, 문제 해결에 대한 다양한 복잡성이 존재한다. 
  - 관련 없는 세부사항 필터링 복잡성 
  - 필요한 것을 찾는 복잡성
  - 데이터 일관성 유지에 대한 복잡성 

> '이것저것 다 잘하는 사람은 단 한 분야에서도 달인이 될 수 없다'

2. 문맥에 따른 용어 앞 접두사(prefix) 추가 방식
- 해당 방법은 두 모델을 코드로써 생성 가능하게 함 
- 단점 : 
  - 1. 인지 부하 발생 : 각 모델을 언제 사용해야 하는 지, 구현할 수록 실수 유발 가능성 높아짐 
  - 2. 모델의 구현이 유비쿼터스 언어와 일치하지 않음 : 커뮤니케이션에서 아무도 접두사를 사용하지 않음 

해당 딜레마를 해결해 줄 방법 : 도메인 주도 설계 패턴 - **바운디드 컨텍스트**


## Bounded Context 
바운디드 컨텍스트

- 유비쿼터스 언어를 여러 작은 언어로 나눠 각 언어를 적용할 수 있는 명시적인 바운디드 컨텍스트에 할당 
- 각 바운디드 컨텍스트에서 단일 의미를 가지며, 이는 세분화된 유비쿼터스 언어 각각 일관성을 띄게 된다 
- 바운디드 컨텍스트 : 유비쿼터스 언어의 일관성이 유지되는 경계 
- **기대효과 : 컨텍스트를 명시적이고 중요한 비즈니스 도메인의 요소로 모델링 가능**

### 모델 경계 
- 모델 : 복잡한 시스템을 이해하는데 도움을 주기 위한 구조화 
- 우리가 해결하려는 문제 = 모델 본연의 목적 
- 모델의 경계(바운디드 컨텍스트)를 정의하는 것 - 모델링 프로세스의 본질적 부분 

### 정제된 유비쿼터스 언어 
- 유비쿼터스 언어는 바운디드 컨텍스트 경계 안에서만 보편적으로 적용
- 유비쿼터스 언어는 바운디드 컨텍스트 포함된 모델을 설명하는 데만 집중 

### 바운디드 컨텍스트의 범위 
- 서로 다른 도메인 전문가들은 동일한 비즈니스 엔티티에 대해 상충되는 멘탈 모델을 가지고 있음 
- 유비 쿼터스 언어의 일관성은 해당 언어의 가장 넓은 경계를 식별하는 데 도움이 될 뿐 
  - 일관성이 없는 모델과 용어가 있기 때문에 더 커질 수 없음 
- 하지만, 바운디드 컨텍스트를 기준으로 분해는 가능

[사진] 그림 3-4 


바운디드 컨텍스트 정의 
- = 유비쿼터스 언어의 범위를 정의하는 것 
- **전략적인 설계 의사 결정**
- 바운디드 컨텍스트의 크기 자체는 의사결정 요소는 아니다 
  - 유비쿼터스 언어의 경계는 넓을수록 일관성을 유지하기는 어려움 
  - 바운디드 컨텍스트를 괜히 작게 만들기 위해 노력하는 것은 설계를 통합하는데 오버헤드가 커질 수 있다. 
- 따라서 컨텍스트의 크기 결정 : 문제 도메인이 무엇이냐에 따라 달라진다. 

세분화된 바운디드 컨텍스트 추출 이유
- 일부 컴포넌트 개발 수명주기 분리
  - 새로운 SW 팀 구성
  - 시스템 일부 비기능 요구사항 해결
- 바운디드 컨텍스트의 나머지 기능과 독립적으로 확장할 수 있기 때문에 
- 모델을 유용하게 유지하고 바운디드 컨텍스트의 크기를 비즈니스 요구사항과 조직의 제약사항에 맞춰라 


## 바운디드 컨텍스트 vs 하위 도메인 

비즈니스 도메인은 두가지 기준으로 분해 가능
- 여러 하위 도메인 (세분화된 문제 도메인)
- 바운디드 컨텍스트 집합

### 하위 도메인 
- 조직이 일하고 경쟁 전략을 계획하는 방법이 결정된다. 
- 정의는 : 비즈니스 담당자가 담당
  - 소프트웨어 엔지니어는 하위 도메인을 식별하기 위해 비즈니스 도메인을 분석할 뿐 

### 바운디드 컨텍스트 
- SW 엔지니어에 의해 설계 된다. 
- 모델의 경계 선택 = 전략적 설계의 의사결정 

### 하위도메인과 바운디드 컨텍스트 상호작용 
- 소규모 시스템 : 단일 모델에 전체 비즈니스 도메인에 적용가능 (모놀리식 바운디드 컨텍스트)
  - 하지만 좀 비현실적 
- 모델이 여전히 크고 유지보수가 어려운 경우 : 더 작은 바운디드 컨텍스트로 분해 가능
- = 각 하위 도메인에 대한 바운디드 컨텍스트로 나눌 수 있음 
- 바운디드 컨텍스트와 하위 도메인의 설계가 1:1 매핑
  - 유연성 하락
  - 바운디드 컨텍스트 안에서 하나의 하위 도메인 모델만 사용된다

하위 도메인
- 발견되는 것 
- 비즈니스 전략에 의해 정의 

바운디드 컨텍스트 
- 설계하는 것 
- SW 엔지니어가 특정 프로젝트의 컨텍스트와 제약 조건을 해결하기 위해 설계 

## 경계 

바운디드 컨텍스트 패턴 : 물리적 경계 / 소유권 경계를 규정하기 위한 도메인 주도 설계 도구 

### 물리적 경계 
- 바운디드 컨텍스트 : 모델 경계 + 구현하는 시스템의 물리적 경계 역할 
- 즉, 구현, 버전 관리 등에 대해 각각의 다른 바운디드 컨텍스트와 독립적 실행
- 바운디드 컨텍스트는 여러 하위 도메인을 포함할 수 있다. 
- **바운디드 컨텍스트 = 물리적 경계 / 하위 도메인 = 논리적 경계**

### 소유권 경계
- 바운디드 컨텍스트 : 한 팀에서만 구현, 발전, 유지 관리해야 함 
- 팀과 바운디드 컨텍스트 간의 관계 = 단방향
  - 단일 팀이 여러 바운디드 컨텍스트를 소유하는 것은 가능 

## 실생활의 바운디드 컨텍스트 

### 시맨틱 도메인(semantic domain)
- 의미 영역과 해당 의미를 전달하기 위해 사용하는 단어 영역으로 구분 
- ex) monitor, port, processor 
- 모델 : 당면한 작업과 관련 없는 정보는 생략해야 한다 

## 결론 
- 유비쿼터스 언어는 여러 바운디드 컨텍스트로 분해해야 한다.
- 유비쿼터스 언어 : 바운디드 컨텍스트의 범위 내에서 일관성을 가져야 함 
- 그러나 서로 다른 바운디드 컨텍스트에서는 동일한 용어라도 다른 의미를 가질 수 있다 
- task : 1) 하위 도메인이 발견되면 2) 바운디드 컨텍스트도 설계
- 바운디드 컨텍스트 & 유비쿼터스 언어는 한 팀에서 만들고 유지보수 
- 바운디드 컨텍스트는 서비스, 하위 시스템 등의 물리적 구성요소로 분해 
- 각 바운디드 컨텍스트의 수명주기는 서로 독립적 