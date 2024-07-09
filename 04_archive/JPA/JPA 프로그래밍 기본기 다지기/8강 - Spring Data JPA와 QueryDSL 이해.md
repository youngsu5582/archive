---
tags:
  - JPA
강의명: JPA 프로그래밍 기본기 다지기
강의_링크: https://www.youtube.com/watch?v=gRqyzi9VGYc
---
### 반복되는 CRUD

```java
public class MemberRepository{
	public void save(Member member){...}
	public Member findOne(Long id){...}
	public List<Member> findAll(){...};
}

public class ItemRepository{
	public void save(Item item){...}
	public Item findOne(Long id){...}
	public List<Item> findAll(){...}
}
```

이들은 결국 중복되는 코드!
-> 반복해서 짜는건 매우 비효율적이다
=> 스프링 데이터 JPA 의 등장

### 스프링 데이터 JPA

- 지루하게 반복되는 CRUD 문제 세련된 방법으로 해결
- 개발자는 인터페이스만 작성
- 스프링 데이터 JPA 가 구현 객체를 동적으로 생성해서 주입

```java
public class MemberRepository extends JpaRepository<Member, Long>{
	Member findByUsername(String username);
} 
```

일반적인 경우는 전부 구현을 미리 해놓는다.

![350](https://i.imgur.com/gSoG3yY.png)

클래스 다이어그램

Spring Data JPA 에서 생성 -> ItemRepository 구현 클래스 -> `<<Interface>>` ItemRepository

- 공통 CRUD 제공
- 제네릭은 `<엔티티,식별자>` 설정
#### 메소드 

```java
List<Member> findByName(String username);
```
메소드 이름만으로 , JPQL 쿼리 생성 가능

추론이 불가능한 메소드명이면?
-> `ReferenceException : No property 이름 found for type Member` 와 같은 에러가 뜬다

##### 검색 + 정렬

```java
List<Member> findByName(String username,Sort sort);
```

##### 검색 + 정렬 + 페이징
```java
List<Member> findByName(String username,Pageable pageable);

PageRequest pageRequest = new PageRequest(1,10);
findByName("meber",pageRequest);
```

#### 쿼리

@Query 사용해서 직접 JPQL 지정도 가능
```java
@Query("select m from Member m where m.username = ?1")
Member findByUsername(String username,Pageable pageable);
```

#### 도메인 클래스 컨버터도 가능

식별자로 , 도메인 클래스 찾음
```java
@RequestMapping("/members/{memberId}")
Member member(@PathVariable("memberId")Member member){
	return member;
}

```


### QueryDSL

- SQL , JPQL 코드로 작성할 수 있도록 도와주는 빌더 API
- JPA Criteria 에 비해 , 편리하고 실용적
- 오픈소스

#### SQL , JPQL 문제점

- 문자 , Type-Check 불가능
- 해당 로직 실행 전까지 작동여부 확인 불가

`SELECT * FROM MEMBERR WHERE MEMBER_ID = 100`
같은 쿼리문은 실행 시점에 오류가 발견된다

#### 장점

- 문자가 아닌 코드로 작성
- 컴파일 시점에 문법 오류 발견
- 코드 자동완성 (IDE단)
- 단순하고 쉬움 : 코드 모양 역시 JPQL 과 거의 비슷 
- 동적 쿼리

#### 동작 원리

Member.java => APT => QMember.java

Member 에 맞게 생성

#### 사용
```java
JPAFactory query = new JPAQueryFactory(em);
QMember m  = QMember.member;

List<Member> list =
	query.selectFrom(m).where(m.age.gt(18)).orderBy(m.name.desc()).fetch();
```

컴파일 단에서 , 실수를 캐치할 수 있다

##### 쿼리 분리 가능 - 이것이 자바다
```java
private BooleanExpression isServiceable(){
	return coupon.status.eq("LIVE").and(marketing.viewCount.lt(markting.maxCount));
}
```


### 실무 경험 공유

- 테이블 중심에서 객체 중심으로 개발 패러다임 변화
- 유연한 DB 변경 & 테스트 
	- JUnit 통합 테스트시 H2 DB 메모리 모드
	- 로컬 PC 에선 H2 DB 서버 모드
	- 개발 운영은 MySQL , Oracle
-> 이렇게 , DB 변화에 신경을 쓸 필요가 없다
- 테스트 , 통합 테스트 시 CRUD 는 생략 가능

- 빠른 오류 발견 가능 ( 컴파일 시점 , 애플리케이션 로딩 시점 )
- 최소한 쿼리 문법 실수 - 오류 발생하진 않는다 ( 대부분 비즈니스 로직 오류 )

#### 실무 경험 - 성능

- JPA 자체 인한 성능 이슈는 거의 없음
- 성능 이슈 대부분은 JPA 잘 이해하지 못해 발생
	- 즉시 로딩 : 쿼리가 튐 -> 지연 로딩으로 변경
	- N+1 문제 : 대부분 페치 조인으로 해결 가능
- 내부 파서 문제 : 2000줄 짜리 같은 동적 쿼리 생성하는데 걸리는 시간
	-> 정적 쿼리로 변경 ( 하이버네이트가 파싱된 결과 재사용 )

#### 실무 경험 - 생산성

- 단순 코딩 시간 줄어듬 -> 개발 생산성 향상 -> 잉여 시간 발생
- 비즈니스 로직 작성시 흐름 끊어지지 않음
- 남는 시간에 테스트 작성 , 기술 공부 , 코드에 금칠 등등 모든게 가능
