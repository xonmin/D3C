<img width="837" alt="image" src="https://github.com/xonmin/D3C/assets/27190617/3f188e0a-a6de-429c-97f7-be57bd99cc4d"># Chapter 07. 상태 저장 스트리밍 

대부분의 애플리케이션은 처리 요건에 있어, state(상태)를 유지해야 하는 경우가 있음. 
따라서 상태 저장 스트리밍은 이벤트 기반 MSA에 있어 가장 중요한 근간이다. 

## 7.1 상태 저장소, 이벤트 스트림에서 상태 구체화 
- materialized state(구체화 상태) : 이벤트 스트림의 이벤트를 투영(projection)한 것. (불변적)
  - MSA 에서 공통 비즈니스 엔티티로 사용할 수 있음. 
- state store(상태 저장소) : 서비스 내에 비즈니스 상태를 저장하는 곳 (가변적)
  - MSA 에서 비즈니스 상태 및 중간 계산 결과를 저장해놓을 수 있음.

 MSA 설계시, 일반적으로 데이터의 상태를 저장 및 접근하는 방법
1. 내부 저장소 사용 : 처리기(비즈니스 로직 처리)와 동일한 컨테이너의 메모리 or disk 에 데이터 내부 저장
2. 외부 저장소 사용 : 처리기 컨테이너 밖에 있는 외부 스토리지 서비스에 network 를 통해 데이터 저장 

<img width="500" alt="image" src="https://github.com/xonmin/D3C/assets/27190617/1184dfde-03c2-44f0-8b3d-ca661a40cc26">


## 7.2 체인지로그 이벤트 스트림에 상태 기록 

상태 저장소의 데이터 변경 사항 기록 사항을 담은 체인지 로그
- 해당 체인지로그는 오프셋 or 타임스탬프를 통해 현재 스냅샷을 가진 상태저장소의 변화를 이벤트 스트림으로 변환하는데 이용 가능 (테이블-스트림 이원성)
- 체인지로그를 통해 상태를 재구성하거나 이벤트 처리 체크포인트 수단으로 활용 가능

 ![image](https://github.com/xonmin/D3C/assets/27190617/e8a47514-371b-474e-be82-3e5787babca3)

> 카프카 스트림즈는 체인지 로그를 기본적으로 제공하고 있지만, 일반적인 프로듀서/컨슈머 클라이언트는 자체 구현해야함.

![image](https://github.com/xonmin/D3C/assets/27190617/4908d202-d5f4-48aa-b87d-c934abb608ec)

## 7.3 내부상태 저장소에 상태 구체화 
앞서 설명한 바와 같이 내부(로컬) 상태 저장소는 MSA 비즈니스 로직(처리기)과 동일한 컨테이너 or VM 에 둔다.
각 MSA 인스턴스는 자신에게 할당된 이벤트 스트림에서 할당된 파티션에서 컨슈밍 후 이벤트를 내부에 구체화 (내부 처리기에 Entity or Data class) 로 구체화하여
상태 저장소 내부에서 개별 파티션 별로 구체화된 데이터를 논리적으로 분리한다. 
- ex. key 값으로 데이터 논리적 분리 (vehicleId)

만약 MSA 컨슈머 그룹에 대해서 rebalance 가 일어난다고 가정
- 해당 컨슈머 그룹에 대한 특정 파티션은 하나의 컨슈머 인스턴스만 가지고 있기 때문에 리밸런스 이전 컨슈머는 기존 파티션으로 들어오던 데이터들에 대한 상태는 제거 (for 진실공급원 문제 발생 방지)
- 리밸랜스 이후 해당 파티션을 할당한 컨슈머에서만 해당 이벤트 발행

> 내부 상태 저장소로는 SSD 최적화된 RocksDB 와 같은 고성능 K-V 저장소가 주로 쓰인다.
> RDB 혹은 Document Storage 로 내부 저장소를 구현한 사례는 현재 모른다.

### 7.3.1 전역 상태 구체화 
- 할당하지 않은 파티션의 데이터를 포함하여 이벤트 스트림의 모든 파티션 데이터를 구체화하여, 이벤트 데이터 전체 사본을 각 MSA 인스턴스에 제공
- 전역 상태 저장소(global state store) 는 각 MSA 인스턴스에 전체 데이터 세트가 필요한 경우 / 자주 쓰이지만 변경되지 않는 소규모 데이터 세트에 사용

<img width="500" alt="image" src="https://github.com/xonmin/D3C/assets/27190617/38d139db-8039-46d0-8918-09f9c07eb513">

### 7.3.2 & 3 내부 상태 사용 시 장점/단점
장점 
- 개발자가 직접 모든 확장 요건에 대해 확장할 필요가 없음.
  - 브로커 및 컴퓨팅 리소스 클러스터에게 모두 맡길 수 있어 개발자는 비즈니스 로직에 전념할 수 있음.
  - 그냥 MSA 인스턴스 수만 늘리고 줄이면 된다.
  - 단, 개별 인스턴스에서 이벤트 처리에 대한 성능요건은 명확히 파악할 것.
- 고성능 디스크 기반 옵션
  - 모든 이벤트의 상태를 메인 메모리에 둘 수 없는 경우에 로컬 디스크를 통해 로컬 메모리와 함꼐 처리량을 매우 높일 수 있으며, 데이터 액세스 병목을 줄일 수 있다.
- NAS를 사용할 정도로 유연하다.
  - 고성능 처리가 필요하지 않는 상태 저장이라면 NAS를 통해서도 로컬 데이터를 저장하여 사용할 수 있음.
  - NAS 의 이점인 상태를 볼륨에 유지할 수 있음 (필요에 따라 새 하드웨어로 갈아 탈 수 있음.)
  - 노드 네트워크 단절 시 중단된 이벤트부터 처리 가능 (로컬 디스크처럼 상태가 완전히 일시적인 것이 아님)
 
단점 
- 런타임에 정의된 디스크를 사용할 수 밖에 없음. (디스크 볼륨 변경 시 서비스 중단 및 재시작 필요)
- 디스크 공간 낭비 (트래픽 시간별 패턴에 따라 순환 스토리지 볼륨이 필요할 때, 유연하지 않은 디스크 볼륨 변화로 인해 효율적이지 않을 수 있음)

### 7.3.4 내부 상태 확장 및 복구 
상태 복구 관점에서는 새로운 인스턴스 확장 & 실패 인스턴스 복구 모두 동일한 행위 
- 결국 두 경우 모두 새로운 이벤트를 처리하기 전 토폴로지에 정의된 상태 모두 구체화 해야한다. (이 말은 신규 데이터 처리 전 이전 데이터 상태들 모두 재컨슘하거나 체인지로그를 통해 상태 원복해야함)
- 가장 빠른 방법은 각 **상태 저장 저장소**에 대해 체인지 로그 토픽을 다시 읽는 것
  - 상태 저장 저장소..? state storage의 상태를 저장한 건가? 뭔말이야
 
**핫 레플리카 사용**
구체화 상태 레플리카는 파티션당 보통 하나만 유지하지만, 상태를 더욱 관리할 경우 레플리카를 더욱 추가할 수 있음. 
- 해당 레플리카는 리더 레플리카의 오프셋 동기화 
ex_) kafka replication fractor 조정 
<img width="540" alt="image" src="https://github.com/xonmin/D3C/assets/27190617/f92368a9-f6e3-44ee-8da7-4781cd2aad58">

replication factor = 2 인 경우 consumer 와 partition 관계 
- 스트림 B 에서 이벤트를 컨슘하여 코파티션된 구체화 상태와 조인 진행
- 이 때 replica 는 컨슈머에 유지되지만 아무 이벤트도 처리하지는 않는다. 
<img width="965" alt="image" src="https://github.com/xonmin/D3C/assets/27190617/16e84133-3d39-4d9c-9033-4a8fa5e047e3">

인스턴스 2가 다운된 상황에서는 리밸런스가 발생 -> 레플리카는 그대로 리더로 승격 및 스트림 B의 파티션 2을 재할당 받음. 

<img width="837" alt="image" src="https://github.com/xonmin/D3C/assets/27190617/8a4f03d8-04ed-4313-a1e0-64a98b81f1d7">

물론 replication-factor 를 맞추기 위해 체인지 로그에서 새로운 레플리카를 다른 곳에 생성하여 다른 인스턴스들에 추가됨.


**체인지로그를 통한 복구 및 확장 **
새로운 인스턴스가 컨슈머 그룹에 조인될 때 해당 인스턴스에 할당된 모든 파티션은 체인지 로그를 소비하여 현 상태를 load 
- 해당 시간동안에는 새로운 이벤트 처리 X (부정확한 상태를 통한 조인 시, 부정확한 결과 초래 가능)

**입력 이벤트 스트림 처음부터 복구 및 확장 **
해당 파티션의 맨 처음부터 다시 모든 입력 이벤트를 소비하는 방법 
일반적으로 체인지 로그보다 훨씬 더 오래걸리므로 입력 이벤트 스트림 보존 기간이 짧고 중복 출력이 문제가 안되는 단순한 토폴로지에 추천 

## 7.4 외부 상태 저장소 상태 구체화 
외부 상태 저장소는 컨테이너 / 가상 머신 외부에 존재하지만 대개 동일한 로컬 네트워크 안에 위치 
개별 여건에 맞는 RDB , DocumentDB , geospatial search System based Lucene , 고가용성 분산 K-V Storage 
- 이 때, 데이터 세트 자체는 여타 모든 MSA 구현체와 논리적으로 분리되어야 한다.
  - MSA 간 구체화 상태 공유는 다양한 비즈니스 니즈에 대응하고자 함께 사용할 순 있지만, 전혀 상관없는 특성이 강결합될 수 있는 안티패턴

### 7.4.1 & 2 외부상태저장소의 장단점 

장점
- 완전한 데이터 지역성 : 각 MSA 인스턴스가 접근 가능한 모든 구체화 데이터를 제공한다. 따라서 파티션 지역성이 필요 없음. 
- 기술 : 이미 익숙하게 쓰고 있는 기술들이므로 리소스가 줄어든다.

단점 
- 기술적 관리포인트 증가
- 네트워크 지연으로 인한 성능 저하
- 외부상태 저장소 서비스 이용료
- 완전한 데이터 지역성
  - 개별 인스턴스에서 동일한 데이터 세트를 사용하기 때문에 관여 및 경합 , 디버깅 등에 대한 추론이 어려움.

 ### 7.4.3 외부 상태 저장소 확장 및 복구 
 - 외부 상태 저장소 접근에 필요한 credential(인증 정보)만 있으면 된다.
 - 소스 스트림 사용 방법
   - 모든 입력스트림 오프셋 초기 시점으로 복귀 및 이벤트 초기 시점부터 소비
   - 중단 시간은 길지만 재연은 가장 쉬움
   - 단, 재 소비로 인한 출력이벤트도 모두 재생성된다.
- 체인지로그 사용 : 내부상태 저장소와 동일함.
- 스냅샷 사용 :
  - 일반적으로 개별 외부저장소 기술에서 자체 백업/복구 솔루션을 제공하고 있음.
  - 저장된 스냅샷이 멱등적이라면 컨슈머 오프셋과 비교할 필요 없음.

 ## 7.5 재구성 대 상태 저장소 마이그레이션 
 기존 저장소의 데이터 구조가 변경된 경우(필드 추가, 다른 구체화 테이블 조인 단계 추가 등) 기존 상태 저장소의 데이터가 반영되도록 재구성 또는 마이그레이션이 필요함. 

 ### 7.5.1 재구성 
 1. consumer 중지 및 input stream (topic) offset reset
 2. reset 된 offset 부터 다시 consume ->  기존 저장소의 중간 상태 제거 & 저장소의 **변경된 데이터 구조**로 다시 재구성
 3. 이렇게 새롭게 만들어진 output stream 을 통해 다운스트림으로 전파됨. 

> 다만, 재구성의 경우 시간 소요에 대해서 MSA에 대한 문제 확인 필요. 
> 재구성의 장점은 MSA 실패로 인해 상태 전부 유실된 경우 필요한 복구 프로세스를 통해 복구 대비 상태를 테스트해볼 수 있음. 
