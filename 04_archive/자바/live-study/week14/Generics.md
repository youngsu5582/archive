---
tags:
  - 자바
추가_정보: 백기선 라이브 스터디
주차: "14"
---
### Generics

JDK 1.5 에서 처음 도입

컴파일 시 타입 체크
-> 객체 타입 컴파일 시 체크하므로 , 객체 타입 안정성 높이고 형변환 번거로움 줄어듬

타입 안정성을 높인다
-> 의도하지 않은 타입 객체 지정을 막고 , 지정된 객체 꺼내올 때 다른 타입으로 형변환 되는 오류 방지

#### 장점

1. 타입 안전성 제공
	-> 컴파일 타임에  타입 체크 , 런타임에 ClassCastException 같은 Unchecked Exception 역시 보장 받음
2. 타입 체크 & 형변환 생략 가능 -> 코드 간결화 가능

#### 왜 사용하는가?

- 잘못된 타입 사용될 수 있는 문제 컴파일 과정에서 제거 가능
- 컴파일러가 코드에서 잘못 사용된 타입 때문에 발생하는 문제점 제거 위해 제네릭 코드 대해 강타입 체크
- 실행 시 타입 에러 나는 것 보단 , 컴파일 시 미라 타입 강체크 -> 에러 사전 방지
- 타입 국한하여 요소 찾을 떄 타입 변환 할 필요 없음 -> 프로그램 성능 향상

#### 선언

```java
public class Student {
	Object attribute;
	public Object getAttribute(){
		return attribute;
	}
	public void setAttribute(Object attribute) {
		this.attribute = attribute;
	}
}
```

=> 

```java
public class Student<T> {
	T attribute;

	public T getAttribute(){
		return attribute;
	}

	public void setAttribute(T attribute)(){
		this.attribute = attribute;
	}

}
```

T 는 타입 변수

##### 타입 네이밍

- T : Type
- E : Element
- K , V : Key , Value
- N : Number
- R : Return
- S , U , V : 두 , 세 , 네 번째 사용 되는 타입 파라미터

모두 타입 네이밍이나 , 편의 및 가독성을 위해 지정 알파벳 사용

```java
Student<String> student = new Student();
student.setAttribute("First Grade");

-----------------------------------

Student<Long> student = new Student();
student.setAttribute(1L);
```

### Generic 작동 원리

타입 소거를 통해 작동
-> 실제 코드는 런타임에 타입에 대한 정보는 소거

컴파일러가 코드를 검사해 타입 오류 없을시 제네릭 -> Object 타입으로 치환해 소거

#### Sample

```java
// 불필요한 코드는 생략
public class ArrayList<E> extends AbstractList<E> implements List<E> {
	E elementData(int index){
		return (E) elementData[index];
	}
}
```
=>
```java
public class ArrayList extends AbstractList implements List {
	Object elementData(int index) {
		return elementData[index];
	}
}
```

타입 소거 이전 코드는 (E) 로 비검사 형변환 ( 필드들이 기본적으로 Object 선언되어 있으므로 형변환 )

- 컴파일 시점에 타입을 특정해 안전하지 않은 타입이 들어오지 못하도록 타입 안정성 확보
-> 런타임에서 임의 "코드 추가" 불가능! ( 컴파일 시점 안정성 확보 -> 런타임 타입 신경쓰지 않고 사용 가능 )

- 제네릭에 넣은 값을 받을 때는 컴파일러가 지정한 타입 형태로 자동 형변환 지원 - Object -> E 로 형변환 ( 컴파일러 자동 처리 )
-> 위와 동일하게 타입에 대한 안정성 확보함 -> 무조건 E 를 받아서 E를 내보냄을 보장 가능

##### 타입 소거

- Type Erasure
- 자바 하위 호환성 유지위해 사용 - Jdk 1.5 이전 버전과 호환

#### Generics 제한

```java
static T genericStatic;
```

static 불가능
-> 모든 객체에 대해 동일하게 동작해야 하므로 ( T 는 인스턴스 변수로 간주 )

```java
T genericVar = new T();
T[] genericAry = new T[];
Class<?> c = T.class;
```

세개 전부 불가능
-> 중요한 점은 T[] 생성이 불가능 하다는 점 ( new 연산자는 컴파일 시점에 타입 T 가 뭔지 명확히 알아야 함 )

##### T[] 생성

```java
public Stack(Class<E> clazz) {  
	elements = (E[]) Array.newInstance(clazz, 16);  
}

Stack<String> stack = new Stack<>(String.class);

```

Reflection API 인 newInstance 사용해 , 동적 객체 생성

```java
public Stack() {  
	elements = (E[])new Object[DEFAULT_INITIAL_CAPACITY]; // 컴파일 에러 발생  
}
```

배열을 Object 배열로 생성 후 제네릭으로 형 변환

`Stack.java uses unchecked or unsafe operations.` 
`Note: Recompile with -Xlint:unchecked for details`

에러가 뜨므로 , @SuppressWarnings("unchecked") 사용해줘야함
	+ 추가로 , 힙 오염 발생 가능!
##### 힙 오염

```java
ArrayList arrayList = new ArrayList();  
List<String> stringList = arrayList;

arrayList.add(42);  
String str = stringList.get(0);  

```

단순 ArrayList 는 Object 를 가짐
-> Integer 값을 String으로 받아 `ClassCastException` 발생

### 사용

```java
public class Box<T> {  
    T item;  
    public T getItem() {  
        return item;  
    }  
    public void setItem(final T item) {  
        this.item = item;  
    }  
}
```

```java
Box<String> stringBox = new Box<>();
```

`<>` 안에 타입 생략 가능 ( JDK 1.7 부터 )

```java
Box<Fruit> fruitBox = new Box<>();
fruitBox.setItem(new Apple());
fruitBox.setItem(new Grape());
```

setItem 의 인자는 Fruit 와 Fruit 의 자손 ( 당연히 자손의 자손 ... 도 가능 ) 들만 가능

#### 타입 제약
```java
public class FruitBox<T extends Fruit> extends Box {
}
```
Box 의 T 를 FruitBox 가 다시한번 제네릭 타입으로 지정
-> 제네릭 타입이 무조건 Fruit 를 상속받게 제한

```java
public class EatableBox<T extends Eatable> extends Box {
}
```

Box 의 T를 EatableBox 가 다시한번 제네릭 타입으로 지정
-> 제네릭 타입이 무조건 Eatable을 구현하도록 제한

##### VS TypeScript

TypeScript 의 구조적 타이핑 과는 다름
```java
public interface Eatable {  
    abstract void eat();  
}
```

해당 interface 가 있다면,
`implemetns Eatable` 을 통해 interface 를 구현해야만 통과

```java
public void eat(){} // => ❌
@Override
public void eat(){} // => ✅
```

```java
public class AdvancedBox<T extends Fruit & Eatable> extends Box {

}
```

Fruit Class 의 자손이면서 , Eatable 을 구현해야만 함
