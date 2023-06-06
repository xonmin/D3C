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
- 히스토그램과 인덱스의 공통점 : 부족한 통계 정보를 수집하기 위해 사용
- 쿼리 실행 계획 수립 시, 사용가능한 인덱스들로부터 조건절에 일치하는 레코드 수 파악 후 최종적으로 가장 나은 실행 계획 선택 
  - **인덱스 다이브(Index-Dive)** : 조건절에 일치하는 레코드 건수 예측을 위한 옵티마이저의 인덱스 B-Tree 샘플링 

```sql 
mysql> select *
from employees
where first_name='Tonny'
and birth_date BETWEEN '1954-01-01' and '1955-01-01';
```
> 만약 first_name 히스토그램이 수집되어있다면, 옵타마이저는 해당 히스토그램을 이용할까? 

- 8.0 부터 인덱스된 칼럼을 검색 조건(where) 로 사용하는 경우 : 해당 칼럼의 히스토그램 사용 x, 인덱스 다이브를 통해 수집한 정보를 활용 
- 이유는 정확성 기대값 : 샘플링 > 히스토그램 
- 따라서 히스토그램은 주로 인덱스되지 않은 칼럼에 대한 데이터 분포도에 대해 참조하는 용도로만 사용 

> 하지만 인덱스 다이브 작업은 어느정도 비용이 소요되는 작업으로, 
> 때로는 (IN 절에 값이 많이 명시된 경우) 실행 계획 수립 만으로도 상당한 많은 인덱스 다이브를 실행하게 되어 비용도 커지게 된다는 점 유의 

## 10.1.3 Cost Model 

MySQL 서버의 쿼리 처리시 필요한 작업들 
- Disk read 
- memory(InnoDB buffer pool) read 
- index 키 비교 
- 레코드 평가 
- 메모리 임시 테이블 작업
- 디스크 임시 테이블 작업 

서버는 쿼리에 대해 각각의 작업들이 얼마나 필요하고 비용이 얼마나 드는지 예측 후, 최적의 실행 계획을 찾는다 
- **Cost Model** : 전체 쿼리 비용을 계산하는 데 필요한 단위 작업들의 비용 
- ~v5.7 : 코스트 모델을 MySQL 내에 상수화하여 사용 (하드웨어에 따라 달라질 수 있어 최적의 실행계획에는 방해요소) 
- v8.0~ : 히스토그램과 각 인덱스별 메모리 내 적재된 페이지 비율 관리 -> 옵티마이저 실행 계획 수립에 사용 

- 코소트 모델은 mySQL DB 내 2개 테이블에 저장되어 있는 설정 값을 통해 사용 
  - `server_cost` : 인덱스를 찾고 레코드 비교, 임시 테이블 처리에 대한 비용 관리 테이블 
  - `engine_cost` : 레코드를 가진 데이터 페이지를 가져오는데 필요한 비용 관리 테이블 
  - 두 테이블에 공통 5개 칼럼 
    - `cost_name` : 코스트 모델의 각 단위 작업
    - `default_value` : 각 단위 작업 비용(기본값, MySQL 서버 소스 코드에서 설정된 값) 
    - `cost_value` : DBMS 관리자가 설정한 값(if null -> use default_value column)
    - `last_update`
    - `comment`
  - `engine_cost`의 추가적인 칼럼
    - `engine_name` : 비용이 적용된 스토리지 엔진 (InnoDB와 MyISAM 작업 비용 별도 설정 가능) 
    - `device_type` : 디스크 타입 



> MYSQL 코스트 모델 단위 작업 
>
> (engine_cost) : `io_block_read_cost`, `memory_block_read_cost`
> 
> (server_cost) : `disk_temptable_create_cost`, `disk_temptable_row_cost`, `key_compare_cost`, 
`memory_temptable_create_cost`, `memory_temptable_row_cost`, `row_evaluate_cost`

`row_evaluate_cost` 스토리지 엔진이 반환한 레코드가 쿼리 조건에 일치하는지를 평가하는 단위 작업 
- 값이 증가할수록 풀 테이블 스캔과 같이 많은 레코드를 처리하는 쿼리의 비용이 높아지고,
- 반대로 레인지 스캔과 같이 적은 수의 레코드를 처리하는 쿼리의 비용이 낮아진다.

`key_compare_cost` : 키 값의 비교 작업에 필요한 비용 값이 증가할수록, 레코드 정렬과 같이 키 값 비교 처리가 많은 경우 쿼리 비용이 높아짐 

> 코스트 모델에서 중요한 것은 각 단위 작업에 설정되는 비용 값이 커지면 어떤 실행 계획들이 고비용으로 바뀌고 어떤 실행 계획들이 저비용으로 바뀌는지를 파악하는 것

# 10.2 실행 계획 확인 
`DESC` 또는 `EXPLAIN` 명령으로 실행 계획 확인 가능 

## 10.2.1 실행 계획 출력 포맷 
- 8.0 이전 : `EXPLAIN EXTENDED` or `EXPLAIN PARTITION` 
- 8.0 이후 : FORMAT 옵션을 사용해 실행 계획의 표시 방법을 JSON, TREE, 단순 테이블 형태로 선택 가능 

## 10.2.2 쿼리 실행 시간 확인 
`EXPLAIN ANALYZE`를 쓰면 쿼리의 실행 계획과 단계별 소요된 시간 정보를 확인 가능 (이 명령은 항상 결과를 TREE 포맷)
```sql 
EXPLAIN ANALYZE 
SELECT avg(s.salary)
FROM employees e INNER JOIN salaries s ON s.emp_no=e.emp_no
					AND s.salary>50000
					AND s.from_date<='1990-01-01'
					AND s.to_date>'1990-01-01'
WHERE e.first_name='Matt'
GROUP BY e.hire_date;
```

```sql
-> Table scan on <temporary>  (actual time=0.002..0.005 rows=48 loops=1)
    -> Aggregate using temporary table  (actual time=3.088..3.095 rows=48 loops=1)
        -> Nested loop inner join  (cost=618.92 rows=2239) (actual time=0.303..2.991 rows=48 loops=1)
            -> Index lookup on e using ix_first_name (first_name='Matt')  (cost=166.76 rows=233) (actual time=0.286..0.736 rows=233 loops=1)
            -> Filter: ((s.salary > 50000) and (s.from_date <= DATE'1990-01-01') and (s.to_date > DATE'1990-01-01'))  (cost=0.98 rows=10) (actual time=0.008..0.009 rows=0 loops=233)
                -> Index lookup on s using PRIMARY (emp_no=e.emp_no)  (cost=0.98 rows=10) (actual time=0.005..0.008 rows=10 loops=233)
```

- TREE 포맷 실행 계획에서 들여쓰기는 호출 순서를 의미하며, 다음 기준으로 읽으면 된다.
  - 들여쓰기가 같은 레벨에서는 상단에 위치한 라인이 먼저 실행
  - 들여쓰기가 다른 레벨에서는 가장 안쪽에 위치한 라인이 먼저 실행

> 1) Index lookup on e using ix_first_name
> first_name 인덱스를 이용하여 first_name='Matt' 인 것만 가져온다.
> 2) Index lookup on s using PRIMARY
> primary key를 이용하여 emp_no가 같은 동일한 레코드를 가져옴
> 3) Filter : ((s.slary > 50000) and ...
> inner join시 만족해야하는 조건에 대한 레코드를 가져온다.
> 4) Nested loop inner join
> inner join을 수행한다.
> 5) Aggregate using temporay table
> 임시 테이블로 부터 집계 결과를 수행한다.
> 6) Table scan on <temporary>
> 테이블을 스캔해서 결과를 반환한다.
  
  
- 실행 계획에 나열된 필드들의 의미
  - actual time 숫자 값이 2개 표시되는데, 
    - 첫 번째 숫자 값은 첫 번째 레코드를 가져오는 데 걸린 평균 시간
    - 두 번째 숫자 값은 마지막 레코드를 가져오는 데 걸린 평균 시간
  - rows :해당 테이블에서 읽은 조건에 일치하는 평균 레코드 건수
  - loops : 해당 테이블에서 읽은 정보를 이용해 해당 레코드를 찾는 작업이 반복된 횟수


# 10.3 실행 계획 분석 
실행 계획에서 위쪽에 출력된 결과일수록 쿼리의 바깥 부분이거나 먼저 접근한 테이블이고, 
아래쪽에 출력된 결과일수록 쿼리의 안쪽 부분 또는 나중에 접근한 테이블에 해당.

## 10.3.1 id 칼럼 
- id 칼럼은 단위 SELECT 쿼리별로 부여되는 식별자 값 (SubQuery 일 경우 id 값 증가) 
- 하나의 SELECT 문장 안에서 여러 개의 테이블을 조인하면 조인되는 테이블의 개수만큼 실행 계획 레코드가 출력되지만 같은 id 값 부여
  - id 칼럼이 테이블 접근 순서를 의미하지는 않는다.
  - 테이블 포맷으로 보면 접근 순서가 혼란스러울 수 있다. FORMAT=TREE로 보면 순서를 더 정확히 알 수 있다.
  
## 10.3.2 select_type 칼럼 
각 단위 SELECT 쿼리가 어떤 타입의 쿼리인지 표시되는 칼럼
표시될 수 있는 값은 다음과 같다.
  
### 10.3.2.1 SIMPLE 
- UNION이나 서브쿼리를 사용하지 않은 단순한 SELECT 쿼리
- 아무리 복잡한 쿼리라도 SIMPLE인 단위 쿼리는 하나만 존재한다. 일반적으로 제일 바깥 SELECT 쿼리가 SIMPLE이다.
  
### 10.3.2.2 PRIMARY 
- UNION이나 서브쿼리를 가지는 SELECT 쿼리의 가장 바깥쪽에 있는 단위 쿼리
- 쿼리에서 PRIMARY인 단위 쿼리는 하나만 존재한다. 제일 바깥 SELECT 쿼리가 PRIMARY이다.

### 10.3.2.3 UNION 
- UNION으로 결합하는 단위 SELECT 쿼리 가운데 첫 번째를 제외한 두 번째 이후 단위 SELECT 쿼리들
- UNION의 첫 번째 단위 SELECT는 select_type이 DERIVED(쿼리 결과에 대한 임시 테이블)이다.
```sql 
  mysql> EXPLAIN 
    -> 
    -> SELECT employee1.emp_no, employee1.first_name, employee1.last_name
    -> FROM employees employee1
    -> WHERE employee1.emp_no = 10001
    -> 
    -> UNION ALL
    -> 
    -> SELECT employee2.emp_no, employee2.first_name, employee2.last_name
    -> FROM employees employee2
    -> WHERE employee2.emp_no = 10002;
+----+-------------+-----------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
| id | select_type | table     | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra |
+----+-------------+-----------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
|  1 | PRIMARY     | employee1 | NULL       | const | PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | NULL  |
|  2 | UNION       | employee2 | NULL       | const | PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | NULL  |
+----+-------------+-----------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
2 rows in set, 1 warning (0.00 sec)
```
