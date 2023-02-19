# MyISAM 스토리지 엔진 아키텍처 

## 키 캐시
- 키캐시:
    - 인덱스만을 대상으로 작동
    - 인덱스의 디스크 쓰기 작업에 대한 버퍼링 역활
- Default 키캐시는 제한된 메모리가 존재 (예. 32비트OS: 4GB)
    - 제한된 메모리 보다 더 할당하고 싶으면 Default 키캐시 외 별도의 키캐시를 설정해야함
        - 예: `key_buffer_size: 4GB, kbuf_board.key_buffer_size: 2GB`
    - 추가 키캐시는 어떤 인덱스를 캐시할지 설정 필요
        - 예: `CACHE INDEX db1.board, db2.board IN kbuf_board

## 운영체제의 캐시 및 버퍼
- MyISAM 테이블 데이터 (즉, row)에 대해서는 캐시/버퍼링 기능이 존재하지 않음
- 데이터 읽기/쓰기 작업은 항상 OS 디스크 읽기/쓰기 작업으로 요청이 됌
- 대부분의 OS에는 자체 캐시/버퍼링 기능이 존재하여 MyISAM 성능은 OS 기능에 의존성이 높음
    - OS 캐시는 애플리케이션들이 사용하고 남은 메모리 공간을 캐시로 사용함
    - 그래서 MyISAM 사용할때는 항상 OS 메모리 공간을 예의주시해야함

## 데이터 파일과 프라이머리 키(인덱스) 구조
- InnoDB: 프라이머리키에 의한 클러스터링 vs. MyISAM: 클러스터링 없이 INSERT 되는 순서대로 데이터 파일에 저장됌
- MyISAM의 모든 레코드는 ROWID (물리적 주솟값) 을 갖고 primary/secondary 인덱스는 레코드의 ROWID 값을 포인터로 갖음
- ROWID: 가변길이 / 고정길이 방식으로 저장 가능
    - 고정길이: MyISAM 테이블 생성할때 MAX_ROWS 옵셜 활성화된 경우
    - 가변길이: 그외 케이스일때 / 2 byte ~ 7 byte 를 갖고 첫번째 바이트는 길이정보를 저장


---

# 4. MySQL log File 

## 에러 로그 파일
- 실행 도중 발생하는 에러/경고 메시지를 담은 로그 파일
- 생성 경로: `my.cnf`의 `log_error`에 정의 되어있는 경로에 생성
- 에러 로그 파일에 기록되는 다양한 메시지 유형:
    1. 설정 파일 변경 혹은 비정상적 종료 이후 재시작인 경우
        - 예. 설정 변경이 있을 경우, 해당 로그를 통해 설정이 의도대로 설정이 되어있는지 확인
    2. 비정상적 종료 이후 트랜잭션 복구 시 발생한 경우
        - 복구에 실패할 경우를 대비하여 에러 로그 확인 필요
        - 해당 에러 로그를 보면서 `innodb_force_recovery` 파라미터 설정 필요
    3. 쿼리 처리 도중 에러가 발생한 경우
    4. 클라이언트 앱에서 정상적으로 연결 종료를 하지 못한 경우
        - 이 로그를 확인하여 앱의 접속 종료 로직을 검토할 수 있음
        - `max_connect_errors`: 클라이언트에서 발생한 에러의 횟수가 넘을 경우 MySQL 서버에서 블락
    5. 테이블 모니터링 / 락 모니터링 / 엔진 상태 조회하는 경우
        - 상대적으로 큰 메시지를 에러 로그 파일에 저장함
        - 모니터링을 계속 활성화해두면 로그 파일이 너무 커져 파일 시스템 공간이 고갈될 수 있음
    6. MySQL 서버의 종료되는 경우
        - 왜 MySQL 서버가 죽는지 확인 가능
        - 누가 해당 서버를 직접 죽였으면 `Received SHUTDOWN from user ...` 로 확인 가능
        - 직접 안죽였으면 `Segmentation fault`로 비정상적으로 종료된것으로 판단 가능

## 제너럴 쿼리 로그 파일
- 쿼리 로그를 활성화해 실행되는 쿼리들의 목록 확인 가능
    - `general_log_file` 파라미터로 제너럴 로그 파일 경로 설정 (혹은 별도 테이블에 저장도 가능)
    - 슬로우 쿼리라는 다르게 실행되기 전에 기록이 되는 로그

## 슬로우 쿼리 로그 파일
- MySQL 서버의 쿼리 튜닝
    - 서비스 적용 전 튜닝
    - 서비즈 적용 후 운영 중 성능 저하 검사 및 정기점검
- 후자일 경우 어떤 쿼리가 문제인지 파악하기 위해 슬로우 쿼리 로그 파일 사용
- `long_query_time` 보다 오래걸린 쿼리들을 모두 기록
    - 여기서 기록되는 쿼리는 모두 정상적으로 실행이 완료된 쿼리들만 기록이 됌
- `log_output` 옵션을 통해 테이블 혹은 파일로 저장할지 선택 가능

```
# Time:2020-07-19T15:44:22.178484+09:00
# User@Host:root[root] @ localhost [] Id: 14
# Query_time:1.180245 Lock_time:0.002658 Rows_sent:1 Rows_examined:2844047 
use employees;
SET timestamp=1595141060;
select emp_no, max(salary) from salaries;
```
- 예제 설명:
    - Time: 쿼리가 종료된 시점 / 시작시간은 Time - Query_time으로 계산 가능
    - Query_time: 쿼리 실행 소요시간
    - Lock_time: MySQL 엔진 레벨에서의 테이블 잠금에 대한 대기시간 (잠금 체크와 같은 코드 실행부분까지 포함)
        - InnoDB: MySQL의 잠금 처리는 엔진 레벨과 스토리지 엔진 레벨의 두가지 레이어로 처리
        - MyISAM: 테이블 단위의 잠금을 사용하여 SELECT 쿼리라도 Lock_time이 길수도 있음
        - 다만, InnoDB에서도 SELECT 쿼리의 Lock_time이 길수 있는데, 이유는 InnoDB의 레코드수준의 잠금이 아닌 MySQL 엔진 레벨에서 설정한 테이블 잠금 때문일수도 있음
        - 즉, InnoDB에서 슬로우 쿼리 로그의 Lock_time은 도움이 별로 안됌
    - Rows_Sent: 실제 몇건의 처리 결과를 보냈는지 나타냄
    - Rows_examined: 쿼리가 처리되기 위해 몇건의 레코드에 접근했는지 나타냄
- 슬로우 쿼리 / 제너럴 로그 파일의 내용이 너무 많아 검토하기 어려울 경우, Percona Toolkit의 pt-query-digest 스크립트를 사용하여 빈도/처리성능별로 쿼리 정렬 가능
    - 슬로우 쿼리 통계
        - 모든 쿼리를 대상으로 슬로우 쿼리 로그의 실행 시간 / 잠금 대기 시간 등에 대해 평균/회소/회대값 표시
        - ![image](https://user-images.githubusercontent.com/27190617/219934932-ab59131e-8de5-40e9-ad92-b5640ddf76fd.png)
    - 실행 빈도 및 누적 실행 시간순 랭킹
        - 같은 모양의 쿼리라면 동일한 Query ID를 갖게 됌
        - ![image](https://user-images.githubusercontent.com/27190617/219934935-224221c2-8b47-4ea0-abac-4535c7275f3f.png)
    - 쿼리별 실행 횟수 및 누적 실행 시간 상세정보
        - 특정 쿼리에 대한 자세한 내용을 보여줌
        - ![image](https://user-images.githubusercontent.com/27190617/219934937-e1d6099f-1dec-4810-89df-9a45a204b9c0.png)
        - ![image](https://user-images.githubusercontent.com/27190617/219934942-11ade0ab-fbcc-4fc4-a6df-d4e4d9494603.png)





