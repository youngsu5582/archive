---
tags:
  - 자바
추가_정보: 백기선 라이브 스터디
주차: "5"
---
Java 에서 Fucntion 에 대한 정의는 다소 애매할 수 있다

함수형 프로그래밍 한참 이전부터 Java 는 존재해왔고 , 
Method 는 다른 프로그래밍 언어에서의 함수의 개념과 비슷하게 동작해왔다.

그래서 , 함수형 프로그래밍의 함수를 의미하지 않는 부분도 존재한다.
### Method

객체 ( 클래스 or 인터페이스 ) 에 선언되어 있는 코드 블록

- 객체의 동작 정의
```java
class Dog{
	String voice;

	public void bark(){
		System.out.println("Bark Bark : "+voice);
	}
}
```

=> 즉 , 특정 클래스 와 구조체에 종속되어 있는 함수가 메소드

### Function

Java 8 이후에 함수형 프로그래밍을 지원한다

단순한 Function 이 아니라 , 익명 함수와 유사

```java
Function<Integer,Integer> square = x -> x * x;

Foo foo = new Foo() { // 실제로는 메소드를 내부 클래스 내에서 구현중임
	@Override
	public void print(String s) {
		System.out.println(s);
	}
}
```

물론 , Lambda 및 화살표 함수는 익명으로 생성후 , 클래스 생성 X

익명 클래스는 컴파일 단위에 , 클래스 생성 O

### 결론

메소드 와 함수의 차이점은 다소 애매할 수 있다.

함수는 호출이 되면 Stack 메모리에 저장 , 종료시 메모리에서 삭제
메소드는 생성 객체 기능 구현 위해 클래스 내부에 구현되어 있다

그냥 함수는 Java8 이후 Lambda 나 함수형 프로그래밍이라 생각하고 , 
메소드는 이전부터 존재했는 Class 에 존재하는 함수라고 생각하자


