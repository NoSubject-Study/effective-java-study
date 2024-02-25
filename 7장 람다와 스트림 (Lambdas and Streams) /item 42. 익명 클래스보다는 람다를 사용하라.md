# 익명 클래스 보다는 람다를 사용하라
Prefer Lambdas to Anonymous Classes 

## 람다란?
메서드를 하나의 식으로 표현한 것, 메서드로 전달할 수 있는 익명함수를 단순화 한 것

#### ASIS
``` java
Collections.sort(words, new Comparator<String> () {
  public int compare(String s1, String s2) {
    return Integer.compare(s1.length(), s2.length());
  }
});
```
- 이전에는 특정 함수나 동작을 나타내는 <b>함수 객체 (functional object)</b>를 만들기 위해 <b>익명 클래스</b>를 사용했다.
- 그러나 길고 깔끔하지 않아 함수형 프로그래밍, 동적 파라미터화를 막는 요소였다. 

#### TOBE
```java
Collections.sort(words, (s1, s2) -> Integer.compare(s1.length(), s2.length());
```

### 람다 표현식
![image](https://github.com/NoSubject-Study/effective-java-study/assets/37797830/3fd0bca3-2d25-43fd-9e1d-56bb1275d413)
- 파라미터 리스트 : Comparator의 compare 메서드 파라미터
- 화살표 : 람다의 파람미터 리스트/바디 구분
- 람다 바디 : 동작, 람다의 반환값에 해당하는 표현식

|사용 사례|람다 예제|
|---|---|
|불리언 표현식|```(List<String> list) -> list.isEmpty()```|
|객체 생성|```()-> new Apple(10)```|
|객체에서 소비|```(Apple a) -> {System.out.println(a.getWeight());}```|
|객체에서 선택/추출|```(String s) -> s.length()```|
|두 값을 조합|```(int a, int b) -> a * b```|
|두 객체 비교|```(Apple a1, Apple a) -> a1.getWeight().compareTo(a2.getWeight())```|


- <람다, 매개변수(s1, s2), 반환 값>의 타입은 컴파일러가 추론하기 때문에 명시하지 않아도 된다.

## 어디에서 람다를 사용할 수 있을까?
함수형 인터페이스라는 문맥에서 람다 표현식을 사용할 수 있다.

### 함수형 인터페이스 (Functional Interface)
정확히 하나의 추상 메서드를 지정하는 인터페이스, 자바 API의 함수형 인터페이스로는 Comparator, Runnable 등이 있다.
``` java
Runnable r1 = () -> System.out.println("Hello World 1");

public static void process(Runnalbe r) {
    r.run();
}

process(r1);
process(() -> System.out.println("Hello World 2");
```

``` bash
Hello World 1
Hello World 2
```


### Lambda를 실용적으로 활용하기 (Enum에 동작이 결합된 경우)
- 동작이 상수마다 달라야 해서 상수별 클래스 몸체를 이용해 각 상수에서 apply 메소드를 재정의
#### ASIS
``` java
public enum Operation {
  PLUS { public double apply(double x, double y) { return x + y;}},
  MINUS { public double apply(double x, double y) { return x - y; }},
  TIMES { public double apply(double x, double y) { return x * y; }},
  DIVIDE { public double apply(double x, double y) {  return x / y; }};

 public abstract double apply(double x, double y);
}
```

- enum의 인스턴스 필드를 이용해서, 상수별로 다르게 동작하는 코드를 쉽게 구현 가능. 
#### TOBE
``` java
public enum Operation {
  PLUS((x,y) -> x + y),
  MINUS((x,y) -> x - y),
  TIMES((x,y) -> x * y),,
  DIVIDE((x,y) -> x / y),

  private final DoubleBinaryOperator operator;

  Operation(DoubleBinaryOperator operator) {
    this.operator = operator;
  }

  public double apply(double x, double y) {
    return operator.applyAsDouble(x, y);
  }
}
```
     
      
### 람다를 쓰지 말아야 하는경우
- 람다는 이름도 없고 문서화도 불가능하다
  - 코드 자체로 동작이 명확히 설명되지 않거나 코드 줄 수가 많아지면 람다를 쓰지 말아야 한다.
- enum 타입 생성자에 넘겨지는 인수들의 타입도 컴파일 타임에 추론되기 때문에 enum 타입 생성자 안의 람다는 enum타입의 인스턴스 필드에 접근 불가능
- 람다는 자신을 참조할 수 없다.
  - 람다에서의 this 키워드는 바깥 인스턴스를 가리킨다. 그래서 자신을 참조하려면 익명 클래스를 사용해야한다.


 ### 💡 Quiz
 다음 중 람다 표현식을 올바로 사용한 코드는?
``` java
// 1.
execute(() -> {});
public void execute(Runnable r) {
    r.run();
}
```

``` java
// 2.
public Callable<String> fetch() {
    return () -> "Tricky example ;-)";
}
```

``` java
// 3.
Predicate<Apple> p = (Apple a) -> a.getWeight();
```
