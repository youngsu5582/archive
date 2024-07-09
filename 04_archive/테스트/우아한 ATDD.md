---
tags:
  - 테스트
테크톡_링크: https://www.youtube.com/watch?v=ITVpmjM4mUE&t=2548s
테크톡_발표자: 브라운(류성현)
---
### ATDD??

- Acceptance Test


### 새로운 개발 문화

화자는 우테코 코치로 오며 새로운 문화를 접함

- 페어 프로그래밍
- 테스트 작성 권장
- 인수 테스트

#### 페어 프로그래밍

기술적인 부족함이 있어도 , 빠르게 익혀나갈수 있었음
-> 계속 맨날 , 할수는 없다! ( 페어도 사람이야 사람! )

### 테스트

- 테스트 코드로부터 받는 피드백 + 심리적 안정감 역시 제공

점심 처럼 노곤한 상황에도 , 테스트 코드는 졸지 않는다!
( 획일적이게 검수 해줌 )

### 인수 테스트

- 시나리오 기반으로 , 기능 테스트
![600](https://i.imgur.com/nLoljSg.png)


> 실제 요청을 날려서

- 배포없이 테스트로 , 대부분 검증 가능
	-> 빠르게 확인 가능
- 인수 테스트로 , 스펙 표현 가능
- 새로운 팀의 도메인 과 흐름 파악에 큰 도움을 받음 + 도메인 이해에 짧은 시간 소요

---

So What?

### 빠른 피드백

- 위 문화들은 빠른 피드백을 받을 수 있다
- 든든한 지원군이 코딩을 도와주는 느낌을 준다
- 자동화된 테스트 자신감을 얻을 수 있다

#### 피드백을 받는 방법
- 페어 프로그래밍
- 테스트 / 인수 테스트
- 코드 리뷰
- 배포
- 출시

### 테스트 주도 개발

- Test Driven Development
-> 결국 이 빠른 피드백을 받게 도와준다


![450](https://i.imgur.com/mwErYZf.png)

실패 테스트 작성 -> 통과하게 작성 -> 리팩토링 의 굴레

- 테스트를 설계 활동으로 바꾸는 효과도 있음
- 이로 인해 설계 품질에 관한 피드백을 빠르게 받게 도와줌

#### 아쉬운 점이라면?

- 어떻게 시작해야 하고 , 언제 끝내야 하는지?
- 각 단위들이 잘 통합이 된건지?

### 인수 테스트 등장

TDD 의 아쉬운 점을 보완해준다

우선 인수 테스트 작성후 , TDD Cycle 로 넘어간다.

![450](https://i.imgur.com/TXWQVgm.png)

#### 인수 테스트 기반 개발을 할 경우?

- 빠른 피드백을 받을 수 있음
- 회귀 오류 잡아줄 꾸준한 테스트를 만들 수 있음
- 기존 기능 망가뜨리지 않고 , 새 기능 추가 가능

- 인수 테스트 작성하며 구현할 대상 대한 이해도 증진
- 작업 시작과 끝이 명확해져 심리적 안정감에 도움을 줌

추가로 , Issue 를 작성할 때 명확하게 작성 가능

### ATDD 란

차근 차근 용어부터 설명해나갈 예정


#### 번외 : 테스트 VS TDD

일반적인 테스트는?
=> 구현 후 테스트를 통해 : 검증

TDD 는?
=> 요구 사항을 테스트로 작성 후 구현 + @

:) 하루를 시작할 때 먼저 계획을 세우고 사는게 TDD

#### 번외 : TDD VS BDD

- TDD 는 Test Driven Development
- BDD 는 Behavior Driven Development

BDD 는 TDD 를 잘 하고 잘 설명하기 위해 나온 개념

검증의 의미로 사용이 되다 보니 , 오해와 생각이 다를 수 있었음
=> 행위에 대해서 작성을 하자

OutSide -> Inside 단위로 , 코드를 구현해나감

#### TDD VS BDD VS ATDD

![250](https://i.imgur.com/VCafIRQ.png)

![250](https://i.imgur.com/LGrhB4U.png)

![250](https://i.imgur.com/k26yO2i.png)

BDD 도 Given - When - Then 으로 작성
ATDD 도 Given - When - Then 으로 작성

ATDD 는 TDD 처럼 꼭 개발 관련 언어는 아님

- 애자일 기법에서 파생된 개발 방법론
( 비개발자들도 포괄적으로 참여하기 위해 나온 방법 )


인수 조건 -> 인수 테스트 -> 구현 으로 동작

#### 왜 다 같이 인수 조건을 정의 하나요?

각 부서가 생각하는게 다를 수가 있으므로

기획은 기획한 대로 , 개발은 개발한 대로
=> 서로 다른 이해관계를 가지고 진행이 된다
( 개발은 산으로~ 기획은 바다로~ )

### ATDD 개발 프로세스

- 인수 조건 정의
- 인수 테스트 작성
- 기능 구현

### 인수 조건

- 인수 테스트가 통과해야 하는 조건
- 인수 조건 표현하는 여러가지 포맷이 있음
-> 시나리오 기반 표현 방식 통해 설명

#### Given

- 강사는 강의를 생성
- 강사는 강의를 신청 가능 상태로 변경
- 강의 모집인원 만큼 신청을 받았다
#### When

- 회원이 수강 대기 신청 요청
#### Then

- 회원은 강의 수강 대기자로 등록 되었다

#### 개발 과정 예시 - 인수 테스트

인수 조건 검증하는 테스트
실제 요청 / 응답 환경과 유사하게 테스트 환경 구성

![500](https://i.imgur.com/ccsRjoS.png)

#### 개발 과정 예시 - 기능 구현

- 인수 테스트 충족 위한 코드 구현
- 기능 구현 TDD 로 진행할 수 있음


## ATDD 관련 고민한 사항들

### 인수 테스트 유래

- 인수를 받을 때 검증하며 확인하는 것에서 유래
- 인수 조건 - 인수 테스트로 구성
- given - when - then 으로 작성
#### Example

요구사항 ( 사용자 스토리 ) 
강사는 
강의료를 환불해주기 위해 
수강생의 수강을 취소할 수 있다

#### 인수 조건 작성

- 검증 하고자 하는 when 구문 먼저 작성
- 기대 결과 의미하는 then 구문 먼저 작성
- when 과 then 에서 필요한 정보를 given 통해 마련

##### Given

수강생이 수강 신청을 하였다.
과정의 남은 기간이 절반 이상이다.

##### When

강사는 특정 수강생의 수강 상태를 취소 요청을 한다.

##### Then

특정 수강생의 수강 상태가 취소 된다.
특정 수강생의 결제 내역이 환불 된다.

### 이슈 관리

![450](https://i.imgur.com/oXLHCql.png)

테스트 용 컨텍스트가 존재하기에
- 상황마다 다르게도 가능
( 상황에 따라 필요할때마다 리팩토링해서 구현 )

서비스로 생성 하는것도 낫배드 - 

구현이 어려우면? -> 모킹으로도 낫배드 ( 나중에 교체 가능 )

모든걸 명시하는것도 낫배드 안 명시하는 것도 낫배드

인수 조건 역시도 요구 사항 명세처럼 비개발자의 시선에서도 명시




- Given - When - Then 형식으로 이슈 작성
![450](https://i.imgur.com/ocPFHd8.png)

- 계속해서 수정해가며 보완해나감

#### 인수 테스트 특징

##### Black - Box Test

내부 구조나 작동과 연관이 없는 테스트
-> 내부 상황을 고려하지 않는다.

##### UI 레벨 대신 API 레벨 인수 테스트

- 백엔드 개발 입장에서 공수가 너무 많이 든다고 판단 ( +깨지기 쉬워질 수 밖에 없음 )
	-> UI 레벨이 아닌 API 레벨에서 인수 테스트 작성

---



Spring 단위에서 테스트

```java
@Target({ElementType.TYPE})  
@Retention(RetentionPolicy.RUNTIME)  
@Documented  
@Inherited  
@BootstrapWith(SpringBootTestContextBootstrapper.class)  
@ExtendWith(SpringExtension.class)  
public @interface SpringBootTest {
	...
}
```

SpringBootTest 의 Docs

##### Target({ElementType.TYPE})
어노테이션이 타입 레벨에서 사용 - 클래스 , 인터페이스 , 열거형에 사용 가능
##### Retention(RetentionPolicy.RUNTIME)
어노테이션 생명 주기 지정 - 런타임에 유지 되고 , 실행 시 반영 되고 , 리플렉션 통해 접근 가능
##### Documented
어노테이션 정보 문서화 - API 문서에 나타나게 선언
##### Inherited
해당 클래스를 상속하는 클래스가 어노테이션 정보 상속 - 서브 클래스도 같은 테스트 구성 상속
##### BootstrapWith(SpringBootTestContextBootstrapper.class)
Application Context 를 어떻게 Bootstrap(초기화) 하는지 알려준다
##### ExtendWith(SpringExtension.class)
JUnit 5 , Test Engiene 을 사용할 때 선언
Test 에서 Spring 의 기능 ( Bean , D.I ) 을 사용하게 해줌

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)  
public class NsAcceptanceTest {  
    @LocalServerPort  
    private Integer serverPort = 0;  
    @BeforeEach  
    void setup(){  
    }  
}
```

- SpringBootTest 를 Annotation 하여 , 구현
### SpringBootTest - webEnvironment

- 총 4가지 설정
여기서는 RANDOM_PORT 설정
	-> 실제 웹 환경과 유사하게 구현하기 위해 random
- random_port , define_port 는 실제 웹 환경 구성
( tomcat , webt - web flux ... )
- mock 이랑 설정하면 모킹된 서버 구성

#### 테스트 객체

테스트를 위한 서버에 요청 보내기 위한 클라이언트 객체 설정
EX) MockMVC , RestAssured , WebTestClient

#### MockMVC vs WebTestClient vs RestAssured

##### MockMVC

- @SpringBootTest 의 webEnvironment.MOCK 과 함께 사용
- mocking 된 web 환경에서 테스트
- 당연히 속도가 빠름
- 근데 , 일일히 찾아서 주입해야 할 수도 있음
##### WebTestClient

- @SpringBootTest 의 webEnvironment.RANDOM_PORT 나 DEFINED_PORT 와 함께 사용
- Netty 를 기본으로 사용

##### RestAssured
우테코는 이걸 사용

- 실제 web 환경에서 테스트
- Apache Tomcat 을 사용

### Example

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)  
public class SampleTest {  
    @LocalServerPort  
    private Integer serverPort = 0;  
  
    @BeforeEach  
    void setup() {  
        RestAssured.port = serverPort;  
    }  
  
    Response 수강_대기_등록_요청(String sessionId,  
                         String name,  
                         String email  
    ) {  
        var createView = SessionWaitingCreateView(name, email);  
        return RestAssured  
                .given()  
                .accept(ContentType.JSON)  
                .body(createView)  
                .when()  
                .post("$SESSION_BASE_URL/$sessionId/waitings");  
    }  
    private String SessionWaitingCreateView(String name, String email) {  
        return "body";  
    }  
}
```

- 해당 동작은 무작위 PORT 에서 수행
- Response 를 통해 , 다른 인수 테스트에서 검증 및 사용

```java
@DisplayName("강의 추가 시 기존 렉쳐 parentId가 같아야 한다.")  
@Test  
void '강의 추가 시 렉처 복사'  
  
{  
    // given  
    var sessionResponse = 강의_생성되어_있음();  
    var lectureView = 랙처_생성_요청(sessionResponse, "단위 테스트 , 코드 리뷰 통한 개선");  
  
    // when  
    var sessionView2 = 강의_생성_요청(sessionResponse);  
    var sessionView3 = 강의_생성_요청(sessionResponse);  
  
    // then  
    렉처_부모_아이디_확인(lectureView, sessionView2, sessionView3);  
}
```

이렇게 , 결과를 수행하고 Response 를 통해 추가적인 연산 작업

=> 만약 , 이런 테스트들이 다른 테스트에 영향을 주는 경우라면?

#### DirtiesContext

- 스프링 테스트 환경에서 캐싱된 Context 를 사용하지 않게 설정
- 매번 Context 를 구성 하기에 시간은 많이 걸림

### 일관적 데이터 초기화

작업을 하면 , 필연적으로 DB 연산을 하게 된다.
-> 다른 테스트에 영향을 줄 수 있다

- EntityManager 를 활용해 테이블 이름 조회
- 각 테이블에 Truncate 수행
- ID Auto Increment 값 초기화

```java
@Service  
@ActiveProfiles("test")  
public class DatabaseCleanUp implements InitializingBean {  
    @PersistenceContext  
    private EntityManager entityManager;  
  
    private List<String> tableNames;  
  
    @Override  
    public void afterPropertiesSet() {  
        try {  
            System.out.println(entityManager.getMetamodel());  
            System.out.println(entityManager.getMetamodel().getEntities());  
            tableNames = entityManager.getMetamodel().getEntities().stream()  
                    .filter(e -> e.getJavaType().getAnnotation(Entity.class) != null)  
                    .map(e -> CaseFormat.UPPER_CAMEL.to(CaseFormat.LOWER_UNDERSCORE, e.getName()))  
                    .toList();  
        } catch (Exception e) {  
            System.out.println(e.getMessage());  
        }  
    }  
  
    @Transactional  
    public void execute() {  
        entityManager.flush();  
        //h2 -> mysql -> postgresql  
//            entityManager.createNativeQuery("SET REFERENTIAL_INTEGRITY FALSE").executeUpdate();  
        //entityManager.createNativeQuery("SET FOREIGN_KEY_CHECKS = 0;").executeUpdate();        entityManager.createNativeQuery("SET CONSTRAINTS ALL DEFERRED;").executeUpdate();  
  
        for (String tableName : tableNames) {  
            entityManager.createNativeQuery("TRUNCATE TABLE " + tableName).executeUpdate();  
  
            //entityManager.createNativeQuery("ALTER TABLE " + tableName + " ALTER COLUMN ID RESTART WITH 1").executeUpdate();  
            //entityManager.createNativeQuery("ALTER TABLE " + tableName+" AUTO_INCREMENT = 1").executeUpdate();            entityManager.createNativeQuery("ALTER SEQUENCE " + tableName + "_seq RESTART WITH 1;").executeUpdate();  
        }  
//            entityManager.createNativeQuery("SET REFERENTIAL_INTEGRITY TRUE").executeUpdate();  
//            entityManager.createNativeQuery("SET FOREIGN_KEY_CHECKS = 1;").executeUpdate();  
        entityManager.createNativeQuery("SET CONSTRAINTS ALL IMMEDIATE;").executeUpdate();  
    }  
}
```

- 내가 코드를 본 후 , h2 / mysql / postgresql 용 으로 추가 작성했다


### For Good Quality
#### 테스트 코드 가독성의 중요성?

당연하게도 , 가독성이 좋지 않으면 방치될 가능성이 높음

```java
152 :: void 전체_프로세스_인수_테스트(){
	...
404 :: }
```
@DisplayName 도 없는데 200줄 씩 되는 경우?
-> 읽기 당연히 싫어진다

#### 다른 인수 테스트에서 사용해야 한다면?

재사용을 위해서 별도 클래스로 분리

```java
public CourseView createCourseView() {
        RequestSpecification request = givenInstructor();
        CourseCreateView body = createCourseCreateView();

        return getCourse(
            createCourseLocation(request, body)
        );
}
```

계속 함수로 파고 들어가는 것도 좋은 방법

#### 메서드 명이 영어일 필요는 없자나?

우리가 아무리 영어를 잘하고 , 똑똑하더라도
팀원 과 협업 + 한눈에 이해하려면 한글이 최고
```java
Response 수강_대기_등록_요청(String sessionId,  
                     String name,  
                     String email  
) {  
    var createView = SessionWaitingCreateView(name, email);  
    return RestAssured  
            .given()  
            .accept(ContentType.JSON)  
            .body(createView)  
            .when()  
            .post("$SESSION_BASE_URL/$sessionId/waitings");  
}
```

이렇게 한글로 드러내주자


#### 중복 검증이 있다면?

다른 곳에서 , 이미 검증한 내용이 있다면?
( 수강 신청 로직 , 수강 신청 로직을 포함한 모집 마감후 수강 신청 )

=> 그냥 하나를 더 만들자

```java
private SessionView 유료_강의_수강_신청되어있음(String userId){
	var sessionResponse = 강의_생성_유료_모집중(1);
	유료_강의_수강_신청(userId,sessionResponse);
	강의_수강_신청됨(userId,sessionResponse);
	return sessionResponse;
}
```

- 물론 , 중복 코드가 발생할 수 있으나 , 인수 테스트에서는 가독성과 로직을 드러내는게 Key Point!
#### 외부 API 호출을 해야하면??

##### Github?
- 테스트 계정으로 실제 요청을 하자 ( 대기 시간 필요 시 , Delay 처리 )
##### 결제?
- 실제 서버로가 아닌 , Fake 서버로 요청하게 조정

### 작성 팁

#### 간단한 성공 케이스 부터 우선 작성하자

- 동작 가능한 가장 간단한 성공 케이스부터 시작
- 테스트가 동작하고 , 수정해나가면 더 좋은 생각이 떠오를 수 있음
( 나쁘지 않은게 매개 변수 타입이나 , 리턴 타입을 생각할 수 있게 되므로 깔끔해져나감 )

#### 인수 테스트 클래스 묶기
- 같은 테스트 Fixture 를 공유하는 인수 테스트를 클래스 단위로 묶자

#### Live Templates 를 활용하자
- 불필요한 중복 코드는 미리 작성하자
```java

```
#### 도메인 부터 TDD 를 이끌어가자

- 인수 테스트 통해 시나리오 및 전반적 기능 이해 했다면
- 핵심 기능 대한 도메인 설계 후 , 도메인 객체 대한 TDD 수행

### 추천 흐름

- Top-Down 으로 방향을 잡고 , Bottom-Up 으로 구현을 해나가자
- 인수 테스트 작성 통해 요구사항 & 기능 전반 대한 이해 선행
- 내부 구현 대한 설계 흐름 구상
- 설계가 끝나면 도메인부터 차근차근 TDD로 기능 구현
- 만약 도메인이 복잡하거나 , 설계가 어려울 경우 이해하고 있는 부분 먼저 기능 구현
- 인수 테스트 요청 처리하는 부분부터 진행 가능

### 성공적인 ATDD 적응을 위해

혼자서 , 토이 프로젝트에서 경험을 쌓고 , 간단한 기능부터 적용
	+ 상황에 맞는 방법을 찾자
=>
함께 , 개발하며 프로세스 룰을 정해나가자 ( 인수 조건 , 포맷 등 )
팀 회의와 회고를 통한 피드백
살아있는 규칙 정하기

- 아주 쉽게 시작할 수 있는 부분부터 도입
- 조금씩 상황에 맞게 조정
- 가벼운 프로세스 부터 시작 , 문제점 발견 시 즉각 민감하게 대응

기획 / QA / 개발자가 함께 인수 조건 회의를 참여해가며 , 인수 조건을 만들어 나가자


##### 장점?
- Common Understanding
	- 다른 포지션의 관점은 물론 , 업무 프로세스도 간접적으로 익힐 수 있음
	- 다른 포지션 진행 상황에 대한 인지 & 이해도 높아짐
##### 단점?
- 인수 조건 정의 어려움 ( 문서 어떻게 관리할 지 고민을 요구함 )
- 리소스 ( 기획 과 개발이 서로 두번 작업하는 느낌을 받을 수 있음 )

---

E2E 연습부터

사용자 기능 명세

테스트 만드는거부터 명세


예약 하는 프로세스
각 단계별로 call 하는 메소드

방탈출 대기 플로우

로그인 회원가입
-> 되있다고 가정

테스트가 중복 제거 & 가독성 둘다 중요

- 가독성이 더 중요

- 테스트간 격리

하나의 DB를 공유한다면
무조건 초기화

처음부터 시작하면 너무 방대하다

API CALL 을 테스트

호출 -> 실제 생성 확인

다른 계층도 어차피 테스트 할꺼니까 상관없다
응답 코드만 확인하는 것도 Not Bad

=> 적절선을 세우자
( 자기가 검증하고 싶은거를 검증 ) - 정답은 없다

---

자기가 할 수 있는 쉬운거부터 테스트를 하라

E2E 는 쉬운거부터 해나가자 - Request / Resopnse 만 고려하자
-> 애초에 RestAssured 로만
실제 call 은 staging 서버에 호출할 수도 있음

중복 제거 은 역시 자신의 취향 - 테스트의 정도는 역시 없기에

=> 그냥 시도를 해나가자
 ( 샌드 박스 )

문제를 마주해서 고쳐나가자
- 미션의 의의는 아름다운 코드가 아닌 그냥 삽질들을 중요하다

TDD 는 결국 미시적인 부분 - 너무 짧게 쪼개다 보니까
ATDD 는 애초에 다른 별개가 아닌 TDD 를 하게 해주는 등대

마스터한다의 느낌이 아닌 필요하니까 배운다
- 그 순간 중요한거를 생각하자
