---
tags:
  - 자바
강의명: 모던 자바 (자바8) 못다한 이야기
강의_링크: https://www.youtube.com/watch?v=-TMcXGdhP4Q
---


```java
List.of(1,2,3,4,5)
	.forEach(System.out::println);  
	
	.forEach(i-> System.out.println());
```

두 개다 사용 가능

#### Why?

```java
default void forEach(Consumer<? super T> action) {  
    Objects.requireNonNull(action);  
    for (T t : this) {  
        action.accept(t);  
    }  
}
```

forEach 의 원형 코드
Consumer 를 받아서 사용

```java
public void println(Object x) {  
    String s = String.valueOf(x);  
    if (getClass() == PrintStream.class) {  
        // need to apply String.valueOf again since first invocation  
        // might return null        writeln(String.valueOf(s));  
    } else {  
        synchronized (this) {  
            print(s);  
            newLine();  
        }  
    }  
}
```

println 의 원형 코드
Object 를 받아서 출력

println 의 void (x) 와 Consumer accept 의 원형이 void(x) 로 같다.
### Method Reference

- Java 8 도입
- 람다 표현식을 더욱 간결하게 해준다 ( 특정 메소드를 호출하는 경우에만 )

##### 정적 메소드 참조 : String::valueOf 는 (obj) => String.valueOf(obj)

##### 특정 객체 메소드 참조 : instance::instanceMethod 는 instance.instanceMethod(args)

##### 특정 유형 인스턴스 메소드 참조 : String::toLowerCase 는 (str) => str.toLowerCase()

##### 클래스 생성자 참조 : ArrayList::new 는 new ArrayList()

=> 코드를 더욱 간결하고 , 다양한 Context 에서 유용하게 사용 가능



```java
List<BigDecimal> list = List.of(new BigDecimal("10.0"), new BigDecimal("23"), new BigDecimal("5"))  
        .stream()  
        .sorted()  
        .toList();
```

배열의 값이 자동으로 정렬 ( sorted 에 값을 안넣어도 , default 로 정렬 )

```java
class BigDecimalUtil{  
    public static final int compare(BigDecimal bd1, BigDecimal bd2){  
        return bd1.compareTo(bd2);  
    }  
}
```

단순 비교해주는 함수

```java
List<BigDecimal> list = List.of(new BigDecimal("10.0"), new BigDecimal("23"), new BigDecimal("5"))  
        .stream()  
        .sorted(BigDecimalUtil::compare)  
        .toList();
```

sorted 에 추가 가능! 

```java
final String targetString = "c";  

System.out.println(  
        List.of("a","b","c","d")  
                .stream()  
                .anyMatch(targetString::equals)  
);
```
targetString 에는 자동으로 문자열이 들어가므로 , 추론 가능