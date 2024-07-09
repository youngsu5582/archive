---
tags:
  - 자바
도서명: 모던 자바 인 액션
---
### 람다의 특징

- Anonymous - 명확한 이름을 가질 필요 X
- Function - 람다는 특정 클래스와 연결되지 않는 , 메소드와 다른 개념
	( 물론 메소드 처럼 매개변수 , 본문 , 반환 타입 , 예외 목록 을 가지고 있음 )
- Passed around - 람다는 메소드 인자로 전달되거나 , 변수에 저장 가능
- Concise - 익명 클래스처럼 보일러 플레이트를 작성할 필요 X - 간결

=> 코드가 명확해지고 유연해짐

```java
Comparator<Apple> byWeight = (Apple a1,Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
```


(Apple a1,Apple a2) -> a1.getWeight().compareTo(a2.getWeight());

- A list of parameters - (Apple a1 ,Apple a2)
- An arrow - ->
- Lambda body - a1.getWeight().compareTo(a2.getWeight());

자바는 앞선 C# , Scala 를 기반으로 , 람다 표현식을 선택했다.

(parameters) -> expression
(parameters) -> { statements; } - return 문 반드시 명시

boolean expression - (List<`String`>list) -> list.isEmpty()
Creating Object - () -> new Apple(10)
Consuming from an object - (Apple a) -> {System.out.println(a.getWeight());}
Select/extract from an object - (String s) -> s.length()
Combine two values - (int a,int b) -> a`*`b
Compare two objects - (Apple a1,Apple a2) -> a1.getWeight().compareTo(a2.getWeight())

### 어디에 어떻게 사용해야 하는가

#### Functional Interface

```java
public interface Predicate<T>{
	boolean test(T t);
}
```

기존에 구현한 Predicate 역시도 Functional Interface!
-> 하나의 추상 메소드만 정의 하므로 

#### Functional descriptor

추상 메소드의 Signature 는 어떤 람다 표현식인지 표현하는 Signatrue

( 그냥 특정 인터페이스를 사용해 , 사용한 람다 표현식을 호출 )



- Predicate<`T`>    T -> boolean    IntPredicate,LongPredicate,DoublePredicate

Predicate 가 있는데 , 왜 IntPredicate 같은게 있는가?
-> 기본 타입 int 가 Integer 로 Boxing 을 해야 사용 가능하므로 , 추가적 메모리 할당 & 해제 필요

- Consumer<`T`>    T -> void    IntConsumer,LongConsumer,DoubleConsumer

위와 동일한 이유

- Function<`T,R`>    T -> R    IntFunction , ToIntFunction , IntToLongFunction ( 매우 많음 )

- Supplier<`T`>    () -> T    BooleanSupplier , IntSupplier , LongSupplier , DoubleSupplier

- UnaryOperator<`T`>    T -> T    IntUnaryOpeartor , LongUnaryOpeartor , DoubleUnaryOpeartor

- BinaryOperator<`T`>    (T,T) -> T    위와 동일

- BiPredicate<`L,R`>    (L,R) -> boolean    기본 제공 X

- BiConsumer<`T,U`>    (T,U) -> void    ObjIntConsumer<`T`> , ObjLongConsumer<`T`> , ObjDoubleConsumer<`T`>
	T t , int value 형식으로 사용

- BiFunction<`T,U,R`>    (T,U) -> R    ToIntBiFunction<`T,U`> , 위와 동일
	T t , U u , int value 형식으로 사용

##### Funtional Interface

위의 Descriptor 은 모두 @FunctionalInterface 애노테이션이 붙어있다
( @Override 처럼 , 함수형 인터페이스를 나타내주는 의미 )

두개의 abstract 메소드가 있으면 컴파일 단에서 에러 발생!

설정과 정리 단계는 항상 비슷하며 처리하는 중요한 코드를 둘러쌈
=> execute around pattern

```java
try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
    return br.readLine();
}
```

해당 부분이 실제 달라지는 부분

- functional interface 를 사용해 람다로 간소화 하자

```java
@FunctionalInterface
public interface BufferReaderProcessor {
	String process(BufferReader b) throws IOException;
}

public static String processFile(BufferReaderProcessor p) throws IOException{
	try (BufferReader br = new BufferedReader(new FileReader("data.txt")){
		return p.process(br);
	})
}

String oneLine = processFile((BufferedReader br) -> br.readLine());
```

![450](https://i.imgur.com/AxnXVKI.png)

해당 4단계를 거쳐 , Execute Around Pattern 을 함수형으로 변환


### 제공 Functional Interface

#### Predicate

T의 객체를 받아 boolean 반환 하는 test 추상 메소드 정의
```java
@FunctionalInterface
public interface Predicate<T>{
	boolean test(T t);
}
```

```java
Predicate<String> nonEmptyStringPredicate = (String s) -> !s.isEmpty();
List<String> nonEmpty = filter(list,nonEmptyStringPredicate);
```

#### Consumer

T의 객체를 받아 void 즉 , 아무것도 반환 하지 않는 accpet 추상 메소드 정의

```java
@FunctionalInterface
public interface Consumer<T>{
	void accept(T t);
}
public static <T> void forEach(List<T> list, Consumer<T> c){
	for(T e : list){
		c.accept(e);
	}
}
```

```java
forEach(List.of(1,2,3,4,5),(Integer i) -> System.out.println(i);)
```

#### Function

T 의 객체를 입력으로 받고 , R의 객체를 반환하는 applt 추상 메소드 정의
( 입력 객체 정보를 추출해 , 출력을 매핑하는 람다 정의 )

```java
@FunctionalInterface
public interface Function<T,R>{
	R apply(T t);
}
public static <T, R> List<R> map(List<T> list, Function<T, R> f){
	List<R> result = new ArrayList<>();
	for(T s: list){
		result.add(f.apply(s));
	}
	return result;
}
```

#### Primitive Specializations

결국 자바 타입은 참조 타입 or 원시 타입 중 하나
제네릭에는 참조 타입만 바인딩이 가능하다
-> 원시 타입은 참조 타입으로 변환되는 매커니즘이 필요 ( Boxing )

```java
public interface IntPredicate{
	boolean test(int t);
}

IntPredicate evenNumbers = (int i) -> i % 2 == 0;
Predicate<Integer> oddNumbers = (Integer i) -> i % 2 == 1;
```

- IntPredicate 는 No Boxing!
- Predicate<`Integer`> 는 Boxing!

Boxing 은 추가적인 메모리 리소스를 사용 비효율적!
=> 그렇기에 , Primitive 타입에 대한 Predicate 를 미리 만들어놨다


#### Type Checking , Type Inference

그러면 , 람다는 어떻게 타입을 알아서 사용하는 걸까?

람다는 사용되는 맥락에서 유추 , 예상되는 타입 ( 전달되는 메소드 매개변수 or 할당되는 지역변수) 을 대상 타입

![400](https://i.imgur.com/j5N44gO.png)

1. 필터 메소드 선언부를 탐색
2. Predicate<`Apple`> p 타입 객체 기대
3. Preicate<`Apple`> 은 test 불리는 단일 추상 메소드 정의 함수형 인터페이스
4. boolean test(Apple apple) 불리는 단일 추상 메소드임을 확인
5. filter 메소드의 실제 인자가 요구 사항과 일치한지 확인

##### Same Lambda , Different Functional Interfaces

말 그대로 , 같은 람다여도 다른 Interface 에 할당이 가능할 수 있다

```java
Comparator<Apple> c1 =
    (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
ToIntBiFunction<Apple, Apple> c2 =
    (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
BiFunction<Apple, Apple, Integer> c3 =
    (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());

```

호환 가능하며 , 유동적인 Interface 사용 가능
본인이 필요한 Interface 를 선언해서 코드에서 사용하면 된다

##### Special void-compability rule

```java
// Predicate는 boolean을 반환
Predicate<String> p = s -> list.add(s);

// Consumer는 void를 반환
Consumer<String> b = s -> list.add(s);
```

둘다 호환 가능
add 함수는 boolean 을 반환하지만 , 단순 void 로 처리해도 상관없기 때문
매개변수의 리스트 역시도 호환 가능
#### Type inference

컴파일러가 람다 표현식에 연관된 함수형 인터페이스를 찾아가 target 의 type 역시도 추론 가능
매개 변수 부분에서 타입 생략 가능

```java
Comparator<Apple> c = (Apple a1,Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
Comparator<Apple> c = (a1,a2) -> a1.getWeight().compareTo(a2.getWeight());
```

이렇게 , 타입을 생략 할 수 있음

```java
Predicate<String> pc = s -> "hi".equals(s);
```

이렇게 매개변수가 하나이면 () 역시 생략 가능
#### Using Local Variables

일반적인 람다 표현식은 자체 매개변수만 사용 했다
익명 클래스 처럼 외부에 정의된 변수도 역시 사용 가능
```java
int portNumber = 1337;
Runnable r = () -> System.out.println(portNumber);
```

람다 표현식에서 외부 portNumber 를 접근 가능

```java
int portNumber = 1337;
Runnable r = () -> System.out.println(portNumber);
portNumber = 1500;
```

컴파일 에러 발생!
람다에서 사용하는 변수들은 final 이거나 , 사실상 final (값이 변하지 않는) 여야 함

- 인스턴스 변수는 힙 메모리 저장 , 모든 스레드가 공유 - 람다 표현식 역시 인스턴스 변수
- 지역 변수는 스택에 저장 , 각 스레드가 자신만의 스택 가짐
		메소드 호출 시 생성 - 실행 종료 시 제거 ( 지역 변수 참조가 유효하지 않게 됨 )

람다 표현식이 지역 변수 직접적 사용 한다면 , 실행 도중 사라질 위험이 존재한다
	-> 원본 메소드 실행이 끝나고 , 지역 변수 스택에서 제거된 후 람다 실행하여 변수 접근하려고 하면 문제 발생!

=> 그렇기에 람다 표현식은 지역 변수의 복사본에 캡쳐 ( final 이거나 , 사실상 final 이여야 함)
변경될 위험 없으므로 , 변수의 일관성 유지!

##### Closure

클로저는 해당 함수 밖의 변수를 제한 없이 참여할 수 있는 인스턴스
즉 , 람다에서도 메소드 인자로 전달될 수 있고 , 범위 바깥 변수에 접근 가능
하지만 , 지역 변수 내용 수정은 불가능 - 변수 자체가 아닌 값에 대한 클로저 생성의 느낌
( JavaScript 의 클로저는 변경 가능 )

#### Method Reference

메소드 참조를 사용해 , 메소드 정의 재사용하고 람다에 전달 가능
```java
inventory.sort((Apple a1,Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
=>
inventory.sort(Comparator.comparing(Apple::getWeight));
```

매우 간결하게 정리 가능
특정 메소드를 호출하는 람다를 위한 축약형
대상 참조는 구분자 ( :: ) 앞에 위치하고 , 메소드 이름은 뒤에 제공

람다 표현식과 동일한 작업 수행하지만 , 더 간결한 형태로 표현

() -> Thread.currentThread().dumpStack() => Thread.currentThread()::dumpStack
(str,i) -> str.substring(i) => String::substring
(String s) -> System.out.println(s) => System.out::println

##### 사용하는 방법

1. 정적 메소드 참조 : Integer 의 parseInt 를 사용하려면 ? => Integer::parseInt
	( String 이 parseInt 에 들어가는게 확실하므로 생략 가능 )
2. 임의 객체 인스턴스 메소드 참조 : String 의 length 메소드는 => String::length
3. 특정 객체 인스턴스 메소드 참조 : Transaction 의 getValue 메소드는 => transaction::getValue

![500](https://i.imgur.com/MI9TtZL.png)

#### Constructor references

```java
Supplier<Apple> c1 = Apple::new;
Supplier<Apple> c1 = () -> new Apple();
```

간편하게 구현 가능

```java
BiFunction<String,Integer,Apple> c3 = Apple::new;
Apple a3 = c3.apply("green",110);
```

String , Integer 를 통해 Apple 생성

만약에 매개변수가 2개보다 더 많다면?

```java
public interface TriFunction<T,U,V,R>{
	R apply(T t,U u,V v);
}
TriFunction<Integer,Integer,Integer,Color> colorFactory = Color::new;
```

직접 만들어서 매핑 가능

```java
BiFunction<Integer,String,Color> temp = Color::new;  
BiFunction<Integer,Integer,Color> temp2 = Color::new;
```

이렇게 , 받는 타입을 통해 생성자 지정 가능

### 복습 : 단계별 진행

#### 1. Pass Code
```java
void sort(Comparator<? superE> c);
```

해당 제공 API를 통해 정렬을 구현

```java
public class AppleComparator implements Comparator<Apple>{
	public int compare(Apple a1,Apple a2){
		return a1.getWeight().compareTo(a2.getWeight());
	}
}
inventory.sort(new AppleComparator());
```

#### 2. Use anonymous class

```java
inventory.sort(new Comparator<Apple>(){
	public int compare(Apple a1,Apple a2){
		return a1.getWeight().compareTo(a2.getWeight());
	}
})
```

#### 3.Use Lambda expression

```java
inventory.sort((Apple a1,Apple a2)->a1.getWeight().compareTo(a2.getWeight()));
```

```java
inventory.sort((a1,a2)->a1.getWeight().compareTo(a2.getWegiht()));
```

```java
inventory.sort(Comparator.comparing(a->a.getWeight()));
```

Comparator 의 헬퍼 메소드 comparing 을 통해, 더욱 간결화 가능

#### 4. Use Method References

```java
inventory.sort(comparing(Apple::getWeight));
inventory.sort(comparing(Apple::getWeight).reversed());
```

reversed 통해 바로 반대도 가능

##### Chaining Comparing

```java
inventory.sort(comparing(Apple::getWeight).reversed()).thenComparing(Apple::getCountry);
```
### Composing Predicate

```java
Predicate<Apple> notRedApple = redApple.negate();
```

negate 는 Prediate 의 부정 Predicate 를 다시 반환

```java
Predicate<Apple> redAndHeavyApple = redApple.and(a -> a.getWeight > 150);
```

and 를 통해 다음 조건문을 연결 가능

```java
Predicate<Apple> redAndHeavyOrGreen = redAndHeavyApple.or(a -> "green".equals(a.getColor()));
```

or 을 통해 또 연결 가능

### Composing Functions

```java
Function<Integer,Integer> f = x -> x + 1;
Function<Integer,Integer> g = x -> x + 1;
Function<Integer,Integer> h = f.andThen(g);
```

그 후 ,실행 명령어를 파이프처럼 연결 가능

```java
Function<Integer,Integer> f = x -> x + 1;
Function<Integer,Integer> g = x -> x * 2;
Function<Integer,Integer> h = f.compose(g);
h.apply(1);
```

compose 는 매개변수로 받은 함수를 호출한 결과에 현재 함수 적용

1 * 2 = 2 -> 2 +1 = 3 -> 3

#### andThen VS compose

andThen 은 한 함수의 출력을 다른 함수 입력으로 전달

compose 는 두 번째 함수의 결과를 다른 함수 입력으로 전달

=> 순서가 서로 반대

그 뒤 내용은 불필요하다 생각하여 정리 X

