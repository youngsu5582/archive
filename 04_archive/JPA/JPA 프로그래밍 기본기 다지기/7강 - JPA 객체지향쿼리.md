---
tags:
  - JPA
강의명: JPA 프로그래밍 기본기 다지기
강의_링크: https://www.youtube.com/watch?v=wt_BEqxjaj8
---
JPA 는 다양한 쿼리 방법 지원
- JPQL
- JPA Criteria
- QueryDSL
- Native SQL
- JDBC API 직접 지원 , MyBatis , SpringJDBCTemplate 함께 사용

### JPQL

- 가장 단순한 조회 방법
	- EntityManager.find()
	- 객체 그래프 탐색 ( a.getB().getC() )

- JPA 를 사용하면 엔티티 객체 중심 개발 ( 생성 , 수정 , 삭제 )
	-> 문제는 검색 쿼리
- 검색 할 때 테이블 아닌 엔티티 객체 대상으로 검색 가능
- 모든 DB 데이터를 객체 변환해서 검색하는 것은 불가능
- 필요 데이터를 DB에서 불러오려면 검색 조건 포함 SQL 필요

=> 
- SQL 추상화한 JPQL 이라는 객체 지향 쿼리 언어 제공
- SQL 문법 유사 , SELECT FROM , WHERE , GROUP BY , HAVING , JOIN 들 지원
- 엔티티 객체 대상 쿼리 ( SQL은 DB 테이블 대상 쿼리 )

```java
String jpql = "select m From Member m where m.name like '%hello%'";
List<Member> result = em.createQuery(jpql, Member.class).getResultList();
```
테이블이 아닌 객체 대상 검색하는 객체 지향 쿼리
SQL 추상화해서 특정 DB SQL 에 의존 X
-> JPQL은 객체 지향 SQL

#### JPQL - SQL

```java
String jpql = "select m From Member m where m.age > 18";
List<Member> result = em.createQuery(jpql, Member.class).getResultList();
```

```SQL
select
    m.id as id,
    m.age as age,
    m.USERNAME as USERNAME,
    m.TEAM_ID as TEAM_ID
from
    Member m
where
    m.age > 18
```

![400](https://i.imgur.com/5vmg9Xu.png)

사진과 같이 작성 가능

- 엔티티 속성은 대소문자 구분 ( Member , username )
- JPQL 키워드는 대소문자 구분 X (SELECT,FROM, where )
- 엔티티 이름 사용 , 테이블 이름 X
- 별칭 필수

#### 결과 조회 API

- getResultList() : 결과 하나 이상 , 리스트 반환
- getSingleResult() : 결과 정확히 하나 , 단일 객체 반환 ( 하나가 아니면 , 예외 발생 )


#### 프로젝션

- SELECT m FROM Member m -> 엔티티 프로젝션
- SELECT m.team FROM Member m -> 엔티티 프로젝션
- SELECT username,age FROM Member m -> 단순 값 프로젝션

- SELECT new jpabook.jpql.UserDTO(m.username,m.age) FROM Member m -> 단순 값 DTO 바로 조회
- DISTINCT 는 중복 제거

##### 페이징 API

- JPA 는 페이징을 두 API 로 추상화
- setFirstResult(int startPosition) : 조회 시작 위치 - 0부터 시작
- setMaxResult(int maxResult) : 조회할 데이터 수

```java
String jpql = "select m from Member m order by m.name desc";
List<Member> resultList = em.createQuery(jpql,Member.class).setFirstResult(10).setMaxResult(20).getResultList();
```

```sql
SELECT
    M.ID AS ID,
    M.AGE AS AGE,
    M.TEAM_ID AS TEAM_ID,
    M.NAME AS NAME
FROM
    MEMBER M
ORDER BY
    M.NAME DESC
LIMIT ?, ?
```

DB 방언에 맞게 변경

#### 조인

- 내부 조인 : SELECT m FROM Member m `[INNER]` JOIN m.teat t
- 외부 조인 : SELECT m FROM Member m `[INNER]` JOIN m.teat t
- 세타 조인 : SELECT count(m) from Member m , Team t where m.username = t.name
	( 하이버네이트 5.1 부터 세타 조인도 외부 조인 가능 )
##### 페치 조인

- 엔티티 객체 그래프 한번에 조회
- 별칭 사용 X
- JPQL  : select m from Member m join fetch m.team
	-> SELECT M.* , T.* FROM MEMBER M INNER JOIN TEAM T ON M.TEAM_ID=T.ID

```java
String jpql = "select m from Member m join fetch m.team [where문]";
List<Member> members = em.createQuery(jpql,Member.class).getResultList();
// 페치 조인으로 , 회원 과 팀을 함께 조회해서 지연 로딩 발생 X
```

왜 사용하는가?
=> List 에 있는 모든 Entity 에서 N + 1 문제 발생!

멤버 받는 쿼리 1번 -> 10명의 멤버 에서 팀을 찾는 쿼리 10번 발생!
N + 1


### JPQL 기본 함수

- concat
- substring
- trim
- lower , upper
- length
- locate
- abs , sort , mod
- size , index
- case 문 가능
- 사용자 정의 함수 호출 가능 - select function('group_concat', i.name) from Item i


### Named Query - 정적 쿼리

- 미리 정의해서 이름 부여하고 사용하는 JPQL
- 어노테이션 이나 XML 에 정의
- 애플리케이션 로딩 시점에 초기화 후 재사용
- 애플리케이션 로딩 시점에 쿼리 검증

왜 사용해야하는가?
=> 런타임 단위 에러 방지를 하기 위해서

```java
@Entity
@NamedQuery(
	name = "Member.findByUsername",
	query = "select m from Member m where m.username = :username"
)
public class Member{}


List<Member> resultList = 
	em.createNamedQuery("member.findByUsername",Member.class).setParameter("username","회원1").getResultList();
```

애플리케이션 시작 전 자동으로 쿼리문으로 변환을 해놓는다. 
-> 배포 전 쿼리문 에러를 캐치할 수 있다.