---
tags:
  - 자바
추가_정보: 백기선 라이브 스터디
주차: "2"
---
### Primitive VS Reference

Reference Type 은 결국 Primitive 값에서 데이터를 가져온다

- 8개의 원시 타입은 자바 언어로 정의
	-> 개발자는 절대 새로운 원시타입 정의 불가능
=> 개발자는 레퍼런스 타입을 정의 가능 ( 수없이 많은 래퍼런스 타입 가능 )

#### Example

```java
public static void main(String[] args) {  
    int i = 2000;  
    int j = 1004;  
  
    Integer var1 = 2000;  
    Integer var2 = 1004;
}
```

해당 값이 있다면?
##### Stack
- int i 는 실제 값 ( 2000 ) 저장
- int j 도 실제 값 ( 1004 ) 저장
- var1 은 주소 값 ( 1 - 주소 값 ) 저장
- var2 은 주소 값 ( 2 - 주소 값 ) 저장

=> 힙에 저장된 1 과 2를 사용

### Literal

- 소스 코드에 사용되는 고정된 값
	-> 데이터의 고유한 값 자체
- -123 , 42 , 'A' , "Hello" 등등

리터럴은 값을 표현하는 방식 , 상수는 프로그램에 사용되는 고정된 값 ( 변수의 불변된 값 )
