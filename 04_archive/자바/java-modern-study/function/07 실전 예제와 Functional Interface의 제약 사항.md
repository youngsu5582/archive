---
tags:
  - 자바
강의명: 모던 자바 (자바8) 못다한 이야기
강의_링크: https://www.youtube.com/watch?v=WQXXrHhLH-M
---

### Example

BigDecimal 에 $ 기호를 붙히는 간단한 예제
```java
@FunctionalInterface  
public interface BigDecimalToCurrency {  
    String toCurrency(BigDecimal value);  
}
```

```java
BigDecimalToCurrency bigDecimalToCurrency = bd -> "$" +bd.toString();  
System.out.println(bigDecimalToCurrency.toCurrency(BigDecimal.valueOf(100000)));
```

기존 , 제네릭 타입이 아닌 타입 지정하여서도 사용 가능!

### Invalid Functional Interface

```java
@FunctionalInterface  
interface InvalidFunctionInterface{  
    <T> String mkString(T value);  
}
```

기대 : 해당 Generic Type 의 문자열을 반환

```java
final InvalidFunctionInterface invalidFunctionInterface = value -> value.toString();
```

이렇게 하면 , value.toString() 에서 `Target method is generic`  에러가 발생한다!

value 만으로는 정보 추론이 힘들기 때문인듯
```java
@FunctionalInterface  
interface InvalidFunctionInterface<T>{  
    String mkString(T value);  
}


final InvalidFunctionInterface<String> invalidFunctionInterface = value -> value.toString();  
System.out.println(invalidFunctionInterface.mkString("5"));
```
이렇게 Class 선언부에 제네릭 타입을 넣어 해결이 가능은 하다.


### Deep Dive

```java
private static final <T> List<T> filter(List<T> list, Predicate<T> predicate) {  
    final List<T> result = new ArrayList<>();  
    for (final T t : list) {  
        if (predicate.test(t)) {  
            result.add(t);  
        }  
    }  
    return result;  
}
```

해당 코드는 , 조건을 만족하는 Element 만 선택

```java
final Predicate<DiscountedProduct> lessThanOrEqual30 
	= product -> product.getPrice().compareTo(new BigDecimal("30")) <= 0;
```

Price 의 값이 30보다 작을때 만족

```java
public class Product {  
    private Long id;  
    private String name;  
    private BigDecimal price;
}
```

```java
@ToString(callSuper = true)  
public class DiscountedProduct extends Product{  
    public DiscountedProduct(Long id, String name, BigDecimal price) {  
        super(id, name, price);  
    }  
}
```

해당 DiscountProduct 는 Product 를 상속 받는다

```java
final Predicate<DiscountProduct> lessThanOrEqual30 = product -> product.getPrice().compareTo(new BigDecimal("30")) <= 0;  
filter(discountedProduct, lessThanOrEqual30);  
filter(sampleProducts,lessThanOrEqual30);
```

위의 lessThanOrEqual30 에서 Predicate 의 Type 을 DiscountedProduct 로 하든 Product 로 하든 타입 불일치 에러가 뜬다!

- DiscountProduct 일 시 ? : 하단 코드에서 , `List<Product> <-> Predicate<DiscountProduct>` 로 에러 발생
- DiscountProduct 일 시 ? : 상단 코드에서 , `List<DiscountProduct> <-> Predicate<Product>` 로 에러 발생

=> 결국 , DiscountProduct 는 Product 를 extends 받음에도 에러가 발생한다!

```java
private static final <T> List<T> filter(List<T> list, Predicate<? super T> predicate) {  
    final List<T> result = new ArrayList<>();  
    for (final T t : list) {  
        if (predicate.test(t)) {  
            result.add(t);  
        }  
    }  
    return result;  
}
```

? super T 를 사용하면 해결이 된다.

클래스 A 가 클래스 B 의 하위 클래스 일 시? ( 즉 , B 가 더 높다 )
=> Predicate<`B`> 는 Predicate<`A`> 의 상위 타입으로 간주


```java
final Predicate<Product> lessThanOrEqual30 = product -> product.getPrice().compareTo(new BigDecimal("30")) <= 0;

filter(discountedProduct, lessThanOrEqual30);  
filter(sampleProducts,lessThanOrEqual30);
```

DiscountProduct 의 Super Type 은 Product!
해당 코드 두개 다 만족 ( 결국 , DiscountProduct 는 Product 의 모든걸 가지고 있기 때문에 가능 )

#### Total

```java
private static final <T> BigDecimal total(List<T> list, Function<T, BigDecimal> mapper) {  
    BigDecimal total = BigDecimal.ZERO;  
    for (final T t : list) {  
        total = total.add(mapper.apply(t));  
    }  
    return total;  
}
```

```java
final BigDecimal total = total(sampleProducts, product -> product.getPrice());  
System.out.println("total : "+total);
```

이렇게 , T 는 상관 없이 BigDemical 을 반환하기만 하면 된다!

```java
public class OrderItem {  
    private Long id;  
    private Product product;  
    private Integer quantity;  
    public BigDecimal getItemTotal(){  
        return product.getPrice().multiply(new BigDecimal(quantity));  
    }  
}
```

```java
public class Order {  
    private Long id;  
    private String orderNumber;  
    private List<OrderItem> items;  
  
    public BigDecimal totalPrice() {  
        return CalculationUtil.total(items, item -> item.getItemTotal());  
    }  
}
```

이렇게 서로서로 연관이 되어서도 구현이 가능!


### Conclusion

해당 내용은 억지로 , Stream 을 사용하지 않고 직접 구현을 했음 ( 답답할 수 있는 이유가 이 때문 )

핵심은 Generic 과 super 로 Generic 의 타입을 강제하여 ,
보편적이면서도 유동적인 함수형 프로그래밍을 할 수 있는거 같다