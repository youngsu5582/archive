---
tags:
  - JPA
강의명: JPA 프로그래밍 기본기 다지기
강의_링크: https://www.youtube.com/watch?v=WfrSN9Z7MiA
---
### 서론

수많은 객체 지향적 언어 존재 ( Java , Scala ... )

DB는 아직까지 관계형 DB가 선호 ( Oracle , MySQL ... )

=> 객체지향적 언어에서 관계형 DB를 관리해야하는 상황이 발생

### JPA 가 없는 SQL 쿼리문을 작성한다면?
#### 무한 반복 , 지루한 코드 요구
CRUD 
자바 객체 -> SQL 로
SQL -> 자바 객체 로
=> 매우 불필요한 작업

#### 엔티티 신뢰 문제
```java
memeber.getTeam();
member.getOrder().getDelivery();
```

Order 가 있다고 확신하는가?
Team 이 있다고 확신하는가?
=> 계층형 아키텍처 - 진정한 의미 분할이 어렵다

#### 패러다임의 불 일치

객체 지향은 OOP 같은 추상화 & 관리를 잘 할지에 포커싱
관계형 DB는 철저히 , 데이터를 잘 정규화해서 저장할 것 인지에 포커싱

필연적으로 객체를 SQL 변환을 하여 SQL 로 DB에 저장을 하게 된다
( 개발자는 SQL 매퍼와 다를게 없어져버린다 )

![450](https://i.imgur.com/qIEQYIU.png)

같은걸 요구하지만 , 구성도는 결국 완전 다르다.
( 관계 DB에는 일반적으로 상속이 존재 X )

##### Album 저장 - In DB
1. 객체 분해
2. Insert into ITEM
3. Insert into ALBUM
##### Album 조회 - In Java
1. 각각 테이블 따른 조인 SQL 작성
2. 각각 객체 생성...
3. 매우 복잡 
=> 그렇기에 DB에 저장할 객체는 상속 관계 쓰지 않는다.
( super Item 을 다 때려받게 됨 )
##### 단순 자바 컬렉션이라면?

```java
list.add(album);
Album album = list.get(albumId);
Item item = list.get(albumId);
```

다형성 활용도 가능하고 , 매우 간편

##### 연관관계

객체는 참조를 사용 : member.getTeam()
테이블은 외래 키 사용 : JOIN ON M.TEAM_ID = T.TEAM_ID
( Java 는 Team -> Member 이동 불가능) - 방향성 존재
( DB 는 TEAM <-> MEMBER 이동 가능 ) - 방향성 존재 X
=> 이는 생각보다 매우 큼

##### 객체를 테이블에 맞춰서 모델링 해볼까?
```java
class Member{
	String id;
	Long teamId;
	String username;
}
class Team{
	Long id;
	String name;
}
```
=> INSERT INTO MEMBER(MEMBER_ID,TEAM_ID,USERNAME) VALUES ...
( 쿼리문은 매우 깔끔하게 작성 가능하네?? )

똑똑한 신입이 나타나서 , Long teamId -> Team team 으로 바꾼다면?

Insert 문에서 memberId 를 `member.getTeam().getId();` 이렇게 추출해야 함

```sql
SELECT M.* , T.*
	FROM MEMBER M
	JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID
```

이는 ANSI Query ( Oracle 과 다소 다름 - 이게 FM )

```java
public Member find(String memberId) {
	//SQL 실행
	Member member = new Member();
	Team team = new Team();

	member.setTeam(team);
	return member;
}
```
이를 위해서는 수많은 코드가 들어간다 사실 ...

그렇기에 , Member_Team 같은 슈퍼 클래스가 만들어진다.
모든 걸 다 넣어서 , 쿼리 한방에 해결되게 함

=> 이게 Java 적인 코드인가?

결국 패러다임이 다른 관계형 DB 와 마주치며 비효율적이게 되어버린다.

- 객체는 자유롭게 객체 그래프를 탐색 가능해야 한다.

```sql
SELECT M.* , T.*
	FROM MEMBER M
	JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID
```

```java
memeber.getTeam(); // OK
member.getOrder(); // null
```

이는 당연하면서도 , 어려운 맥락
( Java 에서는 당연히 getOrder 를 호출할 수도 있지만 , DB에서는 당연히 Order 가 없는걸 암 ) - 둘 사이 간극 발생
getDAO 를 확인해 값을 확인하기 전 까지는 존재 유무 확정을 못함

```java
memeber.getTeam();
member.getOrder().getDelivery();
```

NullPointerException 을 안터트린다고 확정할 수 있을까...?

```java
memberDAO.getMember();  // Member 만 조회
memberDAO.getMemberWithTeam(); // Member,Team 조회
memberDAO.getMemberWithOrderWithDelivery(); // ...
```

사용할 수록 코드가 늘어난다 . ( 노동량 및 반복 업무 증가 - 휴먼 에러 발생! )

### 비교하기

```java
String memberId = "100";
Member member1 = memberDAO.getMember(memberId);
Member member2 = memberDAO.getMember(memberId);

member1 == member2; // 다름
```
당연히 다름 ( 새로운 객체이므로 메모리 주소가 다름 )

```java
String memberId = "100";
Member member1 = list.get(memberId);
Member member2 = list.get(memberId);
member1 == member2; // 같음
```
당연히 같음 ( 같은 객체이므로 메모리 주소도 같음 )

=> 객체답게 모델링 할수록 매핑 작업이 늘어난다 ( @eqauls 오버라이딩 작업 요구 )

#### 객체를 자바 컬렉션에 저장 하듯이 DB에 저장이 불가능할까? => JPA 의 등장
### JPA

- Java Persistence API
- 자바 진영 ORM 기술 표준
##### ORM?
- Object-Relation-Mapping
- 객체는 객체대로 설계
- 관계형 DB 는 관계 DB 대로 설계
	-> ORM 프레임워크가 중간에서 매핑
- 대중적 언어에는 대부분 ORM 기술이 존재 ( TypeORM , DjangoORM , JPA ... )


![350](https://i.imgur.com/U0a9mA9.png)
JPA 는 애플리케이션 과 JDBC 사이에서 동작
패러다임 불일치가 앞서 말한 모든 것 ( 객체 프로그래밍 - 관계 프로그래밍 간 불일치 )
-> JPA 가 객체를 통해 SQL로 직접 쿼리문을 작성해준다
( MyBatis 는 쿼리는 직접 짜야 했다 )

#### JPA 소개

JPA 는 단계별 변천사를 통해 탄생했다.
1. EJB - 엔티티 빈(자바 표준) ( 매우 아마추어적 코드로 부팅 시간 오래 걸리며 , 매우 난잡함 )
2. 하이버네이트 - 오픈 소스 ( EJB 에서 답답함을 느끼고 직접 작성 )
3. JPA  - 자바 표준 (하이버네이트 개발자를 데려 와서 만듬 -> 그렇기에 Interface - 표준화의 느낌 )
( 여담으로 , EJB 가 답답해서 때려쳐서 만든게 Spring , Hybernate 들 )
#### JPA 표준 명세

- JPA 는 인터페이스의 모음
- JPA 2.1 표준 명세 구현 3가지 구현체 존재
	( 하이버네이트 , EclipseLink , DataNucleus ) - 그냥 하이버네이트를 대부분이 사용

#### JPA 를 왜 사용해야 하는가?
- SQL 중심적 개발 -> 객체 중심 개발 
- 생산성
- 유지보수
- 패러다임 불일치 해결
- 성능 최적화 , 표준
- 데이터 접근 추상화 & 밴더 접근성
##### 생산성
- 저장 : jpa.persist(member)
- 조회 : Member member = jpa.find(member)
- 수정 : member.setName("변경 이름")
- 삭제 : jpa.remove(member
##### 유지보수
필드만 추가하면 됨
-> SQL 은 JPA가 모두 처리

#### 패러다임 불일치 해결

##### JPA 상속 - 저장
```java
jpa.persist(album);
```
=> JPA 가 자체 처리
```sql
INSERT INTO ITEM ...
INSERT INTO ALBUM ...
```
##### JPA 상속 - 조회

```java
Album album = jpa.find(Album.class,albumId);
```
=> JPA 처리
```sql
SELECT M.* , T.*
	FROM MEMBER M
	JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID
```

##### JPA 연관관계 , 객체 그래프 탐색
```java
member.setTeam(team);
jpa.persist(member);
```
연관관계 단순 setter 로 작업 가능

```java
Member member = jpa.find(Member.class,memberId);
Team team = member.getTeam();
```
객체 그래프 탐색 보장 ( How? : Team 이 필요할때 즉 , getTeam 단위에서 쿼리 한번 더 보냄 - 물론 Eager Loading 도 가능 )
##### JPA 비교하기
```java
String memberId = "100";
Member member1 = jpa.find(Member.class,memberId);
Member member2 = jpa.find(Member.class,memberId);

member1 == member2; // 같음
```
동일 트랜잭션에서 , 조회한 엔티티는 같음 보장 

#### 성능 최적화 기능

##### 1차 캐싱 & 동일성 보장

같은 트랜잭션 안에서 같은 엔티티 반환 -> 약간의 조회 성능 향상
( 로직이 매우 복잡하면 , 이마저도 매우 효율적 - 배민과 같은 거대 기업은 결제 내에 , 100 번 정도의 트랜잭션을 일으킬 수도 있음 )

DB Isolation Level 이 Read Commit 이여도 애플리케이션단에서 Repeatable Read 보장

##### 트랜잭션 지원하는 쓰기 지연 - INSERT

1. 트랜잭션 커밋 때까지 INSERT SQL 모음
2. JDBC BATCH SQL 기능 사용해 한번에 SQL 전송
```java
transaction.begin();

em.persist(memberA);
em.persist(memberB);
em.persist(memberC);
//SQL 문을 모음

transaction.commit();
//커밋 순간 DB에 INSERT SQL 문을 보냄 
```
##### 트랜잭션 지원하는 쓰기 지연 -UPDATE

1. UPDATE,DELETE 인한 ROW 락 시간 최소화
2. 트랜잭션 커밋 시 UPDATE,DELETE SQL 실행하고 , 바로 커밋

```java
transaction.begin();

changeMember(memberA);
deleteMember(memberB);
비즈니스_로직_수행(); // DB 로우 락 걸지 않음

transaction.commit();
//커밋 순간 DB에 UPDATE,DELETE SQL 문을 보냄
```

#### 지연 로딩 과 즉시 로딩

- 지연 로딩 ( Eager Loading ) : 객체 실제 사용될 때 로딩
- 즉시 로딩 ( Lazy Loading ) : JOIN SQL 로 한번에 연관 객체까지 미리 조회

##### 지연 로딩
```java
Member member = memberDAO.find(memberId);  // SELECT * FROM MEMBER
Team team = member.getTeam();
String teamName = team.getName();  // SELECT * FROM TEAM
```
최대한 미룸 ( 사용을 해야할때까지 )
##### 즉시 로딩
```java
Member member = memberDAO.find(memberId);  // SELECT M.* , T.* FROM MEMBER JOIN TEAM ...
Team team = member.getTeam();
String teamName = team.getName();
```

사실 성능이 모든게 아니다!
큰 규모에서 애플리케이션 전체 숲을 봐야만 한다 ( + 통합 성능 테스트 )

