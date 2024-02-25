## 아티클 링크
https://medium.com/naverfinancial/node-js-vs-java-spring-c4699565918e

### 세줄 요약
- Node.js 는 장점과 단점이 명확한 소프트웨어 플랫폼
- MSA 기반 및 , 서버 관리 및 배포가 용이해지며 가능성 주목
- 당장 , 판도를 바꾸진 않으나 다양하게 사용될 여지가 존재

## 내용

### 개요
- 예전에 비해 , Node.js 개발자 ( + MSA 경험자 ) 우대 채용 공고가 늘어나는 것처럼 느껴짐
- Node.js 에 대한 이야기를 해볼 예정

### Node.js
- 2009년 출시
#### Node.js 단점
- Single Thread 기반 동작
	=> 모든 요청이 하나의 Thread 로 처리 ( 서버 응답성 하락 유래 )
	=> Event Loop 가 결국 Single Thread 이기에 , 서버 성능 100% 활용 힘듬
- Javascript 라는 말도 안되는 언어 사용
	=> 언어적 자체 결함 ( this keyword 의 난해 , 혼잡한 scope 개념 )
	=> 단순 스크립트 언어에서 시작했기에 , 부자연스러운 부분
		( class 가 사실 실체가 없는 syntax sugar , )
	=> 이해하기 어려운 개념 존재 ( closure , prototype chain 등 )
	=> 런타임 전 , 잡을수 없는 에러

#### Node.js 장점
- Single Thread 기반 동작 ( 장점이자 , 단점의 부분 )
	=> 기존 서버는 대규모 트래픽을 받기 위해서만 존재 하였으나 , 작은 서버를 여러대 사용해 거대한 서버로 대체하는 MSA 구조 및 서버 배포 & 관리 지원해주는 k8s ( kubernetes ) 등장에 따라 , node.js 의 장점 부각
	=> 매우 짧은 서버 가동 시간 ( 갑작스러운 트래픽 즉각 대응 유리 )
	=> 하나의 Thread 이기에 , 떨어진 응답성 도 여러대의 서버로 무마 가능 
	=> Single Thread 이기에 , Thread 간 동시성 ( concurrency ) 을 고려할 필요 X 
- Typescript 의 등장
	=> 기존 , any type 이 아닌 , type 을 강제
- Nest Framework 등장
	=> Spring 에서 제공 JPA 이나 Hibernate 와 같은 기능 기대 가능
- Good Quality 의 Testing Tool 등장
	=> Jest , Mocha 등장으로 , DI 나 IoC 구조 제공 장점인 Unit Test 도 편리하게 구현 가능

### 결론
- 지금 당장 , Java Spring -> Node.js 를 의미하는 것은 아님
	=> Spring 은 , 10여년 간 , 현장에서 안전성과 성능을 검증 받아 왔기 때문
	=> 특히 , 대기업 과 같은 매우 큰 규모의 서버는 마이그레이션 역시 매우 힘듬
	=> 차후 , 개발 트렌드를 주도할 가능성이 존재 하다는 의의

- Naver 페이플랫폼 팀은 다양한 서버에서 node.js 도입 시도
	- Naver 쇼핑 구매 목록 API 제공 Timeline Server
	- 카드 정보 API 제공 간편 입력 Server
	- 마이데이터 API 제공 마이데이터 서버
	- 혜택 광고 API 제공 리워드 서버
- 대규모 CPU 연산 요구하는 신규 DB 구축 마이그레이션 작업 역시 , MSA 기반 node.js 서버로 처리해 대규모 CPU 연산 작업 역시 가능하다고 검증
