---
tags:
  - 자바
추가_정보: 백기선 라이브 스터디
주차: "1"
---
## JVM이란 무엇인가

- Java Virtual Machine
- Java 를 O.S 에 상관없이 실행 가능하게 해주는 프로그램
- byte 코드 ( .class ) 를 기계가 사용 가능하게 기계어로 번역해준다


## Byte Code

컴파일 과정에서 생겨난 중간 언어
특정 HW 가 아닌 가상 머신 위에서 실행되게 설계된 코드
-> H/W 에 직접적으로 종속되지 않는다

### 번외 : JVM 은 Java 언어에서만 사용되나?

X!!

Scala , Kotlin , JRUBY 등도 전부 JVM 가상 환경 안에서 작동
-> 한 프로젝트 안에서 같이 사용 가능! ( 같이 컴파일이 가능하므로 )

### Byte Can What?

ByteCode 의 활용성은 현재 개발에선 , 굳이 필요한가 ? 싶을수도 있지만 아래 와 같은 특징을 가지고 있다.

- 1. 프로그램 분석 : ByteCode 를 분석해 프로그램 구조 이해 , 흐름 분석 가능
- 2. 코드 버그 발견 : 잠재적 구조 , 안티패턴 , 비효율적 구조 식별 가능

=> 사실 , 초심자가 배울 요소는 아닌 거 같다. ( 몰라도 잘 해주기 때문 )
### Java 동작 원리

Java Application Start -> JRE 동작 -> Class Loading -> main Method 호출
