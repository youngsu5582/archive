---
tags:
  - 자바
추가_정보: 백기선 라이브 스터디
주차: "2"
---
### 변수 선언

```java
Long var1;
// 타입 변수이름
Long var1,var2,var3;
// 타입 변수이름...
```

- 매우 간단

#### Naming Convention

- 첫번째 글자는 문자이거나 지정된 특수 기호 ( $ , _ )
- 영어 대소문자 구분
- 첫 문자는 소문자 , 다른 단어는 대문자로 지정
- 문자 수의 제한 X , 예약어 사용 불가능

##### 예약어

- 기본 데이터 타입 ( boolean , byte ... )
- 접근 지정자 ( private , protected , public  )
- 클래스 관련 ( class , abstract , interface , extends , implements , enum )
- 객체 관련 ( new , instanceof , this , super , null )
- 메소드 관련 ( void , return )
- 제어문 관련 ( if , else , switch , case , default , for , do , while , break , continue )
- 논리 값 ( true , false )
- 예외 처리 관련 ( try , catch , finally , throws )
- 기타 ( import , synchronized , transient , volatile , package , native , final , static , strictfp , assert )

### 변수

- 지역 변수 < - > 필드 , 멤버 변수 , 전역 변수 로 존재
```java
public class TestClass {
	int var = 1; // 필드,멤버 변수
	static int globalVar; // 전역 변수
	public void testFunction(){
		int var2 = 2;  // 지역 변수
	}
}
```

### 변수 초기화

#### 1. 명시적 초기화
```java
int var = 1004;
```

변수 선언하고 , 바로 뒤 '=' 대입 연산자 사용해 변수에 데이터 초기화

#### 2. 블록 초기화
```java
// 일반적 초기화 블록
int hello = 5;   
{  
    hello = 10;  
    System.out.println(hello);  
}
static int globalVar = 10;
static {
	globalVar = 20;
}
```
명시적 초기화에 한계가 있으므로 , 블록 방식으로 사용

- 일반적 초기화 블록 : Class 가 New 를 통해 인스턴스 생성 순간 초기화 진행
	-> 초기화 블록 후 Constructor
- 정적 초기화 블록 : JVM 이 Class Loader 로 로딩 시점 초기화 진행

```java
{
	// 복잡한 로직 or Logging
	Logger.info("Variable Computing about Value");
	this.value = computeValue();
}
```

이렇게 , 단순 초기화 뿐만 아니라 , 부가적인 로직도 가능
=> 사실 , 그렇게까지 필요한지는 모르겠음
