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
