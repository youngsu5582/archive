---
tags:
  - 리팩토링
테크톡_링크: https://www.youtube.com/watch?v=Sb3fMvIIsqQ
테크톡_발표자: 포비(박재성)
---
켄트백이 처음 제시한 개념

Test Driven Development

테스트 부터 개발을 시작하자
( Not Production )

테스트 케이스 목록을 만들고 , 그것부터 시작을 하자

- 일단 클래스에 고민을 하지 말자 ( 메소드 명 역시 X )
	-> Input 을 넣으면 , Output이 뭐가 나오는지 생각하자

### Sample

숫자 하나 더하기?

```java
public void add_숫자하나(){
	int result = a("1");
	assertEquals(1,result);
}
```

왜 함수명을 a로 하면 안되는가? - Why Not

가장 빠르게 테스트를 통과시키자
- 클래스를 빨간줄 뜨는 컴파일 에러에서 만들어서 시작을 하자! ( 거꾸로 )
=> fail을 본 상태에서 가장 빠르게 시작하자

```java
public class StringCal {
	public static int a(String string) {
		return 1;
	}
}
```

왜 불가능한가?
=> 테스트를 통과하므로 OK

```java
public class StringCal {
	public static int add(String text) {
		return 1;
	}
}
```

그래도 너무 마음에 안드네?
=> 리팩토링! ( 메소드명 변경 , 파라미터 변경 등등 )

```java
public void add_숫자하나(){
	int result = a("1");
	assertEquals(1,result);
	assertEquals(2,result);
}
```

어 테스트가 다시 에러가 뜨네?
=> 다시 프로덕션 코드 수정

```java
public class StringCal {
	public static int add(String text) {
		return Integer.valueOf(string);
	}
}
```

오케이 통과

```java
public void add_쉼표_구문자(){
	assertEquals(3,add("1,2"));
}
```

Fail이네?

=>

```java
public class StringCal {
	public static int add(String text) {
		if(text.contain(",")){
			String[] values = text.split(",");
			int first = Integer.valueOf(values[0]);
			int second = Integer.valueOf(values[1]);
			return first + second;
		}
		return Integer.valueOf(string);
	}
}
```

오케이 통과~

```java
public void add_쉼표_구문자(){
	assertEquals(8,add("1,2,5"));
}
```

또 fail 이 뜨겠지?

=>

```java
public class StringCal {
	public static int add(String text){
		if(text.contains(",")){	
			String[] values = text.split(",");
			int sum = 0;
			for(String value : values){
				sum += Integer.valueOf(value);
			}
			return sum;
		}
		return Integer.valueOf(text);
	}
}
```

포비는 리폭토링 ( 연습 ) 을 극단적으로 한다

어차피 5 같은 단일 숫자도 split을 통해 배열로 나오는거 아닌가?
밑에 Integer.valueOf(text) 를 지워볼까?
어떻게 확인하지?
=> 기존의 테스트를 돌려보자!

OK 통과!

하나의 함수가 너무 많은 기능을 하는거 아닌가?
분리해볼까?
어떻게 확인하지?
=> 기존의 테스트를 돌려보자!

```java
public static int add(String text){
	String[] values = split(text);
	return sum(values);
}
```

OK 테스트 통과!

리팩토링을 진정하게 하려면 , 작동하는 테스트 코드가 필요하다!
테스트 코드가 둿받침을 해준다

```java
public static int add(String text){
	return Arrays.stream(text.split(",")).mapToInt(Integer::valueOf).sum();
}
```

내가 람다를 배웠는데 사용해볼까?
하 겉멋좀 부려볼까??
 => 테스트 코드가 나를 지켜준다


클린 코드 와 리팩토링은 공부로 되는게 아닌 , 이런식 저런식 바꿔가며 만들때 나온다

=> 결국 받쳐주려면 , 테스트 코드가 필요하다

- 모아놓으면 하기 귀찮음 -> 코드가 발전할 수록 리팩토링을 해야하나 테스트 코드를 짜기 귀찮아진다
테스트를 먼저 함으로써 , 변화를 이끌어 나가자 ( 끊임없는 리팩토링 통해 ) - 자존감 , 실력 , 자신감 UP

##### 결론

그렇기에 최대한 빠르게 테스트를 통과하고 , 리팩토링 해나가자

#### 왜 TDD 에 집착하는가?

리팩토링도 극단적으로 해야 , 실력이 는다


- 그 뒤에서는 우테코에서 알려준 개념과 동일

- 사람이기에 , 한번에 하나씩 하고 싶다

- 테스트를 통과함으로써 , 심적 안정감

- 처음부터 완벽한게 아니라 , 점진적으로 설계를 개선 해나갈수 있다

- 자동화된 테스트 개발 사이클을 통해
처음에는 느리나 , 그 다음부터 어마어마 무시하게 빠르다

#### TDD In 인생

##### 책쓰기

1. 현재 상태에서 쓸 수 있는 내용 위주로 빠르게 chapter 마무리
2. chapter 를 마무리 했음으로 심적 안정감
3. chapter 마무리 한 후 편집자에게 공유 피드백
4. 1차 회고
5. 2차 회고
##### 창업

1. 측정 가능 가설 세움
2. 실행 통해 소비자에게 피드백
3. 1차 개선
4. 2차 개선
5. 두 번째로 상품 개발 반복
6. 일정 목표를 달성만 하면 OK
#### TDD , 리팩토링의 핵심

- 큰 단위를 작은 단위로 나눠 지속적 피드백 통해 목표 + 개선
- 달성하기 힘든 것부터 생각하는 일에 도전할 수 있는 용기!
##### 성공한 사람과 실패한 사람의 차이

- 실패한 사람 : 성공 과 실패의 과정은 한번이라 생각 ( 일을 성공하거나 실패하거나 )
- 성공한 사람 : 실패를 과정으로 계속 지속해나감

=> 빠르게 실패하고 빠르게 피드백 받아나가자 ( TDD 를 너무 소프트웨어 적으로 생각 X - 인생에 적용해보는건 어떤가? )