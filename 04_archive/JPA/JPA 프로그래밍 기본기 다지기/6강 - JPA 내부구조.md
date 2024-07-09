---
tags:
  - JPA
강의명: JPA 프로그래밍 기본기 다지기
강의_링크: https://www.youtube.com/watch?v=PMNSeD25Qko
---
### 영속성 컨텍스트

JPA 이해하는데 가장 중요한 용어

- 엔티티 영구 저장하는 환경
- 논리적인 개념
- 엔티티 매니저를 통해 접근

EntityManager.persist(entity);

일반적인 Java 환경에서는 EntityManager -> PersistenceContext ( 1:1 매핑 )
스프링 프레임워크 같은 컨테이너 환경에서는 EntityManager... -> PersistenceContext ( N : 1 매핑 )
-> 같은 트랜잭션이면 같은 영속성 컨텍스트에 접근
( 서비스 계층이 같은 트랜잭션에 묶여 있으면 같은 영속성 컨텍스트 )

### 엔티티 생명주기

![400](https://i.imgur.com/eHn5gjs.png)

#### 비영속 - new / transient : 영속성 컨텍스트와 전혀 관계 없는 상태
```java
Member member = new Member();
member.setId("member1");
member.setUsername("회원");
```
단순 객체만 생성

#### 영속 - managaed : 영속성 컨텍스트에 저장된 상태
```java
em.persist(memeber);
```
객체를 영속 컨텍스트에 저장
#### 준영속 - detached : 컨텍스트에 저장되었다가 분리된 상태
```java
em.detach(member);
```

-> Lazy Loading 때문에 필요
#### 삭제 - removed : 삭제된 상태
```java
em.remove(member);
```
DB에서 날림

### 영속성 컨텍스트의 이점

#### 1차 캐싱
@Id 가 Key - member1
Entity 가 Value - member
Map 개념으로 값이 저장된다

값이 있으면?
1. 1차 캐시에 값이 있는지 확인
2. 반환

값이 없으면?
1. 1차 캐시에 값이 있는지 확인
2. DB 조회
3. 1차 캐시에 저장
4. 반환

#### 동일성 보장

```java
Member a = em.find(Member.class, "member1");
Member b = em.find(Member.class, "member1");

a == b // true
```
내부 1차 캐싱 때문에 같은 Referecne 나옴

#### 트랜잭션 지원 쓰기 지연

```java
em.persist(memberA);
em.persist(memberB);

transaction.commit();
// 커밋 순간 DB에 INSERT SQL 문을 보낸다
```
![400](https://i.imgur.com/8lfN3nR.png)

쓰기 지연 SQL 저장소에 쿼리를 저장하고 있는다. ( 실제 구현은 X - 논리적 )

commit() 을 할 때 , 다 같이 진행
flush -> commit
#### 변경 감지 ( Dirty Check )
```java
Member memberA = em.find(Member.class, "member1");

memberA.setAge(10);

transaction.commit();
```

커밋 시 자동으로 , UPDATE 쿼리 수행
![400](https://i.imgur.com/dzCNRuY.png)

`em.update(member);` 와 같은 값 불필요
스냅샷을 통해 , 기존 값과 비교하여 변경을 감지 ( 물론 메모리 두배 차지의 우려도 존재 )
=> 자바의 컬렉션과 동일하게 구현하기 위해 이런 방식으로 작동
##### Flush

- 변경 감지
- 수정된 엔티티 쓰기 지연 SQL 저장소 등록
- 쓰기 지연 SQL 저장소 쿼리 ( 등록 , 수정 , 삭제 쿼리 ) DB에 전송

em.flush 를 통해 직접 호출 가능
트랜잭션 커밋 - flush 자동 호출
JPQL 쿼리 실행 - flush 자동 호출


```java
em.persist(memberA);
em.persist(memberB);
em.persist(memberC);

query = em.createQuery("select m from Member m",Member.class);
List<Member> members = query.getResultList();
```

getResultList 실행할 때 , 우선 flush 메소드를 호출한 후 query문 실행
( 해야만 , A B C 의 값들이 나오므로 )

#### 지연 로딩

지연 로딩 LAZY 를 사용해 프록시로 조회
![400](https://i.imgur.com/fZ3NSEI.png)

프록시를 Team 을 만든후
필요할 때 , 바로 호출

- 가급적이면 지연 로딩을 사용하자
	-> 즉시 로딩이면 예상치 못한 SQL 발생 ( N+1 문제 유발 가능 )
- ManyToOne , OneToOne 은 기본이 즉시 로딩 -> LAZY로 설정하자
- OneToMany , ManyToMany 는 기본이 지연 로딩 

em.close() 후 지연 로딩을 주의하자
```java
Member findMember = em.find(Member.class,member.getId());

em.close();

Team findTeam = findMember.getTeam();
```

`LazyInitializationException` 발생!
( 스프링 단위에선 , 트랜잭션 끝난 시점에 , 컨트롤러에서 호출하려 할 때 에러 발생한다 )