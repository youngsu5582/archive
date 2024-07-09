---
tags:
  - 자바
추가_정보: 백기선 라이브 스터디
주차: "9"
---
### Error

에러는 컴파일 에러 와 런타임 에러로 구분 가능

#### 컴파일 에러
컴파일 과정에서 일어나는 에러
자바 컴파일러가 문법 검사를 통해 오류 잡아준다

#### 런타임 에러
런타임 과정에서 일어나는 에러
컴파일이 문제없이 실행 되더라도 , 실행 과정에서 오류 발생 가능
예외(Exception) 와 에러(Error) 두가지로 다시 구분

#### 런타임 에러

JVM 이나 하드웨어 등 기반 시스템 문제로 발생 ( 시스템 레벨 발생 , 프로세스 영향 줌 )
-> 프로그래머가 어떻게 할 수 없는 문제 

- OutOfMemory : JVM 이 더 이상 메모리 할당할 수 없는 경우
- StackOverflow : 메소드 호출에 대한 스택 메모리가 너무 깊어져 스택 프레임 할당할 수 없는 경우
- NoClassDefFoundError : JVM 이 클래스 정의를 찾을 수 없는 경우 ( 컴파일 시 가능했으나 , 런타임 시 불가능 )


#### 런타임 예외

컴퓨터 에러가 아닌 사용자의 잘못된 조작 or 개발자의 잘못된 코딩으로 인해 발생
-> 프로그래머가 처리할 수 있는 문제
=> 예외처리를 통해 프로그램을 종료되지 않게 정상 작동되게 만듬
##### Checked Exception

컴파일러가 체크하고 확인하는 예외 ( 코드 작성 동안 컴파일러가 경고 통해 예외 처리 여부 알려줌 )
	-> 반드시 예외 처리 해야함 ( try-catch or throws 선언 )

- IOException
- RuntimeException
=> RuntimeException 을 제외한 모든 예외

##### Runtime Exception

예외가 발생할 것을 미리 감지 못할때 발생

런타임 예외에 해당하는 모든 예외들은 RuntimeException 을 확장
	-> 비검사(unchecked) 분류 - 명시적 예외 처리 강제 X

- NullPointerException
- ArrayIndexOutOfBoundsException

---

### Throwable

모든 예외 & 오류의 최상위 클래스 ( 즉 throwable 을 상속받아 처리 )

```java
try {
	// Simulating a scenario where a custom exception is thrown with a specific message.
	throw new CustomException("Custom Exception Message");
}
```
해당 코드를 통해 수행
#### getMessage

예외에 대한 상세한 정보 반환
예외 출력할 때 , 어떤 예외 발생했는지 확인할 때 유용
( 별도 예외 메시지를 사용자에게 보여줄 때 사용 )
=> `Custom Exception Message` 출력
#### toString

예외 객체에 대한 문자열 표현
예외 클래스 이름 과 예외 메시지 포함
( getMessage 보다 더 자세함 )
=> `CustomException: Custom Exception Message` 출력
#### printStackTrace

예외의 trace 정보 출력
어떤 메소드에서 예외 발생했는지 , 해당 위치 표시
=> `CustomException: Custom Exception Message
    `at Example.main(Example.java:8)` 출력

### 예외 처리

#### try-catch

try - catch - finally 문법으로 구분
```java
try {
    예외를 처리하길 원하는 실행 코드;
} catch (NullPointerException exception) {
    NullPointerException 예외가 발생할 경우에 실행될 코드;
}
finally {
	예외 발생 여부와 상관없이 무조건 실행될 코드;
}
```

매우 간단

catch 문을 여러개 작성 가능 - 해당하는 Exception 에 맞기 분기 처리

##### MultiCatch Block

jdk 1.7부터 도입

```java
catch (IllegalArgumentException | ArithmeticException e){ 
	System.out.println(e.getMessage()); 
	}
```

중복되는 처리 로직일 시 용이
단 , 부모 - 자식 관계 있을시 에러 발생
( 자식 클래스로 잡아내는 예외는 부모 클래스로도 잡을 수 있기 때문 )
#### throws

throws 는 명확하게는 예외 처리는 아님
해당 메소드를 사용하는 사람에게 예외 처리하도록 책임 전가하는 역활

```java
void throwsMethod() throws Exception {
	
}
```

#### re-throwing

한 메소드에 발생하는 예외가 여러개인 경우
일부는 메소드 처리하고 일부는 선언부에 지정해 메소드 호출한 쪽에서 처리하도록 할 수 있다
하나의 예외에 대해 양쪽에서 처리도 가능하게 할 수 있다
```java
try{
}catch(Exception e){
	...처리
}

void method(){
	try{
	}catch(Exception e){
		...처리
		throw e;
	}
}
```
##### chained Exception

한 예외가 다른 예외 발생시킬 수도 있음
예외 A 가 예외 B를 발생시킨다면 , A를 B의 원인 예외
```java
catch(ArithmeticException e){
	IllegalArgumentException exception = new Exception();
	exception.initCause(e)
}
```

```java
catch(IllegalArgumentException e){
	throw new RuntimeException(e);
}
```

이렇게 checked exception 을 unchecked exception 으로 변환 가능
### try-with-resources

jdk 1.7부터 도입

AutoCloseable 인터페이스를 구현한 리소스 사용할 때 , 코드를 간결하게 사용하게 도와줌
( finally 구문을 통해 , 자원을 닫을 필요 없음 )

```java
try(FileOutputStream outputStream = new FileOutputStream("")){

}catch(){

}
```

대부분의 자바 표준 라이브러리들은 AutoCloseable을 구현하므로 손쉽게 처리 가능

### 예외 계층 구조

![400](https://i.imgur.com/7EcwukO.png)

- Throwable 은 Object 직접 상속
- Exception , Error 는 Throwable 상속
#### Checked Exception VS Unchekced Exception

Exception 중 RuntimeException 을 제외하고는 모두 Checked Exception

Checked Exception 은 컴파일 시점 확인될 수 있는 예외
-> 코드 내에서 발생시키면 반드시 처리하거나 선언부에 throws 로 예외를 선언해줘야 함

Unchecked Exception 은 컴파일 시점 강제 확인하지 않는 예외
-> 예외 처리하거나 , 선언하도록 강제하지 않음

#### Why?

>오라클 공식 문서
>RuntimeException 은 catch 하거나 지정할 필요가 없다
>이는 컴파일 오류 없이 코드 작성하거나 예외 지정하거나 catch 하는 불편함 피하게 해준다
>편리해 보일수 있으나 catch , 요구 사항의 의도를 우회할 수 있다

>설계자들은 메소드에 발생할 수 있는 예외를 메소드의 공개 API 일부로 지정해야 한다고 결정했다
>이를 알아야 어떻게 처리할지 결정할 수 있기 때문

```java
public void readFile() throws IOException{

}
```
IOException을 던질수 있다고 명시

>그러면 Runtime Exception 들은 왜 API 를 표시 안하는 걸까?
>Runtime Exception 은 결국 프로그래밍의 실수로 나오는 결과
>어디에서든 발생할 수 있고 일반적인 경우도 많을 것이다
>이를 전부 추가하는게 오히려 가독성이 감소한다 - 실행 예외 catch or 지정할 필요 X

```java
public int divide(int numerator, int denominator) { 
	return numerator / denominator; 
}
```
ArithmeticException을 명시할 필요 X
즉 , RuntimeException 은 사용자가 메소드 잘못 호출할 때

일반적으로 메소드가 던질 수 있는 예외 지정하고 싶지 않아 RuntimeException 로 만들면 안된다

클라이언트가 예외를 합리적으로 복구할 수 있다면 Checked Exception 으로
클라이언트가 예외에서 아무것도 할 수 없다면 UnChecked Exception 으로 만들자

### Custom Error

직접 커스텀 에러를 만들려면 Exception 클래스를 상속 받아 사용하자
( Error 는 개발자가 절대 손되면 안됨 )

```java
public class MyException extends Exception {
    public MyException() {
        super();
    }

    public MyException(String message) {
        super(message);
    }
}

```

Exception 은 두개의 super 메소드 호출 가능

```java
private void throwException() throws MyException{

}
```

```java
private void throwException(){

	...
	throw new MyException();
}
```

해당 메소드를 사용하는 코드는 catch 문을 통해 Exception 해결법을 구현해야 한다

```java
public class MyException extends RuntimeException{
	...
}
```

RuntimeException 을 상속 받으면 해결법 구현할 필요 없음

#### Good Tip

- 실행 시 발생활 확률이 높은 경우에는 런타임 예외로 만드는 것도 좋다
	-> 물론 , 처리는 해주자

- 단순히 catch문만 구현하는건 피하자
```java
try{

}catch (MyException exception){
}
```

catch 문에서 아무런 작업을 하지 않으면 어디서 발생했는지 문제를 찾을 수 없다
