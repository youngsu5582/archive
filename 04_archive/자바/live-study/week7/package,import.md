---
tags:
  - 자바
추가_정보: 백기선 라이브 스터디
주차: "7"
---
### package

클래스를 묶는 하나의 디렉토리
클래스를 유일하게 만들어주는 식별자 역활

클래스의 전체 이름은 : 패키지명 + 클래스
-> FQCN ( Fully Qualified Class Name )
( String 클래스 명: java.lang.String )

.(도트) 를 사용해 표현
비슷한 자바 클래스 모아서 폴더 단위 관리하므로 유지 보수 용이

### 명명 규칙

- 패키지 이름은 모두 소문자
- 자바 예약어 사용 불가 ( int , static 등등 )

### 빌트 인 패키지

자바 표준 라이브러리 ( Java STL ) 로 제공해주는 패키지

java.lang, .util , .io , .net 등등
```java
package java.lang;
```

파일 제일 상단에 속해있는 package 선언

### Import 

소스코드를 짜면 필연적으로 다른 패키지 내 클래스를 사용해야만 함
-> 매번 패키지명.클래스명 을 붙혀 사용하는건 매우 비효율적 

=> import 문을 활용해 클래스 패키지를 미리 명시해주면 패키지명 생략 가능!

```java
import packagename.classname;

import packagename.*;

import static packageName.StaticClassName.staticMember;
```

패키지명.* 은 지정 패키지에 속하는 모든 클래스를 사용 가능

static 함수는 일반적으로 Assertion..asssertEqual() 이렇게 사용한다.
-> 이 Assertion 을 생략하고 싶으면 import static

