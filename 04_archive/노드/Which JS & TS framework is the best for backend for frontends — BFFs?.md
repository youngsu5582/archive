---
tags:
  - 노드
아티클_링크: https://medium.com/@burak.bburuk/which-js-ts-framework-is-the-best-for-backend-for-frontends-bffs-b982b4ddb785
아티클_작성자: Burak Buruk
---
## 아티클 링크


## 세줄 요약

1. 프론트엔드에도 백엔드와 통신을 위한 백엔드가 존재한다 ( Backend for Frontend )
2. BFF 를 위한 다양한 프레임워크 존재
3. SSR 이 필요가 없다면 Nest.JS 도 , BFF 로 고려할만하다

## 내용

### 서론

- 현재 웹이나 모바일 애플리케이션을 가지고 있다고 상상
	-> 앱은 데이터를 가지기 위해 , 백엔드와 통신 해야함
- 각 앱마다 , 특별한 백엔드 프레임워크를 구축해 생성
	-> 이를 , Backend for Frontend ( 일명 BFF )
- 물론 , 다양한 언어에서 서버 프레임워크가 존재하나 , 여기서는 JS / TS 기반 프레임워크에 중점
### Express.js

- 서버 프레임워크의 기능을 제공하는 최소한이며 매우 유연한 프레임워크
![](https://i.imgur.com/lD7isbA.png)


#### 장점

- 노드에서 매우 잘 검증 ( 테스팅 ) 된 프레임워크
- Javscript 에 대해 친숙하면 배우기 매우 쉬움
- package.json 만 있다면 서버 바로 시작 가능

#### 단점

- 이러한 유연성 과 단순함이 단점
- 스스로 구조를 설계해야 하고 , 기능이 필요할 때마다 라이브러리 찾아서 설치
- Typescript 가 인기를 얻어가나 , express 에는 typescript 위한 내장 지원 X

### Nuxt.js

- Server Side Scripting 을 사용해 , 렌더링 된 애플리케이션 구축하고 , 인터페이스 생성에 사용되는 Vue 기반 Framework
![](https://i.imgur.com/hBBjybi.png)

#### 장점

- UI 렌더링에 강한 초점
- 강력한 라우팅 시스템을 통해 , 폴더 사용해 설정 없이 라우팅 쉽게 설정 가능
- SEO 기반 애플리케이션 개발 할 경우 , Nuxt 의 SSR 구현으로 최대한 이점 가능

#### 단점
- Vue framework 기반이므로 , Vue 에 대한 학습 시간이 소요된다
- SSR 이기 때문에 , 트래픽이 몰릴 시 서버에 부하가 많이 가고 , 네트워크 사용량에 의존
	-> 서버가 렌더링 해서 , html 로 넘겨져야 하므로 Client 에 전달하는 byte 양 증가
	( 물론 , 효율성을 위한 dynamic imports 가 있지만 주제 범위 밖 )

### Next.JS

- next 는 full-stack application 생성 가능
- 최신 리액트 기술들 포함 + Rust 기반 Javascript 도구로 빠른 빌드 기능 포함
![](https://i.imgur.com/VoC8igt.png)

#### 장점 & 단점

- React 와 함께 SSR 기능이 가능한 프레임워크
- Nuxt 와 대부분 비슷하나 , React 기반이므로 , 이것이 장점이 될수도 단점이 될수도
### Nest.JS

- 확장 가능한 서버 사이드 애플리케이션 구축 위한 완전한 개발 키드
- UI 라이브러리 / 프레임워크에 의존을 받지 않음

![](https://i.imgur.com/o49tdmE.png)

#### 장점

- Module 구조 로 , 코드르 분리된 모듈에 유기적으로 구성 가능
- 애플리케이션 구조가 아닌 , 디자연 영역에만 집중 가능
- command line interface tool 제공으로 , 보일러플레이트 쉽게 생성 가능
- 스프링 프레임워크 와 매우 유사 ( 애노테이션을 데코레이터로 구현 )
- 프론트 프레임워크에 구애받지 않음 ( express , fasity 를 기본 프레임워크 설정 - 유연성 )
- 의존성 모킹해 테스팅 작성하는게 쉬움

#### 단점

- 백엔드 서비스를 위해 구축된 광범위한 프레임워크 ( 기능 중 작은 부분만 필요하면 BFF 로는 과도함 )
- 어노테이션 , 의존성 주입에 대한 학습 곡선 필요
## 결론

### Nest.JS 를 선택
### 이유
1. SSR 이 필요하지 않기 떄문 ( routing 이나 , seo )
	-> 이는 , next 나 nuxt 에서도 많은 기능을 사용하지 않을 것
2. 특정 front framework 에 구애받지 않는 BFF 설계 원함
	-> 마이크로 프론트 내에 Vue 와 React 를 동시 사용하는데 , 자유롭게 교체 하기 위해서
3. Frontend 언어가 타입스크립트
	-> 타입스크립트 지원 해주는 프레임워크를 원함
4. 다양한 기능 ( Logging , Validation , Caching 등 ) + 편리한 cli

- 번외로 , 프로젝트에서 GraphQL 사용시 , 매우 쉽게 적용 가능

## 사담

- 아무리 그대로 , 프론트엔트 의 백엔드로 nest 를 사용하는 일은 없을 거 같다.
	-> 단순이 아닌 , 복잡한 기능을 들어갈 때 , 부족한 프론트 기능이나 장기적으로 보면 React 와 Next 가 점유할 거 같으므로 , 공부는 필수인거 같다.

- Nest 를 frontend 에서 사용한다는 생각은 참신했던 거 같다.
	-> 필자는 맨날 백엔드 용 으로만 사용했기 때문

## 참고 링크

https://medium.com/@burak.bburuk/which-js-ts-framework-is-the-best-for-backend-for-frontends-bffs-b982b4ddb785