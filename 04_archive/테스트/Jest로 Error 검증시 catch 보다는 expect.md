---
tags:
  - 테스트
아티클_링크: https://jojoldu.tistory.com/656
아티클_작성자: 향로
추가_정보: 개인_블로그
---
Jest 로 테스트를 작성하든 , Java 에서 Mockito 를 통해서 테스트를 작성하든
Exception 에 대한 검증을 작성해야할 때가 있다.

이때 2가지 방법 중 하나를 선호합니다
- try ~ catch
- expect.rejects.toThrowError

##### try ~ catch
```typescript
it("[try/catch] 주문금액이 -이면 BadParameter Exception 을 던진다.", async () => {
    try {
        await acceptOrder({amount: -1000});
    } catch (e) {
        expect(e).toBeInstanceOf(BadParameterException);
        expect(e.message).toBe('승인 요청 주문의 금액은 -가 될 수 없습니다');
    }
});
```
##### expect
```typescript
it("[expect] 주문금액이 -이면 BadParameter Exception 을 던진다.", async () => {
    await expect(async () => {
        await acceptOrder({amount: -1000});
    }).rejects.toThrowError(new BadParameterException('승인 요청 주문의 금액은 -가 될 수 없습니다'));
});
```

향로는 expect 방식을 더욱 선호한다고 한다.

### 1.예외없이 완료 되는 경우

첫번째 코드인 try-catch 테스트가
실제 발생해야 할 예외가 발생하지 않을 경우를 대비해 추가적인 코드가 필요해집니다

```typescript
async function acceptOrder1(order): Promise<void> {
}

it("[try/catch] 주문금액이 -이면 BadParameter Exception 을 던진다.", async () => {
    try {
        await acceptOrder1({amount: -1000});
    } catch (e) {
        expect(e).toBeInstanceOf(BadParameterException);
        expect(e.message).toBe('승인 요청 주문의 금액은 -가 될 수 없습니다');
    }
});
```
해당 코드는 예외 발생하지 않고 , 정상적 수행 되어도 테스트를 통과한다.

##### 검증 방법 1. Exception 발생 여부
```typescript
it('[try/catch] 주문금액이 -이면 BadParameter Exception 을 던진다.', async () => {
    let errorWeExceptFor = null;
    try {
        await acceptOrder1({amount: -1000});
    } catch (e) {
        expect(e).toBeInstanceOf(BadParameterException);
        expect(e.message).toBe('승인 요청 주문의 금액은 -가 될 수 없습니다');
        errorWeExceptFor = e;
    }

    expect(errorWeExceptFor).not.toBeNull();
});
```
- Exception 이 발생했는지 , 안했는지 여부로 검사

##### 검증 방법 2. 의도적 throw 발생
```typescript
it("[try/catch & fail] 주문금액이 -이면 BadParameter Exception 을 던진다.", async() => {
    try {
        await acceptOrder1({amount: -1000});
        throw new Error('it should not reach here');
    } catch (e) {
        expect(e.message).toBe('승인 요청 주문의 금액은 -가 될 수 없습니다');
        expect(e).toBeInstanceOf(BadParameterException);
    }
});
```
- catch 로 빠지지 않으면 , throw 를 발생시켜 catch 로 빠지게  한다
( fail 은 정상적 작동하지 않는 Issue 가 있다고 합니다 - 사실 불필요 )

=> 결국 , try ~ catch 문은 후속 조치가 항상 필요해집니다

### 상세한 에러 트레이스

try ~ catch 로 잡을 경우 , 예상과 다른 예외 발생 시 추적할 정보가 없습니다

catch 영역에서 예외정보를 가져갔기 때문에 
추적 위해서는 로거를 통해 에러 트레이스를 출력해야만 합니다

반면 , expect 방식은 예상치 못한 예외가 발생하더라도 상세 예외 트레이스를 모두 출력시켜줍니다

![450](https://i.imgur.com/yMwvnz8.png)

예상과 다른 예외가 발생한다면 , 무엇때문인지 상세히 알려주므로 원인 파악이 쉬워집니다

### 마무리

테스트 코드를 검증할때는 이 테스트가 통과될 때만 가정을 해서는 안됩니다

- 잘못된 결과로 테스트가 통과하는 경우
- 의도와 다른 에러가 발생하는 경우

등 여러 테스트를 고려해 테스트 검증문을 선택해야만 합니다.