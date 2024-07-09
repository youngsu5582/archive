---
tags:
  - 자바
추가_정보: 백기선 라이브 스터디
주차: "12"
---
### Annotation

소스코드에 메타데이터 를 추가하는 메커니즘

#### 왜 등장했는가?

소스 코드와 관련된 메타 데이터를 직접 코드 내 포함시키고 관리하기 위해서

이전에는 소스 코드 와 문서를 분리해서 관리
-> 코드 변경 후 문서 업데이트 하지 않아서 불일치 발생

설정 파일 , XML 등 별도 관리
-> 관리 복잡성 증가

=> 애노테이션을 통해 문서와 설정 정보를 코드와 함께 관리해나가자

- 컴파일러에게 문법 에러 체크하도록 정보 제공
- 컴파일 타임에 자동으로 코드 생상할 수 있도록 정보 제공 - Annotation Processor
- 런타임에 특정 기능 수행하도록 정보 제공 - 리플렉션

#### 정의

```java
public @interface 애노테이션 이름 {
	타입 요소이름();
}
```

- 타입에는 primitive , String , enum , annotation , class , 이들의 배열만 허용 - 사실상 전부 아닌가? ( Object 는 불가 )
- 매개변수 가질수 없다
- 예외 선언할 수 없다
- 요소를 타입 매개변수로 정의 할 수 없다

정의된 요소가 하나라면 요소 이름 생략 가능

```java
public @interface Annotation{
	String value();
}

@Annotation("SingleValue")
public class MyClass{}
```

#### 자바 표준 애노테이션

#### Override

해당 메소드가 오버라이딩 된 메소드 라는 것을 컴파일러에게 알려준다
-> 애노테이션이 달려 있을 시 부모의 메소드에 맞게 오버라이딩 해야한다

```java
public class Parent {
	void method1(){}
}
class Child extends Parent {
	@Override
	void method2(){
	}
}
```

>java: method does not override or implement a method from a supertype

해당 에러 발생

#### Deprecated

미리 만들어져 있는 클래스 나 메소드가 더 이상 사용되지 않는 경우
컴파일러에게 "더이상 사용하지 않으니 경고" 명시하게 함

```java
public class AnnotationDeprecated{
	@Deprecated
	public void notUse(){}
}
public void useDeprecated(){
	AnnotationDeprecated annotatoion = new AnnotationDeprecated();
	annotation.notUse();
}

```

notuse 는 ~~notUse~~ - 이렇게 밑줄 처리
#### SupressWarings

컴파일 할 때 경고를 안줘도 된다고 말해주는 경우

```java
class NewClass {
    @Deprecated
    void oldMethod() { }
}

@SuppressWarnings("deprecation") 
public static void main(String[] args) { 
	NewClass newClass = new NewClass();
	newClass.oldMethod(); 
}
```

deprecation 에 대한 경고 메시지를 무시

deprecation , unchecked , rawtypes , varargs 존재
#### FunctionalInterface

함수형 인터페이스 정의에 사용 - 람다 표현식 지원 위해 도입
컴파일러가 함수형 인터페이스 계약을 준수 해야한다고 알려줌 - 인터페이스 내 추상 메소드 하나만 존재

```java
@FunctionalInterface interface StringConcat { 
	String concat(String a, String b); 
}
```

#### SafeVarargs

- 자바 7에 도입
- 타입 안정성 문제 인한 경고가 ( 힙 오염 ) 발생하지 않는다고 컴파일러에게 알려줌
- final , static 메소드 , 생성자에만 이용 가능 - 자바 9부터는 private 역시도 가능!

```java
@SafeVarargs
private static void printLists(List<String>... lists) { 
	for (List<String> list : lists) { 
		System.out.println(list); 
	} 
}
```

왜 도입하는가?

제네릭 타입 정보가 런타임에 소거되므로
실제 메모리 저장은 List 객체 원시 타입
-> 어떤 종류 List 도 저장될 수 있음
