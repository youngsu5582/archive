---
tags:
  - 스프링
테크톡_링크: https://www.youtube.com/watch?v=2YbqvLuwiY8
테크톡_발표자: 박용권
---
### 서론

#### Spring 6 and Spring Boot 3 의 도입점

Java 17+ 버전 도입 , 세부 변화 , AOT 기반 테스팅이 추가됐으나 , 해당 부분은 자세히 살펴보지 않는다
발표자는 Native Image 즉 Cloud 부분에서 내용을 다룸
##### Native Executables

- Ahead-Of-Time (AOT)
- GraalVM Native Image
##### Observability

- Micrometer Observation

### Java Application startup steps

1. Application sources
2. Package application 
3. Load & Parse configuration
4. analyze dependenceis
5. build dependency tree
6. execute code

#### Traditional
- 1~2 단계 까지 Build Time
- 3~6 단계 까지 Runtime 으로 동작했다

굉장히 복잡하게 작동하여 , Memory 와 CPU 에 큰 부하를 줌
-> 빌드 타임은 짧으나 , 런타임은 김

#### Native
- 1~5 단계 까지 모두 Build Time
- 6단계 만 Runtime 동작

적은 메모리 , 적은 CPU 부하
-> 런타임도 매우 짧음

### What's Next In Spring

- Project Loom
- Project CRaC
- Project Galahad
- Project Leyden

=> 모두 Application 을 효율적이고 , 빠르게 동작하도록 만들어주는 프로젝트
( faster , smaller , more efficient )

### Cloud Native

시작은 설왕설래가 있으나 , 2010년도 즈음 Cloud 업계 단위에서 처음 언급되며 시작
> Cloud Native Application 은 특별히 설계 되야 하며 , Cloud 환경과 컴퓨팅 모델을 사용하는 속성을 가져야 한다

( 이 당시엔 , Docker 같은 클라우드 서비스 자체가 없었는데도 현재에도 접목가능한 어마어마한 용어 )

### Cloud 변천사

2006 년도 AWS 서비스 첫 출시
2008 년도 Google Cloud 출시
2010 년도 MS Azure 출시

특정 WAS Application ( Tomcat , WebLogic ) 에 배포하는게 일반적 
-> 이에 따라 코드 구현이 조금씩 달라졌음
( 물론 , Spring 단계에서 , 추상화로 이를 벗어나게 해줌 ) - 이식성( Portability ) 증가

2011 년도 MSA 라는 정의가 만들어짐

Netflix 는 2007년도 부터 클라우드 기반 이전 시작하기 함
2012년도 여러 가지 도구를 만들며 2012년오픈소스로 공개를 함

2012년도 Heroku 에서는 클라우드에서 적합한 어플리케이션 만드는 12가지의 원칙 발표 
![350](https://i.imgur.com/2q38ITD.png)

2013년도 , 2014년도 Spring MSA 기반 REST 포용하며 생상성 향상을 찾기 위해 노력함
-> 쉽고 빠르게
=> Spring IO 플랫폼을 발표

![350](https://i.imgur.com/cfYfzgM.png)

- Boot 를 통해 Rest 기반 MSA 를 임베드한 런타임에 동작하도록 도와줌
- Cloud 를 통해 Service 간 상호작용 처리하고 , 시스템 결합 대처 , 분산 시스템 공통 관리 및 운영에 도움을 줌

- Reactor 를 통해 클라우드에서 효율적으로 작동하게 도와줌 - 새로운 프로그래밍 모델
	( MSA 기반 아키텍쳐는 당연히 , 돈이 매우 많이 들어간다 )
=> WebFlux 의 핵심적인 엔진으로 작동

Docker 가 오픈 소스로 공개 - 개발 진입 장벽 을 낯줘줌
( 로컬 작동 , 데이터 센서 작동  , aws 작동 등에 연관을 받지 않음 )

AWS 람다가 출시 ( 2008 년 구글이 있긴 했으나 , 상업적은 AWS 람다 )

2014년 즈음 프로젝트 발할라 시작
- var 타입 도임
- 제네릭 타입 성능 향상 , 메모리 효율성 향상

2014년 Reactive Menifesto 라는걸 발표
-> 더 빠르고 , 일관성 있게 하려면 무언가가 필요하다!
메시징 기반 동작 어플리케이션을 어떻게 만들어야 하는지 논의해보자

2015년 즈음 , Cloud Native Enterprise 가 몰려온다...

Spring 팀 핵심 개발자들도 Cloud Native Java 책 출판하며 , 어떻게 어플리케이션을 잘 만들어야 하는지 책 작성

Google Cloud 도 , 쿠버네티스를 공개하며 CNCF 에 오픈소스로 기부!

클라우드 기반 기술들은 대부분 이 CNCF 의 소속일 가능성이 높음

2015년 클라우드 컴퓨팅을 위한 기반들은 전부 완성이 됐다

- Container
- Orchestration
- Serverless

=> Cloud Native Application 을 만들기 위한 노력이 일어남

Lift & Shift - 들어서 고스란히 옮기는 전략
그러나 , Java Application 은 문제점이 있었다.
- Start , Ramp Up Time - 구동 시간
- Footprint
- Memory limits & imperfect CPU 
=> 클라우드는 쓰는만큼 내야하는데 , 처음 과부하가 매우 비효율적
( 최대한 효율적이고 , 적게 사용해야 한다 )

2016 , 2017 년도 Spring 은 Reactive 키워드에 주목

Spring Cloud Stream 을 통해 ,
백엔드에서 고속으로 메시지 처리하는 방식으로 효율적으로 작동하게 변경
-> Reactive Driven Development

클라이언트 의 요청을 받는 부분도 , 새로운 스택을 만들어 두가지로 접근 가능하게 하자
![400](https://i.imgur.com/2wIx8Rq.png)

![400](https://i.imgur.com/rthDXeq.png)

Container 생성을 Compile 단계애 땡김
Runtime 때는 Scan 하지 않고 바로 실행
-> 메모리 사용량 저하 & 더 빨리 실행

![400](https://i.imgur.com/ap4jGAq.png)

Spring Webflux 위해 공개 ( 끔찍하게 복잡하고 어려움 )
효율적이고 굉장히 고속 동작이긴 함

새로운 프로젝트 두가지 시작
-> Project Room
-> 

Virtual Thread 를 통해 최적화
( Java 19 Preview , Java 21 도입 ) - 코루틴 , 고루틴

동시에 , 자바 라이브러리 전부 재 모듈화
-> 매우 덩어리 크고 비효율적
=> 모듈화를 하여 , 경량화 &속도 향상 - Java 8 에 도입

2018년도 즈음에는 Java를 Container 에 도입하기 위한 노력이 시도됨

- 컨테이너 상황인지 알려주는 +UseContainerSupport
- InitialRAMPercentage=75 , Min(Max)RAMPercentage 같은 메모리 퍼센테이지 제공

실행되는 시점에 , 적정한 상황을 잘 이해 하여 지시

![400](https://i.imgur.com/2DKZU9q.png)

어떤 GC인지 명확하게 지시

2018년도 Eclise 재단에 넘어며 JavaX -> Jakarta EE 로 바뀌면서
플랫폼 적이지 않고 , 현대화 & Cloud Native 적이게 바꿔 나가게 노력하겠다고 함
-> MicroProfile 이 그 산출물

2019년 즈음 GraalVM 의 정책이 출시 ( 2011 년에 나오긴 했음 )
다양한 프로그래밍 언어를 고성능으로 실행할 수 있는 런타임 플랫폼을 만들겠다고 함

![400](https://i.imgur.com/okVrglj.png)

오래된 JVM 을 , Java 단위로 전부 재구성 해나갔음

( 트위터가 VM 을 교체하며 , 10% 가량의 성능 향상 사례가 있었음 )

AOT 컴파일러는 Java 언어 ->  머신 코드로 바로 컴파일 ( 더 효율적 )