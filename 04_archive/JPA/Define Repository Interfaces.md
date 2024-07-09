---
tags:
  - JPA
참고_링크: https://docs.spring.io/spring-data/jpa/reference/repositories/definition.html
추가_정보: Spring JPA 공식 문서
---
Domain Repository 를 선언하려면?
-> Domain Class / Id Class 지정만 하면 끝!

자기가 원하는 Repository 를 extends 하기만 하면 OK!

리액티브 저장소일시? -> `ReactiveCrudRepository`
코루틴 저장소일시? -> `CorutineCrudRepository`

```java
@RepositoryDefinition(domainClass = ReservationTime.class, idClass = Long.class)  
public interface ReservationTimeRepository{  
    boolean existsByStartAt(LocalTime startAt);  
}
```

와 같이 직접 지정도 가능!
```java
@NoRepositoryBean
public interface CommonRepository<T, ID> extends CrudRepository<T, ID> {
    List<T> findBySomeCriteria(String criteria);
}

```

직접적인 Repository 로 동작하는게 아닌 온전한 interface 처럼 명세 가능

---

### 금지 케이스

```java
interface PersonRepository extends Repository<Person, Long> { … }
@Entity
class Person { … }

interface UserRepository extends Repository<User, Long> { … }
@Document
class User { … }
```
NoSQL - SQL 혼합 금지!

```java
interface JpaPersonRepository extends Repository<Person, Long> { … }
interface MongoDBPersonRepository extends Repository<Person, Long> { … }

@Entity
@Document
class Person { … }
```
같이 사용 역시 금지!

```java
interface AmbiguousRepository extends Repository<User, Long> { … }

@NoRepositoryBean
interface MyBaseRepository<T, ID> extends CrudRepository<T, ID> { … }

interface AmbiguousUserRepository extends MyBaseRepository<User, Long> { … }
```
모호하게 지정 금지!

=>

```java
@EnableJpaRepositories(basePackages = "com.acme.repositories.jpa") @EnableMongoRepositories(basePackages = "com.acme.repositories.mongo") 
class Configuration { … }
```
이렇게 구별은 가능!