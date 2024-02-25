## 아티클 링크

https://toss.tech/article/typescript-type-compatibility
## 세줄 요약

- Typescript 만의 Type 호환성
- structural subtyping
- Fresh Literal

## 내용

### 서론

#### TypeScript 의 타입 호환성 이란?
- 구조적 서브 타이핑 ( structural subtyping ) 기반
	-> 오직 멤버만으로 타입 관계
	-> 명목적 타이핑 과는 대조적
- y 가 최소한 x 와 동일한 멤버를 가지고 있을 시 x 와 y 는 호환되는 것

=> 강한 타입 시스템이 필요한데 왜 타입 호환성을 지원하는가?

### 예시

Food Type 객체를 인자로 받아 간단한 칼로리 계산 공식 계산하는 Function 존재
```typescript
type Food = {
	protein:number;
	carbohydrates:number;
	fat:number;
}
function calculateCalorie(food : Food){
	return food.proten * 4
		+ food.carbohydrates * 4
		+ food.fat * 9
}
```

![500*500](https://i.imgur.com/RditRER.png)

해당 Case 인 경우 , Type Checker 는 어떻게 판단해야하는가?
1번은 오류 없음이 명확 ( 당연히 , Type 일치하므로 )
2번은 오류 판단이 명확 ( 다른 Type 이므로 )
3번은 판단의 고민 ( Food 의 한 종류이므로 )
```typescript
type Burger = Food &{
	burgerBrand : string;
}
```

결국 Burger 는 계산을 위한 모든 Property 가 존재!
-> 정상적 동작 하는 코드라면 , 유연하게 대응하는 것 역시 나쁘지 않을 수 있음
-> Computer 와 같은 타입 사례는 타입 오류로 판단해야 하므로 , 명확한 규칙 필요

=> 명목적 서브타이핑 과 구조적 서브타이핑 존재

### 명목적 서브타이핑
- nominal subtyping

타입 정의 시에 상속 관계를 명확히 명시한 경우에만 타입 호환 허용
타입 오류 발생 가능성 배제하고 , 개발자 명확한 의도를 반영
```typescript
type Burger = Food &{
	burgerBrand : string;
}
const burger : Burger = {
	protein: 29,
	carbohydrates: 48,
	fat: 13,
	burgerBrand: '버거킹''
}
const calorie = caluclateCalorie(burger)

=> 타입검사결과 : 오류없음 (OK)
```

- C# , Java 는 명시적 상속 관계를 명시해줘야만 타입 호환 가능
### 구조적 서브타이핑
- structural subtyping

상속 관계 명시되어 있지 않더라도 객체 프로퍼티 기반으로 사용함에 문제 없을시 타입 호한 허용
-> 상속 관계임을 명시하지 않아도 모든 Property 를 포함하고 , 함수 실행과정에서 오류 발생 X
```typescript
const burger = {
	protein: 29,
	carbohydrates: 48,
	fat: 13,
	burgerBrand: '버거킹'
}
const calorie = caluclateCalorie(burger)

=> 타입검사결과 : 오류없음 (OK)
```
- "만약 어떤 새가 오리처럼 걷고 , 헤엄치고 , 꽥꽥거리는 소리 낸다면 그 새는 오리라 부를 것이다. "
	-> Duck Typing 이라고도 불림
### 심화적

```typescript
const calorie = calculateCalorie({
  protein: 29,
  carbohydrates: 48,
  fat: 13,
  burgerBrand: '버거킹'
})

=> 타임검사결과 : 오류 (NOT OK)
```

TypeScript Compiler 역활은 
TypeScript 코드를 AST 로 변환 -> Type 검사 -> JavaScript 로 변환

- parser.ts , scanner.ts 가 AST 변환 과정 담당
- binder.ts , checker.ts 가 Type 검사 담당
- emitter.ts , transformer.ts 가 JavaScript 변환 담당

### Checker

```typescript
/** 함수 매개변수에 전달된 값이 FreshLiteral인 경우 true가 됩니다. */
const isPerformingExcessPropertyChecks =
    getObjectFlags(source) & ObjectFlags.FreshLiteral;

if (isPerformingExcessPropertyChecks) {
    /** 이 경우 아래 로직이 실행되는데,
     * hasExcessProperties() 함수는
     * excess property가 있는 경우 에러를 반환하게 됩니다.
     * 즉, property가 정확히 일치하는 경우만 허용하는 것으로
     * 타입 호환을 허용하지 않는 것과 같은 의미입니다. */
    if (hasExcessProperties(source as FreshObjectLiteralType)) {
        reportError();
    }
}
/**
 * FreshLiteral이 아닌 경우 위 분기를 skip하게 되며,
 * 타입 호환을 허용하게 됩니다. */
```

- 함수에 인자로 들어온 값이 FreshLiteral 여부에 따라 호환 허용 여부 결정
#### Fresh Literal

타입 호환의 예외 조건과 관련해 신선도(Freshness) 라는 개념 제공
- object literal 은 초기에는 fresh로 간주
- 타입 단언(type assertion) 을 할 시 + 타입 추론 의해 object literal 타입 확장되면 사라짐
-> 특정 변수에 object literal 할당하는 경우 freshness 사라짐
-> 함수에 인자로 object literal 바로 전달 시 fresh 한 상태로 전달


##### fresh object 인 경우에는 예외적 타입 호환 허용 X 
-> https://github.com/Microsoft/TypeScript/pull/3823

```typescript
/** 부작용 1
 * 코드를 읽는 다른 개발자가 calculateCalorie 함수가
 * burgerBrand를 사용한다고 오해할 수 있음 */
const calorie1 = calculateCalorie({
  protein: 29,
  carbohydrates: 48,
  fat: 13,
  burgerBrand: '버거킹'
})

/** 부작용 2
 * birgerBrand 라는 오타가 발생하더라도
 * excess property이기 때문에 호환에 의해 오류가
 * 발견되지 않음 */
const calorie2 = calculateCalorie({
  protein: 29,
  carbohydrates: 48,
  fat: 13,
  birgerBrand: '버거킹'
})
```
- 결국 , 나머지 property 의 type 및 value checker 가 불가능!
- 해당 함수에서만 사용되고 , 다른 곳에서 사용 X ( 유연한 보다는 부작용을 발생시킬 가능성만 높아짐 )

##### index signature 포함해 fresh object 에 타입 호환 허용 O
```typescript
type Food = {
	protein:number;
	carbohydrates:number;
	fat:number;
	[x:string]:any;
}
const calorie = calculateCalorie({
	...
	burgerBrand :'버거킹''
})
=> 오류없음 (OK)
```

- tsconfig 상에 suppressExcessPropertyErrors 를 true 로 설정하는 방식도 가능

### Branded Type

특정 타입에 고유 식별자 추가해 해당 타입을 다른 타입과 구분하도록 하는 기법
-> 구조적 타이핑 때문에 발생
( 타입이 구조적으로 동일하면 , 같은 타입으로 구분하므로 )

```typescript
type Brand<K, T> = K & { __brand: T};
type Food = Brand<{
  protein: number;
  carbohydrates: number;
  fat: number;
}, 'Food'>

const burger = {
  protein: 100,
  carbohydrates: 100,
  fat: 100,
  burgerBrand: '버거킹'
}

calculateCalorie(burger)
/** 타임검사결과 : 오류 (NOT OK) */
```
- 의도적으로 property를 추가해 , 정의 타입 외에는 호환 될 수 없도록 강제하는 기법
## 결론

- 타입 검사의 안정성 과 유연함 사이에서 타입 호환성이 발생!
- 프로그래밍 언어마다 타입 검사 및 동작 방식이 다르고 , 이는 논의와 의사결정에 따라 선택된 결과
- 프로젝트에서 , 위 기술한 설정들을 이용해 타입 호환성 범위를 선택하자 

##### Writed By Obisidan