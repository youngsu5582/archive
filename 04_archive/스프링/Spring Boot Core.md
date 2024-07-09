---
tags: 스프링
---
src/main/webapp directory 사용 금지 ( application 이 JAR 로 package 되었을 때 )
=> Maven 기본 디렉토리 이나 , WAR packaging 이랑만 동작 ( JAR 로 생성시 오류 가능성 매우 높음! )
#### Templates
 전부 , resources/templates directory 에서 자동 laod
- FreeMarker
- Thymeleaf ( 현재 가장 핫함 )
- Mustache

#### Qualifier
- 여러개의 상속된 Class 중 하나를 식별할 때 사용 가능
- Coach - BaseballCoach , CriketCoach , BasketballCoach ...
- @Qualifier("baseballCoach")
	- class 랑 이름은 같게하나 , 첫 글자 소문자
	- setter 함수에서도 사용 가능

#### Primary
- 결국 , Qualifier 는 개발자가 문자열로 주입을 해야함
- Component 에 @Primary 를 통해 , 사용할 클래스임을 표기
```java
@Component
@Primary
public class TrackCoach implements Coach{
}
```
- 당연히 하나에만 적용 가능 ( Qualifier 가 더 높은 우선순위 )

#### Conclusion
- 강사는 Qualifier 추천
- 더 높은 우선순위 , 더 알아채기 쉬움

#### Lazy
- 요청이 들어올 때 , 클래스가 생성됨
##### 장점
- 필요할 때만 Object 생성
- 더 빠르게 실행 가능
##### 단점
- Controller 들도 , 요청 되기 전까진 생성 안될수도 있음
- Configuration Issue 늦게 발견 가능
- Memory 계산이 힘듬
=> 굳이 사용할 필요는 없는듯. 그냥 express.js 를 사용하는 느낌
=> Nest.js 에서도 , 그렇기에 이런 기능은 없는듯

### Bean Scope
#### 1. Singleton Scope
- 생명주기동안 하나의 빈 인스턴스를 생성
- Default Scope
- Nest.js 의 Singleton Scope 와 동일
#### 2. Prototype Scope
- 사용할 때마다 새로운 인스턴스 생성
- Nest.js 의 **Transient** Scope 와 동일
#### 3. Request Scope
- 각 HTTP 요청마다 새로운 인스턴스 생성
- Web Application 에서만 가능
- Nest.js 의 Request Scope 와 동일
#### 번외 : Request VS Prototype
##### Prototype
- 상태 유지하는 스프링 빈에 사용
- 요청 , 참조에서 독립적인 인스턴스 필요시 사용
##### Request
- 한 요청 내 상태 유지해야하는 빈에 적합
- HTTP 요청은 독립적 처리 , Request Scope 는 Request 끝날 때 까지 생성
```java
@Controller
public class MyController {
    
    @Autowired
    private RequestScopedBean requestScopedBean;

    @GetMapping("/newPage")
    public String newPageRequest() {
        // 여기서 requestScopedBean은 새로운 HTTP 요청에 대해 새로운 인스턴스를 가짐
        return "newPage";
    }
}
```
#### 4. Session Scope
- HTTP 세션 별로 새로운 인스턴스 생성
- Web Application 에서만 가능
- 각 사용자 세션 개별적 유지 , 각자 만의 인스턴스를 가짐
#### 5. Global-Session Scope
- Globla Http Session 별로 인스턴스 생성
- Web Application 에서만 가능
- 일반적으로 잘 사용 X
#### Conclusion
##### Nest.js 에선 , 왜 Session , Global Session 이 없는가?
- MSA 를 지향하는 프레임워크 상,  Stateless 상태를 준수하기 때문
 ( 사용자의 정보를 가진다는 것은 Stateless 하지 않음 )

## Bean Lifecycle
![](https://i.imgur.com/8gpbZOD.png)
1. Bean Instantiated : Container 가 시작되며 , Bean Instance 화
2. Dependencies Injected : 의존성 주입
3. Internal Spring Processing : Spring 에 의한 내부 처리
4. Custom Init Method : 사용자 정의 초기화 메소드 호출
5. Bean Is Ready For Use : Bean 이 사용될 준비 완료
6. Container Is Shutdown : Container 종료될 때
7. Custom Destory Method : 사용자 정의 종료 메소드 호출

### BeanPostProcessor
- 초기화 이전 과 이후에 커스텀 로직 실행 가능
- postProcessBeforeInitialization , postProcessAfterInitialization
#### @PostConstruct
- 초기화 후 실행 로직 설정 가능
#### PreDestory
- Bean 소멸 전 실행될 로직

=>@Bean(initMethod = "customInit", destroyMethod = "customDestroy") 이렇게도 설정 가능

### IN Nest.js?
#### OnModuleInit
- 모듈의 프로바이더들 인스턴스화 하고 모듈의 의존성이 주입된 후 호출
- Spring 의 @PostConstruct 와 유사
- 다른 모듈에 의존하고 있다면 , 해당 의존성들 모두 로드되고 나서 호출
#### OnApplicationBootstrop
- Application 이 완전히 초기화 된 후 호출
- 모든 모듈 로드 , 모든 서비스 시작한 이후 실행
#### OnModuleDestory
- 모듈이 파괴되기 전 호출
- 다른 모듈들이 의존하고 있다면 , 해당 모듈들 파괴된 후 파괴
#### OnApplicationShutdown
- Application 이 종료되기 전 호출
- k8s 의 원리와 유사
