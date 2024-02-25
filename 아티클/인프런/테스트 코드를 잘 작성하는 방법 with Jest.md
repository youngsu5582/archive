## 아티클 링크

https://tech.inflab.com/20230404-test-code/#%EB%8B%A4%EC%84%AF%EB%B2%88%EC%A7%B8-%ED%85%8C%EC%8A%A4%ED%8A%B8-%EC%9E%90%EB%8F%99%ED%99%94

## 내용

인프런은 typescript 와 jset framework 를 사용!

### 1. 테스트 코드는 DRY 보다는 DAMP 하게!

개발 원칙 중 DRY 원칙이 존재!
중복 코드를 싫어해 중복 코드가 보이면 어떻게든 없애려고 하는 경향
#### 개발자 원칙
##### 1. KISS
- Keep It Simple Stupid!
- SW 설계 작업이나 코딩 하는 행위에서 되도록 간단하게 단순하게 만드는 것이 좋다!
- 소스 코드와 설계 내용이 불필요하게 장황 및 복잡해지는 것을 경계하자
	=> 단순할수록 이해하기 쉽고 , 버그 발생 가능성이 줄어든다. ( 생산성 역시 향상! )
##### 2. YAGNI
- You Ain't Gonna Need It
- 프로그램 작성 시 , 현재 사용하지 않지만 확장성을 고려해 미리 작업해 놓은 것들이 존재!!
	-> 이런 작업들을 지양 하자
	( 현재 사용하지 않는 코드를 작성해놓으면 코드가 불필요하게 장황해짐 , 설계 & 환경 변경시 수정 코드양이 늘어남 )
##### 3.DRY
- Do not Repeat Yourselt
- 동일한 코드가 반복이 되는 건 잠재적인 버그 위협이 증가한다!
	( 내용이 변경될 필요가 있는 경우 , 반복되는 모든 코드에 찾아서 수정해야함 -> 실수 발생 가능! )
- 프로그램 규모가 커질수록 반복되는 코드로 인해 유지 보수 오버헤드가 커진다!


=> 중복 코드를 없애기 전 , DAMP 하게 테스트 코드를 작성하자!

DAMP 는 Descriptive and Meaningful Phrases : 의미있고 설명적인 구문을 사용하라는 원칙

### 문제의 코드!
```typescript
describe('지원자', () => {
  it('지원자를 최종합격시킨다.', () => {
    const name = 'haru';
    const status = JobApplicantStatus.IN_PROGRESS;
    const jobApplicant = JobApplicant.create(name, status);

    jobApplicant.pass();

    expect(jobApplicant.status).toBe(JobApplicantStatus.PASS);
  });

  it('지원서를 불합격시킨다.', () => {
    const name = 'haru';
    const status = JobApplicantStatus.IN_PROGRESS;
    const jobApplicant = JobApplicant.create(name, status);

    jobApplicant.fail();

    expect(jobApplicant.status).toBe(JobApplicantStatus.FAIL);
  });
});
```

- 결국 , const 부분은 중복적인 코드!
=>
```typescript
  beforeEach(() => {
    const name = 'haru';
    const status = JobApplicantStatus.IN_PROGRESS;
    jobApplicant = JobApplicant.create(name, status);
  });
```
- 각각 실행 전 수행되는 로직을 setup 메소드로 작성!

#### 이게 최선의 코드일까?

```typescript
it('지원서를 불합격 상태일때 보관할수 있다.', () => {
  jobApplicant.putStorage();

  expect(jobApplicant.isStorage).toBe(true);
});
```
- 테스트 코드 실행시 실패!
	-> it 내용으로 유추할 시 , Status 상태를 FAIL 로 요구
	-> beforeEach 에서는 IN_PROGRESS 로 초기화
=> 결국 , 더 파악하기 어려워짐!

status 를 전역 변수인 공유 상태로 둠으로 , 수정이 어려워지고 테스트간 영향을 주는 코드가 됨!

∴ 무작정 코드 중복을 줄이기 보단 , 의도를 정확하게 드러내자!

### Fixture Class/Function
##### Fixture?
- 테스트 실행 전 필요한 준비 작업 또는 설정 준비하는 기능!
#### Function-level Fixture
- 각 테스트 함수 & 메소드에 대해 설정 및 해제
- BeforeEach , AfterEach 
#### Class-level Fixture
- 테스트 클래스의 모든 메소드에 대해 설정 및 해제
- BeforeAll , AfterAll

#### Good Code
- 테스트의 중복을 줄이는 게 아니라 서술적이고 의미있게 작성하는 방향으로 리팩토링!
- 테스트는 서로 독립적이고 격리 되어 하므로 테스트 간 수정에 다른 테스트가 영향 받지 않게 하자!
- DAMP 원칙을 지키며 중복 줄이는 방안?
	=> 테스트 픽스쳐 함수나 클래스등을 사용하자!
	
```typescript
  it('지원자를 최종합격시킨다.', () => {
    const jobApplicant = JobApplicantFixture.create(JobApplicantStatus.IN_PROGRESS);

    jobApplicant.pass();

    expect(jobApplicant.status).toBe(JobApplicantStatus.PASS);
  });
```
- JobApplicantFixture 는 JobApplicant 를 생성해주는 코드로 유추!
```typescript
class JobApplicantFixture(){
	// status 를 enum 단위 관리라 생각!
	public static JobApplicant create(status : number){
	    const name = 'haru';
		return JobApplicant.create(name,status);
	}
}
```

- Fixture 파일에 생성자 케이스를 만들어 , 별도 관리
	-> 사실 , 해당 부분은 Factory Method Pattern 에 가까운 거 같음

### 2. 테스트 구현이 아닌 결과를 검증하도록 한다.

테스트 코드를 작성할 때 빠른 테스트를 위해 모킹한 모의 객체를 사용할 떄가 있다!
->mock , spy , stub 등 사용하는 경우 , 테스트 대상의 구현을 알야만 하는 문제가 발생한다.

해당 부분에서 , 남용하거나 잘못 사용할 가능성이 발생한다!!

```typescript
export class JobApplicant {
  updatePass() {
    this.validateIsNotCancel();
    this.validateIsNotFail();
    this.changeStatus();
  }

  validateIsNotFail(): void {
  }

  validateIsNotCancel(): void {
  }
}
```

- 해당 코드는 , 검증을 한 후 , 상태를 변경

```typescript
it('지원서를 합격시킨다.', () => {
  const jobApplicant = new JobApplicant();

  jest
    .spyOn(jobApplicant, 'validateIsNotCancel')
    .mockReturnValue(undefined);
  jest
    .spyOn(jobApplicant, 'validateIsNotFail')
    .mockReturnValue(undefined);

  jobApplicant.updatePass();

  expect(jobApplicant.validateIsNotCancel).toBeCalledTimes(1); // 호출되었는지 검증
  expect(jobApplicant.validateIsNotFail).toBeCalledTimes(1);
  expect(jobApplicant.status).toBe(JobApplicantStatus.PASS);
});
```
- 해당 코드는 완벽한가?
	-> 이론상 , 함수 호출 여부도 확인하고 , 상태 변경에 대한 검증도 하므로 완벽하다고 생각할 수는 있음

- 해당 테스트는 깨지기 쉬운 테스트이며 , 좋은 테스트가 아님!
	( validate 함수의 함수명이 변경 되거나 , 메소드가 삭제 되면 테스트는 깨지게 됨! )
=> 구현에 의존적이며 , 테스트 목적이 구현에 맞쳐져 있는 테스트!
내부 구현 이나 , 비공개 메소드들은 언제든 바뀔 여지가 있는 코드이기 때문에 은닉 위해 숨기고 테스트 하지 않는것이 좋음!

```typescript
  expect(jobApplicant.validateIsNotCancel).toBeCalledTimes(1);
  expect(jobApplicant.validateIsNotFail).toBeCalledTimes(1);
```
- 특히 , 해당 부분도 필요 없다!
	( 원하는 건 , 합격 상태 변경 검증 뿐이므로 + 리팩토링 내성 과 유지보수를 오히려 어렵게 만듬 )
###### 사담
- 이때 신선한 충격이였는데 , 대부분의 예시 테스트 코드는 이런 방식으로 작성이 되어 있기 때문
- 생각해보면 , 해당 메소드의 호출 횟수는 전혀 중요하지 않았음!

#### Good Code

- 테스트 코드를 내부 구현이 아닌 , 실행 결과에 온전히 집중하자!
- 의미 있는 테스트와 검증을 하자!
```typescript
it('지원서를 합격시킨다.', () => {
  const jobApplicant = new JobApplicant();

  jobApplicant.updatePass();

  expect(jobApplicant.status).toBe(JobApplicantStatus.PASS);
});

it.each([
  ['취소', JobApplicantStatus.CANCEL],
  ['불합격', JobApplicantStatus.FAIL],
])(
  '%s 상태의 지원서는 합격시킬수 없습니다',
  (_, status) => {
    const jobApplicant = new JobApplicant();
    jobApplicant.status = status;

    expect(() => jobApplicant.updatePass()).toThrowError();
  },
);
```
- it.eah 를 통해 , 단순 중복되는 코드 방지 가능!
- 테스트 결과들은 온전히 결과 검증에 집중!
	( 궁금한 점은 , 상단 코드인 JobApplicantFixture.create() 해당 코드에 안 넣은 이유 : 단순 보여주기 위함일까? )

### 3. 읽기 좋은 테스트를 작성하라

메인 과 제품을 위한 코드라 할 지라도 , 여전히 책임 지고 관리 해야하는 대상!
테스트 코드 역시 가독성이 좋고 , 명확해야함 ( 유지보수 및 리팩토링 을 위해서라도 )
```ts
it('포스트 글의 좋아요수를 업데이트한다.', () => {
  const a = User.create('test', 1);
  const b = User.create('test', 2);
  const c = Post.create('test', a, 3);
  c.updateLikeCount(300);
  expect(c.like).toBe(300);
})
```
- 해당 코드를 보며 드는 생각?
	- User.create 와 Post.create 는 단순 Domain 생성만 할까?
	- Post.create 에 있는 매개변수들이 의미하는게 뭘까?
	- updateLike는 + 일까 , 초기화 일까?

=> 이처럼 코드 파악하기 위해 , 부가적인 코드를 봐야하는 상황이 발생할 수 있다!

> 좋은 테스트 코드는 읽는 사람 입장에서 이 테스트를 이해하는데 필요한 모든 정보를 , 테스트 케이스 본문에 담고 있는 테스트

#### 테스트의 구조
1. 준비
2. 실행
3. 검증

- 각각 구절을 AAA 패턴 , GWT 패턴으로 나누어 작성 하자
##### AAA
- Arrange-Act-Assert Pattern
- 주석으로 구분
- 단위 테스트에 주로 사용 ( 코드 실행 검증 에 초점을 맞춤 )
##### GWT
- Given-When-Then Pattern
- 행위 주석으로 구분
- 행위 주도 개발에서 사용 ( 테스트를 사용자 단위 연결에 초점을 맞춤 )
- 비 개발자도 이해하기 쉽고 참여할 수 있게 설계

=> 사실 , 일반적인 관점에서 둘은 매우 유사 + 주석을 하지 않고 , 구분해서 가독성 좋게 작성하면 그게 제일 Best

- 테스트 안에 너무 많은 양의 코드가 존재할 시 ?
	=> 재사용을 위해 모듈화 ( Test Factory , Builder ,Helper Pattern ) 를하자

#### Good Code

- 읽기 쉬운 코드를 작성하자
- 무엇이 잘못되어 실패하는지 , 어떻게 고쳐야 하는지 파악하기 쉬운 테스트를 작성하자
- 테스트 구조 나누는 패턴 + 테스트 코드 모듈화 하는 방법 통해 테스트 코드 읽기 쉽게 작성하자
```ts
it('포스트 글의 좋아요수를 업데이트한다.', () => {
  // given 
  const givenLikeCount = 3;
  const post = Post.create('title', createUser(), givenLikeCount);
  
  // when 
  const updateLikeCount = 300;
  post.updateLikeCount(updateLikeCount);
  
  // then 
  expect(post.like).toBe(givenLikeCount + updateLikeCount);
})

function createUser(name = 'name', age = 1) {
  return User.create(name, age);
}
```

- 매개변수에 넣는 인자 값도 알기 쉽게 보여주자
- 가독성과 관리 용이를 위해 createUser 로 분리

### 4. 테스트 명세에 비즈니스 행위를 담도록 하자

마지막으로 테스트 명을 작성할 때도 유의하자!
테스트 명 역시 , 명확하게 의도를 나타내야 한다.
-> 간과하는 점이 , 
개발자 용어가 아닌 비즈니쉬 행위를 담은 비개발자 역시 읽을수 있게 설명되어야 한다!

#### Bad
```ts
it('관리자를 생성한후 관리자 정보를 확인한다.')
```
#### Better
```ts
it('관리자 정보로 가입한다')
```

- 테스트 코드 , 실행 로그 확인하지 않아도 , 동작 유추 가능 + 요구사항 빠르게 공유 가능!
- 영향 범위 빠르게 파악 가능하며 , 유추 통해 빠르게 복구 가능

- 해당 부분은 , 완전 큰 차이점 까지는 모르겠음 ( 비개발자 감성이 부족한 듯 )

### 마무리

- 테스트 코드는 "지속 가능한 서비스 , 프로젝트 " 위해 필수적인 요소임을 기억하자!

## 참고 링크


https://velog.io/@pood/Test-Fixture-%EC%97%90-%EB%8C%80%ED%95%B4%EC%84%9C
https://berrrrr.github.io/programming/2020/09/07/aaa-vs-gwt/
##### Writed By Obisidan