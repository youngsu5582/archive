---
tags:
  - 리팩토링
아티클_링크: https://jojoldu.tistory.com/734
아티클_작성자: 향로
추가_정보: 개인_블로그
---

좋은 예외 처리는 견고한 프로그램을 만들고 , 좋은 사용자 경험을 줍니다

예외 처리를 통해 Application 이 예기치 않게 종료되는 걸 방지하고 ,
갑작스런 종료가 아닌 사용자는 무엇이 잘못 됐는지 , 가능하면 어떻게 바로잡을 수 있는지에 대한 메시지를 받게 해줍니다
또한 , 개발자가 문제를 진단하는데 큰 도움을 줘 문제 해결 시간을 단축 시켜 줍니다

복잡한 여러 단계 프로세스가 있을 시 , 위치 따라 다르게 적절한 처리와 예외는 프로그램 프로세스를 관리하는데 유연성을 제공해줍니다

반면 , 이를 위해 과도하게 사용하면 비즈니스 로직이 뭔지 파악하기 힘들 정도로 너무 많은 오류 처리 코드를 가지는 코드가 될 수도 있습니다

### 복구 가능한 오류와 불가능한 오류 구분하기

가장 먼저 할 것은 복구 가능한 오류 와 복구 불가능한 오류를 구분하는 것입니다.

모든 예외에 대해 동일 방식을 적용할 수 없습니다
어떤 예외는 상시 발생할 수 있어 무시해도 되며 , 어떤 예외는 무시하면 절대 안되는 경우도 있습니다.

이들을 구분하지 않고 , 다룬다면
사용자는 불편하고 , 개발자는 상시 발생하는 알람으로 오류를 등한시 하게 되는 경우가 생길수도 있습니다.

=> 이들을 구분해서 예외 발생시 어떻게 처리할지 결정해나가야 합니다

#### 복구 가능한 오류

복구 가능한 오류는 시스템 외적 요소로 발생하는 치명적이지 않은 ( Non - Critical ) 요소입니다.

사용자가 잘못된 전화번호를 입력했다고 시스템을 멈춘다면?
-> 이는 말도 안되는 사항입니다.

다시 입력하라는 오류 메시지를 제공하고 다시 입력받으면 됩니다.
( 네트워크 오류 역시 이와 같은 맥락입니다 )

- 사용자의 오입력
- 네트워크 오류
- 서비스적 중요하지 작업의 오류

이런 오류들은 프로그램이 감시후 , 적절한 조치 취한 후 정상적으로 계속 실행해도 되는 오류입니다
상시로 발생할 수 있다고 가정하고 , 사용자(고객) 에게 가능한 문제 원인을 인지시켜주면 됩니다

너무 잦은 오류 발생 시 , 개발자의 오류일 수도 있으므로
로그 레벨을 warn 으로 두고 , 임계치 넘으면 모니터링 알람을 보내게 합니다.

#### 복구 불가능한 오류

복구 불가능한 오류는 시스템이 자동으로 복구할 수 있는 방법이 없는 경우입니다.
대부분 경우 , 이 오류를 해결하지 못하면 프로그램이 계속 실행될 수 없습니다

- 메모리 부족 : 시스템이 필요한 메모리를 할당받지 못할 때 발생
- 스택 오버플로우 : 재귀 함수가 너무 깊게 호출되어 스택 메모리가 고갈될 때 발생
- 시스템 레벨 오류 : H/W 문제나 운영체제의 중대한 버그로 인해 발생
- 개발자의 잘못 구현된 코드

자주 발생하는 오류가 아니므로 , 개발자에게 원인을 빠르게 제공해야 하고
로그 레벨을 error로 두고 , 에러 트레이스를 남겨 , 임계치 초과시 개발자에게 알람을 보내도록 구성해야 합니다

### null , -1 , "" 같은 특수값 예외로 사용하지 않기

예외 상황은 Exception 만으로 처리해야 합니다
일부 프로젝트에서는 예외가 아닌 특정 값을 사용하는 경우가 있습니다
```typescript
function divideWrong(a: number, b: number): number {
    if (b === 0) {
        return -1;  // 오류를 나타내는 대신 -1 반환
    }
    return a / b;
```

이렇게 반환할 경우 , 호출자는 항상 반환값을 확인해야 합니다. ( 반환값이 -1은 아닌지 )
-1이 의미하는 바가 뭔지도 알아야 합니다.

```typescript
function divideRight(a: number, b: number): number {
    if (b === 0) {
        throw new Error("Division by zero is not allowed.");  // 오류를 throw
    }
    return a / b;
}
```

호출자는 항상 예외를 처리해야 하며 , 예외의 의미를 예외의 타입을 보고 유추가 가능하게 됩니다
- 정확히 어떤 문제인지 표현할 수 있다
- 해당 문제의 상세 메시지를 포함할 수 있다
- 어떤 경로로 이 문제가 발생한 지 , 확인할 수 있는 Stack Trace 를 얻을 수 있다
- 더 깔끔한 코드를 작성할 수 있다

### 문자열 throw 하지 않기

```javascript
throw '유저 정보를 받아오는데 실패했습니다.';
```

해당 코드처럼 , 문자열을 그대로 throw 할 수도 있다.
다양한 형태의 CustomException 을 구분하는 방법은 오직 메시지 내용만으로 구분해야 합니다

스택 트레이스 같은 다양한 정보가 없어 문제 해결하는데도 어려움이 생깁니다
그래서 다음과 같이 항상 Error ( Exception ) 객체를 사용해야 합니다!

```typescript
throw new NotFoundResourceException ('유저 정보를 받아오는데 실패했습니다.');
```

이렇게 , 오류에 대한 의미 있는 정보를 제공할 수 있게 되고
문제 설명 , 문제 발생 Context & 디버깅 위한 StackTrace 등 유용한 세부 정보를 전달합니다

### 추적 가능한 예외


실패한 코드의 의도를 파악하려면 호출 스택만으로 부족합니다
그렇기에 , 아래 내용들을 예외에 같이 담아줘야 합니다

- 어떠한 값을 사용하다가 실패했는지
- 실패한 작업의 이름과 실패 유형

이들이 있어야만 운영 환경에서 예외 발생했을 때 조금이라도 정확하고 빠르게 대응이 가능해집니다
```typescript
throw new IllegalArgumentException('잘못된 입력입니다.'); ❌
=> 
throw new IllegalArgumentException(`사용자 ${userId}의 입력(${inputData})가 잘못되었다.`);
```

- 물론 , Exception 내용을 사용자에게 그대로 노출할지는 별개의 이야기 입니다
	( Logger 를 통해 남길 내용 과 사용자에게 노출할 메시지는 분리해야 할 수도 있습니다! )

### 의미를 담고 있는 예외

예외의 이름에는 예외의 원인 과 내용을 정확하게 반영해야 합니다
-> 코드를 읽는 사람이 이름과 예외만 보고도 , 왜 발생한지 어느정도 추측이 가능하게 해줘야만 합니다

- 코드의 가독성 향상 : 의미 있는 이름 가진 예외는 코드 읽는 사람에게 문맥을 제공해줍니다
- 디버깅 용이성 : 오류 원인을 빠르게 파악하고 , 수정을 가능하게 해줘야 합니다

```typescript
class CustomException extends Error {}

function connectToDatabase() {
	throw new CustomException("Connection failed because of invalid credentials.");
}
```
- 해당 예외는 너무 포괄적인 의미를 가지고 있다 ( CustomException )
- 좀 더 유의미한 예외로 만들어 개선하자!

```typescript
class InvalidCredentialsException extends Error {}

function connectToDatabase() {
    throw new InvalidCredentialsException("Failed to connect due to invalid credentials.");
}
```
- 예외 이름을 통해 , DB 연결 시 발생한 인증 오류임을 명확하게 나타냅니다

#### Layer에 맞는 예외

일반적인 3계층 Web Application에선 다음과 같은 계층을 가집니다.
- Data Access Layer
- Business Logic Layer
- Presentation Layer

각 계층들이 발생할 수 있는 오류의 성격은 다르므로 , 해당 계층에 맞는 예외를 던지는 게 유용합니다.
```typescript
function fetchUserData(userId: string): any {
	throw new DataAccessException("Failed to fetch user data from database.");
}

function getUserProfile(userId: string): any {

try {
	const userData = fetchUserData(userId);
} catch (error) {
	if (error instanceof DataAccessException) {
		throw new BusinessLogicException("Error processing user profile.");
	}
}
}
```

### 예외 계층 구조 만들기

예외를 가능한 계층 구조로 만들어서 사용하는 것이 좋습니다

수많은 Custom Exception 들이 만들어 집니다.
하지만 , 이들을 무턱대고 만들고 다 따로 사용하는 것보단 분리하여 계층적 Exception 을 만드는 것이 좋습니다
```typescript
class ValidationException extends Error {}
class DuplicatedException extends ValidationException {}
class UserAlreadyRegisteredException extends ValidationException {}
```

상위 Exception 을 가지게 하여 , 추적을 용이하게 만들수 있습니다

### 외부 예외 감싸기

외부 SDK , 외부 API 통해 발생하는 예외들은 전부 하나로 묶어서 관리하자
( 이는 , 앞선 예외 계층 구조 만들기에도 연관이 됩니다 )

```typescript
function order() {
  const pay = new Pay();
  try{
      pay.billing();
      database.save(pay);
  } catch (e) {
      logger.error(`pay fail`, e);
  }
}
```

이렇게 한번에 모든걸 catch 할 경우 , 
구체적으로 어디서 문제가 발생했는지 & 어떤 해결 방법을 해결해야할지 포함시키기가 매우 어려워집니다

외부 라이브러리인 ( pay.billing 과 , database.save 가 같은 방식으로 해결되서는 안됩니다!  - 정말 당연한 이치 )

특히 , 결제 서비스인 경우에는 사용자 문제로 오류가 발생할 수 있는 상시적인 문제들도 있습니다

```typescript
function order() {
  const pay = new Pay();
  try{
      pay.billing();
      database.save(pay);
  } catch (e) {
      if(e instanceof PayNetworkException) {
        ...
      } else if (e instanceof EmptyMoneyException) {
        ...
      } else if (e instanceof PayServerException) {
        ...
      }
      ...
  }
}
```

그럼 이렇게 , 모든 Error 를 처리하는 경우는?

결제 서비스가 바뀌는 경우에는 모든 Exception 을 수정해야만 할 것입니다

```typescript
function billing() {
  try {
    pay.billing();
  } catch (e) {
    if(e instanceof PayNetworkException) {
      ...
    } else if (e instanceof EmptyMoneyException) {
      ...
    } else if (e instanceof PayServerException) {
      ...
    }
    ...
    throw new BillingException (e);
}

function order() {
  const pay = new Pay();
  pay.billing(); 
  
  try{
    database.save(pay);
  } catch (e) {
    pay.cancel();  
  }
}
```
- BillingException 은 서비스 자체 예외이므로 , 맨 마지막 처리 ( Middleware , Global Handler 등 )
- DB에서 저장 실패시 , 결제 요청도 취소 처리한다

- 외부 라이브러리 와 우리 서비스간 의존성이 분리된다. ( 다른 외부 라이브러리로 교체 필요한 상황에도 )

### 다시 throw 할 거면 잡지 않기

``` typescript
function something() {
  try {
    // 비즈니스 로직
  } catch (e) {
    throw e
  }  
}
```

해당 위 코드는 

```typescript
function something() {
  // 비즈니스 로직
}
```

해당 코드와 다를게 없다.
물론 , Exception 을 무시하는 것보다야 , try - catch 하는게 더 나을수도 있는거 아닌가? 라고 생각할 수 있습니다
-> 그래도 굳이 , 불필요한 코드를 추가한 것과 다를게 없습니다

catch 절에는 예외 흐름에 적합한 구현 코드가 있어야만 합니다
( Logging or Layer 에 적합한 Exception 변환등이 필요하다면 try - catch 로 잡아도 괜찮습니다! )

### 정상적 흐름에서 Catch 금지 ( 무분별한 Catch 금지 )

프로그램 정상적 흐름 제어하기 위해 예외를 사용하지 않는게 좋습니다
예외는 오직 예외적인 경우에만 사용해야 합니다

```typescript
function fetchDataFromAPI() {
  // 가상의 API 호출
  if (/* 데이터가 없는 경우 */) {
      throw new NoDataFoundError();
  }
  // 데이터 반환
  return data;
}

function display() {
try {
  const data = fetchDataFromAPI();
  process(data);
} catch (error) {
    if (error instanceof NoDataFoundError) {
        displayEmptyState();
    } else {
        displayGenericError();
    }
}
}
```

NoDataFoundError 예외는 데이터가 없는 일반적인 상황을 나타냅니다
-> 이는 예외를 프로그램 정상적 흐름에 사용하는 잘못된 접근 방식입니다

```javascript
function fetchDataFromAPI() {
  // 가상의 API 호출
  if (/* 데이터가 없는 경우 */) {
    return null;
  }
  
  // 데이터 반환
  return data;
}

function display() {
  const data = fetchDataFromAPI();

  if (!data) {
    displayEmptyState();
    return;
  }

  process(data);
}
```

함수는 null 을 반환하고 , 결과를 확인해 적절한 액션을 취하게 하면 됩니다
( Null Object Pattern 을 활용할 수 있다면 더 Good! )

특정 예외를 처리하기 위해 너무 많은 코드를 사용하거나 , 필요 이상으로 구체적인 예외를 처리할 필요는 없습니다!

### 가능한 늦게 예외를 처리 한다.

Exception 을 Throw 하자마자 잡는게 아닌 , 가능한 최상위 계층에서 처리합니다
( 무조건적으로 , 글로벌 핸들러에서 처리하라는 것은 아닙니다! )

해당 예외를 처리해야 하는 여러 계층 중 가장 최상위 계층에서 처리 하자는 뜻입니다

```typescript
function divide(a: number, b: number): number {
    if (b === 0) {
        // 일찍 예외를 던진다
        throw new DivideZeroError("Cannot divide by zero!");
    }
    return a / b;
}

function calculate() {
  try {
    divide();
  } catch (e) {
    if (error instanceof DivideZeroError) {
       ...
    }
  }
}
```

해당 코드는 , divide 함수를 호출하는 모든 곳에서 항상 예외처리를 해야 합니다
-> 실제 비즈니스 로직 보다 예외 처리 하는 코드가 더 많아질 수 있습니다

```typescript
export class DivideZeroExceptionsFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();
    const status = exception instanceof HttpException ? exception.getStatus() : HttpStatus.INTERNAL_SERVER_ERROR;

    const errorResponse = {
      statusCode: status,
      message: exception['message'],
      // ... add other fields if needed
    };

    response.status(status).json(errorResponse);
  }
}
```

throw 한 후 , 마지막에 Catch 를 하면 , 코드 가독성 , 재사용성 그리고 통일성에 유용합니다!

### 마무리

프로그램을 만들며 , 오류를 피하는 건 불가능합니다! ( 예상치 못하게 발생하는 오류도 수많이 존재하므로 )
좋은 코드란 해당 오류들을 어떻게 다루는지가 중요합니다

##### In 향로

프로젝트에서 예외에 대한 기준들을 하나둘 씩 세우고 기록으로 남겨두자
남겨둔 기록을 추가 / 개선 해나가면 커리어 내내 도움이 된다

특히 , 예외처리 & 로깅 은 프로그래밍 언어와 무관하게 도움이 많이 되니 계속 기록해나가자!!