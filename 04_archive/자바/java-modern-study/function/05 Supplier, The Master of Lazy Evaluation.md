---
tags:
  - 자바
강의명: 모던 자바 (자바8) 못다한 이야기
강의_링크: https://www.youtube.com/watch?v=7e7FCMFrwcg
---

### Supplier
```java
Supplier<T>
```

```java
T get();
```

- 단순 , T 값을 반환

### Example

Hello 문자열을 필요할 때 마다 추가해주자

```java
final Supplier<String> helloSupplier = () -> "Hello ";  
System.out.println(helloSupplier.get() + "world");
```

그냥 문자열을 바로 추가해주면 되는거 아닌가?
-> Lazy Evaluation 을 위해 사용한다!
### Lazy Evaluation

그 값이 필요할 때 까지 지연 시키는 방법

- 성능 최적화 : 실제 필요할 때 까지 값의 계산을 연기함으로써 프로그램 성능 향상
- 무한 구조 생성 가능 : 무한한 길이의 Data Structure 생성 가능! ( 실제 해당 데이터가 필요할 때만 수행되므로 )
- Side Effect 최소화 : 부수 효과 발생 시점을 제어하는데 도움이 됨 ( 필요할 때 순간만 실행하고 끝나므로 )

#### Example

해당 값은 매우 무거운 연산을 한다고 가정 ( Timer 3초 정지 )

```java
private static final String getVeryExpensiveValue() {  
  
    //Very Expensive Calculation  
    try {  
        TimeUnit.SECONDS.sleep(3);  
    } catch (InterruptedException e) {  
        e.printStackTrace();  
    }  
    return "Youngsu";  
}
```

```java
private static final void print(Integer number, String value) {  
    if (number >= 0) {  
        System.out.println("The value is " + value + ".");  
    } else {  
        System.out.println("Invalid");  
    }  
}
```

```java
print(-1, getVeryExpensiveValue());  
print(0, getVeryExpensiveValue());
```

해당 코드에서 , 위의 함수에서 값을 받아 처리한다고 가정

-1인 print 는 Invalid 를 출력
0인 print 는 문자열을 받아 출력한다!

`It Takes 6 Seconds` !!
이렇게 , -1 일 때는 시간을 소요할 필요가 없음에도 기다려서 값을 받아온다.

=> 매우 비효율적!

```java
private static final void printWithSupplier(Integer number, Supplier<String> valueSupplier) {  
    if (number >= 0) {  
        System.out.println("The value is " + valueSupplier.get() + ".");  
    } else {  
        System.out.println("Invalid");  
    }  
}
```

```java
printWithSupplier(-1, () -> getVeryExpensiveValue());  
printWithSupplier(0, () -> getVeryExpensiveValue());  
printWithSupplier(-2, () -> getVeryExpensiveValue());
```
`It Takes 3 Seconds` !!
해당 문자열이 필요한 경우에만 get 해서 받아온다.

=> 자원 낭비 방지 가능!!


#### 사담

Java Stream 에서도 , 이와 유사한 Lazy Execution 을 사용한다!

filter , map , limit 같은 Stream To Stream 연산들은 즉시 처리하지 않고 , 처리 파이프라인을 구축만 한다!
=> forEach , collect 와 같은 최종 연산들이 나올 때 전부 실행

차후 , 설명해볼 예정

###### The Master of Lazy Evaluation

원하는 걸 진짜 필요할 때만 공급해주는 게으름의 마스터