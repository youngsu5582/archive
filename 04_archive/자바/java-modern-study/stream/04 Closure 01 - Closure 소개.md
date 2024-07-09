---
tags:
  - 자바
강의명: 모던 자바 (자바8) 못다한 이야기
강의_링크: https://www.youtube.com/watch?v=pjtk7vvryio
---

### Example

```java
private static final void testClosure(final String name, final Runnable runnable) {  
    System.out.println("========================================");  
    System.out.println(name + ": ");  
    runnable.run();  
}
```

이름을 출력하고 , 단순 실행하는 코드
#### Annonymous Function

```java
testClosure("Anonymous Class", new Runnable() {  
    @Override  
    public void run() {  
        System.out.println(number);  
    }  
});
```

익명 함수로 , Runnable 을 생성하여 실행

#### Lambda Function
```java
testClosure("Lambda Expression", () -> {  
    System.out.println(number);  
});
```

```java
public void test1() {  
    Integer number = 100;  
  
    testClosure("Anonymous Class", new Runnable() {  
        @Override  
        public void run() {  
            System.out.println(number);  
        }  
    });  
    testClosure("Lambda Expression", () -> {  
        System.out.println(number);  
    });  
}
```

```cmd
========================================
Anonymous Class: 
100
========================================
Lambda Expression: 
100
```

해당 코드들은 , 100을 출력

#### Class Parameter ?

```java
public class Main {  
    protected Integer number = 999;

Main main = new Main();  
main.test1();
}
```

이렇게 , class 의 parameter 로 있는
number 를 출력하려면?

```java
testClosure("Anonymous Class", new Runnable() {  
    @Override  
    public void run() {  
        System.out.println(Main.this.number);  
    }  
});  

testClosure("Lambda Expression", () -> {  
    System.out.println(this.number);  
});
```

Annonymous Class 는 Main.this.number 로 지정
Lambda 는 this.number 로 지정

=> Closure 때문!
### Closure 
```java
function closureFunction(){
	Integer i = 10;
	method( x -> x*2 + i );
}
```

method 함수에서 , i로 접근을 가능하게 해주는 것이 closure

Anonymous 함수는 Annonymous Class 를 가지고 있는다.
=> this 는 Annoynmous Class 를 가리킨다.

Lambda 함수는 자체를 this를 가지고 있지 않는다!
=> this 는 Main Class 를 가리킨다.

```java
testClosure("Anonymous Class", new Runnable() {  
    @Override  
    public void run() {  
        System.out.println("this.toString() : " +this.toString());  
    }  
});  
testClosure("Anonymous Class", new Runnable() {  
    @Override  
    public void run() {  
        System.out.println("Closure this.toString() : " + Main.this.toString());  
    }  
});  
testClosure("Lambda Expression", () -> {  
    System.out.println("this.toString() : "+this.toString());  
});
```

```cmd
========================================
Anonymous Class: 
this.toString() : stream.section8.Main$2@7344699f
========================================
Anonymous Class: 
Closure this.toString() : stream.section8.Main@8bcc55f
========================================
Lambda Expression: 
this.toString() : stream.section8.Main@8bcc55f
```

Lambda 의 this.toString 과 , Main 의 this.toString 은 동일하다.

### Example

```java
public class Main {  
    protected Integer number = 999;
	@Override  
	public String toString() {  
	    return new StringBuilder("{Closure-Example")  
	            .append("number=")  
	            .append(number)  
	            .append('}')  
	            .toString();  
	}  
	  
	public static <T> String toString(T value) {  
	    return "This Value is " + String.valueOf(value);  
	}
}
```

toString 을 두개 선언

```java
testClosure("Anonymous Class", new Runnable() {  

    @Override  
    public void run() {  
        System.out.println("ClosureExample this.toString(): " + toString("Test"));  
    }  
});  
testClosure(  
        "Lambda Expression",  
        () -> System.out.println("this.toString(): " + toString("Test"))  
);
```

위의 코드는 에러가 발생한다.
`Expected 0 arguments but found 1`

위의 코드는 Object 의 toString 을 호출하나 , toString 은 메소드가 필요없으므로 에러가 발생한다!
=> Main Class 의 toString 으로 인식을 못한다.
( 메소드의 파라미터 개수에 상관없이 )

결국 , Lambda 를 사용하면 Anonymous 보다 Performance 가 뛰어남!
( 자체 Class 생성이 아닌 , 외부 Class 를 참조하므로 )

#### In GPT

Java에서 클로저(Closure)는 함수의 비지역 변수를 "닫아(capture)"서 사용할 수 있게 하는 개념입니다. 비록 Java에는 일급 함수(first-class functions)가 없어서 함수를 직접적으로 클로저로 사용하는 것은 불가능하지만, 익명 클래스나 람다 표현식을 통해 비슷한 효과를 낼 수 있습니다.

클로저는 다음과 같은 특징을 가집니다:

1. **외부 스코프의 변수 접근**: 클로저는 자신이 생성될 때의 환경을 "기억"합니다. 따라서 클로저는 자신이 만들어진 스코프 외부에 있는 변수에 접근할 수 있습니다.
    
2. **외부 변수의 상태 보존**: 클로저는 외부 변수의 상태를 보존할 수 있습니다. 함수가 실행될 때 외부 변수의 최신 상태를 사용할 수 있습니다.
    
3. **데이터 은닉과 캡슐화**: 클로저는 특정 함수 내부에서만 사용할 수 있는 변수를 외부에서 접근하지 못하도록 은닉합니다. 이를 통해 데이터를 캡슐화할 수 있습니다.
    

자바에서 클로저를 구현하는 방법은 다음과 같습니다:

- **익명 클래스**: 자바 7까지는 익명 클래스를 사용하여 클로저와 유사한 효과를 낼 수 있었습니다.