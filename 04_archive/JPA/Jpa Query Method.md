---
tags:
  - JPA
참고_링크: https://docs.spring.io/spring-data/jpa/reference/jpa/query-methods.html
추가_정보: Spring JPA 공식 문서
---

| Keyword        | Sample                               | JPQL snippet                                                       |
| -------------- | ------------------------------------ | ------------------------------------------------------------------ |
| `Distinct`     | `findDistinctByLastnameAndFirstname` |                                                                    |
| `StartingWith` | `findByFirstnameStartingWith`        | `… where x.firstname like ?1` (parameter bound with appended `%`)  |
| `EndingWith`   | `findByFirstnameEndingWith`          | `… where x.firstname like ?1` (parameter bound with prepended `%`) |
| `Containing`   | `findByFirstnameContaining`          |                                                                    |

> distinct 는 주의해서 사용하자
> -> select distinct u from User u
> -> select distinct u.lastname from User u
> 
> 이 2개는 엄연히 다르다!
> 
> countDistinctByLastname(String lastname)
> countByLastname(String lastname)
> 
>이 2개는 같은 결과
>-> select count(distinct u.id) from User u where u.lastname = ?1
>-( u.id 는 어차피 primary 이므로 )

=> 그렇기에, 가끔은 Query 로 명시해줘야 할 때가 있다
### Query

```java
@Entity
@NamedQuery(name = "User.findByEmailAddress",
  query = "select u from User u where u.emailAddress = ?1")
public class User {
}
```

```java
<named-query name="User.findByLastname">
  <query>select u from User u where u.lastname = ?1</query>
</named-query>
```

```java
public interface UserRepository extends JpaRepository<User, Long> {

  List<User> findByLastname(String lastname);

  User findByEmailAddress(String emailAddress);
}
```

쿼리를 생성하는 대신, 앞서 정의한 Named Query 로 사용

```java
public interface UserRepository extends JpaRepository<User, Long> {

  @Query("select u from User u where u.emailAddress = ?1")
  User findByEmailAddress(String emailAddress);
}
```
와 같이 직접 명시도 가능

#### QueryRewriter
```java
public interface MyRepository extends JpaRepository<User, Long> {

		@Query(value = "select original_user_alias.* from SD_USER original_user_alias",
                nativeQuery = true,
				queryRewriter = MyQueryRewriter.class)
		List<User> findByNativeQuery(String param);
}
```

```java
@Component
public class MyQueryRewriter implements QueryRewriter {
     @Override
     public String rewrite(String query, Sort sort) {
         return query.replaceAll("original_user_alias", "rewritten_user_alias");
     }
}
```

Query 를 EntityManager 에 보내기 전 접근 가능!
#### NativeQuery

실제 query 를 지정
```java
public interface UserRepository extends JpaRepository<User, Long> {

  @Query(value = "SELECT * FROM USERS WHERE EMAIL_ADDRESS = ?1", nativeQuery = true)
  User findByEmailAddress(String emailAddress);
}
```

Query flag 를 true로 변경해서 사용

> Spring Data JPA는 nativeQuery 에 대해서는 dynamic sorting 을 지원하지 않는다

```java
public interface UserRepository extends JpaRepository<User, Long> {

  @Query(value = "SELECT * FROM USERS WHERE LASTNAME = ?1",
    countQuery = "SELECT count(*) FROM USERS WHERE LASTNAME = ?1",
    nativeQuery = true)
  Page<User> findByLastname(String lastname, Pageable pageable);
}
```
자기가 직접 count 문을 구현해야함

#### Named Parameter
```java
public interface UserRepository extends JpaRepository<User, Long> {
  @Query("select u from User u where u.firstname = :firstname or u.lastname = :lastname")
  User findByLastnameOrFirstname(@Param("lastname") String lastname,
                                 @Param("firstname") String firstname);
}
```

param 명을 직접 명시해서 사용 가능

### Other Methods

우선, builder query 를 사용하자
-> 그래도 안된다면? `@Query` 를 사용해 쿼리문을 직접 작성하자

- EntityManager 에 Pure HQL/JPQL/EQL/native SQL QUERY 를 사용
- Jdbc Template 에 native QUERY 사용

또는
`@StoredProcedure` 을 통해 DB에 저장후, `@Query` 에서 CALL 하는 것 역시 방법

#### Modifying


```java
@Modifying  
@Query("delete from ReservationTime rt where rt.id = ?1")  
int deleteById(long timeId);
```
WRITE 문을 날리는 커스텀 쿼리에는 어노테이션을 붙혀줘야 한다
( "message": "Expecting a SELECT query : `delete from ReservationTime rt where rt.id = ?1`" )
해당 에러 발생!

그러면 기존 코드와의 차이점은?
-> 엔티티의 생명주기를 거치지 않는다!
( List의 여러개를 한번에 지울때 유용 - 생명주기 거칠 필요 없을때 )

---

근데 여기서 재밌는 부분이
```java
@Modifying  
@Query("delete from ReservationTime rt where rt.id = ?1")  
int deleteById(long timeId);
```

해당 코드와 같이, 기본 제공 메소드이나 return 타입이 다르면
삭제된 개수도 반환하며, 생명주기를 거치지 않는다

```java
public void deleteReservationTime(final long id) {  
    final ReservationTime reservationTime = reservationTimeRepository.findById(id)  
            .orElseThrow(() -> new NotExistException(RESERVATION_TIME, id));  
  
    if (reservationRepository.existsByTimeId(id)) {  
        throw new ExistReservationException(RESERVATION_TIME, id);  
    }  
    reservationTimeRepository.deleteById(id);  
}
```
해당 코드에서도 `@Transactional` 을 붙히지 않아도 동작한다

```java
public void deleteReservationTime(final long id) {  
    final ReservationTime reservationTime = reservationTimeRepository.findById(id)  
            .orElseThrow(() -> new NotExistException(RESERVATION_TIME, id));  
  
    if (reservationRepository.existsByTimeId(id)) {  
        throw new ExistReservationException(RESERVATION_TIME, id);  
    }  
    reservationTimeRepository.deleteBulkById(id);  
}
```

하지만
```java
@Modifying  
@Query("delete from ReservationTime rt where rt.id = ?1")  
int deleteBulkById(long timeId);
```
이렇게 직접 정의시?
-> `Executing an update/delete query` 가 발생한다. (삭제/갱신 쿼리에는 Transactional 정의 필요)

```java
void deleteByStartAt(LocalTime startAt);
```
메소드명 기반 자동 쿼리 생성은?
->  `No EntityManager with actual transaction available for current thread - cannot reliably process 'remove' call` 가 발생한다.

=> 둘다 `@Transactional` 서비스단에서 필요! ( 레포단에서 `@Transactional` 매우 별로 )

#### Hint
```java
public interface UserRepository extends Repository<User, Long> {

  @QueryHints(value = { @QueryHint(name = "name", value = "value")},
              forCounting = false)
  Page<User> findByLastname(String lastname, Pageable pageable);
}
```

```java
@QueryHints(value = { @QueryHint(name = "jakarta.persistence.cache.retrieveMode", value = "USE"), @QueryHint(name = "jakarta.persistence.cache.storeMode", value = "USE") }) List<Employee> findEmployeesByName(String name);
```
와 같이 비교적 다양하게 힌트 접목 가능
