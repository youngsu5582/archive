---
tags:
  - 자바
추가_정보: 백기선 라이브 스터디
주차: "12"
---
### Meta Annotation

애노테이션을 위한 애노테이션

애노테이션의 적용대상 ( Target ) , 유지기간 ( Retention ) 등 지정하는데 사용

#### @Target

해당 에노테이션이 어디 적용될 수 있는지 지정

- ANNOTATION_TYPE : 애노테이션
- CONSTRUCTOR : 생성자
- FIELD : 필드 ( 멤버 변수 , enum 상수 )
- LOCAL_VARIABLE : 지역 변수
- METHOD : 메소드
- PACKAGE : 패키지
- PARAMETER : 매개변수
- TYPE : 타입 ( 클래스 , 인터페이스 , enum )

- TYPE_PARAMETER : 타입 매개변수 - JDK 1.8
-> 제네릭 타입 매개변수에 적용
```java
public class MyClass<@MyAnnotation T>{}
```
- TYPE_USE : 타입 사용되는 모든 곳 - JDK 1.8
implements , extends 에서도 사용 가능
```java
class MyClass implements @NonNull MyInterface{

}
```


여러 개 값 지정할 때 중괄호 안에 콤마 구분 지정
```java
@Target({TYPE,FIELD,TYPE_USE})
```

### Retention

해당 애노테이션의 유효기간

어느 시점까지 애노테이션 정보를 메모리에 저장할 것인지 설정

enum 타입 RetentionPolicy 를 값으로 넘겨줌 - default 는 CLASS

- SOURCE : 컴파일 전까지 , 컴파일 시 해당 애노테이션 제거
- CLASS : 바이너리 파일에 정보가 저장되나 , Runtime에서 사라짐
- RUNTIME : 런타임에 JVM 이 해당 에너토에신 정보 사용


### Documented

역시 메타 어노테이션

Javadoc 이 문서화 할 때 , 해당 애노테이션 정보도 문서에 포함시키도록 지정
