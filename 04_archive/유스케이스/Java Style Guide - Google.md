---
tags: 유스케이스
---

## 해당 규칙을 준수하는 경우에만 Google 스타일로 설명
## 2. 소스 파일 기본 사항

#### 파일 이름
- 소스 파일 이름은 포함 된 최상위 클래스의 대소 문자 구분 이름과 .java 확장자로 구성  ( Application.java)
#### 파일 인코딩
- 소스 파일은 UTF - 8 로 인코딩

#### 특수 문자들
##### 공백 문자
- 특별 공백 문자들만 이스케이프 처리가 되고 , 다른 문자들은 이스케이프 처리 되지 않는다
- Tab 은 들여쓰기에 사용되지 않는다.
##### 특수 이스케이프 문자들
- 특수 이스케이프 문자들 ( \\b \\t \\n \\f \\r \\" \\'  \\\ ) 에만 이스케이프 처리가 된다.
- 해당 옥텟(\\012 ) , 유니코드(\u000a) 이스케이프 대신 해당 문자가 사용 된다.

##### Non-ASCII 문자
- 유니코드 문자 와 유키노드 이스케이프 문자를 다룰때 아래와 같은 규칙을 따름
1. 그대로 유니코드 문자 사용 : 사실상 가장 이상적
```java
String unicodeSymbol ="∞";
```
2. 프린트할 수 없는 문자 ( 바이트 오더 마크 ) 사용 : 프린트 할 수 없는 문자에 대해 다룰 때는 이스케이프를 사용하고 주석으로 설명하는 것을 권장
```java
return '\ufeff' + content // byte order mark
```

## 3. 소스 파일 구조
1. 라이센스 나 저작권 정보 ( 존재한다면 )
2. 패키지 구축 ( 당연히 하나만 선언 )
3. Import 구문 ( 다른 패키지 사용 클래스나 인터페이스 사용하기 위해 사용 )
4. 정확히 하나의 최상위 클래스 ( 중첩 클래스 사용하는 )

### Example
```java
// 1. 라이센스 나 저작권 정보
/*
 * License: MIT License
 * License URL: https://opensource.org/licenses/MIT
 */


// 2. 패키지 구축
package com.example.package1;

// 3. Import 구문
import java.util.List;
import java.util.Stack;

public class MyClass{

}


```
### 패키지 구문
- 줄바꿈 하지 않음 , 열 제한은 패키지 구문에 적용되지 않음
### 임포트 구문
- 줄바꿈 하지 않음 , 열 제한은 임포트 구문에 적용되지 않음

- 와일드 카드 를 통해 import 하지 않는다 ( 명시적 import 를 권장 )
#### Why?
- 편리하나 , 일반적으로 추천 되지 않는다. 명시적 import 를 권장하는 이유- 
1. 가독성 : 해당 클래스들이 어디서 왔는지 명확하게 파악 가능
2. 충돌회피 : 서로 다른 패키지에 동일 이름 클래스 있는 경우 충돌 발생 가능
3. 코드 최적화 : 모든 클래스 가져와야 하므로 불필요한 클래스 포함 가능성 존재
4. 코드 유지보수 : 유지보수 사용하지 않는 import 문 식별 및 삭제가 쉬움.


- static 같은 것들을 사용하지 않는다 ( 위 와일드 카드와 비슷한 이유 )
- 임포트 순서 및 공간은 단계에 맞춰서 작성한다.
1. 하나의 블럭 안에 static 임포트 포함
2. 하나의 블럭 안에 non-static 임포트 포함

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

import static java.lang.Math.PI;
import static java.lang.Math.abs;

import org.example.MyClass;
import org.example.MyInterface;
```

### 클래스 구문 
- 소스 파일마다 각자 최상위 클래스 존재


- 클래스 안 멤버들과 초기화 순서는 학습 용이성에 영향을 준다.
- 물론 , 하나의 완벽한 방법은 없으므로 각자 방법대로 구성함

##### 클래스 본문 순서 정하기
- 중요한 것은 논리적 순서를 따라서 , 유지보수자가 질문을 받았을 때 설명을 할 수 있어야 한다.
	( 흔히 , 개발자들이 실수하는 것 중 하나가 새로운 메소드 들을 클래스 끝에 붙이는 것이지만 생각해보면 단지 그것은 "연대기적 순서" 이지 논리적 순서는 아니기 때문이다. )

일례로 , 오버로딩된 생성자들 역시 다른 코드들 없이 차례로 나타나져야 한다. ( private 멤버라 할지라도 )

## 4. 포매팅
##### 용어 노트
	block-like construct : 블럭과 같은 구조 , 괄호로 나타내어지는 구조
	- 후에 나오는 배열 초기화 블럭 역시 블럭 과 같은 구조로 간주

##### 괄호
- 괄호는 선택사항에서 쓰인다.
- if , else 구문에서 몸체가 없거나 한 줄 인 경우에도 괄호를 사용하자! -> 가독성

##### K & R 스타일
- 비어있지 않은 블럭 + block-like construct 에서 Kernighan & Ritchie 스타일을 따라야 함
```java
return () -> {
	while (condition()) {
		doSomething();
	}
}
```
- 여는 괄호 앞에 줄 바꿈 없음! 
- 여는 괄호 다음에 줄 바꿈
- 닫는 괄호 전 줄 바꿈
```java
	if(condition()){
		try{
			something();
		} catch(Exception e){
			recover();
		}
		} else if(){
	}

```
- 닫는 괄호 다음 줄 바꿈 ( 단 , 구문 끝나거나 메소드 생성자 , 클래스가 끝난 경우 <-> else & 콤마 뒤에 나오는 부분 줄 바꿈 X )
- enum Class 에는 예외가 있음

##### 블럭 들여쓰기 : +2 스페이스
- 새로운 블록 및 block-like construct 가 열리면 스페이스 2번 들여쓰기
- 블럭이 끝나면 , 들여쓰기 이전의 들여쓰기 단계로 돌아감 ( Stack 의 느낌 )

#### 열 제한 : 100자
- 자바 코드는 100개의 문자의 제한
- 이 숫자를 넘어가면 줄 바꿈을 해야함
- 유니코드 포인트로 인해 길게 써야 한다면 규칙보다 일찍 개행하는 것을 추천
##### 예외
1. 개행이 불가능한 경우 ( Javadoc 에서 참조하는 긴 URL 같은 경우 )
2. package 나 import 구문들
3. 쉘에 복사 붙여넣기 되는 커맨드 라인 대한 주석

#### 줄 바꿈  ( 중요하지 않다고 생략하여 부분 생략 )

##### 용어 노트
	코드가 하나의 줄에서 여러 줄로 바뀔시 줄 바꿈 ( 개행 )이라고 함

- 위에 말한 행 제한을 넘지 않기 위해서 함
- 줄 제한에 걸리지 않더라도 저자의 재량에 따라 줄 바꿈 가능

! 함수나 변수를 빼내는 방법을 사용해 줄 바꿈을 대신해보자
##### 언제 바꾸는가
- 원칙적으론 , 높은 문법 레벨에서 바꾸는 것
1. non assignment 연산자 에서 줄 바꿈이 일어날 경우 기호 이전에 위치한다.
```java
int result = a +
	b +
	c;
```
- 보면 + 다음이라 생각할 수 있지만 , 연산이 되기 전에 바꾼다는 뜻인듯()
	-원본 : When a line is broken at a _non-assignment_ operator the break comes _before_ the symbol.
	- dot ( . )
	- 2개의 콜론 ( :: )
	- 타입 바운딩 엠퍼센드 기호 ( & )
	- catch 블럭의 파이프 ( catch )


#### 변수 선언
- 매 변수 초기화는 ( 필드 or 지역 ) 하나의 변수만 초기화 - int a; b; 는 불가능
	-- 물론 for Loop Header 에는 여러 변수 선언 가능
##### 필요하면 정의
- 지역변수는 블럭이나 block-lke construct 가 시작될 때 쓰일 필요 X ( 처음 사용 되는 가까운 위치에 선언 )




#### 배열
##### 배열 역시 block-like
```java
new int[]{
  0,1,2,3
}

new int[] {
  0, 1,
  2, 3
}

new int[] {
  0,
  1,
  2,
  3,
}

new int[]
	{0, 1, 2, 3}
```
- 이 네가지 방법 모두 가능
- 단 C 언어 스타일 배열 선언문 쓰지 말 것 ( String[] args or String args[] 불가능)

#### Switch 구문
- switch 블럭 들여쓰기 역시 +2 이다.
- switch 라벨 이후 , 개행 온 후 , 들여쓰기는 +2
##### default 케이스가 존재한다
- 각 switch 구문은 default 그룹에 존재 ( 그룹이 코드를 포함하고 있지 않더라도 )
	-- enum 타입의 switch 구문은 다른 모든 경우 처리 다 했을 시 생략 가능 ( IDE 단에서도 경고 가능 )

#### 애노테이션
- documentation 블럭 바로 이후 클래스나 함수 혹은 생성자에 적용
- 각 애노테이션 들은 그들 만의 줄 가짐 즉 애노테이션은 한 줄 소유 - 그래서 들여쓰기 레벨 증가도 역시 X
```Java
@Override
@Nullable
public String getNameItPresent(){ ... }
```
- 예외 : 파라미터 없는 단일 애노테이션은 한줄 쓰기 가능
```java
@Override public int hashCode() { ... }
```
- 필드 적용 애노테이션들은 한 줄 쓰기 가능
```java
@Partial @Mock DataLoader loader;
```

#### 주석

- 블록 주석은 주변 코드와 동일 수준에서 들여 쓰기
- /* ... */ 스타일 이나 , ... 스타일 존재

#### 접근 제한자
- 클래스 및 멤버에 접근 제한자는 Java 언어 사양에서 권장하는 순서로 나타냄

```java
public -> protected -> private -> abstract -> default -> static -> final -> transient -> volatile -> synchronized
-> native -> strictfp
```

- 해당 접근 제한자에 대해서는 차후 다시 작성

#### 숫자 리터럴
- long 값 가지는 정수 리터럴은 소문자가 아닌 대문자 접미사 사용 ( 숫자 1 과 l 의 혼동을 피하기 위해 )

### 5. 식별자
#### 모든 식별자에 공통적 규칙
- ASCII 문자와 숫자만 사용하며 _ 를 사용하여 표시도 함
	( 이에 따라 , 각 식별자 이름은 \\w + 정규식과 일치 ) - \\w 는 워드 문자 , +는 앞 패턴이 하나 이상 일치해야 함. => 하나 이상 워드 문자열 패턴
- Google Style 에서는 특수 접두사 or 접미사 사용 X
```java
name_ ❌ , mName ❌ , s_name ❌ , kName ❌
```
#### 식별자 유형별 규칙
##### Package 이름
- 패키지 이름은 모두 소문자이며 , 연속 된 단어는 단순히 함께 연결 ( 밑줄 X )
```java
com.example.deepSpace ❌ , com.example.deep_space ❌
```
##### Class 이름
- 클래스 이름은 UpperCamlCase
- 일반적으로 명사 or 명사구
=> Character , ImmutableList 등
	( Interface 는 이름이 명사 , 명사구 일수 있지만 형용사 , 형용사 구도 가능  - List , Readable)
- 테스트 클래스 이름은 테스트 중인 클래스 이름으로 시작하고 , Test 를 끝에 붙인다.

##### Method 이름
- lowerCamelCase 로 작성
- 일반적으로 동사 or 동사구
=> sendMessage or stop
- JUnit 테스트 메소드 이름은 밑줄로 이름 과 논리적 구성 요소 구분 가능
=> testLoginFailure_invalidCredentials ( 앞은 어떤 메소드를 , 뒤는 검증할 상태를 )

##### Constant 이름
- 상수 이름은 CONSTANT_CASE : 모두 대문자 , 밑줄 _ 로 각 단어 구분
- 내용은 절대적 불변하며 , 메소드에 감지 가능한 부작용 없는 정적의 최종 필드
- primitives , String , immutable types , immutable collections of immutable types 포함
	( 쉽게 말해서 , 그냥 선언 초기화 한 후 절대 바뀔일 없음 , 그렇기에 프로그램도 알고 있음 )
##### Field 이름
- lowerCamelCase 로 작성
- 일반적으로 명사 , 명사구
##### Paremeter 이름
- lowerCaelCase 로 작성
- 공용 메소드에서 한 문자 파리머티 이름은 피해야함

##### Type 변수 이름
- 각 유형 변수는 다음 두 가지 스타일 중 하나로만 이름 지정
- 단일 대문자 or 단일 숫자 따라올 수 있음 ( T , E , T2 )
- 클래스에 사용되는 형식 이름 뒤에 대문자 T ( RequestT , FooBarT )

#### 카멜 케이스 : 정의 된 단어는?
- 두문자어 or "IPv6" , "iOS" 같은 비정상적 구조에 대해 카멜 케이스로 변환은 어떻게 해야하는가?
##### 결정론적 체계
1. 구를 일반 ASCII 변환 후 어퍼스트로피 제거 ( Müller's algorithm -> Muellers algorithm ) : 유니코드 제거 , ( ' ) 제거
2. 단어로 나누고 공백과 나머지 구두점( 일반적 하이픈(-) ) 으로 분리 ( AdWords -> ad words )
3. 모든 것을 lowercase 후 , 첫 문자만 대문자 표시 
4. 모든 단어 단일 식별자 결합 ( 원래 단어 대소 문자 완전히 무시 )
```java
XML HTTP request -> XmlHttpRequest
supports IPv6 on iOS? -> supportsIpv6OnIos
```

### 6. 프로그래밍 실습
##### @Override
- 메소드를 재정의 할 때 @Override 를 항상 사용하자
- 컴파일러가 해당 메소드가 실제 부모 클래스나 인터페이스 메소드 재정의 하는지 확인 가능
- 부모 메소드가 @Deprecated 일 경우에는 생략 가능
##### Caught exceptions
- 이 규칙은 예외 처리 코드에서 예외 무시하지 않고 , 적절히 처리하게 권장
```java
try {
    int i = Integer.parseInt(response);
    return handleNumericResponse(i);
} catch (NumberFormatException ok) {
    // 예외: 입력이 숫자가 아닌 경우
    // 그냥 계속 진행
}
return handleTextResponse(response);
```
- 이렇게 catch 문에서 아무것도 안하는게 옳을 시 , 주석으로 설명하는 것을 권장
	( 테스트 중 포착된 예외는 주석 없이 무시도 가능 - 편의성 )
##### Static members
- 정적 클래스 멤버에 대한 참조를 해야할 시 , 해당 클래스 이름으로 정규화 ( Foo.static )
##### Finalizers : 사용 X !
- finalize 는 객체 소멸자 메서드를 나타내는 특별한 메소드 ( 가비지 컬렉터에 의해 수집 되기 전 호출 - 추가 작업 기대 가능 )
	- 호출 시점 불확실성 : 언제 수집될 지 정확히 예측 불가능
	- 성능 저하 : 호출될 때마다 추가적 작업 수행되므로 성능 영향 미침!
	- 대안 : 더 좋은 방법들이 있음
		=> 이런 단점들로 사용하지 말자!

### 7. JavaDoc
#### Formatting
```java
/**
* 여러 줄 JavaDoc 텍스트
* 일반적 래핑
*/
public int method(){}
```
- 위와 같은 일반적 형태
- 기본 형식도 허용 가능 ( /** 특히 짧은 */ 이렇게도 가능 )
	-=> @return , @param 같은 블록 태그가 없을 경우에만 적용 가능
##### 문단
```java
/**
 * Javadoc 의 첫번째 문단
 *
 * 두번째 문단 , 빈 줄로 구분
 *
 * And here's the third paragraph.
 */
```
- 빈줄로 구분

##### 블록 태그
- 하단 네가지는 설명과 함께 표시해야함.
```java
    /**  
     * @param - 메소드의 매개 변수에 대한 설명 제공
     * @return - 메소드가 반환하는 값에 대한 설명 제공
     * @throws or @exception - 메소드가 던질 수 있는 예외에 대한 설명 제공 
     * @deprecated - 메소드 , 클래스가 더 이상 사용되지 않거나 권장 되지 않을 때 표시
     */
```
- 블록 태그가 한줄로 설명 못할시에는 연속 줄은 @위치에서 4번 들여쓰기
```java
/**
 * Calculates the sum of two numbers.
 *
 * @param num1 The first number.
 *             This is the explanation of num1.
 */
 public int add(int num1, int num2){
 }
```
##### summary fragment
- 간단한 요약 조각 으로 시작 ( 매우 중요 )
- 클래스 & 메소드 인덱스에 같은 특정 컨텍스트에 나타나는 텍스트 유일한 부분
- 완전한 문장이 아니라 , 명사구 또는 동사 구인 단편

#### Javadoc 의 사용 장소
##### 예외
- 단순하고 명백한 방법에 대해서는 선택 사항 ( getFoo - 누가봐도 Foo를 반환 )
- 독자가 알아야 할 관련 정보에 대해서는 작성해야함!
	( getCanonicalName - Canonical Name 이 무엇인지 모를수도 있으므로 )
- 수퍼 타입 메소드 오버라이딩 하는 메소드에는 항상 존재하는 것은 아님.

