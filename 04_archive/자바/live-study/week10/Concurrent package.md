---
tags:
  - 자바
추가_정보: 백기선 라이브 스터디
주차: "10"
---
java5 에서 추가된 패키지
동기화가 필요한 상황에서 사용 가능한 다양한 유틸리티 클래스 제공

### Locks

상호 배제 사용 가능한 클래스들 제공

- synchronized 블록 사용할 때와 동일 메커니즘 동작
( 내부적 synchronized 사용하며 더욱 유연 & 세밀하게 처리 )

#### Interface

##### Lock
공유 자원 한번에 한 스레드만 read , write 수행 가능하게 함
##### ReadWriteLock
Lock 보다 한단계 발전
공유 자원에서 여러개 스레드에서 read  , 하나의 스레드에서 write
##### Condition
Object 클래스 wait , notify , notifyAll 대체
wait -> await , notify -> signal , notifyAll -> signalAll

#### ReentrantLock , ReentrantReadWriteLock

위 인터페이스들의 구현체

##### Method

- lock : 인스턴스에 잠금 ( 이미 잠겨있을 시 , 잠긴 스레드가 unlock 호출할 때 까지 실행 비활성화 )
- lockInterruptibly : interrupted 상태 아닐 시 잠금 ( interrupted 상태면 , InterruptedException 발생 )
- tryLock : 즉시 Lock 인스턴스에 잠금 시도 , boolean 타입 반환
- unlock : 인스턴스 잠금 해제
- newCondition : 현재 Lock 인스턴스와 연결된 Condition 객체 반환

### VS synchronized

Lock 을 통해 좀더 세밀한 제어 가능
lock 과 unlock 메소드 통해 , 임계 영역 시작 , 끝 명확히 지정 가능
-> synchronized 는 메소드 or 블록 단위 동기화 제공

ReentrantLock 을 통해 공정성 제어 가능
- fairness ( 공정성 ) : 모든 스레드가 자신 작업 수행 기회를 공정하게 가지는 것
Queue 안에서 스레드가 무조건 순서가 지켜지며 , lock 확보
-> 불공정일시 , lock 이 필요한 순간 release 발생하면 대기열 건너뛰는 가능성 생김

```java
public ReentrantLock(boolean fair) {
	sync = fair ? new FairSync() : new NonfairSync();
}
```

생성자 인자를 통해 fair - nonfair 생성 가능

### Atomic

동기화 가능한 변수 제공

- atomicity ( 원자성 ) : 쪼갤수 없는 가장 작은 단위

Atomic Type 은 Wrapping 클래스들의 일종 ( Integer , Long ... )
-> 참조 타입 , 원시 타입 두 종류 변수 모두 적용 가능

CAS ( Compare-And-Swap ) 알고리즘 사용해 lock 없이 동기화 처리 가능

- AtomicBoolean
- AtomicInteger
- AtomicLong
- AtomicIntegerArray
- AtomicDoubleArray

#### 주요 메소드

- get : 현재 값 반환
- set(T newValue) : newValue 로 값 업데이트
- getAndSet(T newValue) : 값을 업데이트 하며 , 원래 값 반환
- CompareAndSet(expect,update)
	현재 값이 exepct 값과 동일하면 값 update 후 true 반환 , 동일하지 않으면 update 하지 않고 false 반환

#### CAS

메모리 위치 내용을 주어진 값과 비교해 동일 경우에만 메모리 위치의 내용을 새로 주어진 값으로 수정
즉 , synchornized 처럼 blocking 메커니즘이 아님
