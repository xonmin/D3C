## 10.1.2 히스토그램(Histogram)
- MySQL ~5.7 : 통계 정보 - 단순한 인덱스된 칼럼의 유니크한 값의 개수 정도만 지님 
  - 옵티마이저에게 최적의 실행 계획에는 부적합 
  - 따라서 실행 계획 수립시, 실제 인덱스의 일부 페이지를 랜덤으로 get & reference 하여 부족한 점 보충 
- ~8.0 : 칼럼 데이터 분포도를 확인할 수 있는 히스토그램 정보 활용

### 10.1.2.1 히스토그램 정보 수집 및 삭제 

히스토그램
- 칼럼 단위로 정보 관리 
- 자동 수집이 아닌, `ANALYZE TABLE ... UPDATE HISTOGRAM` 명령어 실행하여 수집 및 관리 
  - 시스템 내 딕셔너리에 함께 저장
  - MySQL 서버 시작시, 딕셔너리에서 `infromation_schema` DB 내 `column_statistics` TABLE 로 load 

**히스토그램 정보 수집 및 조회**
```sql 
-- 히스토그램 수집
mysql> ANALYZE TABLE employees
UPDATE HISTOGRAM ON gender,hire_date;
+-------------------+-----------+----------+------------------------------------------------------+
| Table             | Op        | Msg_type | Msg_text                                             |
+-------------------+-----------+----------+------------------------------------------------------+
| test_db.employees | histogram | status   | Histogram statistics created for column 'gender'.    |
| test_db.employees | histogram | status   | Histogram statistics created for column 'hire_date'. |
+-------------------+-----------+----------+------------------------------------------------------+

-- COLUMN_STATISTICS 조회
mysql> select *
from information_schema.COLUMN_STATISTICS
where table_name='employees'\G

*************************** 1. row ***************************
SCHEMA_NAME: test_db
 TABLE_NAME: employees
COLUMN_NAME: gender
  HISTOGRAM: {"buckets": [[1, 0.5998776829658968]
, [2, 1.0]], "data-type": "enum", "null-values": 0.0, "collation-id": 45,
 "last-updated": "2021-11-21 12:06:45.393325", 
 "sampling-rate": 0.34783905681630983, "histogram-type": "singleton",
  "number-of-buckets-specified": 100}
  
*************************** 2. row ***************************
SCHEMA_NAME: test_db
 TABLE_NAME: employees
COLUMN_NAME: hire_date
  HISTOGRAM: {"buckets": [["1985-01-01", "1985-02-27", 0.010096009164069856, 29]
, ["1985-02-28", "1985-03-25", 0.01992020269679937, 26]
, ["1985-03-26", "1985-04-23", 0.02990942714855695, 29]
, ["1985-04-24", "1985-05-20", 0.03984040539359874, 27]
, ["1985-05-21", "1985-06-16", 0.04983933754647562, 27]
, ["1985-06-17", "1985-07-15", 0.06011008533069284, 29]
, ["1985-07-16", "1985-08-10", 0.06989544805894517, 26]
, ["1985-08-11", "1985-09-07", 0.07998174952189573, 28]
, ["1985-09-08", "1985-10-05", 0.08999038937589189, 28]

...

, ["1997-12-14", "1998-07-27", 0.9899816524448846, 224]
, ["1998-07-28", "2000-01-28", 1.0, 450]], 
"data-type": "date", "null-values": 0.0, 
"collation-id": 8, "last-updated": "2021-11-21 12:06:45.393773", 
"sampling-rate": 0.34783905681630983, "histogram-type": "equi-height", 
"number-of-buckets-specified": 100}
```

**히스토그램 타입**
1. Singleton Histogram 
  - 칼럼값 개별로 레코드 건수 관리하는 히스토그램, (a.k.a : Value-Based 히스토그램, 도수 분포)

2. Equi-Height Histogram(높이 균형 히스토그램) 
  - 칼럼값의 범위를 균등한 개수로 구분하여 관리하는 히스토그램 (a.k.a :Balanced 히스토그램) 

히스토그램은 Bucket 단위로 구분 -> 레코드 건수나 칼럼값의 범위가 관리된다. 
  - 싱글톤 : 
    - 칼럼이 가지는 값별로 할당
    - 각 버킷이 칼럼의 값과 발생 빈도의 비율이란 2가지의 값을 가짐
  - 높이 균형 : 
    - 개수가 균등한 칼럼값의 범위별로 하나의 버킷 할당 
    - 각 버킷이 범위 내 min, max 값, 발생빈도율, 각 버킷에 포함된 유니크 값 개수(카디널리티) : 총 4가지 값을 가짐 
    
**싱글톤 히스토그램**
![image](https://github.com/xonmin/D3C/assets/27190617/9a7c3fe0-5fa5-422b-9e39-90e1a3df9679)

gender 칼럼에 대한 히스토그램 
- M의 record 비율 : 0.5998, F 레코드의 비율은 1 로 표시되지만, 
- 히스토그램의 모든 레코드 건수 비율은 누적으로 표시되므로, gender 컬럼의 값 F의 비율은 (1-0.5998)


![image](https://github.com/xonmin/D3C/assets/27190617/c0c15fab-efa7-4cb7-8889-a5900f80a84a)
hire_date 칼럼에 대한 높이 균형 히스토그램 
- 높이 균형 히스토그램은 컬럼의 각 범위에 대해 레코드 건수 비율 : 누적으로 표시 (따라서 마지막이 1)
- 범위 별로 비율이 같은 수준의 hire_date 컬럼의 범위가 선택된다. (그래서 그래프의 기울기가 일정)

**INFORMATION_SCHEMA.COLUMN_STATISTICS 테이블의 HISTOGRAM 컬럼에서 포함된 정보**

Sampling-rate
- 히스토그램 정보를 수집하기 위해 스캔한 페이지의 비율을 저장
  - ex_ 0.35 = 전체 데이터 페이지중 35% 스캔하여 정보 수집 
- 샘플링 비율이 높을수록 정확도 증가, 리소스 증가 
- 해당 변수 : `histogram_generation_max_max_size(default - 19MB)`

histogram-type : 히스토그램의 종류를 저장 

number-of-buckets-specified
- 히스토그램을 생성 할 때 설정 했던 버킷의 개수
- default : 100, max : 1024개 

> ~8.0.19 미만 버전 
> 샘플링 비율(Sampling-rate) 과 `histogram_generation_max_mem_size` 시스템 변수의 크기와 관계 없이 
> MySQL 서버는 풀 스캔을 통해 데이터 페이지를 샘플링 후 히스토그램 생성 


**히스토그램 삭제 및 미사용**
히스토그램 삭제 시 쿼리의 플랜 변경에 의해서 처리 성능에 영향을 받을 수 있는 점 고려

삭제 
```sql 
mysql> ANALYZE TABLE employees
DROP HISTOGRAM ON gener, hire_date;
```

전역적 사용 제한 
```sql 
mysql> SET GLOBAL optimizer_switch='condition_fanout_filter=off';
```
-  optimizer_switch 를 변경하면 모든 쿼리가 히스토그램을 사용하지 못함
-  `condition_fanout_filter=OFF` 따른 다른 영향을(플랜의 변경) 받을 수도 있으므로 optimizer_switch 값을 변경에 유의할 것 

선택적 사용 제한 
```sql 
-- 접속한 커넥션(세션) 에서만 히스토그램을 사용하지 않음
mysql> SET SESSION optimizer_switch='codition_fanout_filter=off';
-- 수행하는 쿼리에서만 히스토그램을 사용하지 않음
mysql> select /*+ SET_VAR(optimizer_switch='condition_fanout_filter=off') */ *
FROM ...
```

### 10.1.2.2 히스토그램의 용도 
MySQL에 히스토그램이 도입되기 전에도 테이블과 인덱스에 대한 통계정보는 존재
- 위에서 언급한 바와 같이, 통계 정보는 테이블의 전체 레코드 건수와 인덱스 된 컬럼의 카디널리티 
  - ex_ row = 1000, 특정 칼럼의 카디널리티 : 100 :
   해당 칼럼에 대해 동등 비교 검색(WHERE COL=값) 하게 되면 대략 10개의 레코드가 일치할 것으로 예측

히스토그램을 통해서는 버킷별로 레코드 건수와 유니크 값의 개수 정보를 알기 때문에 훨씬 정확 

ex_ birth_date 칼럼에 대한 히스토그램 유무 차이 
없을 때 
```sql 
mysql> explain
select *
from employees 
where first_name='Zita'
and birth_date between '1950-01-01' and '1960-01-01';
+----+-------------+-----------+------+----------------+----------------+---------+-------+------+----------+-------------+
| id | select_type | table     | type | possible_keys  | key            | key_len | ref   | rows | filtered | Extra       |
+----+-------------+-----------+------+----------------+----------------+---------+-------+------+----------+-------------+
|  1 | SIMPLE      | employees | ref  | idx_first_name | idx_first_name | 58      | const |  224 |    11.11 | Using where |
+----+-------------+-----------+------+----------------+----------------+---------+-------+------+----------+-------------+
```
- 만족하는 조건 수 : 224, filtered 항목 : 11.11% = 24.88명 (1950년대 생으로 예측)

있을 때 
```sql 
mysql> ANALYZE TABLE employees
UPDATE HISTOGRAM ON first_name, birth_date ;
+-------------------+-----------+----------+-------------------------------------------------------+
| Table             | Op        | Msg_type | Msg_text                                              |
+-------------------+-----------+----------+-------------------------------------------------------+
| test_db.employees | histogram | status   | Histogram statistics created for column 'birth_date'. |
| test_db.employees | histogram | status   | Histogram statistics created for column 'first_name'. |
+-------------------+-----------+----------+-------------------------------------------------------+
mysql> explain
select *
from employees 
where first_name='Zita'
and birth_date between '1950-01-01' and '1960-01-01';
+----+-------------+-----------+------+----------------+----------------+---------+-------+------+----------+-------------+
| id | select_type | table     | type | possible_keys  | key            | key_len | ref   | rows | filtered | Extra       |
+----+-------------+-----------+------+----------------+----------------+---------+-------+------+----------+-------------+
|  1 | SIMPLE      | employees | ref  | idx_first_name | idx_first_name | 58      | const |  224 |    60.95 | Using where |
+----+-------------+-----------+------+----------------+----------------+---------+-------+------+----------+-------------+
```
- 60.95% (136.5명) 
- 실제 데이터 조회시 143명이 1950년대생 

이처럼 히스토그램을 통해 특정 범위의 데이터가 많고 적음을 알 수 있으며, 이는 쿼리 성능에 영향 
- 히스토그램을 통해 데이터 건수 예측으로 드라이빙 테이블을 선택


### 10.1.2.3 히스토그램과 인덱스 



