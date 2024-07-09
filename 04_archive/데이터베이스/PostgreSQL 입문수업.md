---
tags:
  - 데이터베이스
강의_링크: https://www.youtube.com/watch?v=dKuLA5BGPTY
---
## PostgreSQL
- RDBMS
 ![](https://i.imgur.com/qsw17IU.png)

- 4번째 점유율을 가질 정도로 인기 있음( 꾸준한 성장세 )


### Client

다양한 Client 들 존재
#### pgAdmin 4

- GUI 로 , 서버를 원격으로 관리하게 도와줌
#### psql

- CLI 로 , 서버를 원격으로 관리하게 도와줌

### Server
- 대부분의 일이 처리되는 곳
![](https://i.imgur.com/QX1Fltz.png)

- SQL 통해 처리
- Cluster 에 연결하여 작업 수행

### Install

- 간단한 테스트 확인만 했으므로 , docker-compose 통해 install

```yml
version: '3.8'
services:
  postgres:
    image: postgres:latest
    container_name: postgres_container
    environment:
      POSTGRES_USER: root 
      POSTGRES_PASSWORD: 1234 
      POSTGRES_DB: test  
    volumes:
      - postgres_data:/var/lib/postgresql/data  
    ports:
      - "5432:5432"  
    # restart: always 

volumes:
  postgres_data:  
```
- GPT 참조
- docker exec -it postgres_container psql -U root -d test
	-> 터미널에서 , 해당 명령어로 , 접속 가능

### CREATE

- 다른 RDBMS 와 매우 유사
#### 기본적인 게시글 생성 sql 문
- Mysql 과 차이적인 부분만 설명
```postgresql
create table public.topic
(
	id serial NOT NULL,
	title character varying(50) NOT NULL,
	body text,
	created timestamp with time zone NOT NULL DEFAULT now(),
	PRIMARY KEY (id) 
);
ALTER TABLE IF EXISTS public.topic
	OWNER to postgres;
```
- public.topic : <Schema명> : <Table명>
- serial : Auto Increment 처럼 , 정수 자동 증가
- character varying(50) : 최대 50자 까지 저장 가능한 가변 문자열
- text : 길이 제한 없는 텍스트 데이터
- DEFAULT now() : 기본저긍로 , 현재 시간 사용

- OWNER to postgres : 테이블의 소유자 변경

### SELECT , UPDATE , DELETE 

- 사실상 , 완전히 동일
```postgresql
select id,title from topic where id <> 1 order by id DESC;
```
- <> 은 NOT 연산자
```postgresql
update topic set title ='test' , body = 'delete' where id = 4;
```

```postgresql
delete from topic where id = 4;
```

## MYSQL VS PostgreSQL
### ACID

- Psql 은 ACID 원칙 엄격히 준수 하며 , 다양한 격리 수준을 지원 ( READ Uncommitted , READ Committed , Repeatable Read , Serializable )
	-> 더 복잡한 트랜잭션 처리 가능
- Mysql 은 InnoDB 엔진은 ACID ( + 격리 수준도 지원 ) 를 지원하나 , 다른 엔진은 완전히 지원하지 않을수도 있음.
### SQL 표준 준수

- Psql 은 CHECK 제약 조건 완전히 지원
	-> 특정 컬럼 값이 , 사용자 정의 조건 충족하는지 대한 규칙 등 사용 가능
- Mysql 은 초기에는 무시했으나 , 최근 지원 ( Procedure 나 Trigger 도 다소 밀림 )

### 성능 & 확장성

- Psql 이 복잡한 쿼리 실행하거나 , 대규모 데이터 세트 다룰 떄 유리함
	- Query Planner , Optimizer 통해 복잡한 조인 , 서브쿼리 , 윈도우 함수 등 효율적이게 설계 가능
	- MVCC ( Multi-Version-Concurrency-Control ) , 테이블 파티셔닝 , 병렬 처리 쿼리 기능등 대용량 처리 지원
	- EX ) EXPLAIN ANALYZE 로 , 쿼리 실행 계획 분석 가능
```postgresql
EXPLAIN ANALYZE select id from public.users;
                                            QUERY PLAN                                            
--------------------------------------------------------------------------------------------------
 Seq Scan on users  (cost=0.00..13.00 rows=300 width=4) (actual time=0.073..0.113 rows=1 loops=1)
 Planning Time: 0.662 ms
 Execution Time: 0.229 ms
```
- Mysql 은 간단한 읽기 작업에 최적화된 설계 ( PostgreSQL 에 비해선 , 성능이 뒤떨어짐 )
	- 물론 , InnoDB 는 트랜잭션 및 MVCC 도 지원

### JSON 지원

- Psql JSON 을 완벽히 처리 ( 일급 시민 : 아무 제약 없이 사용 가능 )
	- BSON 과 JSON 두가지 지원
```postgresql
SELECT json_data->>'key' AS value FROM my_table WHERE json_data->>'key' = 'some_value';
```
- json_data->>'key' : <JSON 타입의 Column명> : < Key 에 해당하는 값 >
#### JSON Example
```postgresql
CREATE TABLE user_profiles (
    id SERIAL PRIMARY KEY,
    profile_info JSON NOT NULL
);

INSERT INTO user_profiles (profile_info) VALUES 
('{"name": "John Doe", "age": 30, "interests": ["football", "coding"]}');

SELECT * FROM user_profiles WHERE profile_info->>'name' = 'John Doe';
```

- 여기서 주의해야할 점은 쌍 따옴표 ( " ) 사용시 , 에러뜸!

- Mysql 도 JSON 지원 ( BSON 은 지원 X ) 하나 , 함수 및 연산자가 부족

