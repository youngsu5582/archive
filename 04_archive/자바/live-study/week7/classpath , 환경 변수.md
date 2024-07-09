---
tags:
  - 자바
추가_정보: 백기선 라이브 스터디
주차: "7"
---
### 클래스패스?

JVM 이 클래스 파일을 찾을 때 사용하는 경로 집합

클래스 파일은 결국 Java 소스 코드를 컴파일한 결과

JVM 이 실행할 때 해당 클래스 파일을 찾아야 함


주로 환경 변수 , 명령형 매개변수 통해 지정

### 환경 변수

#### In LINUX  / MAC

export CLASSPATH=/path/to/directory1:/path/to/directory2:/path/to/my.jar
해당 구문을 통해 경로 매핑 가능

### 명령형 매개변수

java -cp /path/to/directory1:/path/to/directory2:/path/to/my.jar com.example.MyClass
해당 구문 통해 Application 실행할 때 지정 가능

-cp 대신 --class-path 도 가능

```shell
VM(자바 실행 프로그램)이 이렇게 얻어진 byte code (.class 파일)을 실행할 때에는 컴파일 과정에서 진행한 일들을 진행하지 않고 말 그대로 실행만 한다.

때문에 소스코드보다 이해가 쉽기 때문에 속도가 더 빠를 뿐만 아니라, 매번 소스코드 문법을 검사하는 등 불필요한 작업을 생략할 수 있어서 효율적이다.
```

즉 파일 지정하면 , 일련 단계를 거치지 않고 바로 진행


### 환경 변수?

운영체제가 참조하는 변수

처음 java 를 세팅하고 실행하다보면
`javac 는 내부 또는 외부 명령 , 실행할 수 있는 프로그램이 아닙니다`
에러가 뜨는 경우가 많다

javac 라는 명령어를 운영체제가 찾게 도와주는게 환경변수이다

1. javac 입력
2. 운영체제가 javac 라는 내부 명령어가 있는지 검사
3. 없는 경우 Path 환경변수 검색
4. Path 에 설정된 경로들 모두 검사
5. javac.exe 발견하면 실행

맥북도 이와 비슷한 원리

### 클래스패스에 사용할 수 있는 값

- /export/home/username/java/classes와 같은 디렉토리
- myclasses.zip과 같은 zip 파일
- myclases.jar와 같은 jar(자바 아카이브) 파일
