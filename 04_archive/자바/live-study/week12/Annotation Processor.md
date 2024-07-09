---
tags:
  - 자바
추가_정보: 백기선 라이브 스터디
주차: "12"
---
### Annotation Processor

자바 컴파일러의 컴파일 단계에서
애노테이션 정보를 스캔하고 처리하는 Hook
-> 컴파일 도중 애노테이션 만나면 특별 동작을 하도록 만들어진 코드


소스코드 파일 (.java) 과 클래스 파일(.class) 을 입력 받아
새로운 소스코드 or 클래스 파일 생산

대표적인 예시로 Lombok

```java
@Getter
@Setter
public class Book {
	private String title;
	private String author;
}
```


- Processor 는 AbstractProcessor 를 상속받음

```java
public class Book {
    private String title;
    private String author;

    public String getTitle() {
        return this.title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getAuthor() {
        return this.author;
    }

    public void setAuthor(String author) {
        this.author = author;
    }
}

```

#### AbstractProcessor

- getSupportedOptions() : 어떤 어노테이션에 대해 동작 수행할 지 지정
- process(Set`<? extends TypeElement>` annotations, RoundEnvironment env) : 프로세서의 메인 메소드
	-> 애노테이션 스캐닝 하고 처리하는 작업 수행
- getSupportedSourceVersion() {} : 지원하는 소스 버전
- init(ProcessingEnvironment env) : 

### Custom 생성

#### 1. AbstractProcessor 상속받은 클래스 생성

```java
@SupportedAnnotationTypes("Annotation")
public class CustomProcessor extedns AbstractProcessor{
	@Override
	public synchronized void init(ProcessingEnvironment env){
		super.init(env);
	}
	@Override
	public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment env){
		...
		return true;
	}
	@Override public SourceVersion getSupportedSourceVersion(){
	     return SourceVersion.latestSupported(); 
	}
}
```

- SupportedAnnotationTypes : 프로세서가 처리할 어노테이션 지정
- init : 메소드에서 프로세서 초기화에 필요한 작업 수행 - ProcessingEnvironment 통해 처리 환경 접근 가능
- process : 등록된 어노테이션 대한 처리 수행 - 프로세서가 관심 가지는 어노테이션 수신
- getSupportedSourceVersion : 프로세서가 지원하는 Java 소스 코드 버전 지정

#### 2. 파일 지정 & 배포

작성한 프로세서 파일 ( .class ) 과
javax.annotation.processing.Processor 파일을 META-INF/services 에 위치 시킨후

processing.Processor 파일에는 FQCN 을 개행으로 구분해서 작성

```
lombok.launch.AnnotationProcessorHider$AnnotationProcessor
lombok.launch.AnnotationProcessorHider$ClaimingProcessor
```

.jar 파일로 패키징해서 제공
