---
tags:
  - 자바
강의명: 모던 자바 (자바8) 못다한 이야기
강의_링크: https://www.youtube.com/watch?v=Duvm8pgjYGo
---

### Java 8 이전

Interface 가 가질 수 있는 Method 는 Abstract Method 만 가질 수 있었다

```java
public interface ICustom{
	void implMethod();
}
```
### Java 8 이후

```java
public interface ICustom{
	default void defaultMethod(){
		System.out.println("Default Method");
	}
	static void staticMethod(){
		System.oout.println("Static Method");
	}
}
```


```java
public interface ICustom {  
    void doSomething(Function<Integer,String > f);  
}
```

이렇게 doSomething 을 구현해야 할 경우?
#### Non - Lambda
```java
default void doSomethingDefault(){  
    doSomething(new Function<Integer, String>() {  
        @Override  
        public String apply(Integer integer) {  
            return String.valueOf(integer);
        }  
    });  
}
```

매우 비효율적
#### Lambda
```java
default void doSomethingDefaultWithLambda(){  
    doSomething(i -> String.valueOf(i));  
}
```

람다로 간단하게 구현 가능
### Function Interface
```java
interface Function <T,R> {
	R apply(T t);
}
```

이렇게 , abstrat Method 가 하나인 경우에는 Functional Interface 라고 한다!

```java
@FunctionalInterface
```

해당 Annotation 이 있을 시 아래 Interface는 함수형 Interface 이다.

```java
public interface Function<T, R> {  
  
	R apply(T t);
}
```

이렇게 , 함수 하나만 있을시 `FunctionalInterface` 임을 유추한다!

```java
@FunctionalInterface
public interface Function<T, R> {  
  
	R apply(T t);
	void printFunction();
}
```

이렇게 , 함수가 하나 더 있을 시 FunctionalInterface Annotation 이 Error 를 유발!
Lambda 가 둘 중에 무슨 함수인지 , 유추를 할 수 없으므로! ( SAM 원칙에 위배 )
##### SAM
- Single Abstrat Method
- 무조건 하나의 추상 메소드만 가져야 함
=> 람다 언어 자체 규칙

### Example

매개변수 세개를 받아 , Logic 을 수행하는 Function 을 구현하라

```java
@FunctionalInterface  
public interface TriFunction<T1, T2, T3, R> {  
    R apply(T1 t1, T2 t2, T3 t3);  
}
```

매우 간단하게 구현 가능

사용하려면 ?

```java
private static <T1, T2, T3> void println(T1 t1, T2 t2, TriFunction<T1, T2, String, String> f) {  
    System.out.println(f.apply(t1, t2, " !!"));  
}  
  
private static <T1, T2, T3> void println(T1 t1, T2 t2, T3 t3, TriFunction<T1, T2, T3, String> f) {  
    System.out.println(f.apply(t1, t2, t3));  
}
```

이렇게 Overloading 을 통해서도 사용 가능!

```java
TriFunction<String, String, String, String> f1 = (s, s2, s3) -> s + " and " + s2 + s3;  
println("Youngsu", "Yejin", f1);
```

```java
TriFunction<Long, String, String, String> f2 =  
        (id, name, email) -> "User info: ID=" + id + ", name=" + name + ", email=" + email;  
println(1L, "Youngsu", "test@email.com",f2);
```

물론 , println 에 바로 넣어서도 사용 가능!
( 가독성 , 및 타입 Human Error 방지 위해서 이렇게 분리하고 먼저 지정하는게 더 괜찮은 거 같다 )

