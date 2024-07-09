---
tags:
  - 자바
추가_정보: 백기선 라이브 스터디
주차: "10"
---
#### Process VS Thread

우리가 사용하는 일반적인 프로그램들은 프로세스
( 프로그램 실행하면 OS로부터 자원 할당을 받아 프로세스가 된다 )
- 멀티 태스킹 : 여러 개의 프로세스를 동시에 실행 가능

이 프로세스는 자원 ( 프로그램 수행에 필요한 ) 과 스레드로 구성되어 있다
- 멀티스레드 프로세스 : 둘 이상 스레드를 가진 프로세스
( 사용하면 CPU 사용률 향상 , 자원 효율적 사용 , 응답성 향상 등 장점이 있다 - 동기화 , 교착상태 같은 단점도 존재 )

=> 즉 프로세스 자원을 이용해 실제 작업을 수행하는게 스레드

---

Java 는 스레드를 구현하려면 
1. Thread 클래스 상속
2. Runnable 인터페이스 구현
두가지 방법을 제공한다

### Thread

```java
public class Thread implements Runnable {
	...
}
```

```java
public class MyThread extends Thread{	  
	@Override  
	public void run() {  
	}  
}
```

Thread 도 결국 Runnable 을 implements 해서 구현
### Runnable

```java
@FunctionalInterface  
public interface Runnable {
	public abstract void run();  
}
```

추상 메소드 run 만 가진 매우 간단한 Interface

#### Thread VS Runnable

##### 상속 , 인터페이스 구현

Thread 는 Thread Class 를 직접 상속 받아 사용 -> 다중 상속 불가능
Runnable 은 Runnable 인터페이스를 구현해 실행 코드 포함 객체를 포함하는 객체 생성 -> 다중 상속 가능
##### 공유 데이터

Thread 는 해당 클래스의 인스턴스 변수로 공유 - 각각 스레드가 별도 인스턴스를 가지므로 독립적 변수
Runnable 을 구현한 클래스의 인스턴스 변수로 공유 - 여러 스레드가 동일 Runnable 인스턴스를 가지므로 공유 변수

---

#### Start

실제 스레드 실행
새로운 스레드가 생성되어 실행할 호출 스택을 만듬 -> run 메소드 호출 -> 호출 스택 번갈아가며 반복

- start 를 호출해 한 번 실행한다면 , 다시 생성후 start를 호출해야 한다
	-> 생성 한번하고 , start 두 번 호출시 `IllegalThreadStateException` 발생
