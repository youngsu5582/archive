---
tags:
  - 스프링
테크톡_링크: https://www.youtube.com/watch?v=1WT6oxchM9M
테크톡_발표자: 김지헌
---
### 간단 설명

- 시니어 개발자가 자신이 겪은 스프링에 대한 변천사 소개
- Java 8 에서 17까지의 사용할만한 Update 소개
- 스프링 프레임워크 6에서 주요 변화점 소개

### Maven vs Gradle

- 당시 , 가독성 적인 면에서 gradle 선택

### application.yaml vs application.properties
##### properties
- 열거형으로 쭈욱 나열 (env 느낌으로 key-value)
- java 기존부터 사용되어 온 방식

##### yaml
- 계층형 속성 정리로 인한 가독성 때문에 yaml 결정

### 스프링 부트 따라잡기
- 기본 Spring 이 유지보수 단계로 넘어가며 , Spring Boot 는 활발히 발전중
- 변화를 원활히 확인하자

#### 스프링 새 기능 관련
https://spring.io/blog/category/releases/ - Spring 공식 Blog
https://github.com/spring-projects/spring-boot/wiki - Spring Boot 공식 Github Wiki

#### 새로운 기술 접목
https://github.com/springrunner/fastcampus-class-201 - 결국 , 변경사항을 토이 프로젝트에 접목시키는게 베스트!

### 스프링 부트란?
- Java 17 ( 20 까지 호환은 가능 )
- Jakarta EE 10
- Spring Framework 6.0 이상
- Build : Grade 7.5 이상 , 8 / Maven 3(3.5)

### Java 10~17 Good Updates
- 8 이후로 , 좋은 기능들이 많이 생성 되었음
#### Java 10 : Local variable Type Interface

```Java

// Origin
List<Hobby> h =new ArrayList<Hobby>();

// Good
var hobbyList = new ArrayList<Hobby>();
```
- 간편해진 만큼 변수명은 유용하고 의미있게
- 당연히 , Local 에서만 가능 ( 바로 선언후 초기화가 가능하니까 - 타입 추론 가능한 충분한 정보 제공)

#### Java 14 : Switch Expression

```Java
// Origin
Day day = Day.WEDENSDAY;
int numLetters=0;
switch(day){
	case MONDAY:
	case TUESDAY:
	case WEDNESDAY:
		numberLetters=7;
		break;
	case THURSDAY:
		numberLetters=8;
}

// Good
var numLetters = 0;
numLetters = switch(day){
	case MONDEY,FRIDAY,SUNDAY -> 6;
	case THURSDAY -> 7;
}

```
- Switch 문을 간결하게 변경해줌.
- IDE 를 통해 , enum 과 switch 를 활용할 시 자신이 빠뜨린게 없는지 체킹도 가능


#### JAVA 15 : Text Blocks

```java

var OldLiteralString = "Mr Kim";
var NewLiteralString = """
Mr Kim""";

Old == New
// True 가 나온다.

String OldJson = "{\n" + " \"name\" : \"Young su\",\n" + " \"age\" : 25\n" + "}";

String NewJson = """
{
 "name" : "Young su",
 "age" : 25
}
""";

```
- 새로운 문자열 리터럴 형식
- 기존에는 줄 바꿈 문자 사용하거나 문자열 연결하는 방식을 사용했음
	-> 긴 문자열 더 쉽게 작성하고 읽게 해줌.
- 기존에 비해 , json 만들기 매우 쉬워짐 ( 물론 매우 큰 JSON 은 Resource 로 만들어서 호출하자 )

#### Java 16 : Record

``` java
record Rectangle(double length,double width){}
//=> 아래와 동일

public final class Rectangle {
 private final double length;
 private final double width;
 public Rectangle(double length, double width) {
 this.length = length;
 this.width = width;
 }
 double length() { return this.length; }
 double width() { return this.width; }
 // Implementation of equals() and hashCode(), which specify
 // that two record objects are equal if they
 // are of the same type and contain equal field values.
 public boolean equals...
 public int hashCode...
 // An implementation of toString() that returns a string
 // representation of all the record class's fields,
 // including their names.
 public String toString() {...}
}
```
- record 선언 시 자동 생성
	- 접근자
	- 생성자
	- equals
	- hashCode
	- toString
- 괄호 안에는 static Method 로 추가 함수 생성 가능 
- 용도? : final 필드 간결히 생성 , DTO 쉽게 생성

#### Java 16 : Jackson 직렬화
- 2.12.3 에서 Record 직렬화 자동 지원 ( 기존에는 복잡했음 )
	-> https://docs.oracle.com/en/java/javase/16/language/records.html#GUID-6699E26F-4A9B-4393-A08B-1E47D4B2D263 참고

#### Java 16 : Pattern matching instanceof


```Java

// Old
	if (shape instanceof Rectangle){
		Rectangle r = (Rectangle) shape;
	}
	else if(shape instanceof Circle){
		Circle c = (Circle) shape;
	}

// New

	if (shape instanceof Rectangle r){
	}
	else if (shape instanceof Circle c){
	}

```
- instanceof 가 true 인 범위 내 사용가능!
- instacne 의 type 을 확인하고 , 해당 타입 변수 선언 및 할당 하는 방법 제공

#### Java 17 : Sealed class

``` Java
public sealed class Shape permits Circle, Sqaure , Rectangle {

}

```
- 다른 Class 는 확장하려고 할 시 , 컴파일 오류 발생!
- API 안정성 및 계층 구조 제어 등 효과 기대 가능
- 사실상 잘 안씀

그 후에는 Spring 버전 변화에 따른 주요 체인지 기능들 나열

#### 정리 ( 사담 )
- Java 17 로 올라가고 신기능이 생기며 코틀린으로 옮길 필요성 에 대한 의문 가지고 있음

#### 개인 느낀점
- Java 스러움에서 벗어나려고 하는 노력들이 보이는 Java 8 -> Java 17
- nest framework 와 또 다른 점들이 매우 많음 ( 편리한 기능 record , 다양한 Volume 의 라이브러리 )