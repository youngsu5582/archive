
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
