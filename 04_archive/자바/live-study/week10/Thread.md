---
tags:
  - 자바
추가_정보: 백기선 라이브 스터디
주차: "10"
---
#### Demon Thread

스레드는 일반 스레드 <-> 데몬 스레드로 나뉨

데몬 스레드는 일반 스레드의 보조 역활 수행 스레드 ( 일반 스레드와 작성 방법은 같음 )
일반 스레드 종료 시 , 데몬 스레드 강제적 종료

=> GC , 자동 저장 , 화면 자동 갱신 등에 사용

- 데몬 스레드가 생성한 스레드는 자동 데몬 스레드

```java
boolean isDemon()  // 스레드가 데몬 스레드인지 아닌지 반환
void setDemon(boolean on)  // 스레드를 데몬 스레드 또는 사용자 스레드로 변경
```

### Thread State

- NEW : 스레드가 생성되고 , start() 가 호출되지 않은 상태
- RUNNABLE : 실행 중 or 실행 가능 상태
- BLOCKED : 동기화 블록에 의해 일시정지된 상태 ( lock 이 풀릴 때 까지 기다리는 상태 )
- WAITING , TIMED_WAITING : 작업 종료되지 않으나 , 실행 가능하지 않은 일시정지 상태
- TERMINATED : 스레드의 작업이 종료된 상태

##### BLOCK VS WAITING

BLOCK 은 synchronized 키워드 사용해 동기화된 코드에 들어가려는 스레드가 다른 스레드에 의해 블로킹
WAITING 은 sleep , wait , join 같은 메소드를 통해 일시정지 상태

getState 메소드 통해 스레드 상태 확인 가능 - jdk 1.5 도입

### Thread Method

#### 제어 메소드

- sleep(long mills) , sleep(long mills, int nanos) : 지정 시간 동안 스레드 일시정지
	-> 주어진 시간 지나면 자동 실행 대기 상태
- join(long mills) , join(long mills,int nanos) : 다른 스레드가 실행되도록 지정된 시간 동안 대기
	-> 다른 스레드가 종료되길 기다리며 , 시간 지나거나 작업 종료되면 다시 실행
- interrupt() : 현재 진행중인 스레드 중단 - InterruptedException 예외 발생시키며 중단

#### 상태확인 메소드

- checkAccess() : 현재 수행중인 스레드가 해당 스레드 수정할 권한 있는지 확인
	-> 없다면 SecurityException 예외 발생
- isAlive() : 스레드가 살아 있는지 확인 , 해당 스레드의 run 메소드가 종료되었는지 확인
- isInterrupted() : 정상적 종료가 아닌 , interrupt() 메소드 호출 통해 종료되었는지 확인
- static boolean interrupted() : 현재 스레드가 중지되었는지 확인 
#### static 메소드
- static int activeCount : 현재 스레드가 속한 스레드 그룹 의 스레드 중 살아있는 스레드 개수 리턴
- static Thread currentThread : 현재 수행중인 스레드 객체 리턴
- static void dumpStack() : 콘솔에 현재 스레드 스택 정보 출력
---
### 우선순위

스레드가 가지고 있는 멤버 변수
우선순위를 다르게 해 어떤 스레드에 더 많은 작업 시간 부여할 지 설정 가능

- 1에서 10사이 값을 지정할 수 있음
- 기본값은 5

```java
public class Thread implements Runnable {

	void setPriority(int newPriority)

	int getPriority();

	public static final int MIN_PRIORITY = 1;
	
	public static final int NORM_PRIORITY = 1;

	public static final int MIN_PRIORITY = 1;
}
```

