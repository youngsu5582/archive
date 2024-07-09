---
tags:
  - 자바
아티클_링크: https://techblog.woowahan.com/2527/
아티클_작성자: 향로
추가_정보: 우아한형제들 테크 블로그
---
우선 , Enum 의 장점?

- 문자열과 비교해 , IDE 의 적극적 지원 받기 가능 ( 자동완성 , 오타검증 , 텍스트 리팩토링 ... )
- 허용 가능 값 제한
- 리팩토링시 변경 범위 최소화 - Enum 코드외 수정할 필요 X

뿐만 아니라 , C/C++ 의 경우 int 였지만 , Java 는 완전한 기능을 갖춘 class

### 데이터들 간 연관관계 표현

#### 기존 문제점

배치를 돌며 하나의 테이블에 있는 내용을 2개 테이블에 등록해야 하는 기능
origin 테이블의 값은 "Y" , "N" 인데 , table1 / table2 는 "1"/"0" , true/false 형태

```java
public class Legacy {
	public String toTable1Value(String originValue){
		if("Y".equals(originValue)){
			return 1;
		}
		return 0;
	}
	public boolean toTable2Value(String originValue){
		if("Y".equals(originValue)){
			return true;
		}
		return false;
	}
}
```

기능상으로는 구현 가능하나

- Y , 1 , true 는 모두 같은 의미
-> 이를 알려면 , 항상 위 선언된 클래스 & 메소드를 찾아야만 함

- 불필요한 코드량 생산
-> Y , N 외 R , S 추가 값 필요할 시 if 포함 메소드 단위 코드 중가 - 반복성 코드 발생

#### Enum 구현

```java
public enum TableStatus {

	Y("1", true),
	N("0", false);

	private String table1Value;
	private String table2Value;

	TableStatus(String tableValue, boolean table2Value) {
		this.table1Value = table1Value;
		this.table2Value = table2Value;
	}

	public String getTable1Value(){
		return table1Value;
	}
	public boolean isTable2Value(){
		return table2Value;
	}
}
```

추가 타입이 필요하더라도 , Enum 상수와 getter 로 구현 가능 ( Lombok 을 통해서 더욱 깔끔하게 가능 )

```java
public void test(){
	TableStatus origin = selectFromOriginTable();

	String table1Value = origin.getTableValue();
	boolean table2Value = origin.isTable2Value();
}
```

TableStatus 만 받으면 , table 에 맞는 값 얻을 수 있음

### 상태 , 행위 한곳에서 관리

서로 다른 계산식 적용해야 할 필요 존재

DB에서 code 값에 따라 연산 수행할 시
CALC_A 일시 값 그대로
CALC_B 일시 * 10 한값
CALC_C 일시 * 3 한값

```java
public static long calculate(String code, long originValue){
	if("CALC_A".equals(code)){
		return originValue;
	}
	else if("CALC_B".equals(code)){
		return originValue * 10;
	}
	else if("CALC_C".equals(code)){
		return originValue * 3;
	}
	return 0;
}
```

```java
public void test(){
	String code = selectCode();
	long originValue = 10000L;
	long result = LegacyCalculate.calculate(code, originValue);
}
```

서로 관계가 있음을 코드로 표현 불가능
-> code 에 따라 지정 메소드에만 수행해야 하나 불가능
( 히스토리를 모를 시 , 실수할 확률이 높음 )

- 똑같은 기능 중복 생성 가능
- 화면 계산 로직 변경 시 , 신규 인력은 2개 메소드 로직 전부 변경해야 하는지 해당 화면만 변경해야하는지 파악 불가능
- 계산 메소드 누락 가능 - 문자열 과 메소드로 분리 되어 있으므로

>DB의 테이블에서 뽑은 특정 값이 지정된 메소드와 관계가 있다

해당 내용을 구두로만 표현하는건 매우 이상한 방법

```java
public enum CalculatorType {

	CALC_A(value -> value),
	CALC_B(value -> value * 10),
	...;

	private Function<Long, Long> expression;

	CalculatorType(Function<Long, Long> expression){
		this.expression = expression;
	}
	public long calculate(long value){
		return expression.apply(value);
	}
}
```

Code 가 본인만의 계산식을 가지게 구현

```java
@Column
@Enumerated(EnumType.STRING)
private CalculatorType calculatorType;
```

EnumType.STRING 으로 저장 시 , CALC_A 가 저장

```java
public void test() {
	CalculatorType = selectType();
	long originValue = 10000L;
	long result = code.calculate(originValue);
}
```

Enum 상수가 직접 연산 수행 가능

### 데이터 그룹 관리

결제 데이터는 결제 종류 - 결제 수단 2가지 형태로 표현

신용 카드 결제 ?
결제 수단 : 신용카드 결제
결제 종류 : 카드

![400](https://i.imgur.com/fprRvB2.png)

결제된 건이 어떤 결제 수단으로 진행 됐으며
해당 결제 방식이 어느 결제 종류에 속하는지 확인해야함

```java
public class LegayPayGroup {
	public static String getPayGroup(String payCode){
		if("ACCOUNT_TRANSFER".equals(payCode) || "REMITTANCE".equals(payCode) || ...){
			return "CASH";
		}
		else if("PAYCO".equals(payCode) || "CARD".eqauls(payCode) || "KAKAO_PAY".equals(payCode)){
			return "CARD";
		}
		else if("POINT".equals(payCode) || "COUPON".equals(payCode)){
			return "ETC";
		}
		return "EMPTY";
	}

}
```

- 둘의 관계 파악 어려움
-> 메소드는 포함관계를 나타내는지 , 단순 대체값 리턴하는기 파악 어려움
( 결제 종류 가 결제 수단을 포함하는 관계이나 - 메소드로 표현 불가능 )
- 입력값 과 결과값 예측 불가능
-> 결제 수단 범위 지정할 수 없어 문자열이면 전부 파라미터로 전달 가능
- 그룹별 기능 추가 어려움 ⭐️
-> 결제 종류 따라 추가 기능 필요할 시? ( if 문으로 메소드를 또 추가해야 함 )

결제 종류 기준 Print 기능 과 Push 하는 기능이 필요하면?

```java
public void pusyByPayGroup(fianl String payGroupCode){
	if("CASH".equals(payGroupCode)){
		pushCashMethod();
	} else if("CARD".equals(payGroupCode)){
		pushCardMethod();
	}
	...
}
public void printByPayGroup(final String payGroupCode){
	if("CASH".equals(payGroupCode)){
		doCashMethod();
	} else if("CARD".equals(payGroupCode)){
		doCardMethod();
	}
}
```

원할 때 사용하기 위해 독립적 구성해야 하며
결제 종류 분기 코드가 필수적 필요
=> 매우 비효율적

```java
public enum PayGroup {  
  
    CASH("현금", List.of("ACCOUNT_TRANSFER", "REMITTANCE")),  
    CARD("카드", List.of("PAYCO", "CARD")),  
    ETC("기타", List.of("POINT", "COUPON")),  
    EMPTY("없음", Collections.EMPTY_LIST);  
  
    private String title;  
    private List<String> payList;  
  
    PayGroup(String title, List<String> payList) {  
        this.title = title;  
        this.payList = payList;  
    }  
    public static PayGroup findByPayCode(String code) {  
        return Arrays.stream(PayGroup.values())  
                     .filter(payGroup -> payGroup.hasPayCode(code))  
                     .findAny()  
                     .orElse(EMPTY);  
    }  
    public boolean hasPayCode(String code) {  
        return payList.stream()  
                      .anyMatch(pay -> pay.equals(code));  
    }  
    public String getTile(){
	    return title;
    }
}
```

본인들이 갖고 있는 문자열 확인하며 문자열 인자값이 어느 Enum 상수에 포함되있는지 확인 가능

```java
public void test(){

	String payCode = selectPayCode();
	PayGroup payGroup = PayGroup.findByPayCode(payCode);
}
```

코드가 간결해졌으나,
결제수단이 문자열 - DB 결제수단 컬럼에 잘못된 값 입력 or 전달된 값 잘못될 수 있음

##### Refactoring

```java
public enum PayType {

	ACCOUNT_TRANSFER("계좌이체"),
	TOSS("토스")
	...;

	private String title;

	PayType(String title){
		this.title = title;
	}

	public String getTitle(){
		return title;
	}
}
```

Enum 을 하나 더 사용해 결제 종류를 만들어 사용 가능

```java
public enum PayGroupRefactoring {  
  
    CASH("현금", List.of(PayType.ACCOUNT_TRANSFER, PayType.REMITTANCE)),  
    CARD("카드", List.of(PayType.PAYCO,PayType.KAKAO_PAY)),  
    ETC("기타", List.of(PayType.POINT,PayType.COUPON)),  
    EMPTY("없음", Collections.EMPTY_LIST);  
  
    private String title;  
    private List<PayType> payList;  
  
    PayGroup(String title, List<PayType> payList) {  
        this.title = title;  
        this.payList = payList;  
    }  
    public static PayGroup findByPayType(PayType payType) {  
        return Arrays.stream(PayGroup.values())  
                     .filter(payGroup -> payGroup.hasPayCode(payType))  
                     .findAny()  
                     .orElse(EMPTY);  
    }  
    public boolean hasPayCode(PayType payType) {  
        return payList.stream()  
                      .anyMatch(payType::equals);  
    }  
    public String getTitle(){  
        return title;  
    }  
}
```

```java
public void test(){
	PayType payType = selectPayType();
	PayGroupRefactoring payGroupRefactoring = PayGroupRefactoring.findByPayType(payType);

}
```

타입 안정성 확보

### 관리 주체 DB -> 객체

정산 플랫폼은 수많은 카테고리 존재
UI 에서 select box 로 표현하는 부분이 많았음


- 코드명 으로 파악 불가능
-> 01 , 1 같은 매직 넘버와 if 문을 통해 수행을 한다면?
-> 01이란 코드가 뭔지 알기 위해 서버단 실행 코드를 보고 DB에서 조회해야 함
( 문서화가 되어 있더라도 , 확신할 수 없으므로 DB를 찾아봐야 함 )

- 항상 코드 테이블 조회 쿼리 실행
-> 특별 조회 하지 않음에도 UI 실행 위해 코드 테이블 조회 ( 캐시로도 가능은 함 )

- 카테고리 코드 기반 서비스 로직 추가시 위치 애매
-> 코드 따라 수행 기능이 있을 시 위와 같이 메소드 위치는 Service or Util 단에서 분기로 처리해야함

추가로 , 카테고리는 6개월에 1~2개 추가될까 말까한 빈도 낮은 영역
-> 굳이 카테고리로 관리를 해야하나?

=> Enum 으로 전환하고 팩토리 & 인터페이스 타입 선언해 일관된 방식으로 관리 및 사용하자

#### With Enum

Enum 을 JSON 으로 return 시 상수 name 만 출력
Enum 의 name 과 View Layer 에 출력할 title 필요
=> Enum 을 인스턴스 생성하기 위한 클래스 선언 필요

```java
public interface EnumMapperType {
	String getCode();
	String getTitle();
}
```

클래스 생성자를 일관되게 받게 하기 위해 인터페이스 생성

```java
public class EnumMapperValue {
    private String code;
    private String title;

    public EnumMapperValue(EnumMapperType enumMapperType) {
        code = enumMapperType.getCode();
        title = enumMapperType.getTitle();
    }

    public String getCode() { return code; }
    public String getTitle() { return title; }

    @Override
    public String toString() {
        return "{" +
            "code='" + code + '\'' +
            ", title='" + title + '\'' +
            '}';
    }
}
```

생성자는 EnumMapperType 을 통해 균일하게 생성된다

```java
public enum FeeType implements EnumMapperType {
    PERCENT("퍼센트"),
    MONEY("금액");

    private String title;

    FeeType(String title) {
        this.title = title;
    }

    @Override
    public String getCode() { return name(); }

    @Override
    public String getTitle() { return title; }
}

```

```java
@GetMapping("/no-bean-categories")  
public List<EnumMapperValue> getNoBeanCategories() {  
    return Arrays.stream(FeeType.values())  
                 .map(EnumMapperValue::new)  
                 .toList();  
}
```

의도한대로 JSON 결과 사용 가능

##### Perfect?

Enum.values 를 통해 Value 인스턴스가 생성되는 과정이 반복된다
런타임때 Enum 의 상수들이 변경될 일은 없다!
=> Bean 에 등록하여 사용하자

```java
public class EnumMapper {

    private Map<String, List<EnumMapperValue>> factory = new LinkedHashMap<>();

    public EnumMapper() { }

    public void put(String key, Class<? extends EnumMapperType> e) {
        factory.put(key, toEnumValues(e));
    }

    private List<EnumMapperValue> toEnumValues(Class<? extends EnumMapperType> e) {
        return Arrays.stream(e.getEnumConstants())
                     .map(EnumMapperValue::new)
                     .collect(Collectors.toList());
    }

    public List<EnumMapperValue> get(String key) {
        return factory.get(key);
    }

    public Map<String, List<EnumMapperValue>> get(List<String> keys) {
        if (keys == null || keys.size() == 0) {
            return new LinkedHashMap<>();
        }
        return keys.stream()
                   .collect(Collectors.toMap(Function.identity(), key -> factory.get(key)));
    }

    public Map<String, List<EnumMapperValue>> getAll() {
        return factory;
    }
}
```

getEnumConstants 를 통해 , 상수값들 전부 받음
값들을 put , get 으로 사용 가능


```java
@Bean
public EnumMapper enumMapper(){
	EnumMapper enumMapper = new EnumMapper();

	enumMapper.put("FeeType",FeeType.class);

	return enumMapper;
}

@GetMapping("/categories")
public List<EnumMapperValue> getCategories(){
	return enumMapper.get("FeeType");
}
```

### 마무리

Enum 을 통해 

해당 코드가 어디에서 쓰이는지
필드에서 어떤 값만 허용 가능한지
A값 과 B값이 실제 동일한지

확실한 부분 과 불확실한 부분을 분리 가능하다

또한 , 문맥을 담는다

A라는 상황의 a 와 B라는 상황의 a 는 다를 수가 있음
코드 의미 용도 파악하기 위해 , 레거시 정보들을 찾을 수 있음
문서화 역시 생략되거나 누락되는 내용이 있을 수 있음

Enum 은 이를 표현 가능하게 해준다

#### 단점?

Enum의 허들은 "변경이 어렵다"

코드 추가하거나 변경해야 하는 일이 빈번할 시
매번 Enum 코드 변경하고 배포하는거 보다
관리자가 직접 변경하는게 훨씬 편리할 수 있음

- 사용되는 테이블이 많아 변경 되면 관련된 테이블 데이터 전부 다 변경해야 하면?
- 한번 생성된 코드테이블 코드를 자주 변경해야 하면?
	- 1년에 몇번 발생하는지?
- 하루에 배포를 몇번 하는지?

=> 위 같은 상황일 시 테이블 로 얻는 장점이 정적언어 활용하는 장점보다 큰지 고민해보자

