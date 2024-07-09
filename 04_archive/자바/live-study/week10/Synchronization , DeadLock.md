---
tags:
  - 자바
추가_정보: 백기선 라이브 스터디
주차: "10"
---
### Synchronization

멀티 스레드 프로세스에서는
여러 프로세스가 메모리 공유하기 때문에 다른 스레드가 간섭하는 문제가 발생 가능

특정 스레드가 진행중인 작업을 다른 스레드가 간섭하지 못하게 할 필요가 있다
=> 동기화 ( Synchronization )

#### critical section

다른 스레드가 간섭하지 못하는 영역
`synchronized` 키워드를 사용해서 설정

##### Method Synchronization

```java
public synchronized void method(){
	...
}
```

메소드가 한번에 한 스레드에서만 실행되게 보장

##### Block Synchronization

```java
synchronized (object){

}
```

특정 영역을 임계 설정 -> 해당 영역이 한 스레드에서만 실행되게 보장
##### lock

모든 객체라 lock 을 가짐
해당 객체 lock 을 가지는 스레드만 임계 영역 코드 수행 가능
-> 다른 스레드들은 lock 을 얻을 때까지 기다림

임계 영역은 멀티 스레드 성능을 좌우
=> 메소드 전체 locking 보다 synchronized 블록으로 임계 영역 최소화

###### locking 이 없다면?

```java
if (balance >= money) {
	try {
		// 문제 상황을 만들기 위해 고의로 쓰레드를 일시정지
		Thread.sleep(1000);
	} catch (InterruptedException e) {}

	balance -= money;
}
```
해당 코드는 음수 발생

```java
public void withdraw(int money) {
	synchronized (this) {
		if (balance >= money) {
			try {
				// 문제 상황을 만들기 위해 고의로 쓰레드를 일시정지
				Thread.sleep(1000);
			} catch (InterruptedException e) {
			}

			balance -= money;
		}
	}
}

```
synchronized 를 통해 병렬 수행 방지

### Deadlock

한 자원을 여러 시스템이 사용하려 할 때 발생할 수 있음


Process 1 , Process 2 가 모두 자원 A,B가 필요한 경우?

Process1 이 A에 먼저 접근 , Process2가 B에 먼저 접근
Process 1 이 B에 접근하기 위해 B의 락이 풀리기를 대기
Process 2 가 A에 접근하기 위해 B의 락이 풀리기를 대기

=> 서로 원하는 리소스가 상대방에 할당 되어 있어 무한히 대기 = 데드락

4가지 중 하나라도 성립하지 않으면 교착 상태 해결 가능
- 상호 배제 : 자원은 한 번에 한 프로세스만 사용 가능
- 점유 대기 : 최소 하나 자원 점유 + 다른 프로세스에 할당된 자원 추가 점유 위해 대기하는 프로세스
- 비선점 : 다른 프로세스 할당된 자원은 사용 끝날 때 까지 강제로 빼앗을 수 없어야 함
- 순환 대기 : 프로세스 집합에서 , P0이 P1점유 자원 위해 대기 -> P1이 P2점유 자원 위해 대기

=> 동기화를 하면 , 하나의 스레드에서 밖에 작업을 못하므로 비효율적
#### wait , notify

```java
void wait()
void wait(long timeout)
void wait(long timeout,int nanos)
```

객체의 lock 을 풀고 , 스레드를 해당 객체 waiting pool 에 넣음

```java
void notify()
void notifyAll()
```

waiting pool 에서 대기 중인 스레드를 깨운다
- notify 는 하나
- notifyAll 은 전부

wait 를 한 후
다른 곳에서 연산을 한 후 notify
