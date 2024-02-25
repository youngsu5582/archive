
### 테크 동영상 링크

https://www.youtube.com/watch?v=mhGz8q-aOZ0

### 목차 

1. API 에 대한 과거 경험과 문제점
	a. API로부터 고통받는 순간들
	b. 실제 경험했던 API 문제점

1. API First Design 이야기 하기
	a. API First Design 정의
	b. API Firrst Designg로 문제 해결하기
	c. API First Design 더욱 잘 활용하기
	d. 아하! 이런 오해가 있었구나!
	e. 결론
## Part 1. API 에 대한 과거 경험과 문제점
### a. API로부터 고통받는 순간들

#### API 문서와 실제 동작의 불일치로 당황스러웠던 경험은?
##### In Front
- Backend 개발자에게 전달받은 API 명세가 적혀 있는 대로 동작하지 않아 당황스러운 경우
- 개발 일정에 쫓겨 API 문서를 언제쯤 받을 수 있는지 물어보는 경우
##### In Backend
- API 설계를 하기 위해 페이지를 열자 빈 페이지인 경우
	-> 어떻게 API 설계를 해야 할까?
	-> 어떤 형식으로 만들어야 할까?
	-> 참고할 만한 템플릿이 없나?

- API 코드 수정 이후 , 변경된 API 명세를 API 문서에 정확히 반영하고 있나요?
	( Parameter 수정 , 기능 수정 , Return 수정 등 ... )
- 더 이상 사용되지 않는 API , 그대로 방치 하고 있지는 않나요?
	( Deprecated ... )
### b. 실제 경험했던 API 문제점

#### 기존 개발 프로세스

![500](https://i.imgur.com/xmLE2zw.png)

1. 설계 문서를 먼저 초안으로 작성
2. 문서에 따라 코드 구현 + 어노테이션을 활용한 코드 기반 문서화 도구 이용
3. 도구로 생성된 Swagger 를 Frontend 에 전달!

#### 이런 과정에서 발견된 문제점
##### 일관성 없는 API 설계 문서로 , 일관성 있게 관리하기 위해 관리 비용 발생!
- 설계 문서 작성의 통일성 부족!
	-> 어떤 사람은 Compliance Wiki , 어떤 사람은 Excel , 어떤 사람은 Github Wiki ...
	->  스타일이 전부 다른 일관성 없는 API 설게 문서
=> API 설계 문서를 일관성 있게 관리하기 위한 관리 비용 증가!!
##### 2번에 걸쳐 작성되는 API 문서
- API 설계 문서 작성 단계에서 작성하는문서
- Frontend 에 전달되는 최종 API 문서
- 첫 번째 문서는 최종 API 문서가 만들어지면 방치 되기 십상!
=> 처음 보는 개발자들은 API 문서에 혼란을 느낀다!
##### 코드 변경사항이 최종 문서에 반영되지 않는 문제
- 문서화 도구를 이용하여 API 문서 생산!
-> 코드를 바꾸나 , 위 Valid 와 문서화 어노테이션에는 수정을 안 할수가 있다!
=> 최종 변경사항이 API 문서에 반영되지 않는 경우도 발생!! ( 결국 , Frontend 와 불필요한 소통 시간 증가 )
##### 이외 추후 발견되는 문제
- API 변경 사항을 API 명세 보다 코드 구현 작업에 중점 
	-> 서로 다른 형태 API Path 지만 , 모두 같은 기능
	( v1/users/register , api/users/create , api/users/user ... )
	-> 의미하는 바는 같지만 , 코드에 사용되는 용어가 다른 문제점 발생
	( Backend : UserDetail , Frontend : UserInfo ... )
	=> 이해관계자들 간 불필요한 의사소통 증가!

## Part 2. API First Design 이야기 하기
### a. API First Design 정의

##### 서론

계약서에 대해서 생각해보자! ( 월세 , 전세 , 근로 계약서 등등 ... )
-> 계약서는 , 반드시 명시되어 있는대로 지켜져야 한다.

API는 기계들의 대화방법! - 하나의 계약서라고 생각하자
#### API Contract
- yaml 로 정의를 해보자
```yaml
paths:
	/apis/oder:
		get:
		...

		response:
		'200':
			description:"데이터 전송 성공"
			content:
				application/json:
					example:
						id: 1L
						amount: 1000L
```

- 기준과 사항이 명확하게 작성되어 있음
=> Open API Specification ( OAS )
#### OAS
- 사람과 컴퓨터가 소스 코드 , 문서에 액세스하거나 네트워크 트래픽 검사를 통하지 않고 
	서비스 기능 발견 이해 하도록 도와주는 명세서
=> 언어에 구애 받지 않는 HTTP API 표준 인터페이스
#### 구조
![500](https://i.imgur.com/OF2LnVh.png)

- OAS 는 단순 Specificatin ( 명세 ) 의 단계
	<-> Swagger , Postman , Google-apigee 는 Implementation ( 구현 ) 의 단계

##### API First Design?
- Open API 명세 기반 API 계약서를 우선순위 1순위 로 고려해 협업하여 설계하는 것

### b. API First Design 로 문제 해결하기

#### API First Design 개발 프로세스
![](https://i.imgur.com/UfLFulh.png)

- Open API 명세서 작성할 때는 , Frontend 와 Backend 가 함께 작성
- 반복적 설계시에는 , 이해관계자도 함께 포함하여 토론 및 공유!
##### Open API 도구 활용 & 구현
###### Code Generators
- API 문서를 기반으로 , Server - Client 코드를 자동으로 생성해줌.
##### Mock Server
- 실제 API 구현되지 않은 경우에도 , Client 가 사용 가능
- API Gateway 도 가능

=> CODE First 가 아닌 API First 로 변경

#### 문제 해결하기
- API 명세 작성된 계약서 통해 문제점 해결 가능
##### 일관성 있는 API 문서
- Open API Specification 에 맞게만 작성하면 Okay!
- 필수 요소들이 있어야만 , 올바른 계약서가 된다!
=> 일관된 품질의 API 문서 유지
##### 하나의 API 문서 유지
- 처음 작성한 Open API 명세서로 모든걸 해결!
- 문서를 중복 작성할 필요 없다!
##### 코드 변경사항 과 API 문서 불일치
- Opean API 명세서를 먼저 변경
	-> 해결 가능
##### 서로 다른 API Path , 같은 기능 + 같은 기능이지만 다른 용어
- API 변경사항을 API 명세보다 , 코드 구현 작업 중점으로 인해 발생!
=> API 명세 설계에 중점 + 메시지 타입 , 문서 형식 , 용어 등 일원화
=> 이해관계자 들 간 원활한 의사소통 가능 

### c. API First Design 더욱 잘 활용하기

#### API 변경에 대한 버전 히스토리 관리를 할 수 있다

- 기존 API 설계 방식에서 변경 발생시 히스토리 관리가 어려움!
	-> 버전이 바뀜에 따라 , 번거로운 복붙 + 버전 관리
=> info 내에 version metadata 를 변경 후 VSC 통해 관리
##### 공통의 API 문서를 통해 함께 협업할 수 있다.

- 기존의 코드 기반 API 는 결국 , 동기화된 API 문서를 보장할 수 없다!
	-> 
- API 명세 확립화 통한 원활한 공유 ( One Source , Multi Using )
	- 타 서비스 개발자들도 수정된 API 계약서를 보고 수정 가능!
	=> SSOT!
##### SSOT
- Single Source Of Truth
- 서로 다른 팀 , 서로 다른 시스템 , 하나의 진실 
##### 이해관계자들과 함께 풍성한 내용이 담긴 API를 만들 수 있다.
- 가장 유용함!
- 함께 토론해가며 , 고 퀄리티의 API 문서가 완성이 된다.
	- 어떤 용어를 사용해서 표현할 것인가?
	- URL 이 내포하고 있는 의미가 적절한가?
	- 적절한 요청 / 응답의 데이터를 가지고 있는가?
=> 빠른 API 설계 피드백과 '함께 서비스를 만들어 간다!' 라는 공감대 형성
### 아하! 이런 오해가 있었구나
#### Swagger 는 API 문서화 도구이다

- Swagger 는 사실 매우 다양한 기능을 가지고 있다!
	( API Design , API Development , API Document , API Testing , API Mocking ... ) 
- OAS 도 Swagger API Specification 에서 시작!
- Swagger 의 개발자는 API 를 간단한 JSON(Yaml) 으로 표현 설계해 ,
	  API 문서화와 클라이언트 SDK 생성을 자동화 했다.
=> Swagger 가 의도한 API 문서화는 결국 , Open API 명세서 기반의 API 문서를 만드는 것을 의미한다!!
	( Annotation 기반이 아님 )
#### API First Design 은 API 를 한번에 결정하는 것이다.

- API First Design 이 처음부터 완벽한 API 명세를 만들라는 것이 아니다!
- 단계 진행 중 , 문제점이나 수정사항이 있을시 과감하게 처음으로 돌아가자!
-> 유지 관리 와 기능 개선에서 핵심적인 부분이므로 , 수정을 해도 상관이 없다
#### Codegen 은 주어진 템플릿만 사용할 수 있다.

- codegn 의 mustache 라는 Template System 을 통해 Customaize 가능!
	( 이번 내용에서는 생략 )
- Nestia 와 유사한 느낌인듯

### 결론
- 앞서 언급하는 API 의 문제들을 공감하고 있다면 , API First Design 을 통해 API 문제를 해결해보자!
#### 사담 ( 개인 의견 )

사실 , 백엔드 와 프론트엔드를 원수지게도 , 웃게 만드는 것이 API 이자 API 명세라고 생각합니다.
프론트 입장에서는 , 명세 없이 개발을 시작 한다는건 너무 막막하고 ,
백엔드 입장에서는 , 불필요한 설명을 프론트에게 하는것을 방지할 수 있습니다.
하지만 , 바빠지는 개발 일정과 잦은 수정으로 인해 API 명세는 쉽게 방치되는 경우가 잡다합니다.

이런 잘못된 명세는 모두를 민감하게 만드는 경향이 있습니다.


![500](https://i.imgur.com/f6fxQAa.png)

![500](https://i.imgur.com/ROHFzYG.png)
=> 상단에서 , 하단의 코드를 자동으로 생성이 되는게 참 매력적인거 같습니다!
( 초안이자 , Mocking Server 의 역활 + 다양한 언어 지원 자동 생성 )

다들 
단순히 Code 에서 -> Swagger 를 만드는 게 아닌
OAS ( Open Api Specification ) 에서 -> Code 를 만드는 API Design First 를 시도해 보는것도 좋은거 같습니다!

### 코드 출처

[OpenAPI Generator로 API의 안전한 Model과 정형화된 구현코드 자동생성하기](https://velog.io/@kdeun1/OpenAPI-Generator%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EC%97%AC-API%EC%99%80-%EB%8F%99%EC%9D%BC%ED%95%9C-Model%EA%B3%BC-%EC%A0%95%ED%98%95%ED%99%94%EB%90%9C-API%EC%BD%94%EB%93%9C-%EC%9E%90%EB%8F%99%EC%83%9D%EC%84%B1%ED%95%98%EA%B8%B0)

https://github.com/LenKIM/openapi-code-gen