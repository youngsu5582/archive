---
tags:
  - 자바
추가_정보: 백기선 라이브 스터디
주차: "1"
---
#### Interpreter , Compiler in Java

![450](https://i.imgur.com/O6cvvMn.png)

![450](https://i.imgur.com/xqSmIMb.png)

=> 핵심 원리는 , Caching 과 비슷한 원리!
### Class Loader

![450](https://i.imgur.com/lj1OYib.png)

- 로딩 : JVM 이 Loader 를 통해 , .class 파일을 찾아 메모리에 로드 , 세 가지로 Loader 에 의해 수행
	- Bootstrap Loader : rt.jar 같은 핵심 Java Library Loading ( rt - runtime jar , java 핵심 라이브러리 포함 )
	- Extension Loader : 표준 핵심 Class 확장 Load ( jre/lib/ext 내 class 등등 ) - 암호화 , xml 처리 등등
	- Application Loader : Application Level 의 class Loading ( 일반적인 우리가 작성한 Class )
- 링킹 : 로딩된 Class File 을 검증하고 , JVM 내부 데이터 구조 와 올바르게 연결 , 세 가지 단계로 구성
	- Verify : ByteCode 검증기가 .class 의 바이트 코드가 정확하고 안전한지 확인
	- Prepare : 클래스 변수 ( static variable ) 메모리에 할당하고 기본값 초기화
	- Resolve : 모든 심볼릭 참조 -> 실제 참조로 변환
- 초기화 : 클래스 변수를 코드에 초기한 정의값으로 설정 ( static block 실행 )

### JVM Memory

Java Application 실행 위해 할당된 메모리 영역

#### 힙 영역
모든 객체 , 배열이 동적으로 할당되는 곳
GC에 의해 관리 , 참조되지 않는 객체들 자동 정리

##### Young Generation
- 새롭게 생성되는 객체들이 할당되는 영역 ( 대부분 객체가 해당 생명주기 부터 시작 ) , 일시적 객체가 다수
- Eden Space : 객체가 처음 생성되는 영역
- Survivor Space : Eden 에서 살아남은 객체가 이동되는 영역 ( S0 , S1 로 구성 )
- 지역 변수 , 임시 객체 , 루프 내 객체들이 생성

##### Old Generation
- Young Generation 에서 살아남은 객체들이 이동하는 영역
- 일반적으로 더 긴 생명주기 , Young 영역보다 크기가 큼
- Olg Generation 의 GC 는 Major GC ( Full GC ) 라고 불림
- 싱글톤 객체 , DB 연결 객체 , 캐싱 객체
#### 스택 영역

각 스레드 마다 , 별도 호출 스택 가진다.
메소드 호출 정보 ( 스택 프레임 - 메소드 종료시 스택에서 제거 ) , 로컬 변수 저장(?)

#### Heap VS Stack ⭐️⭐️

그러면 , 객체가 Heap 에 저장되는가 , Stack 에 저장되는가 헷갈릴 수 있다.
- Stack : Primitive Type 저장 , 참조 (주소) 저장
- Heap : 객체 자체 저장 ( 객체 멤버 변수 포함 )
#### Method Area

모든 클래스 와 인터페이스에 대한 메타데이터 저장
( 클래스 이름 , 부모 클래스 이름 , 메소드 변수 정보 )
Native Memory ( JVM 이 관리하지 않는 외부 메모리 ) 사용해 클래스 메타데이터 저장
-> 메타데이터 : 클래스 이름 , 생성자 정보 , 필드 정보 , 인터페이스 정보 등등 ( 아는 모든 것 )

#### PC Register

수행 중인 , JVM Instruction 의 주소 저장 , 스레드마다 하나씩 존재

##### Native Method Stack

Java 외 작성된 네이티브 메소드 호출 때 사용

### Execution

해당 과정에서 , Interpreter 와 JIT Compiler 를 사용한다.

JNI ( Java Native Interface ) , Native Method Library 사용
-> JNI 가 Native Method 호출하는 방법 제공

##### Why Native Mthod?
- 다른 언어로 상당히 크고 중요한 코드 작성 후 , 자바에서 동일 코드 재작성을 원하지 않을 때
- 성능향상을 위해서
- 시스템 디바이스 , 플랫폼 특정적 작업을 할 때


### Conclusion

JDK 를 사용해 , 바이트코드 (.class) 를 만들고 , 
JRE 를 사용해 , 바이트코드를 실행하고 ,
JVM 이 가동되어 바이트코드를 실질적 실행 ( 실제 OS 할당 / 회수 , 명령 호출 등 ... ) 을 담당한다.

- jdk , jre 는 플랫폼에 독립적이나 , jvm 은 플랫폼 종속적
