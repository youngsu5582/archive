---
tags:
  - 자바
강의명: 모던 자바 (자바8) 못다한 이야기
강의_링크: https://www.youtube.com/watch?v=KO0HWQ0vNRg
---


### Stream

Stream 이란 결국 , Collection 의 흐름

### Example

숫자 배열에서 2보다 크고 , 큰 값에서 2를 곱한 후 , 숫자 앞에 "#"을 붙힌후 Set 으로 만드는 코드를 작성해라

```java
List<Integer> numbers = List.of(1, 3, 3, 5, 5);
	numbers
		.stream  
        .filter(i -> i > 2)  
        .map(i -> i * 2)  
        .map(i -> "#" + i)
        .collect(Collectors.toSet());

```

```java
private static final Stream<String> parsingStream(final Stream<Integer> stream){  
    return stream  
            .filter(i -> i > 2)  
            .map(i -> i * 2)  
            .map(i -> "#" + i);  
}

Stream<String> stream = parsingStream(numbers.stream());  
System.out.println(stream.collect(Collectors.toSet()));
```

해당 코드와 같이 , stream 을 받아서 중간 연산 까지만도 가능하다!

```java
stream = parsingStream(numbers.stream());  
System.out.println(stream.collect(Collectors.joining(", ","[", "]")));
```
작성한 Stream 은 여러곳에서도 사용 가능하나 , 
주의해야할 점은 사용한 stream 은 GC 대상이므로 , 다시 새로 할당을 해서 사용해야 한다!


### 여담

#### Identity VS Equality

##### Identity?

- == 을 사용해서 메모리 참조를 비교
=> 결국 , 같은 메모리르 사용하는지 확인
=> 같은 값이더라도 , 참조하는 메모리가 다르면 False 로 나온다

#### Equality

- equals() 를 통해 확인
=> 같은 값인지를 비교

#### Example

```java
Intenger number1 = 100;
Integer number2 = 100;
number1 == number2 
// => True!

Integer number1 = 1004;
Integer number2 = 1004;
number1 == number2;
// => False!
```

Integer.valueOf 는 127 ( Default ) 까지의 값을 Caching 하고 있다
