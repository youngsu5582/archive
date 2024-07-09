---
tags:
  - 자바
도서명: 모던 자바 인 액션
aliases:
---
요구사항은 변해감
( 앞서 말한 예제처럼 , 초록 사과 -> 무거운 사과 -> 초록색이면서 무거운 사과 ... )

### Behavior Parameterization

자주 변화하는 요구 사항 변경에 대처할 수 있는 SW 개발 패턴

코드 블록을 바로 실행하는게 아닌 , 메소드 인자로 코드 블록을 전달 가능하게 한다.
-> 메소드 행동이 해당 코드 블록 기반 파라미터화

- 리스트의 모든 요소에 "무언가"를 수행할 수 있는 경우
- 리스트 처리를 마친후 "다른 무언가" 를 수행할 수 있는 경우
- 오류 만날 때 "또 다른 무언가"를 수행하는 경우

#### Example

초록색 사과만 선택을 하는
```java
public static List<Apple> filterGreenApple(List<Apple> inventory){
	List<Apple> result = new ArrayList();
	for(Apple apple : inventory){
		if("green".eqauls(apple.getColor()){
			result.add(apple);
		}
	}
	return result;
}
```

여기서 주의할 점은 , "green".eqauls(...) 를 해야 한다 - NullPointerException 을 피하기 위해서

=> 빨간색 사과를 찾는 코드로 바꿔야 하면?

단순 코드 복제 가능

##### 1. parameterizing the color

```java
public static List<Apple> filterAppleByColor(List<Apple> inventory,String color){
	List<Apple> result = new ArrayList();
	for(Apple apple : inventory){
		if(apple.getColor().equals(color)){
			result.add(apple);
		}
	}
	return result;
}
```

```java
List<Apple> greenApples = filterApplesByColor(inventory, "green"); 
List<Apple> redApples = filterApplesByColor(inventory, "red");
```

이렇게 color 를 주입함으로써 , 색깔에 맞게 필터링 가능

만약 일정 무게 이상인 사과를 찾는다면?
```java
public static List<Apple> filterAppleByWeight(List<Apple> inventory,Integer weight){
	List<Apple> result = new ArrayList();
	for(Apple apple : inventory){
		if(apple.getWeight() > weight){
			result.add(apple);
		}
	}
	return result;
}
```

이대로 괜찮은가?
중복 된 코드로 , DRY (Don't Repeat Yourself) 하지 않음
검증 요소를 Flag 에 따라 검증해볼까?
##### 2. filtering with all attribute
```java
public static List<Apple> filterApple(List<Apple> inventory,String color,Integer weight,boolean flag){
	List<Apple> result = new ArrayList();
	for(Apple apple : inventory){
		if((flag && apple.getColor().equals(color) > weight)||(!flag && apple.getWeight() > weight)){
			result.add(apple);
		}
	}
	return result;
}

```

매우 쓰레기 같은 코드 - 더럽고 , true/false 가 뭘 의미하는지 모름

지금까지는 String , Integer , boolean 같은 값들을 파라미터로 전달
=> 결국 코드간 중요한건 검증하는 메소드

### Behavior Parameterization

사과의 속성을 기반으로 , boolean 을 반환하는 것이 중요하다
( is it green? is it heavier than 150g? )
이를 Predicate 라고 부름

```java
public interface ApplePredicate{
	boolean test(Apple apple);
}
```

이를 통해 , 기존의 로직을 상속받아 구현할 수 있다
```java
public class AppleHeavyWeightPredicate implements ApplePredicate{
	public boolean test(Apple apple){
		return apple.getWeight() > 150;
	}
}
public class AppleGreenColorPredicate implements ApplePredicate{
	public boolean test(Apple apple){
		return "green".equals(apple.getColor());
	}
}
```

실행 시간에 전략을 선택 가능하게 해준다
ApplePredicate 를 파라미터로 넣고 , 런타임에 주입

##### 3. filtering by abstract criteria

```java
public static List<Apple> filterAppleByWeight(List<Apple> inventory,ApplePredicate predicate){
	List<Apple> result = new ArrayList();
	for(Apple apple : inventory){
		if(predicate.test(apple)){
			result.add(apple);
		}
	}
	return result;
}
```

이제 , 어떤 조건이든 ApplePredicate 를 implement 하기만 하면 된다

```java
public class AppleRedAndHeavyPredicate implements ApplePredicate{
	public boolean test(Apple apple){
		return "red".equals(apple.getColor()) && apple.getWeight() > 150;
	}
}
```

```java
List<Apple> redAndHeavyApples = filter(inventory, new AppleRedAndHeavyPredicate());
```

코드를 인라인으로 전달하는 것과 유사
-> 하지만 , 여전히 클래스를 생성해서 전달해야 하므로 매우 비효율적

![450](https://i.imgur.com/uqtszGF.png)


##### Example

Pretty 하게 출력하는 함수를 작성하라
```java
public static void prettyPrintApple(List<Apple> inventory, ???){ 
	for(Apple apple: inventory) {
		String output = ???.???(apple);
		System.out.println(output);
	}
}
```

```java
public interface AppleFormatter{
	String execute(Apple apple);
}
```
=>
```java
public class AppleFancyFormatter implements AppleFormatter{
	public String accept(Apple apple){
		String characteristic = apple.getWeight() > 150 ? "heavy" : "light";
		return "A" + characteristic + " " + apple.getColor() + "apple";
	}
}
```
```java
public static void prettyPrintApple(List<Apple> inventory, AppleFormatter formatter){
	for(Apple apple: inventory){  
		String output = formatter.accept(apple); 
		System.out.println(output);
	} 
}
```

매우 간편하게 구현 가능

```java
public class AppleHeavyWeightPredicate implements ApplePredicate {
    public boolean test(Apple apple) {
        return apple.getWeight() > 150;
    }
}

public class AppleGreenColorPredicate implements ApplePredicate {
    public boolean test(Apple apple) {
        return "green".equals(apple.getColor());
    }
}

public class FilteringApples {
    public static void main(String... args) {
        List<Apple> inventory = Arrays.asList(new Apple(80, "green"),
                                              new Apple(155, "green"),
                                              new Apple(120, "red"));

        List<Apple> heavyApples = filterApples(inventory, new AppleHeavyWeightPredicate());
        List<Apple> greenApples = filterApples(inventory, new AppleGreenColorPredicate());
    }

    public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
        List<Apple> result = new ArrayList<>();
        for (Apple apple : inventory) {
            if (p.test(apple)) {
                result.add(apple);
            }
        }
        return result;
    }
}

```

이 코드 역시도 최선인가?

- 불필요한 클래스 키워드 작성
- 한번 사용하는 값이나 , new 를 통해 객체 생성
- 클래스에 메소드가 하나밖에 없지만 클래스를 사용해야 함

#### 익명 클래스

익명 클래스는 로컬 클래스와 유사 ( block 안에 정의된 클래스 )
하지만 익명 클래스는 이름 X + 클래스 선언 과 동시에 생성 가능

##### 4. using an anonymous class

```java
List<Apple> redApples = filterApples(inventory, new ApplePredicate(){
	public boolean test(Apple apple){
		return "red".equals(apple.getColor())
	}
})
```

익명 클래스는 GUI Application 에서 주로 사용

```java
button.setOnAction(new EventHandler<ActionEvent>() {
	public void handle(ActionEvent event) {
		System.out.println("Woooo a click!!"); 
	}
});
```

하지만 , 이것 역시 수많은 보일러 플레이트 코드가 필요하다
 new ApplePredicate(){
	public boolean test(Apple apple){
해당 부분은 사실상 불필요

이러한 점들은 이해하기 어렵게 하고 , 불필요한 작성 시간 증가
=> 람다 표현식이 이러한 점을 해결해줌

##### 5. lambda expression

```java
List<Apple> result = filterApple(inventory, apple -> "red".equals(apple.getColor()));
```

매우 간결하게 표현 가능
단순히 검증하는  코드 작성만 하면 끝

![400](https://i.imgur.com/o1HgkOk.png)

##### 6. abstracting over List type

```java
public interface Predicate<T>{
	boolean test(T t);
}
public static <T> List<T> filter(List<T> list,Predicate<T> p){
	List<T> result = new ArrayList<>();
	for(T e : list){
		if(p.test(e)){
			result.add(e);
		}	
	}
	return result;
}
```

제네릭 타입을 사용함으로써 , 타입에 구애받지 않고 어떤 연산이든 filter 가능

### Real World Example

행동 파라미터를 통해 요구 사항 변화에 쉽게 적용을 할 수 있음을 확인

Java API 역시도 파라미터화 가능

#### Sorting with a Comparator

List 의 내장 sort 메소드는 Comparator 객체를 통해 파라미터화 
```java
public interface Comparator<T>{
	public int compare(T o1,T o2);
}
```

```java
inventory.sort(new Comparator<Apple>(){
	public int compare(Apple a1,Apple a2){
		return a1.getWeight().compareTo(a2.getWeight());
	}
})
```

익명 클래스를 통해 , 자체 구현 가능하나?

```java
inventory.sort(
	(Apple a1,Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
)
```

람다로 더욱 편하게 구현 가능

#### Executing with Runnable

스레드는 경량 프로세스
-> 여러 스레드가 다른 코드 실행 가능
```java
public interface Runnable{
	public void run();
}
```

```java
Thread t = new Thread(new Runnable(){
	public void run(){
		System.out.println("Hello Modern Java!");
	}
})
```

이렇게 , 익명 Runnable class 로 생성 가능

```java
Thread t = new Thread(()->System.out.println("Hello Stream!"));
```

더욱 간편하게 가능!

#### GUI event handling

```java
Button button = new Button("Send"); 
button.setOnAction(new EventHandler<ActionEvent>() {
	public void handle(ActionEvent event) { 
		label.setText("Sent!!");
	} 
});
```

button.setOnAction((ActionEvent event)->label.setText("Sent!"));

역시 가능!

메소드에 함수를 즉 람다를 파라미터로 넣는다는 건 어마어마한 발전
익명 클래스 또는 , 불필요한 보일러 플레이트를 전부 생략

해당 인터페이스의 람다 함수라는 것을 암시적으로 표현하게 하여 어마어마한 코드 줄 생략을 이끌어냈다

