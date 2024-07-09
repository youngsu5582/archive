---
tags:
  - 자바
추가_정보: 백기선 라이브 스터디
주차: "14"
---
### 바운디드 타입

특정 타입의 서브 타입으로 제한
Class , Interface 설계할 때 가장 흔하게 사용하는 개념

```java
public class BoundType <T extends Number> {}
```

Number의 서브 타입만 허용
#### 와일드 카드

`?` 를 사용해서 정의
제네릭 타입 매개변수가 불확실 할때 사용

#### T VS ?

둘다 Upper Bounded Wildcard 를 나타내나 , 문맥 과 제약이 다름

```java
public <T extends Fruit> void processFruit(List<T> list) {
}
```

- T 는 메소드 호출 시 구체적 타입으로 지정
- processFruit(new ArrayList<`Apple`>()) 처럼 호출하면 T는 Apple Classs 로 결정
	( 즉 , 컴파일 시 타입이 고정 )
=> 읽기 , 쓰기 모두 가능
=> 구체적 타입을 명시해 타입 안정성 보장 가능


```java
public void processFruit(List<? extends Fruit> list) {
}
```

- 메소드 호출 시 구체적 타입을 알 수 없음
- processFruit 를 호출할 때 컴파일러가 Fruit 의 어떤 하위 Class 가 있는지 알 수 없음
	( 즉 , 컴파일 시 타입 고정 X )
=> 읽기는 가능하나 , 쓰기는 불가능
=> 어떤 타입이든 받을 수 있어서 유연성 높아지나 읽기 전용으로만 사용해야 함
