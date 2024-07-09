---
tags:
  - 리팩토링
아티클_링크: https://toss.tech/article/engineering-note-2
아티클_작성자: 나재은
추가_정보: 토스 테크 블로그
---
## 아티클 링크



## 세줄 요약

- null 이 나오는게 나쁜건 아니다!
- null 이 나올 시 , 처리 와 분기 방법에 대해 제공하자!
- 주석이 모든걸 해결 해준다고 착각에 빠지지 말자!
## 내용

#### 서론 : 개발자의 주 고객?

- 일반적으로 , 제품을 사용하는 사용자 ( End - user )
- 80% 의 정답
=> 컴파일 타임 고객 , 동료 개발자 역시 포함
- 복잡하고 , 나쁜 코드는 사용자 고객에게 버그 및 장애 유발
- 개발자 고객에게는 낮은 생산성을 줌

### 코드의 체크리스트

1. 코드를 읽고 있을 때 누군가 말을 걸면 어디까지 읽은지 놓쳐서 처음부터 다시 읽어야 하는가?
2. 코드 한 줄을 바꾸기 위해 바꿔야 할 다른 코드가 많다.
3. 새로운 사람이 팀에 합류하면 그 사람이 몇 주 내내 프로젝트 코드를 읽을 시간을 확보해야한다
4. 메서드 인자에 값 전달 위해 , 지나가는 모든 메서드 인자 값 추가 한 적이 있다.
5. 프로젝트 코드가 너무 복잡해 처음부터 다시 만들면 적어도 지금보다 나을 거라는 생각을 한 적 있다.

## Null 리턴은 왜 나쁜가?

### 문제 : 의미 축약한 코드 표현

```typescript
val user: User? = userRepository.findByName("김토스")
println(user)
```
- 해당 경우 user 가 Null 인 경우는?

1. DB에 "김토스" 라는 사람이 없을시
2. DB 와의 연결이 불안정 했을시
3. "김토스" 란 회원이 탈퇴했을시
4. "김토스"는 운영 환경 단위에서만 존재하는 사용자 일시

=> null 하나만으로 모든 것을 추론 해야함.
	( 세부 구현을 들여다 보는 순간 개발자의 생산성은 이미 떨어진 것과 다름 없음 )

#### 이는 꼭 null 개념에 한정되는것이 아니다.
##### 빈 문자열 "" 코드를 읽었을 시
- 사용자가 입력 시도를 하지 않았나?
- 사용자가 무언가를 입력했지만 잘못된 입력인가?
- 사용자가 실제 빈 문자열을 입력했는가?
##### getAge 함수가 Int 타입 -1 을 반환할 시
- 함수 실행은 잘 됐으나 , person age 데이터가 실수로 누락되어서 알 수 없나?
- 함수 실행은 잘 됐으나 , person age 입력이 선택 사항이라 알 수 없나?
- 함수 실행 도 실패했고 , 원인도 알 수 없나?
##### person.getPhoneNumbers() 함수가 리스트 반환하는 코드 읽을 시
- 빈 리스트가 돌아오면 , 함수 실행은 잘 되었을까 or 핸드폰 번호가 없는 걸까?
- null 이 돌아오면 , 함수 실행은 잘 되었을까 or 핸드폰 번호는 없는 걸까?

### 해결 1단계 : 로그에 맥락 남기기

코드의 다양한 의미를 축약하거나 없애지 않고 , 자세히 풀어 코드에 녹여내야함
-> 주석으로 충분하지 않음! ( 실제 코드가 아니므로 )

``` typescript

class UserRepository {
  db =  new Database();
  public findByName(name: string): User {
    var result: User | null = this.db.getUserBy(name)
	  if (result == null) {
		throw Error(`[ERROR] : 1014
		
	      인사관리 시스템과 동기화 되지 않은 유저의 이름을 입력한 경우 이 메시지를 볼 수 있습니다.
	      매주 월->화 넘어가는 자정에 인사 관리 시스템과의 데이터 동기화가 수행되므로, 새로운 사람이 월요일이 아닌 다른 날짜에 입사하지 않았는지 확인하십시오.
	      다음 주 월요일까지 기다리거나, 수동 동기화를 실행하면 문제가 해결될 수 있습니다.
	      
	      인사 관리 시스템과의 데이터 동기화 로직은 UserRepositorySync 클래스를 참고하십시오.
	      문제가 된 name=[$name]
	      `
    )
    }
      return result
  }
}
```

- 코드를 보지 않아도 , user 가 null 인 이유 및 배경 파악 가능
- Error 를 Handler 및 예외 던지지 않고 , 로깅 남기기 가능

### 2단계 : 맥락 처리 위한 기능 만들기

- findBy 가 null 을 return 하는 경우를 밖에서 알고 싶을시?
	( 재시도 로직 & 동기화 시도 )
```typescript
const user: User|null = userRepository.findByName("김토스")
if (user == null) {
  // user가 null이면 유저가 인사관리 시스템과 동기화 되지 않은 경우임.
  // 동기화를 한 번 트리거 시켜주도록 하자
  userRepositorySync.trigger()
}

const user2: User|null = userRepository.findByName("김토스")
print(user2) // 위에서 동기화를 한 번 시켜주었기 때문에 null일 수 없다
```
- 위 주석처럼 , null이 나올수 없는 경우
	-> null 이 나온다면?
	- 재시도 수행 후 , null 이 리턴될리가 없는데?
=> 이 처럼 , null 리턴하는 다른 상황 표현할 수 없는 문제
=> 특히 , 주석으로 인해 정보의 혼란성을 줄 수 있음!

```typescript
var user: User = userRepository.findByName(
    name = "김토스",
    retryHandlerWhenMissing={ userRepositorySync.trigger() }
)

print(user) // non-null type
```
- user 가 nullable 하지 않음
- 분기문도 사라졌기에 가독성 역시 좋음
=> retryHandlerWhenMissing 을 무조건 넣어야 하는 단점 발생!

### 3단계 : 필요할 때만 제공하기

- 결국 , retryHandlerWhenMissing 은 개발자가에게 부담을 줄 수 있음 
```typescript
val user: User = userRepository
  .withRetryPolicy(ResyncWhenUserMissing()) // 이 라인을 삭제해도 findByName() 호출에는 문제 없음
  .findByName("김토스")

print(user) // non-null type

// 이렇게 사용해도 문제 없음
val user2: User = userRepository
  .findByName("김토스")

print(user2) // non-null type
```
- default Argument 사용 시 , 생략 역시 가능
- 지정됐기에 , 사용자는 세부 사항에 대해 알 필요 없이 , 사용 가능!
=> 결국 , ResyncWhenUserMissing 을 위해 , UserRepository 에는 추가로 기능을 구현 해야함.

```java
import org.springframework.data.jpa.repository.JpaRepository

interface UserRepository : JpaRepository<User, Long> {
    // 이건 Spring Framework가 알아서 해주는데,
    fun findByName(name: String): User

    // 이건 어떻게 하지?
    fun withRetryPolicy(retryPolicy: RetryPolicy): UserRepository
}
```
- JPA 가 간단한 기술은 자동으로 해주나 , 하단 추가 기능은 직접 구현!
``` typescript
import { Injectable, NestInterceptor, ExecutionContext, CallHandler, HttpException } from '@nestjs/common';
import { Observable, throwError } from 'rxjs';
import { catchError, retryWhen, delay, mergeMap } from 'rxjs/operators';

@Injectable()
export class RetryInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      retryWhen(errors =>
        errors.pipe(
          delay(1000), // 1초 후 재시도
          mergeMap((error, i) => i < 3 ? throwError(error) : throwError(error)) // 최대 3회 재시도
        )
      ),
      catchError(err => throwError(new HttpException('Request failed', 500)))
    );
  }
}

```
- 해당 구문과 같이 구현
- AOP 와 같은 맥락인 Interceptor 로 구현 가능

## 참고 링크
https://toss.tech/article/engineering-note-2

##### Writed By Obisidan