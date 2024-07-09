---
tags:
  - 자바
추가_정보: 백기선 라이브 스터디
주차: "5"
---
- 말 그대로 , 상수 Pool 이다.

### String Constant Pool

```java
public class CatService {  
    public String format = "Cat's name : ";  
    public CatService() {}  
}
public class CatController {  
    public String format = "Cat's name : ";  
    public CatController() {}  
}
```

이런 , 같은 값을 가지는 format이 있을시?

```java
CatService catService = new CatService();  
CatController catController = new CatController();

System.out.println(System.identityHashCode(catController.format));  
System.out.println(System.identityHashCode(catService.format));

String format = "Cat's name : ";  
System.out.println(System.identityHashCode(format));
```

3가지의 identityHashCode 값들은 모두 동일하다.

#### IdentityHashCode

- 실제 메모리 주소를 기반으로 하는 해시 코드 반환
```java
/**  
 * Returns the same hash code for the given object as * would be returned by the default method hashCode(), * whether or not the given object's class overrides * hashCode(). * The hash code for the null reference is zero. * * @param x object for which the hashCode is to be calculated  
 * @return  the hashCode  
 * @since   1.1  
 * @see Object#hashCode  
 * @see java.util.Objects#hashCode(Object)  
 */@IntrinsicCandidate  
public static native int identityHashCode(Object x);
```

=> 즉 , 메모리 값을 모르더라도 두 개가 동일한지 확인이 가능!

이렇게 String Literal 로 생성된 값은 Heap 의 String Constant Pool 에서 관리된다!

```java
String newFormat = new String("Cat's name : ");
```
이렇게 , New 를 통해 생성된 문자열은 다른 메모리 값을 가진다.

```java
String newFormat = new String("Cat's name : ").intern();
```

이렇게 intern 을 사용하면 , Pool 에 해당 값이 있음을 보장 ( 즉 없으면 자기가 넣음 ) 후 반환한다.
=> 같은 메모리 주소

하지만 , intern 은 매우 비싼 작업이며 굳이 불필요한 코드
##### 추가로!

절대 사용하지 않는것이 좋다
intern 을 사용하면 , 의도하지 않은 채 String constant pool 의 크기가 계속 커질 수 있습니다.
-> 메모리 부족 초래
GC 의 대상이 되지 않고 , 메모리에 남아있음 

- ORM , MyBatis 등 DB 에서 조회해 오는 String 값들도 String Constant Pool 에 저장되있는게 아님
### Class Constant Pool

컴파일 시 , 클래스파일 내부에 존재하는 영역
ClassLoader 에 의해 JVM 에 로드될 때 메모리에 같이 로드
클래스의 구성 요소 ( 상수 , 문자열 , 클래스 / 인터페이스 참조 값 ) 들에 대한 데이터를 저장하고 있다

```java
class Hello {
    public static void main( String[] args ) {
        for( int i = 0; i < 10; i++ )
            System.out.println( "Hello from Hello.main!" );
  }
}
```

```shell
#1 = Methodref          #6.#16         // java/lang/Object."<init>":()V
#2 = Fieldref           #17.#18        // java/lang/System.out:Ljava/io/PrintStream;
#3 = String             #19            // Hello from Hello.main!
#4 = Methodref          #20.#21        // java/io/PrintStream.println:(Ljava/lang/String;)V
#5 = Class              #22            // Hello
#6 = Class              #23            // java/lang/Object
#7 = Utf8               <init>
#8 = Utf8               ()V
#9 = Utf8               Code
#10 = Utf8               LineNumberTable
#11 = Utf8               main
#12 = Utf8               ([Ljava/lang/String;)V
#13 = Utf8               StackMapTable
#14 = Utf8               SourceFile
#15 = Utf8               Hello.java
#16 = NameAndType        #7:#8          // "<init>":()V
#17 = Class              #24            // java/lang/System
#18 = NameAndType        #25:#26        // out:Ljava/io/PrintStream;
#19 = Utf8               Hello from Hello.main!
#20 = Class              #27            // java/io/PrintStream
#21 = NameAndType        #28:#29        // println:(Ljava/lang/String;)V
#22 = Utf8               Hello
#23 = Utf8               java/lang/Object
#24 = Utf8               java/lang/System
#25 = Utf8               out
#26 = Utf8               Ljava/io/PrintStream;
#27 = Utf8               java/io/PrintStream
#28 = Utf8               println
#29 = Utf8               (Ljava/lang/String;)V
```

컴파일 타임에 생성

### Runtime Constant Pool ( JVM )

Java 8 이전에는 JVM - perm 영역에 저장
Java 8 이후에는 JVM - metaspace 영역에 저장

#### Metaspace

Perm 영역에 저장될 때는 Class,  Metadata 로딩 과정에서 Memory Leak 가 발생했다
특히 , 해당 크기를 고정적 설정했기에 Out Of Memory 도 발생했다.

=> 이를 개선하기 위해 MetaSpace 영역으로 이관

JVM 의 Natvie Memory 를 활용해 JVM 이 관리
메모리가 동적 관리되며 , 필요할 시 OS 에 요청해 메모리 추가 할당 가능

클래스가 로딩되어 , 메모리에 올라갈 때 와 인스턴스가 생성될 때 동적으로 구성

동적 프로그래밍을 사용할 떄 기능을 지원한다.

#### 리플렉션

런타임 시간에 메타데이터 검사하거나 수정할 수 있는 기능
클래스 메소드명 , 필드명 , 접근 제어자 등을 조회할 수 있는 이유

#### 동적 작업

동적 로딩 , 동적 바인딩 , 리플렉션 등 동적인 특성 작업 지원가능
