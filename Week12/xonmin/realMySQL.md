## 4.2.7.4 buffer pool flush 

- 8.0 upgrade 이후, dirty page를 disk에 sync 하는 부분(dirty page flush)에서 예전과 같은 디스크 write 폭증 현상은 발생하지 않음 
- innoDB 스토리지 엔진 : buffer pool 에서 아직 disk에 기록되지 않은 dirty page를 disk에 동기화 하기 위해 백그라운드로 플러시 실행 
  - Flush list Flush 
  - LRU list Flush 


### 4.2.7.4.1 플러스 리스트 플러시 
- InnoDB 스토리지 엔진은 redo log 공간 재활용을 위해 오래된 redo log 엔트리 공간을 비워야함 
- 이 때 비우기 위해서는 선순위로 dirty 페이지가 디스크로 동기화 되어야 함 
- 따라서 이를 위해 innoDB 스토리지 엔진은 주기적으로 FLush List Flush 함수 호출 (디스크 동기화 실행) 


- Cleaner Thread : InnoDB 스토리지 엔진에서 dirty page 를 disk로 sync 하는 스레드 (`innodb_page_cleaners` variable)
- InnoDB 버퍼 풀은 DML(`INSERT, UPDATE, DELETE`) 에 의해 변경된 더티 페이지도 함께 가지고 있다. 
- dirty page 생성 속도 == redo log 생성 속도 


### 4.2.7.4.2 LRU 리스트 플러시 
LRU 리스트 Flush 함수 : LRU 리스트에서 사용 빈도가 낮은 데이터 페이지들을 제거하여 새로운 페이지 공간을 만듦 
- InnoDB 스토리지 엔진 : LRU 리스트[-1 : `-innodb_lru_scan_depth`] 까지 페이지 스캔 
  - 스캔하면서 더티 페이지는 disk에 syn, clean page는 즉시 free list로 페이지를 옮긴다. 

> 따라서 버퍼 풀 인스턴스 별로 최대 `innodb_lru_scan_depth` 개수만큼 스캔하기 때문에 총 LRU 리스트의 스캔은 (`innodb_buffer_pool_instance` * `inndb_lru_scan_depth`) 수 만큼 수행 

## 4.2.7.5 버퍼 풀 상태 백업 및 복구 

InnoDB 서버의 버퍼풀은 쿼리의 성능과 매우 밀접한 관계 
- 쿼리요청이 매우 빈번한 서버 : shut down 이후, 서비스 시작시, 쿼리 처리 성능 = 평상시 성능에 비해 1/10 
- WarmUp : disk에 데이터가 buffer pool에 적재돼 있는 상태 
- `innodb_buffer_pool_dump_now` : innodb 버퍼 풀의 상태를 백업 가능 


## 4.2.8 Double Write Buffer 

**Partial-Page(Torn-Page)** 
- 원인 : 
- redo log는  InnoDb 스토리지 엔진의 redo log 공간 낭비를 막기 위해, **페이지의 변경 내용만** 기록 
- 이 때문에, dirty page를 disk로 flush 할 때, 일부만 기록되는 문제가 발생할 수 있음 
  - 해당 페이지의 내용은 복구가 불가할 수 있음 
> 다른 원인으로는 HW 오작동 / System 비정상 종료 


**Double-Write 기법(Dobule Write Buffer)**
목적 : 
- DoubleWrite buffer에 더티 페이지를 작성함으로써, 데이터 파일 쓰기 중간에 실패할 때(비정상 종료) 해당 `버퍼의 내용`과  `file의 page들의 내용`을 비교하여 안정성과 무결성을 높인다.
- 이름은 버퍼이나 Doublewrite Buffer 는 메모리가 아니라 데이터 파일
- 시스템 변수 : `innodb_doublewrite (default : ON)`

![image](https://user-images.githubusercontent.com/27190617/216042355-ef7bcfbd-b3bb-4e3e-8e89-2ae1aade8e38.png)

상황 : InnoDB buffer pool에서 A~D 까지의 더티 페이지 -> disk 로 flush 해야하는 상황

task : 
1. A~D 까지의 더티 페이지를 묶어, 시스템 테이블스페이스의 **DoubleWrite 버퍼**에 기록 
2. InnoDB 엔진은 각 더티 페이지를 파일의 적당한 위치에 하나씩 랜덤하게 write 

if : 
- A와 B 페이지는 정상적으로 데이터 파일쓰기에 정상적으로 성공 
- C 페이지 기록 도중, 비정상 종료 

then : 
- InnoDB 스토리지 엔진 재시작시, 항상 DoubleWrite 버퍼 내용과 데이터 파일의 페이지 모두 비교 
- 다른 내용이 담긴 페이지 존재 -> [ DoubleWrite 버퍼 내용 -> 데이터 파일 페이지로 복사 ]

**DoubleWrite 버퍼와 저장장치**
데이터의 무결성이 중요한 서비스에서는 doublewrite 활성화를 권장 
- DB 성능을 고려하여 redo log 동기화 설정을 1이 아닌 0, 2로 할 경우 : [1초마다 자동적으로 log file을 flush 하므로](http://minsql.com/mysql/innodb_flush_log_at_trx_commit-%EA%B0%9C%EB%85%90%EB%8F%84%EC%99%80-%ED%8A%9C%EB%8B%9D-%ED%8F%AC%EC%9D%B8%ED%8A%B8/), doublewrite도 비활성화하여도 무방 
- HDD : 자기 원판(Platter)의 회전 -> 결국 한 번의 순차 write 작업 
  - 따라서 큰 overhead가 아님 
- SSD : 랜덤 IO와 순차 IO의 비용이 비슷한 특징으로 overhead로 여겨질 수 있음  

> [HDD, SSD와 랜덤/순차 IO의 관계](https://velog.io/@keywookim/MySQL-Index-%EC%BF%BC%EB%A6%AC%ED%8A%9C%EB%8B%9D%EC%9D%98-%EA%B8%B0%EB%B3%B8-1)

## 4.2.9 Undo Log (언두로그)

Undo Log : InnoDB 스토리지 엔진에서 tx와 isolation level을 보장하기 위해 DML(`INSERT, UPDATE, DELETE`)로 변경되기 이전 버전의 데이터가 별도로 백업된 로그
- tx 보장 : 롤백시, tx도중 변경된 데이터 복구를 언두 로그에 저장한 버전을 통해 복구한다. 
- isolation level 보장 : A에서 데이터를 변경하는 도중 B에서 데이터 조회시, tx 격리 수준에 맞게 A가 핸들링하는 데이터를 직접 보는 것이 아닌, 언두 로그의 백업된 데이터를 return 한다. 

