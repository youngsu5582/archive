---
tags:
  - 자바
강의명: 모던 자바 (자바8) 못다한 이야기
강의_링크: https://www.youtube.com/watch?v=PZVBTQph-5I
---

### Identity Function

```java
static <T> Function<T, T> identity() {  
    return t -> t;  
}
```

앞에서 말했듯 , 정말 자신의 타입을 그대로 반환해주는 함수

왜 사용해야 할까?

### Example

배열을 받아 , 배열 의 값들을 특정 행위를 한 후 새로운 배열을 만들자!

```java
private static final <T, R> List<R> map(List<T> list, Function<T, R> mapper) {  
    final List<R> result = new ArrayList<>();  
    for (final T t : list) {  
        result.add(mapper.apply(t));  
    }  
    return result;  
}
```

배열을 선언한 후 , mapper 를 활용해 단순 타입 변환 가능!

```java
final List<Integer> numbers = List.of(1, 2, 3, 4, 5);  
System.out.println(  
        mapOld(numbers, i -> i * 2)  
);
```
간단하게 수행이 가능하다!

##### IF?
값을 변환하지 않고 , 단순 사용해야 하는경우가 생긴다면??

```java
private static final <T, R> List<R> map(List<T> list, Function<T, R> mapper) {  
    if (mapper == null) {  
        return new ArrayList<>((List<R>) list);  
    }  
    final List<R> result = new ArrayList<>();  
  
    for (final T t : list) {  
        result.add(mapper.apply(t));  
    }  
    return result;  
}

System.out.println(  
        map(numbers, null)  
);
```

mapper 가 null 인지 확인 후 , 해당 배열을 변환후 반환해야 한다.
=> null 을 사용하며 , 매우 비효율적인 연산!


```java
private static final <T, R> List<R> map(List<T> list, Function<T, R> mapper) {  
    final List<R> result = new ArrayList<>();  
    for (final T t : list) {  
        result.add(mapper.apply(t));  
    }  
    return result;  
}
```

기존 코드에서 , 

```java
System.out.println(  
        map(numbers,i -> i)  
);  
```

이렇게 , 자기 스스로를 반환하는 코드를 사용하자!

이마저도 생략 하기 위해
```java
System.out.println(  
        map(numbers, Function.identity())  
);
```
identity Function 이 나왔다.

### 결론

특히  , `return new ArrayList<>((List<R>) list);` 이런 강제 형변환을 하지 않아도 된다는 점이 좋은거 같다.

