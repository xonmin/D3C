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

### 4.2.9.1 언두 로그 모니터링 
Undo 영역 : DML (`INSERT, UPDATE, DELETE`) 를 통해 데이터가 변경되었을 때, 변경 전의 데이터를 보관하는 곳 
- commit 시 변경된 데이터의 상태 유지 / rollback : undo 영역의 백업 데이터가 데이터 파일로 복구된다. 

Undo Log의 용도 
1. Tx 롤백 대비용 
2. Tx 격리 수준을 유지하면서 높은 동시성을 제공 (이후 5.4.3 `REPEATABLE_READ` 참고) 

Undo log 공간 
- 대용량 데이터 처리에 관하여 :(mysql ~ v5.5) 한 번 증가한 언두로그 공간은 다시 줄어들지 않음 (100GB 테이블 delete -> undo에 100GB 저장)
  - 서버 새로 구축하지 않는 한 줄일 수 없음   
- 오랜 시간 실행되는 tx : tx가 오래 지속될 경우 언두 로그의 양은 급 증가
  - ex_ ) 순차적으로 A,B,C 의 tx가 시작되었고, B,C는 종료되었지만 A가 tx의 커밋이 되지 않았을 때 : B와 C의 언두 로그는 삭제되지 않음 
  - tx가 오래 유지되는 동안 빈번하게 변경된 record 조회 쿼리 -> undo log 이력을 필요한만큼 스캔해야함 -> **쿼리 성능 저하**
  
> tx 관리가 어플리케이션 레벨에서 잘못된 경우 & 사용자 실수로도 발생할 수 있음 

MySQL v8.0 ~
- undo log를 돌아가면서 순차적으로 언두 로그를 사용하여 디스크 공간 줄임 + 필요 시점에 사용 공간 자동으로 줄임
- MySQL 언두 로그 건수 확인 
```sql 
sql > SHOW ENGINE INNODB STATUS \G 

or 

sql > SELECT count FROM information_schema.innodb_metrics 
      WHERE SUBSYSTEM='transcation' AND NAME='trx_rseg_history_len';
      
# UPDATE와 DELETE 에 의한 언두 로그 개수 
```

> MySQL 서버엔 `INSERT` 와 `UPDATE(DELETE)` 에 대한 언두 로그가 별도로 관리 
> - `UPDATE & DELETE` 언두 로그는 MVCC와 데이터 복구(롤백)에 사용 
> - `INSERT` 는 MVCC에는 사용되지 않고, 롤백 및 데이터 복구에만 사용된다. 

### 4.2.9.2 언두 테이블스페이스 관리 

Undo Tablespace : 언두 로그가 저장되는 공간 
- ~v5.6 : undo 로그는 시스템 테이블 스페이스(`ibdata.ibd`)에 저장 -> 서버 초기화시 생성되므로 확장의 한계 존재
- v5.6 : `innbodb_undo_tablespaces`에서 2 이상 설정시, 별도의 언두 로그 파일 사용 (현재는 deprecated)
- v8.0~ : 항상 시스템 테이블스페이스 외부의 별도 로그 파일로 기록 
-
**undo 테이블 스페이스 구조**
[사진 넣기]


- 언두 테이블스페이스: 1~128개 이하의 롤백 세그먼트 구성
- 롤백 세그먼트 : 1개 이상의 언두 슬롯(Undo Slot)으로 구성
  - InnoDB 페이지 크기를 16Byte 로 나눈 값의 개수만큼의 언두 슬롯을 가진다. 
  - ex_ Innodb Page size : 16KB : 16KB / 16Byte = 1024개 언두 슬롯 
- tx가 필요로 하는 언두 슬롯 개수 : `INSERT, UPDATE, DELETE` 특성에 따라 최대 4개까지의 언두 슬롯 사용  

> if) 하나의 트랜잭션이 약 2개의 언두 슬롯이 필요할 때, 최대 동시 처리 가능한 tx 수 
> then) max_concurrent_tx = (innodb page size) / 16 * (롤백 세그먼트 개수) * (언두 테이블 스페이스 개수)
> ex_ default option인 16KB InnoDB (undo_table_spaces=2, innodb_rollback_segments = 128) 의 최대 동시 처리 tx
> -> (16 * 1024 / 16 * 128 * 2) / 2(언두 슬롯 개수(?))  = 131072 개 [왜 마지막 나누기 2가 들어갔을까]


언두로그 공간이 남는 것은 문제 X / **언두 로그 슬롯이 부족한 경우 tx를 시작할 수 없는 문제**
- ~ v8.0 : 한 번 생성된 언두 로그 변경 허용 불가 (정적 사용)
- v8.0 ~ : `CREATE UNDO TABLESPACE , DROP TABLESPACE`를 통해 언두 테이블 스페이스 동적 추가 및 삭제 가능  

```sql
// 동적 생성 
sql > CREATE UNDO TABLESPACE extra_undo_003 ADD DATAFILE '/data/undo_dir/undo_003.ibu';

//언두 테이블 스페이스 비활성화
sql > ALTER UNDO TABLESPACE extra_undo_003 SET INACTIVE;

// 비활성화된 테이블 스페이스 삭제
sql > DROP UNDO TABLESPACE extra_undo_003;
```

**Undo tablespace truncate**
- 언두 테이블스페이스 공간을 필요한 만큼만 남기고 불필요하거나 과도하게 할당된 공간을 운영체제로 반납하는 것 
- 방법은 2가지 종류 존재
- 자동 모드 : 
  - 트랜잭션이 커밋되면 더 이상 언두 로그의 이전 값은 필요 없음
  - **Undo Purge** : InnoDB 스토리지 엔진의 퍼지 스레드(Purge Thread) 는 주기 적으로 불필요한 언두 로그를 삭제 
  - 언두 퍼지 빈도 수 줄이는 방법 : `innodb_purge_rseg_truncate_frequency`
- 수동 모드 : 
  1. `innodb_undo_log_truncate = OFF`(언두 테이블 스페이스를 비활성화 -> Purge Thread는 비활성 상태의 언두 테이블 스페이스를 찾아 OS에 공간 반납) 
  2. 반납 완료 -> 언두 테이블스페이스 다시 활성화 
  - 참고 : 수동 모드는 언두 테이블 스페이스가 최소 3개 이상 되어야 작동 

```sql
// 언두테이블스페이스 비활성화
sql > ALTER UNDO TABLESPACE tablespace_name SET INACTIVE;

// 퍼지 스레드에 의해 언두 테이블스페이스 공간 반납 후, 활성화 
sql > ALTER UNDO TABLESPACE tablespace_name SET ACTIVE
```

 
###4.2.10 체인지 버퍼 
참조 링크 : https://omty.tistory.com/59

![image](https://user-images.githubusercontent.com/27190617/217253116-c5c9cf0f-2f18-49b6-8c8c-6f5bc8788dd5.png)

체인지 버퍼 
- 정의:  MySQL InnoDB에는 보조 인덱스 I/O 성능 향상을 위해 존재하는 임시 메모리 공간
- index update 시, 랜덤하게 disk read가 필요하므로, 
