---
tags:
  - 자바
추가_정보: 백기선 라이브 스터디
주차: "2"
---
Java 의 데이터 타입은 Primitive ( 원시 ) Type 과 Reference ( 참조 ) Type 두가지가 있다.

우선 , 원시 타입에 대해서만 설명한다.

### 정수 타입

5개의 타입으로 정해져 있다.

#### Byte

- 8 bit = 1 byte
- -128 ~ 127 ( 0 때문에 127 까지 )
- 간단한 숫자를 저장할 때 사용

1 0 0 0 0 0 0 0 = -128 ~
0 1 1 1 1 1 1 1 1 = 127

( 앞이 1 이면 음수 , 0 이면 양수 )

- 값의 범위를 벗어나면 , 컴파일 에러가 발생 ( byte error = 128 -> Compile Error )
- 값의 최대값에서 +1 을 할 시? : 음수 최소값으로 변경
```java
    byte localB;  
    public void test() {  
        byte x;  
        System.out.println("b = " + localB);  
        System.out.println("x = " + x);    // Error!
    }
```

초기화 되지 않은 Class Local 변수는 0 으로 자동 초기화
초기화 되지 않은 Method Local 변수는 에러 발생!

#### Char

- 16 bit = 2 byte
- 0 ~ 65,535 ( 부호 없는 Unsigned 정수 )
- 모든 문자를 저장할 때 사용 ( 이모티콘 역시 포함 가능 ) - 모든 유니코드 저장 가능
	-> 44032 ~ 55203 한글 범위 ( 가 = 44032 )

- 최대값에서 +1을 할 시 ? : 0으로 초기화

초기화 되지 않은 값은 \u0000 으로 자동 초기화 ( 10진수는 0 으로 기본값 )

#### Short

- 16 bit = 2 byte
- -32,768 ~ 32,767 ( 부호 있는 Signed 정수 )
- 작은 숫자를 사용할 떄 사용 

- C언어 , Low 레벨 , 임베디트 코드 에 사용하는 용

#### Int

- 32 bit = 4 byte
- -2,147,483,648 ~ 2,147,483,647 ( -2^31 ~ 2^31 -1)
- 가장 많이 사용하는 원시 타입

int = int + short ( O )
shrt = int + short ( X )
short = short + short ( X )
=> byte , short , char 타입은 산술 연산 사용시 암시적으로 형 변환 ( Up-Casting ) - int 로 변환
( 오버플로우 , 언더플로우 방지 + 높은 정밀도로 연산 수행하기 위해 )
#### Long

- double 과 같이 가장 큰 메모리 양 차지
- 64 bit = 8 byte
- 9,223,372,036,854,775,808 ~ 9,223,372,036,854,775,807 - 어마어마한 큰 값
- DB 칼럼과 자바 필드 매핑때 많이 사용
```java
long l = 99999L;
long l2 = 9999l;
```
-> 소문자 l 이나 대문자 L로 Long 으로 명시

### 실수 타입

- 부동 소수점 아래에 이루어짐
#### Floating Point
- 부동 소수점
###### Example
1.2345
부호 : +
기수(m) : 0.12345
곱하기(x) : x
지수(exp) : 10^1
= 0.12345 * 10 = 1.2345

- 유동적인 소수 표현을 가능하게 해준다.

#### Float

- 32 bit = 4 byte
```java
float f = 0.123f;
float f2 = 1.37F;
```

#### Double

- 64 bit = 8 byte
```java
double d = 0.1234d;
double d2 = 1.5823D;
```

#### Float VS Double

대부분 Double을 많이 사용
- float 는 7자리 까지 정밀도 ( +- 3.40282347 * 10^38 범위 표현 가능 )
- double 은 15자리 까지 정밀도 ( +- 1.7976931348623157 * 10^308 범위 표현 가능 )
=> 상대적 풍부

### 논리 타입

#### Boolean

- 1byte 표현 - true , false 로 표현
- 태초의 C언어에는 Boolean 이 없었다 - `#define FALSE 0 ` 처럼 선언해 사용

