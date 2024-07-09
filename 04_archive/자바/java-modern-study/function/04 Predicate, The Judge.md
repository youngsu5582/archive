---
tags:
  - 자바
강의명: 모던 자바 (자바8) 못다한 이야기
강의_링크: https://www.youtube.com/watch?v=qzUwveRjiEg
---

### Predicate

```java
Predicate<T>
```

```java
boolean test(T t);
```

- T Type 을 받은 후 , boolean 을 반환

### Example

받은 정수가 , 짝수인지 판별

```java
Predicate<Integer> isPositiveWithPredicate = new Predicate<Integer>() {  
    @Override  
    public boolean test(Integer integer) {  
        return integer%2 ==0;  
    }  
};
```

이 역시도 , Function 과 동일

```java
Predicate<Integer> isPositiveWithLambda = (element) -> element%2 ==0;
```

역시 Lambda 도 가능


```java
Function<Integer,Boolean> isPositiveWithFunction = new Function<Integer, Boolean>() {  
    @Override  
    public Boolean apply(Integer integer) {  
        return integer %2 ==0;  
    }  
};
```

Consumer 랑 다르게 , Function 으로도 구현이 가능하다.

#### Why Using?

그러면 , 왜 Predicate 로 따로 만들어서 사용을 할까?

-> 이유는 굳이 불필요한 변환을 만들어서 사용할 필요 없이 바로 사용하기 위해서

```java
List<Integer> numbers = List.of(-5,-4,-3,-2,-1,0,1,2,3,4,5);  
  
Predicate<Integer> isPositive = (element) -> element%2 ==0;  
List<Integer> positiveNumbers = new ArrayList<>();  
for(Integer num : numbers){  
    if(isPositive.test(num)){  
        positiveNumbers.add(num);  
    }  
}  
  
Predicate<Integer> isLessThan3 = i -> i<3;  
List<Integer> numbersLessThan3 = new ArrayList<>();  
for(Integer num : numbers){  
    if(isLessThan3.test(num)){  
        numbersLessThan3.add(num);  
    }  
}
```

해당 코드는 , for 구문 안의 if 문에 Predicate 만 다를 뿐 , 다른 부분이 동일하다.
=> 매우 비효율적

```java
private static<T> List<T> filter(List<T> list , Predicate<T> filter){  
    List<T> result = new ArrayList<>();  
    for(T element : list){  
        if(filter.test(element)){  
            result.add(element);  
        }  
    }  
    return result;  
}

System.out.println(filter(numbers,isPositive));  
System.out.println(filter(numbers,isLessThan3));
```

중복 되는 코드를 통일할 수 있다.


```java
private static <T> List<T> filter(List<T> list, Function<T, Boolean> function) {  
    List<T> result = new ArrayList<>();  
    for (T element : list) {  
        if (function.apply(element)) {  
            result.add(element);  
        }  
    }  
    return result;  
}
```

Function 으로도 구현 가능하나 , 
`Function<Integer,Boolean> isPositiveWithFunctions =  (element) -> element%2==0;`

이렇게 불필요한 Boolean 타입 지정을 강제한다.
=> 가독성을 위해서라도 Predicate 를 사용하자
