---
tags:
  - 자바
추가_정보: 백기선 라이브 스터디
주차: "8"
---

객체와 객체 사이에 일어나는 상호 작용의 매개로 사용

모든 기능을 추상화로 정의한 상태로 선언
- class 대신 interface 키워드를 사용
- 접근 제어자로는 public or default 로 사용

#### Java 8 이전

```java
interface 인터페이스명{
	타입 상수명 = 값;
	타입 메소드명(매개변수);	
}
```

상수 와 추상 메소드만 가능했다

- 인터페이스 내 존재하는 메서드는 public abstract 으로 선언 , 생략 가능
- 인터페이스 내 존재하는 변수는 public static final 로 선언 , 생략 가능

#### Java 8 이후

```java
public interface 인터페이스명{
	default 타입 메소드명(매개변수){
		//구현
	}
	static 타입 메소드명(매겨변수){
		//구현
	}
}
```

default 메소드는 구현해서 기본적 제공해주나 , 
구현 내용을 원하지 않는다면 오버라이딩 해 재구현 해서 사용

static 메소드는 인터페이스에서 제공해주는 것으로 무조건 사용

---
추상 클래스와 같이 추상 메소드르 가지므로 , 추상 클래스와 매우 유사한 느낌
( 단 , 인터페이스는 상속받은 클래스에서 구현하여 사용)

#### Java 9 이후
```java
public interface 인터페이스명{
	default 타입 메소드명(매개변수){
		//구현
	}
	static 타입 메소드명(매겨변수){
		//구현
	}
	private 타입 메소드명(매개변수){}
}
```
private 함수를 default 함수에서 사용 가능

### Interface 사용 이유

##### 다중 상속
클래스는 하나만 상속 가능 , 인터페이스는 여러개 받고 구현 가능
-> 여러 기능 동시에 확장 가능
=> 클래스 간 결합도 낮추고 , 유연성 높여줌

##### 계약 정의
클래스가 특정 메소드 & 행위 구현하도록 강제
-> 인터페이스에 정의된 메소드를 제공한다는 것과 같음

##### 다형성 , 코드 재사용
여러 클래스가 같은 인터페이스 구현하면 , 구체적 타입 알 필요 없이 메소드 호출 가능
동일 인터페이스를 통해 공통 메소드 상속받아 사용 가능

##### 표준화 , API 디자인
표준화 된 인터페이스를 통해 코드 작성 가능
라이브러리 & 프레임워크 API 설계할 때도 사용 가능

### 구현 클래스

```java
public class 구현 클래스명 implements 인터페이스 며ㅇ{
	//구현
}
```

### 익명 구현 객체

위의 구현 방법이 일반적이나,
일회성으로 만들고 사용하는 것도 가능하다
```java
인터페이스 변수 = new 인터페이스(){
	인터페이스 선언된 추상 메소드 의 실체 구현
}
```

### 다중 인터페이스 구현
```java
public class 구현 클래스 implements 인터페이스A, 인터페이스 B{
	// 인터페이스 A의 추상 메소드 실체 메소드 선언
	// 인터페이스 B의 추상 메소드 실체 메소드 선언
} 
```

단 , 주의해야할 점이 있다.
```java
interface A{
	void m1();
}
interface B{
	Integer m1();
}

interface C extends A,B {}
// => 에러 발생
```

unrelated Return Type 에러가 발생한다.

### Q. 두개 인터페이스 구현하는 경우 , 메소드 시그니쳐가 같은 케이스라면?

추상 메소드를 재정의해서 사용 가능

```java
public interface JoinGroup {
    default void preJoin(){
        System.out.println("pre group");
    }
}
package me.ssonsh.interfacetest;

public interface JoinGroup {

    default void preJoin(){
        System.out.println("pre group");
    }
}

public class Member implements JoinMember, JoinGroup{
    @Override
    public void preJoin() {
        JoinMember.super.preJoin();
        JoinGroup.super.preJoin();
    }
}
```

- preJoin 메소드를 재정의 하지 않으면 컴파일 에러가 발생한다.
