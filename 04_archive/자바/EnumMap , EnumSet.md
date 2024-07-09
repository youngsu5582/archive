---
tags: 자바
---
### EnumMap

EnumMap 은 내부적으로 배열 사용

```java
public EnumMap(Class<K> keyType) {  
    this.keyType = keyType;  
    keyUniverse = getKeyUniverse(keyType);  
    vals = new Object[keyUniverse.length];  
}
```

keyUniverse 에서 Key 집합 받음
배열 생성

```java
public Collection<V> values() {  
    Collection<V> vs = values;  
    if (vs == null) {  
        vs = new Values();  
        values = vs;  
    }  
    return vs;  
}
```

V 가 Element 들
싱글톤 유지

---

- Enum Type 만 Key 로 가진다
- Null Key 없음을 보장 ( Null value 는 가능 )
- 배열이므로 , hash 충돌 할 일이 없음
- hashMap 보다 빠름

동기화 하려면

```java
collections.synchronizedMap(new EnumMap<EnumKey, V>(...));
```

### EnumSet

EnumSet 은 내부적으로 Bit Vector 사용

```java
public void add(E e) {
        elements |= (1L << e.ordinal());
    }

    // 열거형 상수 제거
    public void remove(E e) {
        elements &= ~(1L << e.ordinal());
    }

    // 열거형 상수 포함 여부 확인
    public boolean contains(Object e) {
        if (e == null)
            return false;
        int index = ((Enum)e).ordinal();
        return (elements & (1L << index)) != 0;
    }

```

---

- EnumType 의 값만 넣을 수 있다.
- null 요소 허용 X
- hashSet 보다 빠름

```java
EnumSet<CardStatus> enumSet = EnumSet.of(CardStatus.SPACE);
EnumSet<CardStatus> enumSet = EnumSet.allOf(CardStatus.class);
EnumSet<CardStatus> enumSet = EnumSet.noneOf(CardStatus.class);
```

