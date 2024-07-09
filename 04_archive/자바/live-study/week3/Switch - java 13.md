---
tags:
  - 자바
추가_정보: 백기선 라이브 스터디
주차: "3"
---
Switch 문은 필수적인 존재
-> Switch 문이 없으면 if / else if / else 문을 남발할 수 밖에 없음

switch 에는 byte , short , int , char , enum , String 가능
### Switch - Before java 13

```java
```java
switch (foo) {
    case 0:
    case 1:
    case 2:
        System.out.println("0, 1, 2일 때 실행");
        // - > fall through
    case 3:
        System.out.println("0, 1, 2, 3일 때 실행");
        break;
    default:
        System.out.println("그 외 경우에만 실행");
        break;
}
```

```java
int date = 0;  
switch (day) {  
    case "MONDAY":  
    case "FRIDAY":  
    case "SUNDAY":  
        date = 5;  
        break;    
    case "NOT":  
        date = 10;  
        break;    
    default:  
        date = -1;  
}
System.out.println(date);
```
#### 기존의 문제점

- 제한된 표현식 강조 : 불필요한 표현식 남발 ( case , break , default 등등 .. )
- fallthrough : break 문을 사용하지 않으면 , 다음께 진행되는 fall through 발생
- switch 문을 통한 결과 저장 : 변수를 하나 만들고 , 해당 변수를 조작해야 했음
- 디버깅 시 추적 어려움 : StackTrace 에 충분한 정보 제공하지 않을 수 있음 + fall through 때문에 의도하지 않은대로 작동 가능
### Switch - java 13

Lambda 를 활용한 표현식 간결화

```java
int lastDate = switch (month) {
    case 1, 3, 5, 7, 8, 10, 12 -> 31;
    case 2 -> !isLeapYear ? 28 : 29;
    case 4, 6, 9, 11 -> 30;
    default -> throw ... ;
};
```

- 화살표 함수를 통해 , 매우 간결하게 표현 가능
- 대입 연산자를 통해 변수를 안전하게 표현 가능

#### Yield

변화된 switch 문에서는 기존 문과 중복해서 사용은 불가능하다

그러면 , 복잡한 값들을 수행한 후 , 특정 값을 리턴하려면?
-> yield 를 사용하자

```java
int lastDate = switch (month) {
    case 1, 3, 5, 7, 8, 10, 12 -> 31;
    case 2 -> {
        // 올해가 윤년인가(true or false)
        boolean isLeapYear = 
            (year % 4 == 0)
            == (year % 100 == 0)
            == (year % 400 == 0);
        
        // 예시에서는 삼항 연산자의 결과를 반환합니다.
        yield !isLeapYear ? 28 : 29; // yield 28; 또는 yield 29;
    }
    case 4, 6, 9, 11 -> 30;
    default -> throw ... ;
};
```

해당 yield 를 통해 , 값을 반환 하며 break 효과 기대 가능

### Pattern Matching - java 21

- Java 21 에서 도입

```java
if (obj == null) {
    
} else if (obj instanceof SubType1) { // [1] type 체크
    SubType1 s1 = (SubType1) obj; // [2] 캐스팅
    // ... [3] 사용
} else if (obj instanceof SubType2) {
    SubType2 s2 = (SubType2) obj;
    
} else if (obj instanceof SubType3) {
    SubType3 s3 = (SubType3) obj;
    
} else {
    // ...
}
```

기존에는 타입에 따라 , 분기 구조를 선택할 때 switch 문이 불가능했으므로 매우 비효율적 이였다

```java
switch (obj) {
    case SubType1 s1 -> [실행문]; // s1을 사용할 수 있음.
    case SubType2 s2 -> [실행문]; // s2를 사용할 수 있음.
    case SubType3 s3 -> [실행문]; // s3를 사용할 수 있음.
    case null -> [실행문];
    default -> [실행문];
}
```

패턴 매칭을 통해 매우 효율적으로 표현 가능

