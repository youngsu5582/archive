---
tags:
  - 자바
강의명: 모던 자바 (자바8) 못다한 이야기
강의_링크: https://www.youtube.com/watch?v=fGpZFwxRvCA
---


Steram 이란 Collection Builder 똑같다고 생각하자! ( 엄밀히 말하자면 아니지만 , 이해를 위해 단순하게 설명 )

```java
final List<Integer> numbers = IntStream.rangeClosed(1, 10).boxed().toList();
```

Int형 Stream 을 1부터 10까지 반복 -> boxed 를 통해 Int -> Integer 로 Boxing -> List 로 변환
### Example

숫자가 3보다 크고 , 9보다 작은 값 중 
숫자 * 2 를 한 값 중에서 10보다 큰 가장 첫 번째 값을 구하라

#### Non - Stream
```java
for (final Integer number : numbers){  
    if (isSatisfaction(number)){  
        final Integer newNumber = number *2;  
        if(newNumber>10){  
            System.out.println(newNumber);  
            break;        
        }  
    }  
}
```

해당 코드는 변수명을 사용하기 때문에 헷갈릴 여지가 있다! ( 그 정돈가 ... )

#### Stream
```java
final Integer result = numbers  
        .stream()  
        .filter(Main::isSatisfaction)  
        .map(number -> number * 2)  
        .filter(number -> number > 100)  
        .findFirst()  
        .orElseThrow(() -> new IllegalArgumentException("Not Found!"));
```

filter 에서는 무조건 Integer 자료형이 넘어가므로 , 해당 함수에 넣어주지 않고 함수명만 넣어도 정상 작동한다!
하나씩 단계별로 , 수행
findFirst 로 가장 첫 번째 값 검색
orElseThrow 로 없을시 연산 수행 가능

### 그러면 성능은?

기본적인 시간 복잡도는 동일하다

단순 생각하면 , 
Stream 이란게 배열 전부를 순회하여 조건을 만족하는 값 선택!
-> 그 다음 조건 에 맞는 값 선택!

=> 매우 비효율적! 

하지만 , 해당 코드는

1 : 3보다 크고 9보다 작은지 검사 ( X )
2 : 3보다 크고 9보다 작은지 검사 ( X )
3: 3보다 크고 9보다 작은지 검사 ( X )
4 : 3보다 크고 9보다 작은지 검사 ( O )
	=> 4 * 2
	=> 10보다 큰지 검사 ( X )
5 : 3보다 크고 9보다 작은지 검사 ( O )
	=> 5 * 2
	=> 10보다 큰지 검사 ( X )
6 : 3보다 크고 9보다 작은지 검사 ( O )
	=> 6 * 2
	=> 10보다 큰지 검사 ( O )
=> findFirst 를 통해 종료

계속 다음 조건을 확인해가며 매우 효율적으로 작동한다!

앞서 말한 , Lazy Execution 때문!

> Java Stream 에서도 , 이와 유사한 Lazy Execution 을 사용한다!
> 
> filter , map , limit 같은 Stream To Stream 연산들은 즉시 처리하지 않고 , 처리 파이프라인을 구축만 한다!
> => forEach , collect 와 같은 최종 연산들이 나올 때 전부 실행

```java
final List<Integer> myOwnMethodResult =  
        filter(  
                map(  
                        filter(  
                                filter(numbers,  
                                        i -> i > 3),  
                                i -> i < 9  
                        ),  
                        i -> i * 2  
                ),  
                i -> i > 10  
        );
```

물론 이렇게도 가능!

### 기존 배운 코드들 변환

#### filter

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
```

=>

```java
private static final <T> List<T> filter(List<T> list,Predicate<T> predicate){  
    return list.stream().filter(predicate::test).toList();  
}
```

매우 간결해진다!

#### Map

```java
private static final <T, R> List<R> map(List<T> list, Function<T, R> mapper) {  
    final List<R> result = new ArrayList<>();  
    for (final T t : list) {  
        result.add(mapper.apply(t));  
    }  
    return result;  
}
```

=>

```java
private static final <T, R> List<R> map(List<T> list, Function<T, R> mapper) {  
    return list.stream().map(mapper::apply).toList();  
}
```

이 역시 매우 간결해진다!