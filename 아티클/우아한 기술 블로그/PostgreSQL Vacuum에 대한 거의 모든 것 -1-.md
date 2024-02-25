## 아티클 링크
https://techblog.woowahan.com/9478/

## 세줄 요약

## 내용
### Vacuum 이란?
- 사전적 의미 : 진공 청소기 (진공)
- GC 의 역활을 함
- PostgreSQL 만의 특별한 동작 ( MVCC 구현 방법이 MySQL , Oracle 과 달라서 )
### MVCC 란?
- Multi Version Control Concurrency
- 동시성을 위해 제공
- 여러 트랜잭션 수행 환경에서 트랜잭션의 쿼리 수행 시점의 데이터를 제공해 읽기 일관성 보장
		+ Read / Write 간 충돌 및 lock 방지해 동시성을 높임!
+ 기본적 원리는 시작된 시점 Transaction ID 와 같거나 작은 Transaction ID 가지는 데이터를 읽음!

### Oracle & MySQL MVCC
- UNDO Segment ( Rollback Segment ) 사용!
#### Oracle MVCC
![500*500](https://i.imgur.com/tuZzcO2.png)

##### SCN
- System Change Number
- 시스템 변화 추적 위한 일련번호 or 타임스탬프

- 쿼리 시작 이후 다른 트랜잭션 의해 변경된 블록 만나면 원본 블록으로 복사본 ( CR copy ) 만들고 , 
	그 복사본 블록에 UNDO Segment 적용해 쿼리 시작된 시점으로 되돌려서 읽는 방식 사용
=> 결국 , UNDO 에서 그전 Version 을 찾아서 수행 ( 일관성 과 격리는 보장 )

#### MySQL MVCC
![500*500](https://i.imgur.com/ZdFLm1z.png)

- Oracle 과 유사
- 트랜잭션 변경된 데이터가 UNDO 영역 저장
- 변경된 내용들은 앞선 내용들을 포인터로 가리킴
- 신규 트랜잭션 수행 시 , 포인터로 연결된 리스트 searching 하며 
	trx_id 를 비교해 자신이 읽어야 하는 시점 데이터 확인

### PostgreSQL MVCC

- 데이터 Page 내에 변경되기 이전 Tuple 과 신규 Tuple 같은 Page 에 저장
- Tuple 별 생성된 시점 과 변경된 시점 기록 및 비교하는 방식으로 제공
	( PostgreSQL 은 record 대신 Tuple 용어 사용 )

- Tuple 이 생성되거나 변경된 시점을 Tuple 내 xmin , xmax 
	라는 Metadata field 에 기록해 어떤 Tuple 읽을 수 있는지 버전 관리 하게 함
##### xmin : 그때부터 존재
- Insert , Update 하는 시점 Transactional ID 갖는 메타데이터
- Insert 경우 , 신규 Tuple의 xmin 에 해당 시점 Transactional ID 할당
- Update 경우 , update 된 신규 Tuple의 xmin 에 해당 시점 Transactional ID 할당
##### xmax : 그때까지 존재
- Delete , Update 하는 시점 Transactional ID 갖는 메타데이터
- Delete 경우 , 변경되기 이전 xmax 에 해당 시점 Transactional ID 할당
- Update 경우 , 이전 xmax 와 신규 xmin에는 해당 시점 Transactional ID 할당,
	update 되는 신규 Tuple xmax 에는 NULL 할당
![500*500](https://i.imgur.com/2mZBv10.png)

=> Old , New Version 은 Transaction ID 가 저장된 xmin & xmax 값 비교해 판단
```
 xmin  | xmax  |  value
-------+-------+-----
  2010 |  2020 | AAA
  2012 |     0 | BBB
  2014 |  2030 | CCC
  2020 |     0 | ZZZ
```
- Transaction 2015?
	-> ZZZ 는 xmin 이 2020 이므로 미래값이므로 보지 못함 ( AAA BBB CCC 는 가능 )
- Transaction 2021?
	-> AAA 는 xmax 가 2020이므로 과거값이므로 보지 못함 ( BBB CCC ZZZ 는 가능 )
- Transaction 2031?
	-> BBB , ZZZ 만 볼수 있음 (위와 같은 이유 )
### Vacuum의 필요성
#### 1. Dead Tuple
##### Update 동작
- FSM ( Free Space Map )에 사용 공간 있는지 확인 -> 없으면 FSM 추가 확복
- FSM의 사용 가능 공간에 update 된 데이터 기록 ( insert 와 유사 )
- update 된 데이터 저장 완료 시 , 이전 Tuple 가르키는 포인터 새로 update 된 Tuple 가리키게 변경
=> 기존 데이터 저장한 Tuple 은 어디에도 참조되지 않은 Dead Tuple 가 된다!

![500](https://i.imgur.com/FCrjSpS.png)

- Dead Tuple 은 쓸데없는 공간 차지
- Dead Tuple 때문에 더 많은 Page 를 읽고 -> Query 성능 저하!
###### In Detail
- PostgreSQL 은 page 단위로 읽어 Memory 에 올림
- page 에는 live 뿐만 아니라 dead 도 포함되어 있음!
	-> page 크기는 8KB(default) 한정되어 있고 , Dead Tuple 이 쓸데없는 공간 차지!
	-> 필요 live tuple 을 위해선 , 더 많은 page 를 읽어야 하고 더 많은 디스크 I / O 발생으로 이어짐!

##### Vacuum 수행 전 Dead Tuple 존재하는 상황
```
select count(*) from tb_test;
  count
----------
 10000000
(1 row)
Time: 548.779 ms

delete from tb_test where ai != 1;
DELETE 9999999
Time: 30142.916 ms (00:30.143)

select count(*) from tb_test;
 count
-------
     1
(1 row)
Time: 497.835 ms
=> 1건을 count하는데도 오래걸림

SELECT relname, n_live_tup, n_Dead_tup, n_Dead_tup / (n_live_tup::float) as ratio
FROM pg_stat_user_tables;
 relname  | n_live_tup | n_Dead_tup |  ratio
----------+------------+------------+----------
 tb_test  |          1 |    9999999 |  9999999
=> Dead Tuple이 9999999개로 증가함, 반면에 live Tuple은 1건

dt+ tb_test
                            List of relations
 Schema |  Name   | Type  | Owner  | Persistence |  Size   | Description
--------+---------+-------+--------+-------------+---------+-------------
 public | tb_test | table | master | permanent   | 1912 MB |
=> 1건짜리 테이블임에도 거의 2GB인 상황
```

- Dead Tuple 들이 , 공간을 전부 차지하므로 매우 오래 걸림 + 공간을 전부 차지하고 있음!
```
vacuum tb_test;
VACUUM
Time: 11273.959 ms (00:11.274)
testdb=> dt+ tb_test;
                           List of relations
 Schema |  Name   | Type  | Owner  | Persistence | Size  | Description
--------+---------+-------+--------+-------------+-------+-------------
 public | tb_test | table | master | permanent   | 1912 MB |
=> vacuum 수행 후 dead tuple은 정리되었지만 테이블 사이즈는 그대로임

vacuum full tb_test;
VACUUM

dt+ tb_test;
                           List of relations
 Schema |  Name   | Type  | Owner  | Persistence | Size  | Description
--------+---------+-------+--------+-------------+-------+-------------
 public | tb_test | table | master | permanent   | 40 kB |
=> vacuum full 수행 후에는 테이블 사이즈도 줄어들었음
```
- Vacuum full 과 Vacuum 의 차이점이 있음!
	( 단순 Vacuum 으로는 물리 디스크로 할당된 크기는 회수되지 않음! )

![500](https://i.imgur.com/5DKp0JL.png)

- Vacuum : OS 디스크 공간 반환 처리 X ( FSM 에만 반환해 이 공간 재사용 가능하게 처리 )
- Vacuum Full : OS 디스크 공간 반환
=> Vacuum Full 이 무조건 최고 아닌가?
### ❌
#### Vacuum Full
- OS 디스크 공간까지 반환
- Access Exclusive Lock 획득해 DML ( Select 포함 ) 도 대기하게 만들어 운영 중에는 불가능한 작업
	( Vacuum 은 online 작업 지원 + DML 가능 - DDL 은 불가 )
- Table 을 COPY  하는 식으로 동작!
	( 디스크 가용량 여유롭지 않을 시 , 시도조차 불가능 )
->  pg repack 이라는 오픈소스 툴이 있음!
	( 이 역시도 , 여유 디스크 가용량 + 여러 번 Metadata Lock 필요! )
=> 적절한 튜닝 과 정책을 잘 세워 평소 Dead Tuple 정리를 원활히 해 불필요 공간 늘어남을 방지하자!!

#### 2. Transaction ID Wraparound 방지

위 문제도 심각하나 , 서비스 중단 일으킬 정도는 아님!
제때 vacuum이 수행되지 않아 , Transaction ID 정리가 안될 시 모든 Write 작업이 멈출 수 있음!
	-> DB에서 AutoVacuum 을 Off 해도 , 임계치 초과 시 DB에서 강제 수행
##### Transaction ID
- xmin , xmax field 에 저장된 해당 ID 값 통해 MVCC 구현
- 각각 4byte 할당
- 40억 ( 2^32 ) 개 트랜잭션 표현 ( 20억 개는 과거 , 20억 개는 미래 )

Transaction ID 소진해 다시 1부터 시작하면?
=> Transaction ID wraparound 발생!
##### Transaction ID wraparound

- 한 바퀴 돈 Transaction ID 1은 기존 데이터보다 최신 데이터임에도 불구하고 , 
	기존 Data 가 모두 1보다 크므로 과거 데이터인 미래 데이터가 모두 미래에 있는 거처럼 보이지 않게 됨
=> 과거 데이터들이 모두 손실되는 wraparound 현상 발생!

- Transaction ID 를 재사용 하기 위해 과거 Data의 Transaction ID 를 계속 증가시킨게 아닌,
	특정 시점에선 모두 fronzen XID = 2 라는 특별 Transaction ID 로 바꿈!
![500](https://i.imgur.com/HWgwbFX.png)
( freeze , Anti Wraparound Vacuum 이라고 불림 )

- ID 가 더 증가하지 않게 하여 , DB가 안전하게 작동하게 함
- VACUUM 과정에서 무시 가능

##### Age

- Table 같은 Object 와 Tuple 의 나이
- Table 생성 or Tuple 처음 Insert 할 때 age는 1부터 시작
- 해당 테이블 대한 트랜잭션 아니더라도 , DB에서 트랜잭션 발생 마다 모든 Object 와 Tuple 의 age가 1씩 증가
- age 가 증가하다 , 임계치에 도달하면 Wraparound 방지 위한 대상이 되고 ,
	Vacuum이 수행된 후에는 Table 과 Age 는 다시 회춘! ( freezing )
##### Tuple age
- Anti Wraparound Vacuum 이 수행 시 freeze 되는 대상 자체
- vacuum_freeze_min_age ( default : 5천만 ) 설정값 보다 높은 Tuple 이 대상이 됨
##### Table Age
- 존재하는 Tuple 중 가장 오래된 Tuple 의 나이
- vacuum 시 , Table 의 Age 만 보고도 판단 가능!


## 결론


## 참고 링크



##### Writed By Obisidan