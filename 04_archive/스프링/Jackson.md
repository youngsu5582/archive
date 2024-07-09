---
tags: 스프링
---
```java
public class SampleResponse {  
    final String value;  
  
    public SampleResponse(final String value) {  
        this.value = value;  
    }  
}
```
`2024-04-25T16:27:01.149+09:00  WARN 7971 --- [nio-8080-exec-1] .w.s.m.s.DefaultHandlerExceptionResolver : Resolved [org.springframework.web.HttpMediaTypeNotAcceptableException: No acceptable representation]

2024-04-25T16:30:32.115+09:00  WARN 8136 --- [nio-8080-exec-1] .w.s.m.s.DefaultHandlerExceptionResolver : Resolved [org.springframework.http.converter.HttpMessageNotWritableException: No converter for [class roomescape.controller.SampleResponse] with preset Content-Type 'null']`

single value 일때 getValue 이면 property 즉, json으로 인식해 return

```java
new ObjectMapper().setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
```

```java
public enum PropertyAccessor {  
    GETTER,  // 게터
    SETTER,  // 세터
    CREATOR,  // 생성자
    FIELD,  // 필드
    IS_GETTER,  // isXXX
    NONE,  // 아무것도X
    ALL;   // 전부
}
public static enum Visibility {  
    ANY,  // 전부
    NON_PRIVATE, // private 제외  
    PROTECTED_AND_PUBLIC,  // protected 와 public
    PUBLIC_ONLY,  // public만
    NONE,  // 아무것도X
    DEFAULT;  // default만
}
```


---

```java
public class SampleResponse {  
    private final String value;  
    private final String sampleValue;  
  
    public SampleResponse(final String value, final String sampleValue) {  
        this.value = value;  
        this.sampleValue = sampleValue;  
    }  
  
    @JsonProperty("value")  
    public String getValue() {  
        return value;  
    }  
    public String getSampleValue() {  
        return sampleValue;  
    }  
  
    @JsonProperty("isValue")  
    public boolean isValue() {  
        return value.equals("Hello World");  
    }  
}
```

```java
public record SampleResponse(@JsonProperty("value") String value,  
                             String sampleValue) {  
    @JsonProperty("isValue")  
    public boolean isValue() {  
        return value.equals("Hello World");  
    }  
}
```

```json
{
	"sampleValue": "Bye World",
	"isValue": true,
	"value": "Hello World"
}
```
두개는 동일
=> 왠만하면 record 를 쓰자
