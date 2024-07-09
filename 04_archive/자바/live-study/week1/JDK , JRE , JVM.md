---
tags:
  - 자바
추가_정보: 백기선 라이브 스터디
주차: "1"
---
![500](https://i.imgur.com/YI53H5y.png)

해당 사진으로 설명 가능

JDK -> JRE -> JVM

#### JDK

- Java Development Kit
- Java Program 개발할 때 필요한 Software 개발 환경

- javac : Java Compiler , 자바 코드 -> 바이트 코드
- javap : Java Disassembler , 바이트 코드 -> 사람이 읽을 수 코드 ( 자바 코드 X )
- jar : Java Archive , 여러 Java Class File 과 Metadata 과 Resource 를 하나로 압축한 파일
- jdb : Java Debugger , Java Program 을 대화형으로 디버깅 가능하게 해주는 도구
- jdeps : Java Dependency Analyzer , 자바 클래스 파일 & JAR 의존성 분석에 사용

#### JRE

- Java Runtime Environment
- 자바 프로그램이 실행되기 위한 환경

- java : Java , Application 시작하는 Command Line 실행 도구
- javaw : javaw , java 와 유사하나 GUI 기반 애플리케이션에서 사용 ( 콘솔 출력 필요 없을 때 사용 )
- Class Loader : JVM 이 프로그램 실행 중 Class 를 동적으로 Load 해주는 도구

- Java API : 표준 Java Library , Interface 포함 ( 실행할 때 필요한 요소 제공 )
- Native Library : Flatform 에 특정 기능 제공 Library ( JVM 이 특정 시스템 콜 & 기능 사용하게 해줌 )
- Runtime Environment : 애플리케이션에 실행될 때 필요한 환경 ( 메모리 관리 , GC , 보안 관리 등 설정 포함 )
#### JVM

- Java Virtual Machine
- Java Byte Code 를 실행하는 Virtual Machine
- OS 에 독립적으로 실행되는 추상층

- Interpreter : Byte Code 를 한 줄씩 읽고 실행하는 요소
	-> 즉시 실행 : 프로그램 시작할 때 즉시 코드 실행 ( 기계어로 변환 X - 바로 실행 )
	-> 간단 구현 : 간단하게 구현 가능 , 새로운 & 변경 되는 코드 빠르게 실행할 때 유리
- JIT Compiler : 프로그램 실행 속도 향상 위해 소스코드를 바로 컴퓨터가 읽을 수 있는 기계어로 변환
- JVM Memory , Class Loader , Execution Engine
