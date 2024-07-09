---
tags:
  - 자바
추가_정보: 백기선 라이브 스터디
주차: "2"
---
Java 10 부터 타입추론 ( type - inference ) 가능

### 타입 추론

개발자가 변수의 타입을 명시적으로 적지 않고
컴파일러가 알아서 대입된 리터럴로 추론

- 대표적 타입 추론 언어는 자바스크립트 , 코틀린 , 스위프트 등이 있다

```java
var str2 = "Hello Type Inference";
```

- var 은 초기화 값이 있는 지역 변수 로만 선언 가능
- 멤버변수 , Method Parameter , 리턴 타입으로 사용 불가능

```java
var map2 = new HashMap<String,Integer>();
```

var 를 사용해 , 복잡한 구조를 컴파일러가 추론하게 두고
변수명에 신경을 써서 이름에 집중 할 수 있다 ( ? - 해당 부분은 잘 모르겠음 )

```java
int var = 3;
var i = 3;
```
#### var 은 키워드가 아니다.
어떤 타입도 아니고 , 클래스에 사용하는 예약어도 아니다. ( 변수명으로 사용 가능 )
##### How?
컴파일러가 바이트 코드로 변경할 때 , var 에는 타입이 명시
( 바이트코드에는 int i  = 3 으로 변환된다 )

```java
var i = 3;
i = "gell";  // ❌
```

#### var 은 런타임 오버헤드가 없다

컴파일 시점에 , 초기화 된 값을 보고 타입을 추론한다.
-> 변수를 읽을 때 마다 타입을 알아내기 위한 연산을 하지 않는다는 뜻

JavaScript 처럼 중간에 다른값으로 변경 불가능! ( TypeScript 는 Java 와 원리 동일 )

### Var 사용법

#### False Case
```java
var i;  // var 타입은 선언함과 동시에 초기화를 해줘야 한다.
var i =null // var 타입 초기화 값은 null 이 될 수 없다.
private var i = "Private Variable"; // 멤버 변수에 사용 불가능
var lambda = (String s) -> System.out.println("s = " + s); // Lambda 표현식에는 명시적 타입 지정 가능
var arr = {1,2,3}; // 배열 선언할 때는 타입 명시해야 한다. -> 
```
var 는 초기화 없이 사용할 수 없다.

#### Usually Case

```java
Consumer<String> conFoo = s -> System.out.println("s = " + s);
=>
Consumer<String> conFoo = (var s) -> System.out.println("s = " + s);
=>
Consumer<String> conFoo = (@Nonnull var s) -> System.out.println("s = " + s);
```

그냥 s 에는 Annotation 을 붙히지 못하나 ,
var 을 추가함으로써 Annotation 추가 가능

### Reference

https://www.youtube.com/watch?v=tjj-XLk4CSA
https://catch-me-java.tistory.com/19
