## 아티클 링크

https://techblog.woowahan.com/6550/
## 세줄 요약
- 구체적인 성능 비교
- PostgreSQL 만의 특징
- 총평

## 내용

### 단순 CRUD Query

![500*500](https://i.imgur.com/87ib70P.png)

( 하단 숫자는 , Thread + 10tables , 100scale => 90GB 추정 )
##### QPS
- Query For Second
- 초당 실행하는 Query 총량 (  조회량 / 초 )
- 주로 읽기 , 쓰기 , 업데이트 등 단일 처리 능력 평가에 사용
##### TPS
- Transactions Per Second
- 초당 실행하는 Transaction 총량 ( 처리량 / 초 )
- 데이터 무결성 유지하는 작업이나 , 여러 단계 포함하는 작업 처리 능력 평가에 사용

=> MySQL 이 성능 결과가 더 좋음
	- Update 시 ,  Insert & Delete 로 인한 성능 저하!
	- Insert , Select 위주 서비스에서 Postgre 선호

### 복잡 Query

1000만 건 데이터 조인 쿼리를 HASH JOIN 으로 비교 실행

#### Data Setting
```sql
create table users (
     id int auto_increment primary key,
     id2 int, 
     Name varchar(100),
     Address varchar(512)
);

insert into users(id2,Name,Address)
select floor(1 + rand() * 50000000),
A.table_name,A.table_name
from information_schema.tables A
   cross join information_schema.tables B
   cross join information_schema.tables C
   cross join information_schema.tables D
limit 10000000;
```
- Postgre , Mysql 동일 Data 로 수행
##### Cross Join
- 두 테이블 간 카테시안 곱 생성
- 첫 테이블 행들이 두번째 테이블 모든 행에 결합 ( A : 3 , B : 4 => A`*`B = 12 )

=> MySQL 은 22초 , PostgreSQL 은 3초 ( Nestsed Loop 일 시 , 더 오래 걸릴 예정! )

### Partial Index
- 전체 데이터 중 , 부분집합에 대해서만 인덱스 생성하는 기능
- 특정 범위만 인덱싱 하므로 , 인덱스 크기가 작고 리소스 줄일 수 있는 이점 존재

=> Partial Index 와 B- Index 간 조회 속도 차이는 별로 없었음!
=> MySQL ( 755M ) < PostgreSQL ( 57M ) 정도로 10배 크기 차이

### Secondary Index
#### In MySQL
- 5.7이 되며 Online ddl 기능 범위가 넓어져 성능 개선은 됐으나 , 대량 테이블 인덱스 생성 & 컬럼 추가 작업이 부담스러운 작업!
	-> 데이터가 많은 만큼 시간소요 예측 과 실패할 시 rollback 에 대한 위험도가 매우 커짐!
		+ 백업데이터로 테스트를 진행해도 , 라이브에서 장담 못함
#### In PostgreSQL
- 어마어마한 성능 차이
- 200G 테이블에 Index , Column 추가 작업
##### 인덱스 생성
- 40여분 만에 생성! ( MySQL 은 100G 만 넘어도 1시간 경과 )
##### 칼럼 추가
- 바로 완료(!)
=> Online ddl 컬럼 추가는 meta data를 저장하는 시스템 카탈로그에 추가된 정보만 반영
#### Online ddl
- Data Define Language
- Schema 변경 수행 동안 DB의 가용성 유지하는 기능
- 테이블 구조 변경하거나 인덱스 추가/제거 작업 수행 동안 중단되지 않고 온라인 상태 유지해 변경 사항 적용 가능하게 함
- 기존에는 테이블 잠그거나 , 서비스 중단 경향이 있으나 -> 중단 없이 변경 가능!

### 그 외 PostgreSQL 특징
#### SP 생성
- 저장 프로시저 ( Stored Procedure )
- DB에 저장되어 있는 하나 이상 SQL 문의 집합 , 필요할 떄 호출해 실행
- C , C++ , Java , Ruby , Python 등 많은 프로그램이 언어로 생성(? 호출인듯) 가능

#### PostGIS
- Geographic Object 지원
- oracle GIS 와 성능이 비견할 정도로 우수!
#### Vacuum
- MVCC 를 MGA 방식으로 구현
- Update -> Delete 시 새로운 영역 할당해 사용
( 이전 공간은 재사용 될 수 없는 dead tuple 상태! )
-> 공간 부족 및 Data IO 의 비효율을 유발 해 성능저하의 원인!
=> 주기적으로 vacuum 기능을 수행해 재사용 가능하게 관리해줘야 함
#### Materialized View 지원
- materialized view 는 생성 시 설정 조건의 쿼리 결과를 별도 공간에 저장
	-> 실행될 때 미리 저장된 결과 보여줌 ( 성능 향상! )
- 실시간 노출 적은 통계성 Query + 자주 Update 되지 않는 테이블에서 유용
#### 상속기능
- 부모 테이블을 상속해 하위 테이블 생성 가능
	- 하위 테이블 : 부모 테이블 컬럼 제외한 컬럼만 추가 생성 가능!
	- 상위 테이블 조회 시 하위 테이블 데이터 까지 모두 가능! + 데이터 변경 시 , 하위 테이블 까지 모두 반영
#### 다양한 사용자 기반 활용 가능
- 연산자 , 복합 자료형 , 집계 함수 , 자료형 변환자 , 확장 기능 등 다양한 객체를 사용자가 임의로 만들 수 있음



## 결론

- 단순히 , PostgreSQL 이 더 좋다. MySQL이 더 좋다의 관점이 아님!
- 복잡한 로직과 대규모 데이터를 관리할 때는 PostgreSQL 을 고려해보자!


## 참고 링크



##### Writed By Obisidan