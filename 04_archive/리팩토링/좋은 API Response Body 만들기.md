---
tags:
  - 리팩토링
아티클_링크: https://jojoldu.tistory.com/720
아티클_작성자: 향로
추가_정보: 개인_블로그
---
### 서론 
기존 SSR ( jQuery + Server Template ) 시스템을 -> API 기반 신규 전환 작업을 하며 ,
Response Body 관해 종종 코멘트를 한다.

#### 들어가기 앞서 : SSR?
- Server Side Lendering
- <-> CSR ( Client Side Rendering ) - React 는 기본적으로 SPA + CSR
	( CSR 은 클라이언트가 처음 HTML , CSS 를 받을때는 빈 화면만 보이고 , 전부 구성 까지 화면을 대기해야함 )
=> 사실 , 이런 생각이 당연할수도 있다고 생각 ( 서버는 데이터를 보내주고 , 클라이언트는 화면을 구성해나갔으니 )

- HTML , CSS 에 원하는 웹 페이작 만들어 놓고 , JS 는 네트워크 작업 & DOM 조작만 하면 되는거 아닌가?
	-> SSR 발전의 초석
##### SSR 순서
1. 주어진 정보를 가지고 , 서버에서 HTML 파일을 어느정도 구성
2. 클라이언트는 요청 따라 웹서버로부터 HTML 을 받음
3. 클라이언트는 HTML 에 들어있는 내용을 우선 보여주고 필요한 JS 를 가져온다.
4. 가져온 Javscript 와 HTML 을 매칭 ( Hydration )
5. Hydration 과정과 Javacsript 가 해줘야 하는 렌더링 까지 마치면 , 비로소 유저는 완성된 화면을 본다.

- 검색 엔진 최적화 ( SEO - Search Engiene Optimization ) 해결
	-> CSR 은 HTML 내용이 비어있으므로 , Crawler Bot 이 수집할 내용이 없음 ( SSR 은 HTML 집어 넣어 주므로 해결 )

API 역시 , 사용자 인터페이스다! - 프론트엔드 개발자 or 다른 서버 백엔드 개발자들이 사용할 수 있으니까!
인터페이스 라는 측면을 무시하고 , 자신이 편한 방법 or 기준 없이 API Body 를 만들다 보면 정작 사용하는 쪽에서 불퍈할 때가 많다.
이제 , 선호하는 API Response Body 에 대한 설명을 해볼 예정이다.
( 물론 , 사람마다 다양한 관점이 있을수 있음 )
### YAGNI
#### 사전적 의미
- You aren't gonna need it
- 프로그래머가 필요하다고 간주할 때까지 불필요한 기능을 추가하지 않는 것이 좋다!
- 익스트림 프로그래밍 ( XP ) 의 원칙
- 가능한 API 의 크기를 최소화 하자! ( 사담 : 전부 다 용량으로 되기 때문 아닐까? )
	-> API 에 신규 필드 ( 기능 ) 을 넣는 것은 전혀 어렵지 않다.
		( 기존 사용자들에 영향을 끼치지 않으며 추가 가능 )
	-> 이미 존재하는 필드 수정 / 삭제 하는 것이 너무나 어려움 & 버저닝 등 신경 써야할 일 들이 많음
	( 사담 : 팀 프로젝트를 하며도 느낌 , Swagger 바꾸고 , 프론트 알려주고 , 코드 바꾸는 등 일의 증가 )

- API 를 만들때는 현재 필요한 것들만 포함
	-> 항상 필요 스펙에 한해서만 추가 , 확실하지 않은 필드를 미리 추가하지 않는다.
	-> 사용하지 않는 필드가 확실하면 , 해당 필드는 꼭 삭제! ( 사담 : 이것도 진짜 필수인듯 , 프론트의 혼란성 유래 + 본인의 기억력에 의존 )
### calmeCase
```typescript
//bad
{
	group_name :'inflab'
}
// good
{
	groupName : 'inflab'
}
```
- 매우 많은 언어에서 camelCase 는 관례처럼 사용
	-> camelCase 사용시 , 다양한 플랫폼 & 언어에서 일관성 유지 가능
- OAS ( Open API Specification ) 스펙에서 자동 코드 생성하는 도구 codegen 도 camelCase 에 최적화
- 물론 , 내부 시스템 간 API 통신 + codegen 등 도구를 쓰지 않을거라면 사내 기준에 맞추어도 OK
### 표준 포맷 사용하기
```typescript
// bad
{
    createdAt: 1423314000
}

// good
{
    createdAt: '2015-02-07T13:00:00'
}
```
- API 데이터는 누가 봐도 직관적으로 , 무슨 데이터인지 이해할 수 있는 형태여야 한다.
- 특별한 이유가 없다면 표준을 사용하자
	-> UNIX 타임스탬프를 사용한다면? : 1423314000 과 같은 값이 무슨 뜻인지 바로 알아채기 매우 어려움
- 날짜 데이터는 RFC 3339 or ISO-8601 등 정의된 날짜 및 시간 형식을 사용하자
	- ISO 8601 : 날짜 , 시간 및 시간 간격 나타내는 다양한 형식 국제 표준 
	- RFC 3339 : ISO 8601 의 확장 버전 , 인터넷에서 날짜와 시간을 전송하는데 사용
### Boolean 타입에는 null 이 있으면 안된다.
```typescript
// bad
{
    isPassed: null // or true or false
}

// good
{
    passStatus: 'READY' // PASS or FAIL
}
```
- Boolean 은 기본적으로 참 과 거짓 두 값을 가지고 있는 Enum과 다를바 없다.
- Boolean 의 null 이 의미가 있다면 , Enum 으로 대체하는것이 좋다.
EX ) null , true , false 에 의미가 있다면 -> YES , NO , UNKNOWN 등으로 표한하는 것도 좋다.
( 사담 : javascript 라는 언어의 특성상 , null이 들어갈 수도 있다. 해당 경우를 최대한 처리하고 의미있게 사용하자 )

### 제한된 문자열 값은 열거형 ( enum ) 으로 표현한다.
```typescript
enum UserRole{
	ADMIN = 'ADMIN',
	MANAGER = 'MANAGER',
	USER = 'USER'
}

const user.role = UserRole.ADMIN
```

- Enum 은 제한된 문자열 값의 집합 ( 무조건 , 그 중에서 하나가 나와야 함 )
- ADMIN , MANAGER , USER 같이 제한된 문자열 값 있는 경우 Enum 으로 표현하는 것이 좋음
	-> 단순 문자열 표현 경우 , 무슨 값이든 나올수 있으므로 사용자 측에서 값을 예상하기 힘들다!!
- API 문서에서도 , 역시 제한된 값 외 없음을 표현해줘야 한다.
### 열거형은 문자열로 표현되어야 한다.
``` typescript
// bad
{
    passStatus: 1
}

// good
{
    passStatus: 'READY'
}
```
- Enum 은 기본적으로 ordinal 값을 가지고 있으며 , oridnal 은 숫자값이다
	-> Enum 의 순서를 의미하며 , Enum 의 순서가 바뀌면 oridnal 값도 바뀐다.
- Enum 을 사용할때는 항상 문자열 값을 사용하자!
#### 사담 : 금액을 return 해줘야 하는 경우에는?
- 개인적인 궁금함으로는 , 금액들이 모여있는 Enum 의 경우에는 숫자를 사용하는 것이 더 좋지 않나 생각이 든다.
	-> 이유로는 불필요한 형변환을 유발하기 때문이다.
```typescript
enum Grade {
	First = "1000000",
	Second = "100000",
	Third = "5000"
}

const price = Number(Grade.Second);
console.log(price+5000);
```
##### In GPT
	그러면 , 위에 내가 준 enum 의 경우에도 First : "1000000" 이런식으로 하라는 거야?
	
	ChatGPT
	네, 일반적으로 Enum 값이 숫자를 표현할 때도 숫자 대신 문자열로 표현하는 것이 권장됩니다. Enum은 주로 서로 다른 상태 또는 타입을 나타내는 데 사용되며, 숫자값을 사용하면 가독성과 유지보수성이 떨어질 수 있습니다. 따라서 First, Second, Third와 같은 Enum 값도 문자열로 표현하여 사용하는 것이 좋습니다.
=> 해당 부분은 납득하기 힘든 거 같다.

### 복수형 빈 값은 빈 배열로
``` typescript
// bad
{
    orders: null,
}

// good
{
    orders: [],
}
```
- Null 다루는 방법으로는 Null Object Pattern 이 있다.
- Null 대신에 대체 객체를 사용하는 패턴
	-> ORM 들을 보면 , 조회할 때 값이 없으면 빈 배열 반환
	-> 복수형 조회시 , Null 과 Undefined 문제에 대응할 필요가 없어진다!
	=> 복수형 필드는 조회 결과가 없을 시 , `[]` 을 사용하자!!
( 사담 : 해당 부분이 중요하다고 생각하는게 , ! 과 ? 를 사용하면 Typescript 를 사용하는 의미가 어쩔수 없이 퇴색되는거 같다. )

- 복수형은 빈 배열을 사용하나 , 단일형은 Null이 유의미한 경우도 있을수 있으므로 상황에 따라 선택한다.

### 일관성 유지하기

- API 필드명은 예측 가능 해야한다.
	-> 그러므로 , 같은 뜻을 나타내는 필드 이름에는 일관성을 유지하자.
```typescript
// bad
{
    accountNumber: '0012571125123', // number
    bankNo: '012' // no
}

// good
{
    accountNo: '0012571125123',
    bankNo: '012'
}
```
- 둘다 Number 로 통일을 하든 , No 로 하든 일관성을 유지하자.
```typescript
// bad
Request Body
{
    accountNumber: '0012571125123',
}
Response Body
{
    accountNo: '0012571125123',
}
// good
Request Body
{
    accountNo: '0012571125123',
}
Response Body
{
    accountNo: '0012571125123',
}
```
- 응답 필드 뿐만 아니라 , 요청 필드와 응답 필드 간에도 네이밍은 일관성을 유지하자
### 필드명 축약 금지
- 필드명을 축약하지 않는다.
- EX ) count -> cnt , name -> nm , table -> tbl  ❌
- API 를 사용하는 입장에서 , 전혀 의미를 알 수 없기 때문!!
- 축약어가 모두가 알고 있는 보편적인 단어라면 , 사용 가능!
	-> id 는 Identifier ( 식별자 ) 의 축약어 이지만 , 모두가 알고 있다.
	( 우리 조직에서만 알고 있는 축약어라면 절대 선택하지 않고 , 풀네임을 사용하자 - 신입 러닝 커브 , API 공개 시 혼동 )

### 타입에 맞는 필드명
```typescript
// bad
{
    discount: true,
    created: '2023-06-10T14:30:00',
    order: [..],
}

// good
{
    isDiscounted: true,
    createdAt: '2023-06-10T14:30:00',
    orders: [..],
}
```
- 필드가 표현되는 데이터 타입에 맞는 필드명 사용
	-> 적절한 필드명 선언하지 않으면 해당 값이 채워져 있을때만 그 값의 의미를 알 수 있다
	( discount : null 로 API 응답이 왔을 시 , 할인 금액을 나타내는지? 할인 여부 나타내는지? 등등 유추하기 힘듬 )
	=> API 문서를 항상 찾아봐야만 한다.
- API 문서를 찾아보지 않더라도 , 직관적으로 보자마자 알 수 있는 필드명을 사용해야 한다.
	- Boolean 타입이라면 is , has 등을 Prefix 로 선언
	- 일시 타입이라면 At 을 suffix 로 선언
	- 복수형은 복수형으로
### 마무리
- 이런 주제는 정답이 존재하지 않는다! ( 다양한 기준이 존재하므로 )
- API 가 인터페이스 라는 점은 분명해서 사용자가 직관적이게 이해할 수 있는 스펙을 유지하는 것이 중요하다!