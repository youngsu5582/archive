---
tags:
  - JPA
참고_링크: https://docs.spring.io/spring-data/jpa/reference/repositories/core-concepts.html
추가_정보: Spring JPA 공식 문서
---
결국 다른 Spring 기술들 처럼 추상화를 위한 것!

추상화의 핵심인 interface -> Repository!
```java
@Indexed  
public interface Repository<T, ID> {  
}
```
여기서 부터 시작
-> 마커 인터페이스 ( 이를 확장한 레포지토리를 찾게 해줌 )

해당 Repository 를 CrudRepository 가 상속 받음
( fidnAll 시 Iterable 로 반환 )
-> CrudRepository 를 ListRepository 가 상속 받음 - Iterable 대신 List

=> JPARepository 는 결국 여러 interface 들을 extends 한 클래스!
```
<S extends T> S save(S entity);
Optional<T> findById(ID primaryKey);
Iterable<T> findAll();
long count();
void delete(T entity);
boolean existsById(ID primaryKey);
```
와 같은 로직들 기본 제공




