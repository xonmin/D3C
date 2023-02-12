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

 
### 4.2.10 체인지 버퍼 
참조 링크 : https://omty.tistory.com/59

![image](https://user-images.githubusercontent.com/27190617/217807669-0958f454-32f4-4ccc-a779-2328b0e39e7a.png)


**체인지 버퍼** 
- 정의:  MySQL InnoDB에는 **보조 인덱스** I/O 성능 향상을 위해 존재하는 임시 메모리 공간
- index update 시, 랜덤하게 disk read가 필요 
  - 따라서 테이블에 index가 많다면, 필요 resource 증가 
  - 변경해야할 index 페이지를 disk에서 읽어와 업데이트를 해야한다면 즉시 실행하는 것이 아닌,
 **체인지 버퍼**에 저장해두고 사용자에게 결과 바로 반환 

- 체인지 버퍼에 임시 저장된 인덱스 레코드 조각들은 추후 백그라운드 스레드(`체인지 버퍼 머지 스레드`)에 의해 병합

> 유니크 인덱스와 체인지 버퍼 
> 결과를 전달하기 전, **중복 여부**를 체크해야하는 유니크 인덱스의 경우, 체인지 버퍼를 사용할 수 없음
> - 체인지 버퍼를 통하면 디스크 내부에 있는 인덱스들에 대한 체크가 되지 않기 때문에 둘은 서로 알맞은 용도가 되지 않음 

- v8.0부터는 DML(`INSERT, DELETE, UPDATE`)에 대해서 선택적으로 체인지 버퍼 활성화 가능 
  - `innodb_change_buffering` 시스템 변수를 통해 설정 가능 
```sql
SHOW variables LIKE 'innodb_change_buffering'
```

- `INSERT와 UPDATE`가 빈번할 경우 : `innodb_change_buffer_max_size`를 통해 체인지 버퍼의 메모리 공간 확보 가능  
  - (default : 25% , max 50%)
```sql
SHOW variables LIKE 'innodb_change_buffer_max_size'
```

> 체인지 버퍼의 경우 DML 후 보조 인덱스를 최신 상태로 유지하는데 사용되는 상당한 I/O를 줄일 수 있습니다. 
다만 과거 디스크가 느린 시절에 큰 효과가 있었지만 SSD 등 디스크의 성능이 개선됨에 따라 극적인 성능 개선을 가져오지는 않습니다. 
또한 변경 사항이 많을 경우 재부팅, 복구 작업이 지연될 수 있기에 주의해야합니다.
추가로 이러한 이유 때문인지 Aurora MySQL의 경우 체인지 버퍼를 사용하지 않습니다.(None으로 설정되어있으며 변경 불가)

### 4.2.11 리두 로그 및 로그 버퍼 
리두 로그 : 여러 문제점에 의해 비정상 종료시, 데이터 파일에 기록되지 못한 데이터를 잃지 않도록 하는 역할 (WAL - Write Ahead Log)
- **리두 로그 내용을 통해 종료(장애) 직전의 상태로 복구**

비정상 종료의 경우 : InnoDB 스토리지 엔진의 데이터 파일에서의 일관되지 않은 데이터 종류 
1. 커밋됐지만 데이터 파일에 기록되지 않은 데이터
- 리두 로그 내용을 통해서 데이터 파일에 복사
2. 롤백됐지만 데이터 파일에 이미 기록된 데이터
- 언두 로그(변경되기 이전 내용)을 데이터 파일에 복사 + **리두 로그**를 통해 변경작업에서 tx가 어떠한 상태였는 지 파악(commit or rollback or 진행중..) 

redo log를 disk 동기화 주기 설정 
- tx가 commit 될 때 마다 disk 에 기록하는 작업은 부하가 심함 
- disk 동기화 주기 설정 : `innodb_flush_log_at_trx_commit`
  - `0` : 1초에 한 번씩 리두 로그를 기록(write)/동기화 진행 - 서버가 비정상 종료되면 최대 1초동안의 tx가 커밋됐다해도, 해당 tx에서 변경된 데이터는 사라질 가능성 존재
  - `1` : 매번 tx가 commit 할 때 마다 disk로 기록/동기화 수행 - tx가 commit 되면 tx에서 변경한 데이터는 사라진다. 
  - `2` : 매 tx가 commit 될 떄마다 disk로 기록 진행 but **동기화(sync)는 1초에 한 번씩** 실행. tx가 커밋되면 os 메모리 버퍼로 기록되는 것은 보장 
    - mysql이 비정상 종료되더라도 os가 정상이라면 해당 tx의 데이터는 사라지지 않지만, os또한 비정상 종료라면 최근 1초동안의 tx 데이터는 사라짐  

![image](https://user-images.githubusercontent.com/27190617/218233248-0e04371f-c647-458a-871a-d7744d280565.png)

리두 로그 파일의 전체 크기는 InnoDB 스토리지 엔진의 버퍼 풀의 효율성을 결정하게 된다.
- 리두 로그 파일의 전체 크기 = `innodb_log_file_size` * `innodb_log_files_in_group` 

**로그 버퍼**

로그 버퍼 : 디스크의 리두 로그파일에 기록될 데이터를 보유하는 메모리 영역
- 로그 버퍼 크기 : `innodb_log_buffer_size`로 설정 가능 (일반적으로 4~16MB size)
- 로그 버퍼크기를 크게 조정하면 트랜잭션을 커밋 하기 전에 디스크에 로그를 쓰지않아도 큰 트랜잭션을 실행할수 있다.
- 많은 행을 업데이트, 삽입, 삭제하는 트랜잭션이 있는 경우 로그버퍼를 크게조정하면 DISK I/O 가 절약.

![image](https://user-images.githubusercontent.com/27190617/218234182-ac8ce01f-72ec-4594-b682-f86101e82acc.png)

### 4.2.11.1 리두 로그 아카이빙 
8.0 버전부터 InnoDB 스토리지 엔진의 리두 로그를 아카이빙 할 수 있도록 기능 추가
-  리두 로그 아카이빙을 통해 데이터 변경이 너무 많아 리두 로그가 덮어쓰인다하더라도 백업툴이 백업이 실패하지 않도록 기능

> 중요한 부분인지 잘 모르곘음..

### 4.2.11.2 리두 로그 활성화/비활성화
MySQL 서버에서 tx가 커밋되어도, Data 파일은 디스크로 즉시 sync 되지는 않음 / redo log(tx log)는 항상 디스크로 기록 
- `innodb_flush_log_at_trx_commit`이 0이나 2이면 리두 로그 또한 disk 동기화가 바로 일어나지 않을 수 있음

v8.0 부터 리두 로그 비활성화 가능 : 데이터 복구 및 대용량 데이터 적재시 비활성화를 통해 적재 시간 단축 가능
```sql
# 비활성화 
sql > ALTER INSTANCE DISABLE INNODB REDO_LOG;

# 활성화
sql > ALTER INSTANCE ENABLE INNODB REDO_LOG;
```
- 만약 redo 로그를 비활성화하고 비정상 종료되었다면 MySQL 의 마지막 체크 포인트 이후 시점의 데이터는 모두 복구할 수 없음 

### 4.2.12 어댑티브 해시 인덱스 

어댑티브 해시 인덱스(Adaptive Hash Index) : InnoDB 스토리지 엔진이 사용자가 자주 요청하는 데이터(자주 사용되는 칼럼)에 대해 바로 접근하기 위해 옵티마이저가 판단하여 자동 생성되는 인덱스 
- 즉 전체 데이터를 대상으로는 해시값을 생성하지는 않는다.
- `innodb_adaptive_hash_index`시스템 변수를 통해 인덱스 기능 활성화/비활성화 
- MySQL 내에 B-Tree 검색 시간을 줄여주기 위해 도입된 기능
- 자주 읽히는 데이터 페이지의 키 값을 통해 해시 인덱스를 생성후 필요시 어댑티브 해시 인덱스를 검색하여 레코드가 저장된 데이터 페이지 즉시 찾아가기 가능 
  - B-Tree를 root ~ leaf 까지 찾아가는 비용 절약 & 쿼리 성능 개선 & 더 많은 쿼리 동시 처리 가능 
  
![image](https://user-images.githubusercontent.com/27190617/218296543-073812d1-8401-4d64-8b1a-9349206f3152.png)


> **Hash index 구조**
- Hash index 는 (index key : data 페이지의 메모리 주소) 로 구성
- index key는 **B-Tree index id와 B-Tree 인덱스의 실제 key** 로 생성
  - adaptive hash index key에 B-Tree index id가 포함되는 이유 : InnoDB 스토리지에 어댑티브 해시 인덱스는 하나만 존재하므로
  - 즉, B-Tree 인덱스에 대해 어댑티브 해시 인덱스가 하나의 해시 인덱스에 저장 및 특정 키 값이 어느 인덱스에 속한 지 구분

어댑티브 해시 인덱스도 결국 메모리 객체이므로 인덱스 경합(Contention)이 심했지만, 내부 잠금(세마포어) 경합을 줄이기 위해 파티션 기능 제공
- `innodb_adaptive_hash_index_parts`를 통해 파티션 개수 변경 (default - 8개) 


**어댑티브 해시 인덱스 효과 및 주의사항**
성능 향상에 효과적일 때 : 
- disk의 데이터가 InnoDB 버퍼 풀 크기와 비슷한 경우(disk read가 많지 않을 때)
- 동등 조건 검색이 많은 경우(범위 쿼리 등이 없을 때) 
- 쿼리가 데이터 중 일부 데이터에만 집중된 경우 

 주의 사항 : 
 - 어댑티브 해시 인덱스 활성화 시, 해시 인덱스의 효율이 없는 쿼리에도 InnoDB는 해시 인덱스 사용 
 - 특정 테이블의 인덱스가 해시 인덱스에 존재할 때 `DROP or ALTER`한다면 어댑티브 해시 인덱스에서도 제거해주어야함 
 - 옵티마이저가 판단하여 해시 키로 만들기 때문에 제어가 어려우며, 수 개월 동안 사용되지 않던 테이블일지라도 기존 해시 자료 구조에 데이터가 남아 있게 되면, 테이블 Drop 시 영향을 줄 수 있음

참조 ref: https://tech.kakao.com/2016/04/07/innodb-adaptive-hash-index/


### 4.2.13 InnoDB와 MyISAM, MEMORY 스토리지 엔진 비교
InnoDB가 짱!
