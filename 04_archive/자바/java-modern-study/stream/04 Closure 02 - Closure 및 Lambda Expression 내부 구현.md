---
tags:
  - 자바
강의명: 모던 자바 (자바8) 못다한 이야기
강의_링크: https://www.youtube.com/watch?v=bKzMl7LKIO0
---



```java

private int number = 999;  
public static void main(String[] args) {  
    int number = 100;  
    Runnable runnable = new Runnable() {  
        @Override  
        public void run() {  
            System.out.println(number);  
        }  
    };  
    runnable.run();  
}
```

앞선 예제와 같이 , Runnable Anonymous Class 를 작성하면 무슨 일이 생길까?

```java
dragonsu@iyeongsuui-MacBookAir section2 % ls -l

total 16

-rw-r--r--@ 1 dragonsu  staff  694  1  5 15:07 Main$1.class

-rw-r--r--@ 1 dragonsu  staff  656  1  5 15:07 Main.class
```

원래 Build 되야 하는 Main.class 외 , Main$1.class 가 생긴다!

```java
Runnable runnable1 = new Runnable() {  
    @Override  
    public void run() {  
        System.out.println(number);  
    }  
};  

...

Runnable runnable5 = new Runnable() {  
    @Override  
    public void run() {  
        System.out.println(number);  
    }  
};
```

이렇게 , Runnable 을 5개를 만들면?

```java
dragonsu@iyeongsuui-MacBookAir section2 % ls -l

total 48

-rw-r--r--@ 1 dragonsu  staff  694  1  5 15:09 Main$1.class
-rw-r--r--@ 1 dragonsu  staff  694  1  5 15:09 Main$2.class
-rw-r--r--@ 1 dragonsu  staff  694  1  5 15:09 Main$3.class
-rw-r--r--@ 1 dragonsu  staff  694  1  5 15:09 Main$4.class
-rw-r--r--@ 1 dragonsu  staff  694  1  5 15:09 Main$5.class
-rw-r--r--@ 1 dragonsu  staff  926  1  5 15:09 Main.class
```

$ class 가 5개가 생긴다.

=> 매우 비효율적이게 작동!

```java
Runnable runnable1 = () ->System.out.println(number);  
runnable1.run();  

...

Runnable runnable5 = () ->System.out.println(number);  
runnable5.run();
```

```java
dragonsu@iyeongsuui-MacBookAir section2 % ls -l

total 8

-rw-r--r--@ 1 dragonsu  staff  1846  1  5 15:14 Main.class
```

$ class 가 생기지 않는다.

=> 파일 공간 & 최적화 면에서 효율적이게 작동

` javap -c -p Main.class ` 명령어 수행
	( Class 파일을 역 Compile - c Option 은 Method 를 ByteCode 로 보여줌 , p Option 은 private 포함해서 보여줌 )

```java
public class closure.section2.Main {
  private int number;

  public closure.section2.Main();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: aload_0
       5: sipush        999
       8: putfield      #7                  // Field number:I
      11: return

  public static void main(java.lang.String[]);
    Code:
       0: bipush        100
       2: istore_1
       3: iload_1
       4: invokedynamic #13,  0             // InvokeDynamic #0:run:(I)Ljava/lang/Runnable;
       9: astore_2
      10: aload_2
      11: invokeinterface #17,  1           // InterfaceMethod java/lang/Runnable.run:()V
      16: return

  private static void lambda$main$0(int);
    Code:
       0: getstatic     #21                 // Field java/lang/System.out:Ljava/io/PrintStream;
       3: iload_0
       4: invokevirtual #27                 // Method java/io/PrintStream.println:(I)V
       7: return
}
```

=> 해당 코드의 핵심은 Lambda Method 로 만들어진다는 것


```java
Runnable $r = indy[boostrap=LambdaMetaFactory,
					staticargs=[Runnable,lambda$0],
					dynargs=[number]
				  ]
private static void lambda$0(int number){
	System.out.println(number);
}
	
```

해당 코드의 맥락에서 실행


- indy : invoked dynamic
	람다 표현식의 실행 시점에서 메소드 타입 & 행동 결정
	- LambdaMetaFactory 부트스트랩 메소드 사용
	- Runnable Interface 구현하는 람다 표현식 생성
- dynargs : dynamic args
	동적으로 변하는 파라미터
=> 람다 표현식으로 전달되는 동적 인자 & 런타임 결정되는 인자


이때 , Closure 가 아니더라도 Lambda Expression에 parameter 가 있다면
desugar 된 메소드에도 역시 parameter 가 필요함

```java
private static String lambda$0(Product product){
	return product.getName();
}
```

이런 형태의 메소드 일 시 , 파라미터가 추가 되지만 product 가 dynargs 에는 추가되지 않는다!


핵심은 Runnable 이나 Anonymous Function 보다 Lambda 가 60배 더 빠르다!