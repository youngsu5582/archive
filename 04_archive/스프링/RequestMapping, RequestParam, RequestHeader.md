---
tags: 스프링
---
### RequestMapping

```java
@RequestMapping(  
    method = {RequestMethod.GET}  
)  
public @interface GetMapping {  
    @AliasFor(  
        annotation = RequestMapping.class  
    )  
    String name() default "";  
  
    @AliasFor(  
        annotation = RequestMapping.class  
    )  
    String[] value() default {};  
  
    @AliasFor(  
        annotation = RequestMapping.class  
    )  
    String[] path() default {};  
  
    @AliasFor(  
        annotation = RequestMapping.class  
    )  
    String[] params() default {};  
  
    @AliasFor(  
        annotation = RequestMapping.class  
    )  
    String[] headers() default {};  
  
    @AliasFor(  
        annotation = RequestMapping.class  
    )  
    String[] consumes() default {};  
  
    @AliasFor(  
        annotation = RequestMapping.class  
    )  
    String[] produces() default {};  
}
```

```java
public @interface RequestMapping {  
    String name() default "";  
  
    @AliasFor("path")  
    String[] value() default {};  
  
    @AliasFor("value")  
    String[] path() default {};  
  
    RequestMethod[] method() default {};  
  
    String[] params() default {};  
  
    String[] headers() default {};  
  
    String[] consumes() default {};  
  
    String[] produces() default {};  
}
```

결국, RequestMapping 에서 파생된 요청 매핑 어노테이션들 ( GetMapping, PostMapping ... )

`@RequestMapping("/test")` = `@RequestMapping(path = "/test")`


- headers : key=value 가 일치하는 경우에만 수신


- consumes : 클라이언트가 보낸 Content-Type 헤더가 일치하는 경우에만 수신
-> 클라이언트가 서버에게 보내는 데이터 타입을 명시

```java
@RequestMapping(consumes=MediaType.APPLICATION_JSON_VALUE)
@RequestMapping(consumes="application/json")
```

Enum 값으로도 지정 가능
-> 서버에서 Json 만 받는다

- produces : 서버가 생성하는 응답의 Content-Type 헤더 값이 클라이언트의 Accept 헤더와 일치할때 송신
-> 서버가 클라이언트에게 반환하는 데이터 타입을 명시

```java
@RequestMapping(produces=MediaType.APPLICATION_JSON_VALUE)

@RequestMapping (produces = "application/json;charset=UTF-8")
```
 
 2. return값을 json으로 보내고 인코딩은 UTF-8 지정

### RequestParam, RequestHeader

RequestParam 과 RequestHeader 코드가 동일하다
-> RequestParam 으로만 설명
#### 코드

```java
@Target({ElementType.PARAMETER})  
@Retention(RetentionPolicy.RUNTIME)  
@Documented  
public @interface RequestParam {  
    @AliasFor("name")  
    String value() default "";  
  
    @AliasFor("value")  
    String name() default "";  
  
    boolean required() default true;  
  
    String defaultValue() default "\n\t\t\n\t\t\n\ue000\ue001\ue002\n\t\t\t\t\n";  
}
```


required 가 default 가 true 이므로 없으면
```java
{
	"timestamp": "2024-04-18T11:19:35.683+00:00",
	"status": 400,
	"error": "Bad Request",
	"path": "/reservations/test"
}
```
400 발생!

name 과 value 는 같은 역활!
-> 변수명과 다르게 선언을 할때 사용한다
( ex: value="sample" String code ) - sample 경로로 인식

두개를 다르게 하면? ( ex: @RequestParam(value = "sample", name = "code") )
-> `attribute 'name' and its alias 'value' are declared with values of [code] and [sample].`
위와 같은 중복 에러 발생 ( 이 역시도 컴파일 단 캐치 못하는, 런타임 에러 발생! )

defaultValue 를 통해 없을 때 값 지정 가능
-> 선언시, 사실상 required 는 무조건 통과 + null 절대 불가능 기대 가능

사실 간단한 자료형들은 `RequestParam` 지정 없이도 인식 한다.
```java
@GetMapping("/test")  
@ResponseBody  
public Sample some(String code) {
	...
}
```

`http://localhost:8080/reservations/test?code=code` 정상 작동!

```java
public Sample some(@RequestParam(value = "sample", required = false) int code){
	...
}
```

이렇게, 원시형인데 값이 안드가면? - 500 에러 발생!
`java.lang.IllegalStateException: Optional int parameter 'sample' is present but cannot be translated into a null value due to being declared as a primitive type. Consider declaring it as object wrapper for the corresponding primitive type.`

만약 자료형을 다르게 넣었다면? ( Integer 를 요구하나, String 이 들어갈시 ) - 400 에러 발생!
`Resolved [org.springframework.web.method.annotation.MethodArgumentTypeMismatchException: Failed to convert value of type 'java.lang.String' to required type 'java.lang.Integer'; For input string: "hihi"]
`
```java
public Sample some(@RequestParam List<String> params) {
	...
}
```

`http://localhost:8080/reservations/test?params=1,2,3,4` 이렇게 콤마를 통해 넣을시 배열로 들어간다.

---

- 의식적으로, required false 를 했으면, Optional 로 선언을 하자!
- 왠만하면, Wrapper Class 를 사용하자!
- 모든 값을 받고 싶다면? -> Map<String, String> 을 활용하자!