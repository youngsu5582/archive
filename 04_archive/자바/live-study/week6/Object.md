---
tags:
  - 자바
추가_정보: 백기선 라이브 스터디
주차: "6"
---
클래스가 다른 클래스를 상속하지 않으면 상속 받는 최상위 클래스

Object Class 는 필드가 없고 메소드로만 구성되어 있다
-> 모든 클래스에서 당연히 사용 가능

### equals (Object obj)

객체의 동등성 비교
기본 구현은 객체의 참조(주소)를 비교
-> 오버라이드 하여 , 논리적 비교 하는식으로도 수정 가능

```java
public class Member{
	public String id;	

	@Override
	public boolean equals(Object obj){
		if (obj instanceof Member){ // 인스턴스 검사
			Member member = (Member) obj; // 캐스팅
			if(id.equals(member.id)){
				return true;
			}
		}
	}
}
```
### hashCode()

객체의 해시 코드 반환( 객체의 메모리 주소를 이용해 만듬 )
equals 메소드를 사용하는 컬렉션에서도 사용 가능
-> HashSet , HashMap , Hashtable 들은 hashCode 값 -> equals 값 등으로 동등한지 비교

=> 즉 Object 의 equals 메소드만 재정의 하는게 아닌 hashCode 메소드도 재정의해서 논리 동등하게 해야한다.
```java
@Override
public int hashCode(){
	return id.hashCode();
}
```

### toString()

객체의 문자 정보 리턴
일반적으로 , 클래스명@16진수해시코드 구성
```java
public String toString() {  
    return getClass().getName() + "@" + Integer.toHexString(hashCode());  
}
```

오버라이딩해서 간결 & 유익한 정보 리턴하도록 구현


### clone()

원본 객체의 필드값 과 동일 값을 가지는 새로운 객체 생성

##### Why?
원본 객체를 안전하게 보호하기 위해서
-> 원본 객체 넘겨 작업할 경우 , 원본 객체 데이터 훼손 가능

#### 얕은 복사

단순 필드값 복사해 객체 복사
필드가 
기본 값인 경우 , 값 복사
참조 타입인 경우 , 객체 주소 복사

=> object.clone() 은 얕은 복사된 객체 리턴

( java.lang,Cloneable 인터페이스를 구현하고 있어야 함 )
#### 깊은 복사

얕은 복사는 객체 주소를 복사하므로 , 원본 객체 필드 와 복사 객체 필드는 같은 객체를 참조
-> 참조 객체를 변경하면 , 원본 객체도 변경된다!

즉 , 참조하고 있는 객체들의 값도 복제를 하는게 깊은 복사! ( 객체 주소 복제 X )

=> 깊은 복사하려면 직접 구현해야한다.

```java
public class Member implements Cloneable {
    public String name;
    public int age;
    public int[] scores; // 점수 타입 필드 (깊은 복사 대상)
    public Car car; // 참조 타입 필드 (깊은 복사 대상)
    
    public Member(String name, int age, int[] scores, Car car) {
        this.name = name;
        this.age = age;
        this.scores = scores;
        this.car = car;
    }
    
    @Override // clone() 메소드 재정의
    protected Object clone() throws CloneNotSupportedException {
        // 부모의 clone 메소드를 호출하여 name, age 필드를 복제한다.
        Member cloned = (Member) super.clone(); // Object의 clone() 호출
        // scores 깊은 복제
        cloned.scores = Arrays.copyOf(this.scores, this.scores.length);
        // car 깊은 복제
        cloned.car = new Car(this.car.model);
        // 깊은 복제된 Member 객체 리턴
        return cloned;
    }
}
```

이런식으로 , 일일히 복제를 해야함
CloneNotSupportedException 해당 throw 문 처리해줘야함
### finalize()

참조하지 않는 배열 , 객체는 GC 가 힙 영역에서 자동 소멸
-> GC 가 객체 소멸 직전에 마지막으로 실행 시키는 객체 소멸자

객체 소멸 전 마지막 사용한 자원 ( 데이터 연결 , 파일 ) 을 닫고 싶거나,
중요한 데이터 저장하고 싶으면 finalize() 재정의

무작위로 소멸 , 메모리 상태 보고 일부 소멸하므로
프로그램 종료할 때 , 즉시 자원 해제 & 데이터 최종 저장해야 한다면 , 
프로그램 종료될 때 명시적으로 호출하는 것이 좋다

```java
@Override
protected void finalize() throws Throwable {
	try {
		// 객체 소멸 시 수행할 작업
		System.out.println("Finalizing the object");
	} finally {
		super.finalize();
	}
}

```
