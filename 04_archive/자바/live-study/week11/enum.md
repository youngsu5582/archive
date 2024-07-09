---
tags:
  - 자바
추가_정보: 백기선 라이브 스터디
주차: "11"
---
Java 1.5 에서 도입
열거형 클래스
상수의 집합

#### Enum이 왜 만들어졌는가?

```java
public class Demo {

	public static final int APPLE = 1;
	public static final int PEACH = 2;
	public static final int BANANA = 3;

    public static void main(String[] args) {
	int type = APPLE;
        switch (type) {
            case APPLE:
                System.out.println("32 kcal");
                break;
            case PEACH:
                System.out.println("52 kcal");
                break;
            case BANANA:
                System.out.println("16 kcal");
                break;
        }
    }
}
```

로직 에서 명확하게 사용하기 위해 상수를 선언해야 한다
상수 를 지정하기 위해 의미없는 값인 1,2,3 을 넣어야 한다

```java
public class Demo {

	public static final int APPLE = 1;
	public static final int PEACH = 2;

	...

	public static final int APPLE = 1;
	public static final int GOOGLE = 2;
}
```

이름의 충돌이 발생할 가능성이 존재
-> 이름을 다르게 해주거나 , 인터페이스로 구분

```java
public class EnumDemo {
    public static final int FRUIT_APPLE = 1;
    public static final int FRUIT_PEACH = 2;

    public static final int COMPANY_APPLE = 1;
    public static final int COMPANY_GOOGLE = 2;
}
```
이는 매우 비효율적

```java
interface Fruit {
    int APPLE = 1, PEACH = 2, BANANA = 3;
}

interface Company {
    int APPLE = 1, GOOGLE = 2, FACEBOOK = 3;
}
```

interface 로 해결 가능하나 , 이는 안티패턴
( 인터페이스는 규약을 정하기 위함 )

=> 추가로 , 완벽한 해결책이 아님
```java
Fruit.APPLE == Company.APPLE
```

이는 비교조차 안되야 하는 개념이나 ,
컴파일 경고가 뜨지 않으며 심지어 true 까지 나올수 있는 상황

```java
class Fruit {
    public static final Fruit APPLE = new Fruit();
    public static final Fruit PEACH = new Fruit();
    public static final Fruit BANANA = new Fruit();
}

class Company {
    public static final Company APPLE = new Company();
    public static final Company GOOGLE = new Company();
    public static final Company FACEBOOK = new Company();
}
public class EnumDemo {

    public static void main(String[] args) {
        if (Fruit.APPLE == Company.APPLE) {}
    }
}
```

이렇게 클래스 단위로 하면 컴파일 단에서 탐지 가능
=> Enum 의 만들어진 이유
( 클래스로 관리할 때 이점 과 더욱 간단하게 사용 가능하게 하기 위해 )

#### 특징

enum 에 정의된 상수들은 해당 enum type 의 객체

```java
enum Fruit { APPLE , PEACH , BANANA }

=>

class Fruit {
	public static final Fruit APPLE = new Fruit("APPLE");
	public static final Fruit PEACH = new Fruit("PEACH");
	public static final Fruit BANANA = new Fruit("BANANA");

	private String name;

	private Fruit(String name){
		this.name = name;
	}
}
```

( 실제 구현과는 다르나 , 같은 맥락 )

#### 제공 메소드

- compareTo(E e) : 다른 Enum 상수와 순서 비교 ( 굳이...? )
- values : Enum 타입의 모든 상수 포함하는 배열 반환
- valueOf(String name) : 주어진 이름과 일치하는 Enum 상수 반환 - 없으면 `IllegalArgumentException` 발생
- ordinal() : Enum 상수의 선언 순서 반환
- name() : Enum 상수 이름 문자열 반환

---
### 