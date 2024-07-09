---
tags:
  - 자바
추가_정보: 백기선 라이브 스터디
주차: "3"
---
### Instaceof

```java
Integer i = 5;
i instanceof Integer
```

- instanceof 는 Reference Type ( Wrapper Type ) 만 가능

```java
i instanceof Object
```
- instanceof Object 는 항상 true
```java
Integer nullValue = null;
nullValue instanceof Object;
```
- null 은 항상 false

```java
class Animal {}
class Dog extends Animal {}
class BullDog extends Animal {}

BullDog bullDog = new BullDog();

System.out.println(bullDog instanceof Dog);  
System.out.println(bullDog instanceof Animal);
```

해당 결과는 true 를 반환한다.
BullDog 이 Dog 와 Animal 을 상속받아온 관계이므로

```java
Cat cat = new Cat();  
System.out.println(cat instanceof BullDog);  

Object objCat = cat;  
System.out.println(objCat instanceof BullDog);
```

위 구문은 컴파일러 단위에서 에러 발생 `Inconvertible types; cannot cast 'org.example.Cat' to 'org.example.BullDog'`

-> 무조건 false 가 나올뿐 더러 , 굳이 비교를 할 필요 없으므로

아래는 , object 로 만들어서 검사를 통과하나 , false 가 나온다.

### instanceof Example Code

instanceof 의 기본적 동작 원리를 GPT 의 도움을 받아 작성
object 의 class 를 받은 후 , 최상위 super class 까지 탐색
null 이면 , 아닌것이므로 종료 ( object -> null )

```java
private static final boolean mockInstanceof(Object object,Class target){  
    if(object==null){  
        return false;  
    }  
    Class objectClass = object.getClass();  
    while(objectClass!=null){  
        System.out.println(objectClass);  
        if (objectClass == target){  
            return true;  
        }  
        objectClass = objectClass.getSuperclass();  
    }  
    return false;  
}
```

그 중 일치시 true , 반복문 종료시 false 리턴

