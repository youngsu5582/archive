---
tags:
  - 자바
강의명: 모던 자바 (자바8) 못다한 이야기
강의_링크: https://www.youtube.com/watch?v=74nuREHD7p8
---


### Example

1 부터 n 까지 합을 구하라

#### Sample Code
```java
public static final long iterativeSum(long n) {  
    long result = 0;  
    for (long i = 0; i <= n; i++) {  
        result += i;  
    }  
    return result;  
}  
  
public static final long sequentialSum(long n) {  
    return Stream  
            .iterate(1L, i -> i + 1)  
            .limit(n)  
            .reduce(Long::sum)  
            .get();  
}  
  
public static final long parallelSum(long n) {  
    return Stream  
            .iterate(1L, i -> i + 1)  
            .limit(n)  
            .parallel()  
            .reduce(Long::sum)  
            .get();  
}  
  
public static final long rangedSum(long n) {  
    return LongStream  
            .rangeClosed(1, n)  
            .reduce(Long::sum)  
            .getAsLong();  
}  
public static final long parallelRangedSum(long n){  
    return LongStream  
            .rangeClosed(1, n)  
            .parallel()  
            .reduce(Long::sum)  
            .getAsLong();  
}
```
- iterativeSum : for구문 단순 반복
- sequentialSum : 순차 반복 reduce Stream
- parallelSum : 반복 reduce 를 하나 , parallel Stream
- rangeSum : 1부터 n 까지 범위 reduce Stream
- parallelRangedSum : 1부터 n 까지 범위 reduce parallel Stream

```java

Iterative Sum : 50000005000000
25
Sequential Sum : 50000005000000
138
Parallel Sum : 50000005000000
424
Ranged Sum : 50000005000000
14
Parallel Ranged Sum : 50000005000000
18
```

실행 결과 시 ,Parallel 이 매우 형편없다
##### Why?

parallel 은 순서 지정없이 , 계속해서 값을 쪼갠후 연산하고 합쳤다가 연산하고 쪼개는 것을 반복
=> 시간 오래 소모

```java
private static final void slowDown() {  
    try {  
        TimeUnit.MICROSECONDS.sleep(10L);  
    } catch (InterruptedException e) {  
        throw new RuntimeException(e);  
    }  
}
```

위 코드에 이렇게 , 10ms 대기를 한다면??

```java
Iterative Sum : 500500
1313
Sequential Sum : 500500
1293
Parallel Sum : 500500
200
Ranged Sum : 500500
1270
Parallel Ranged Sum : 500500
165
```

아래와 같은 결과가 나온다.

parallel 과 reduce 사이 복잡한 연산이 포함이 되어 있을때 +
map or filter 처럼 순서가 중요하지 않을 때 유용하다!

=> 사실 당연한게 , parallel 를 통해 상단과 같은 간단한 로직을 수행할 리가 없으며 이유도 없다.

### Example

수많은 Product 에서 조건을 만족하는 값만 합쳐서 계산하라

```java
    final List<String> priceStrings = List.of("1.0", "100.99", "35.75", "21.30", "88.00");  
    final Random random = new Random(123);  
    final Integer length = 8_000_000;  
    final List<Product> list = new ArrayList<>();  
    IntStream.range(0, length)  
            .forEach(  
                    i -> list.add(new Product(  
                            (long) i,  
                            "Product" + i,  
                            new BigDecimal(priceStrings.get(random.nextInt(5))))));  
    return list;  
}
```

무작위 800 만개의 Product 생성

```java
private static final BigDecimal imperativeSum(final List<Product> products,  
                                              final Predicate<Product> predicate) {  
    BigDecimal sum = BigDecimal.ZERO;  
    for (final Product product : products) {  
        if (predicate.test(product)) {  
            sum = sum.add(product.getPrice());  
        }  
    }  
    return sum;  
}
```
- For 구문 통한 연산

```java
private static final BigDecimal streamSum(final Stream<Product> stream,  
                                          final Predicate<Product> predicate) {  
    return stream.filter(predicate).map(Product::getPrice).reduce(BigDecimal.ZERO, BigDecimal::add);  
}
```
- Stream 통해 연산

```java
private static final BigDecimal parallelStreamSum(final Stream<Product> stream,  
                                                  final Predicate<Product> predicate) {  
    return streamSum(stream.parallel(), predicate);  
}
```
- Stream Parallel 통해 연산

```java
private static final void test1() {  
    System.out.println("========================================");  
    System.out.println("\tTest1");  
  
    for (int i = 0; i < 5; i++) {  
        BigDecimal price = targetPrices[targetPriceRandom.nextInt(3)];  
  
        imperativeTest(price);  
        streamTest(price);  
        streamParallelTest(price);  
        System.out.println("\n");  
    }  
}
```

해당 방법을 통해 , Seed 고정후 Random 하게 테스트

```cmd
========================================
	Test1
========================================
	Imperative

Sum: 393416804.09
It took 344 ms.
========================================
========================================
	Stream

Sum: 393416804.09
It took 178 ms.
========================================
========================================
	Stream Parallel

Sum: 393416804.09
It took 66 ms.
========================================
```

```cmd
========================================
	Test2
========================================
	Imperative

Sum: 393416804.09
It took 99 ms.
========================================
========================================
	Stream

Sum: 393416804.09
It took 170 ms.
========================================
========================================
	Stream Parallel

Sum: 393416804.09
It took 34 ms.
========================================
```

```cmd
========================================
	Test3
========================================
	Imperative

Sum: 359284746.89
It took 111 ms.
========================================
========================================
	Stream

Sum: 359284746.89
It took 157 ms.
========================================
========================================
	Stream Parallel

Sum: 359284746.89
It took 33 ms.
========================================
```

전반적으로 ,  Stream > Imperative > Stream parallel 의 performance 가 나왔다.

결국 , 상황에 맞게 Stream , Imperative , Parallel 을 선택해야 함

#### In GPT

이 사례는 병렬 처리와 순차 처리의 성능 차이를 비교하는 좋은 예입니다. 여기서 얻을 수 있는 주요 교훈은 다음과 같습니다:

1. **단순한 연산에서의 병렬 처리**: 첫 번째 예시에서는 간단한 덧셈 연산을 수행합니다. 이 경우, 병렬 처리는 순차 처리보다 성능이 현저히 떨어집니다. 이는 병렬 처리가 발생하는 오버헤드(스레드 생성 및 관리, 스레드 간 통신 등)가 간단한 연산의 이점을 상쇄하기 때문입니다.
    
2. **복잡한 연산에서의 병렬 처리**: 두 번째 예시에서는 더 복잡한 연산(여러 조건을 만족하는 제품의 가격 합계)을 수행합니다. 이 경우, 병렬 처리가 순차 처리보다 성능이 우수합니다. 복잡한 연산에서는 각 스레드가 수행하는 작업의 양이 많기 때문에 병렬 처리의 이점이 더 크게 나타납니다.
    
3. **적절한 사용 사례 선택**: 병렬 처리는 모든 경우에 유리한 것이 아니며, 사용 사례에 따라 적절하게 선택해야 합니다. 간단한 연산에서는 순차 처리가, 복잡한 연산에서는 병렬 처리가 더 효율적일 수 있습니다.
    
4. **시스템 리소스의 고려**: 병렬 처리는 시스템의 CPU 코어 수와 리소스를 고려해야 합니다. 코어 수보다 많은 스레드를 사용하는 경우, 컨텍스트 스위칭으로 인한 성능 저하가 발생할 수 있습니다.
    

결론적으로, 병렬 처리는 특정 조건(복잡한 연산, 적절한 리소스 사용 등) 하에서 성능 향상을 가져올 수 있지만, 모든 상황에 적합한 것은 아닙니다. 따라서 성능을 최적화하려면 연산의 복잡성, 데이터의 양, 시스템 리소스 등 여러 요소를 고려하여 순차 처리와 병렬 처리 중 적합한 방법을 선택해야 합니다.

제시한 예시와 결론은 병렬 처리의 이점과 한계를 잘 보여주고 있으며, 주어진 상황에 따라 최적의 접근 방법을 선택하는 것이 중요함을 강조합니다.