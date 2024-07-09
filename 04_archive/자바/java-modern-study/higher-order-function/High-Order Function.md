---
tags:
  - 자바
강의명: 모던 자바 (자바8) 못다한 이야기
강의_링크: https://www.youtube.com/watch?v=7jvhfVGzKG4
---

### High Order Function

- 고차 함수
- 두 가지  중 하나 이상 조건을 만족하는 함수

	- 함수를 인자로 받는다
	- 함수를 결과로 반한한다

=> 유연하게 코드 작성이 가능해지며 , 추상화 및 코드 재사용이 가능하다.

```java
Function<Function<Integer, String>, String> applyFunction = func -> func.apply(10);
```

이렇게 , Function 내 Function을 받아서 사용 가능

```java
Function<Integer, String> intToString = n -> "Converted number: " + n;  
String result = applyFunction.apply(intToString);
```

함수를 적용해 , 사용 가능
```cmd
Converted number: 10
```

```java
Function<Integer,Function<Integer,Function<Integer,Integer>>> f3=  
        i1 -> i2 -> i3 -> i1 + i2 + i3;  

System.out.println(f3.apply(1).apply(2).apply(3));
```

더욱 복잡한 구조도 가능

Stream 의 메소드도 역시 고차 함수로도 가능

- map : ( Function<T,R> )
- filter : ( Predicate<`T`> )
- forEach : (Consumer`<T>`)
- reduce : (BinaryOperator`<T>`)
