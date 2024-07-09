---
tags:
  - 데이터베이스
아티클_링크: https://techblog.woowahan.com/6550/
아티클_작성자: 정지원
추가_정보: 우아한형제들 테크 블로그
---
## 아티클 링크




## 세줄 요약

## 내용
### 서론 : 왜 MySQL -> PostgreSQl 로 넘어갔는가?
데이터가 누적될수록 ( 특히 통계 정산 등 고성능 대량 필요 ) 작업 수행 속도는 느려진다.
MySQL 에서 동작중인 서비스는 쿼리 개선 작업을 진행했으나 , 드라마틱 효과는 볼 수 없었다!
=> 대량 데이터 처리에 특화 되어있는 PostgreSQL 로 이관 고려

### Aurora ( MySQL vs PostgreSQL )
#### Aurora?
- AWS 에서 제공해주는 관계형 DB Service
- 고성능 & 고가용성 목표 설계 + MySQL / PostgreSQL 두 가지 버전 제공!
- 자동 확장 & 빠른 백업 & 복제 & 장애 복구 기능등 제공
##### Aurora VS Rds
자칫 생각하면 , 그러면 Aurora 랑 Rds 는 같은게 아니야? 라고 생각할 수 있다.
- Aurora 는 AWS 에서 자체 개발한 DB Engiene ( 따라서 더 높은 성능 제공 )
	-> 클라우드 이점을 활용해 대규모 처리 작업 & 자동 장애 복구에 최적화!
=> RDS 의 상위 버전의 느낌 ( 대규모 에 적합 )

#### DB 특성
- MySQL : RDBMS ( 일반적인 Relation DataBase Management System )
- PostgreSQL : ORDBMS ( Object Relation Database Management System )
	-> RDBMS  에 객체 지향 DB 기능을 추가함 ( 기존 type 에서 확장 type 자유롭게 정의해 사용 가능 + 상속 기능 )
#### 방식
- MYSQL : 멀티 스레드
- PostgreSQL : 멀티 프로세스
	-> 더 나은 격리 단위를 제공하나 , 더 많은 자원 소비 역시 가능!

#### 사용환경
- MYSQL : OLTP 에 적절
- PostgreSQL : OLTP , OLAP 에 적절
##### OLTP
- Online Transaction Processing
- 일상적인 트랜잭션 처리에 사용
- 빠른 쿼리 처리 속도 , 데이터 무결성 , 동시성 , 빈번한 갱신 작업에 필요
##### OLAP
- Online Analytical Processing
- 대량 데이터 분석 , 보고서 생성에 사용 ( 데이터 마이닝 , 복잡한 쿼리 등 )
- 대규모 데이터 처리 , 복잡한 쿼리 & 분석 작업에 최적화

#### MVCC ( Multi Version Concurrency Control ) 지원
- MySQL : Undo Segement 지원
	-> update 된 최신 데이터는 기존 레코드에 반영 , 이전 값은 undo 라는 별도 공간에 저장
- PostgreSQL : MGA ( Multi Generation Architecture ) 방식
	-> 튜플을 update 할 때 , 새로운 튜플을 추가하고 이전 튜플은 유효 범위를 마킹해 처리
=> Undo 는 기존 데이터 블록 내 레코드 실제 업데이트 , MGA 는 새로운 레코드 추가

#### Update 방식
- MySQL : Update
- PostgreSQL : Insert & Delete ( check )
	-> Update 시 , 새 데이터가 Insert 되고 , 이전 데이터는 삭제 표시
		( 행이 업데이트 되면 , 변경 위치값에 대한 인덱스 정보도 업데이트 )
=> Update 시 , Mysql 보다 성능 떨어짐
#### Join 
- MySQL : NL Join , HASH Join ( 8.0 이후 지원 )
- PostgreSQL : NL Join , HASH Join , SORT Join
##### NL Join
- 한 테이블 각 행을 다른 테이블 모든 행과 비교해 조건 맞는 행 찾음
- 당연히 , 작은 데이터 Set 일때만 효과적
##### Hash Join
- 한 테이블 행을 Hash Table 로 변환 후 , 다른 테이블 행과 비교해 조건 맞는 행 찾음
- 대규모 Set 일때 빠른 Join 제공
##### Sort Merge Join
- PostgreSQL 전용
- 두 테이블을 각각 정렬 해 , 순서대로 조건 맞는 행 합병
- 두 테이블이 이미 정렬 되 있거나 , 정렬에 유리한 상황일때 효과적

#### Parallel Query for SELECT
- MySQL : 8.0 부터 지원 ( Aurora 에선 5.7 부터 지원 ) - 활용이 어려웠음
- PostgresSQL : 9 부터 지원 ( 오래 됨 )
##### Parallel Query for SELECT?
여러 CPU 코어를 활용해 , Select Query 성능을 향상 시키는 기능
- 한 Query 를 여러 부분으로 나누어 동시 처리
- 큰 데이터 Set 처리하거나 , 복잡한 조인 , 집계등을 수행할 때 유용
장점 : 응답 시간을 줄이고 , 처리량을 늘림!
단점 : CPU & 리소스 사용량을 증가시키므로 , 전반적 부하 와 리소스 가용성 고려해야 함

#### Default Transaction Isolation
- MySQL : REPEATABLE READ
- PostgreSQL : READ COMMITTED
##### REPEATABLE READ
- 트랜잭션이 시작된 시점 데이터 스냅샷 기반 작업 수행
- 다른 트랜잭션이 수행한 변경 사항 고려 하지 않음
- 데이터의 일관성을 보장하나 , 동시성이 떨어질 수 있음!
- Phantom Read 는 발생 X

##### READ COMMITTED
- 트랜잭션 중 다른 트랜잭션에 의해 커밋된 변경 사항 볼 수 있음
- 각 쿼리는 실행 시점 커밋된 데이터 조회
- 동시성은 높으나 , 트랜잭션 내 같은 쿼리의 결과는 다를 수 있음! - Non-Repeatable Read
- 동시성은 높으나 , 일관성이 높지는 않음!
- Phantom Read 는 발생할 수 있으나 , Dirty Read 는 발생 X

=> Phatnom Read 와 Dirty Read 는 부록에서 설명
#### Table Default Index
- MySQL : CLUSTERED INDEX
- PostgreSQL : Non-CLUSTERED INDEX
##### Clustered Index
- 데이터의 인덱스를 물리적순서로 저장
- 테이블에는 하나의 Clustered Index 만 존재 가능
- 순서대로 저장이 되므로 , 검색이 빠름
- 재구성 할 필요 없이 , 데이터 삽입 과 업데이트가 빠름

##### Non-Clustered Index
- 별도 공간에 인덱스를 저장 , 실제 데이터 다른 위치 저장
- 테이블에 여러 Non-Clustered Index 존재 가능


## 결론


## 참고 링크



##### Writed By Obisidan