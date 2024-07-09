---
tags:
  - 자바
강의명: 모던 자바 (자바8) 못다한 이야기
강의_링크: https://www.youtube.com/watch?v=UxrP-e1Ujfo
---


### Parallel Programming

여러 계산을 동시에 수행함으로써
컴퓨팅 작업을 더 빠르게 처리하는 프로그래밍 기법

=> Multicore Process & System + 분산  컴퓨팅 환경에서 유용합니다

- Stream API : parallel 메소드를 사용해 , 병렬 처리 가능
- 병렬 처리 : parallelStream 메소드 사용해 , 스트림 처리를 병렬로 수행 가능

#### 주의점

- 스레드 안전성 : 여러 스레드가 동시 접근하므로 , 스레드 안전성을 보장해야 함
- 상태 관리 : 여러 스레드에서 공유 자원 접근할 때 , 발생할 문제점 관리해야 함
- Context Switching 비용 : 스레드 많을수록 , Context Switching 에 따른 오버헤드가 증가함
- 데이터 분할 : 효율적으로 분할하기 위해 각 스레드 처리할 수 있게 설계해줘야 함

#### 이점

- 성능 향상 : 적절히 병렬 처리 시 , 멀티코어 프로세서 이점을 통해 성능 향상
- 자원 활용 : 시스템 자원을 보다 효율적 사용

### Example

0부터 10000까지의 숫자를 더하라

```java
//Stream Sum ( side-effect )  
final int[] num1 = {0};  
IntStream.range(0,10000)  
        .forEach(i -> num1[0] +=i);  
System.out.println(num1[0]);
```

해당 값은 `49995000` 이다

```java
// Parallel sum ( side-effect )  
final int[] num2 = {0};  
IntStream.range(0,10000)  
        .parallel()  
        .forEach(i -> num2[0] +=i);  
System.out.println(num2[0]);
```

해당 값은 계속 불일치 하게 나온다. ( `37722682`  이렇게 나옴 )

##### Why?

병렬 프로그래밍을 통해 여러 Thread 에서 사용하므로
num2 가 여러 곳에서 사용되어 순서대로 일치하지 않을 수 있다


### Example

1부터 8까지 배열의 값을 1초를 대기한 후 출력하라

#### Non - Parallel

```java
final long start1 = System.currentTimeMillis();  
List.of(1,2,3,4,5,6,7,8)  
        .stream()  
        .map(i->{  
            try {  
                TimeUnit.SECONDS.sleep(1);  
            } catch (InterruptedException e) {  
                throw new RuntimeException(e);  
            }  
            return i;  
        })  
        .forEach(System.out::println);  
System.out.println(System.currentTimeMillis() - start1);
```

```cmd
1
2
3
4
5
6
7
8
8043
```

1초씩 대기를 하고 연산하므로 , 
순서대로 & 8초의 시간대가 나온다.

#### Parallel

```java
final long start2 = System.currentTimeMillis();  
List.of(1,2,3,4,5,6,7,8)  
        .parallelStream()  
        .map(i->{  
            try {  
                TimeUnit.SECONDS.sleep(1);  
            } catch (InterruptedException e) {  
                throw new RuntimeException(e);  
            }  
            return i;  
        })  
        .forEach(System.out::println);  
System.out.println(System.currentTimeMillis() - start2);
```

```cmd
8
4
6
2
3
1
7
5
1032
```

1초씩 대기를 하나 , 각 요소들이 각자 수행되므로
순서 대로 되지 않고 & 1초의 시간대가 나온다.


- Java 의 기본적 parallelism 값은 컴퓨터의 Core 개수 에 따라 작동한다.
( 하이퍼 스레딩이 작동하는 경우 : Core 의 2배 )

Core 개수가 8 개일 시 , 기본적으로 작업용 병렬 스레드는 8개 가능!

- 메인 스레드 : 1개
- 병렬 스레드 : 8개
=> 총 9개의 스레드에서 작업 가능!

하지만 계산용 스레드는 8개 이므로 , 
9개의 요소일 시 ? => 2초 발생

##### Property
```java
System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism","0");
```

해당 값으로 설정 가능
- 0 이면 , 메인 스레드에서 전부 수행
- 이 값을 코어 수보다 더 키우면 ? => 스레드가 많이 생성되고 , Context Switching 이 일어난다!

병렬적으로 값이 수행된다.
```java
final long start3 = System.currentTimeMillis();  
IntStream.range(0, 512).boxed().toList()  
        .parallelStream()  
        .map(i -> {  
            try {  
                TimeUnit.SECONDS.sleep(1);  
            } catch (InterruptedException e) {  
                throw new RuntimeException(e);  
            }  
            return i;  
        })  
        .forEach(System.out::println);  
System.out.println(System.currentTimeMillis() - start3);
```

```cmd
464
32
224
96
160
112
80
336

288
113
97
33
81
225
161
337

289
465
98
226
82
34
114
338

466
162
290
227
83
115
35
467
```

값들을 보면 , 무질서 속에서도 1씩 증가를 하는게 보인다.