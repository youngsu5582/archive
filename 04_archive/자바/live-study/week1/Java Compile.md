---
tags:
  - 자바
추가_정보: 백기선 라이브 스터디
주차: "1"
---
### Java Compile

#### 1. 어휘 분석

- 키워드 , 리터럴 , 오퍼레이터 , 식별자 수집

```java
public class App
{
	public static void main(String[] args)
	{
		String str = "Study With! ";
		System.out.println(" Hello World" + str);
	}
}
```
해당 코드가 있을시
- 키워드 : public class
- 리터럴 : Hello World ( GPT 의 의견으로는 Study With! 도 포함 - 나도 이게 맞는듯 )
- 오퍼레이터 : +
- 식별자 : App , main , args ...
	=> 이들을 어휘소 라고 한다.

토큰 스트림 ( 어휘소를 하나의 스트림으로 만듬 ) 까지 뽑아내는 것을 어휘 분석이라고 한다.

public -> class -> App -> {  -> public -> static -> void -> main ... -> }
#### 2. 구문 분석

- 토큰 스트림을 통해 , 문법에 맞는지 확인
- 토큰 스트림 입력 -> 문법 규칙 -> 구문 트리 생성 -> 에러 처리

-> Parsing 통해서 AST - Abstract Syntax Tree ( 구문 트리 ) 생성
- 노드 : 문법의 구성 요소 ( 식 , 문장 , 선언 )
- 리프 노드 : 토큰 ( 식별자 , 리터럴 )
- 내부 노드 : 연산자 , 구문 구조 ( if 문 , for 루프 )

```shell
(ClassDeclaration)
         |
         +--(Modifiers) "public"
         |
         +--(Class Name) "App"
         |
         +--(MethodDeclaration)
             |
             +--(Modifiers) "public static"
             |
             +--(Return Type) "void"
             |
             +--(Method Name) "main"
             |
             +--(Parameters)
             |   |
             |   +--(Parameter Type) "String[]"
             |   |
             |   +--(Parameter Name) "args"
             |
             +--(Method Body)
                 |
                 +--(VariableDeclarationStatement)
                 |   |
                 |   +--(Type) "String"
                 |   |
                 |   +--(VariableDeclarator)
                 |       |
                 |       +--(Variable Name) "str"
                 |       |
                 |       +--(Variable Initializer) "Study With! "
                 |
                 +--(ExpressionStatement)
                     |
                     +--(MethodInvocation)
                         |
                         +--(Expression Name) "System.out"
                         |
                         +--(Method Name) "println"
                         |
                         +--(Arguments)
                             |
                             +--(String Literal) " Hello World"
                             |
                             +--(Expression Name) "str"

```

이렇게 구성

구문 분석을 하는데 문법이 틀릴 시? -> Synatx Error
- 일반적으로 세미클론 ( ; ) 누락 , 괄호 ( { or }) 생략 등등

#### 3. 의미 분석

- 의미가 맞는지 검사 -> SynmaticAnalysis Error

- 타입 검사 : 타입 규칙 준수하는지 확인
- 변수 & 함수 선언 검사 : 모든 식별자 적절 범위 내 선언되는지 확인
- 이름 바인딩 : 각 식별자가 하나의 유일한 개체 참조하는지 확인
- 유효성 검사 : 루프 , 조건문 등이 올바르게 사용되었는지 확인
- 흐름 제어 검사 : 모든 코드 경로가 반환 값을 가지고 있는지 확인
- 메모리 접근 검사 : null 참조 , 범위 벗어난 배열 인덱스 접근하는지 확인

#### 4. 바이트 코드 생성


모든 검사가 끝난 코드를 중간 단계인 바이트 코드로 생성
