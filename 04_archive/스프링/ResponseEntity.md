---
tags: 스프링
---
- HttpEntity 상속
##### HttpEntity

- body,headers 포함
```java
public ResponseEntity(@Nullable T body, @Nullable MultiValueMap<String, String> headers, HttpStatusCode statusCode) {  
    super(body, headers);  
    Assert.notNull(statusCode, "HttpStatusCode must not be null");  
    this.status = statusCode;  
}
``` 
- 다른 생성자 오버로딩 역시 포함

아래 빌더 메소드들은 BodyBuilder 호출
- ok() - 200
- created(Url location) - 201
- accepted() - 202
- noContent() - 204
- badrequest() - 400
- notFound() - 404
- internalServerError() - 500
- status(int value) or status(HttpStatus status) - 임의 statusCode

그후 header, body 설정 가능

header(),headers() 는 여전히 BodyBuilder
body() 는 ResponseEntity`<T>` 반환

ResponeEntity 를 넘기면, Spring Application 이 알아서 JSON 변환 담당

```java
ResponseEntity.accepted()  
   .contentType(MediaType.APPLICATION_JSON)  
   .body(new Reservation(Long.valueOf(1), "조이썬", "2023-04-02", "10:00"));
```
######  Optioanl 값을 넘기면?

```java
return ResponseEntity.ok()  
                     .body(new Sample(3, Optional.of("name"), Optional.of("content")));
```

```java
{
	"id": 3,
	"name": "name",
	"content": "content"
}
```


```java
return ResponseEntity.ok()  
                     .body(new Sample(3, Optional.empty(), null));
```

``` json
{
	"id": 3,
	"name": null,
	"content": null
}
```

JSON 변환 라이브러리가 알아서 변환 해준다.
#### VS ResponseBody

##### ResponseBody?

메소드에 붙일 수 있는 어노테이션
데이터 반환할 떄, `HttpMessageConverter` 를 사용해 body 를 만들어 반환하게 도와준다.

```java
@RequestMapping(value = "/message")
@ResponseBody
public Message get() {
    return new Message(penguinCounter.incrementAndGet() + " penguin!");
}
```

( Optional,null 시는 동일 )

ResponeEntity 에 비해서 유연성이 떨어진다.
( 상태 코드와 응답 메시지를 줄 수 없으므로 )

=> 컨벤션의 기호에 따라 결정 가능할 거 같다.
