# 표준 함수형 인터페이스를 사용해라
Favor the use of standard functional interfaces 

## Java 8, 람다 등장 후 API 디자인 패턴의 흐름
### Java 8 이전
- 템플릿 메서드 패턴을 많이 사용함
  - 템플릿 메서드 패턴 : 상위 클래스의 기본 메서드를 재정의해 원하는 동작 구현
### Java 8 이후
- 같은 효과의 함수 객체를 받는 정적 팩터리, 생성자를 제공하는 것
-  = *함수 객체를 매개변수로 받는 생성자와 메서드를 더 많이 만들어야 한다.*
  - 정적 팩터리 메소드 : 객체 생성의 역할을 하는 클래스 메서드, 직접적으로 생성자를 통해 객체를 생성하는 것이 아닌, 메서드를 통해서 객체를 생성하는 것
  - of, from, getInstance ...

## 함수형 인터페이스 (@FunctionalInterFace)
``` java
@FunctionalInterface
public interface MyFunctionalInterface {
    public void method();
}
```
- <b>하나의 추상메소드가 선언된 인터페이스</b>   
람다식이 하나의 메소드를 정의하기 때문에, 하나의 추상메소드가 선언된 인터페이스만이 람다식의 타겟 타입이 될 수 있다.   
람다식은 함수형 인터페이스가 가지고 있는 추상 메소드의 선언 형태에 따라서 작성 방법이 달라짐.

# 표준 함수형 인터페이스
java.util.function 표준 API 패키지에 다양한 용도의 표준 함수형 인터페이스가 있으니   
<b>❗️필요한 용도에 맞는게 있다면, 직접 구현하지 말고 표준 함수형 인터페이스를 활용해라❗️</b>    
이 패키지에서 제공하는 함수형 인터페이스의 목적 : 메소드, 생성자의 매개타입으로 사용되어 람다식을 대입할 수 있도록 하기 위함

| 종류 | 추상 메소드 특징 |  |  예 |
| - | - | - | - |
| `Consumer` | 매개값 O, 리턴값 X | 매개값⇒[Consumer] | `System.out::println` |
| `Predicate` | 매개값 O, 리턴값 O (boolean), 매개값을 조사해서 true/false 리턴 | 매개값⇒[Predicate]⇒boolean | `Collection::isEmpty` |
| `Operator` | 매개값 O, 리턴값 O, 주로 매개값을 연산하고 결과 리턴 | 매개값⇒[Operator]⇒리턴값 | `String::toLowerCase` |
| `Function` | 매개값 O, 리턴값 O, 주로 매개값을 리턴값으로 변환하고 결과 리턴 |매개값⇒[Function]⇒리턴값 | `Arrays::asList` |
| `Supplier` | 매개값 X, 리턴값 O |[Supplier]⇒boolean  | `Instant::now` |
- java.util.function 패키지에는 총 43개의 인터페이스가 담겨 있어 전부 기억하기는 어렵지만 위 기본 인터페이스만 기억하면, 나머지 변형을 충분히 유추할 수 있음
  - 각 기본 인터페이스는 기본 타입인 int, long, double 용으로 3개의 변형이 생겨남 (+ ⍺)
  - ex) Consumer ⇒ IntConsumer, LongConsumer, DoubleConsumer
    
### Consumer
리턴값이 없는 accept()메소드를 가지고 있으며, 단지 매개값을 소비하는 역할만 한다. 

``` java
        // Consumer 인터페이스 정의
        Consumer<String> printUpperCase = str -> System.out.println(str.toUpperCase());
        Consumer<String> printLength = str -> System.out.println("Length: " + str.length());

        // andThen 메소드를 사용하여 두 Consumer를 연결
        Consumer<String> combinedConsumer = printUpperCase.andThen(printLength);

        // 결과 출력
        combinedConsumer.accept("hello"); // HELLO \n Length: 5
```
### Predicate
매개변수들을 조사해서 true, false로 리턴하는 역할을 한다. 

``` java
        // Predicate 인터페이스 정의
        Predicate<Integer> isEven = x -> x % 2 == 0;

        // negate 메소드를 사용하여 Predicate 반전
        Predicate<Integer> isOdd = isEven.negate();

        // 결과 출력
        System.out.println(isOdd.test(5)); // true (5는 홀수)
```

### Operator
매개변수와 리턴값이 있는 apply메소드를 가지고 있다. Function과 비슷하지만 매개값을 리턴값으로 변환/매핑하기보다는 연산을 수행한 후 동일한 타입으로 리턴하는 역할을 한다
``` java
        // UnaryOperator 람다 표현식
        UnaryOperator<Integer> doubleIt = x -> x * 2;

        // 값 적용
        int result = doubleIt.apply(5);

        // 결과 출력
        System.out.println(result); // 10
```

### Function
매개변수와 리턴값이 있는 apply메소드를 가지고 있다. 매개값을 리턴 값으로 매핑하는 역할을 한다. 
``` java
        // Function 람다 표현식
        Function<String, Integer> strLength = str -> str.length();

        // 값 적용
        int result = strLength.apply("hello");

        // 결과 출력
        System.out.println(result); // 5
```

### Supplier
``` java
        // Supplier 인터페이스 정의
        Supplier<String> supplier = () -> "Hello, World!";

        // get 메소드를 사용하여 값을 제공
        String result = supplier.get();

        // 결과 출력
        System.out.println(result); // Hello, World!
```
## 주의 할 점
- 📌 기본 함수형 인터페이스에 박싱된 기본 타입을 넣어 사용하지 말자.
  - 동작은 하지만 item 61(박싱된 기본 타입 대신 기본 타입을 사용하라)에 위배, 계산량이 많을 땐 성능이 느려진다
- 📌 대부분의 경우에는 직접 작성하지 말고 표준 함수형 인터페이스를 사용하라

### 함수형 인터페이스를 직접 작성해야 하는 경우 주의점!
- 표준 인터페이스 중 필요한 용도에 맞는게 없다면 직접 작성해야 할 수도 있다.
- 하지만! 이런 경우 아래 규칙을 꼭 지켜서 설계해라
  1. API에서 자주 쓰이며, 이름 자체가 용도를 명확히 설명해준다
  2. 구현하는 쪽에서 반드시 지켜야할 규약이 있다.
  3. 유용한 디폴트 메소드를 제공할 수 있다.
- @FunctionalInterface 어노테이션을 꼭 사용해라
이 어노테이션이 없더라도 하나의 추상 메소드만 있다면 함수형 인터페이스로 작동하지만, 꼭 명시적으로 작성해주는 것이 좋다
1. 해당 클래스의 코드를 읽을 이에게 이 인터페이스가 람다용으로 설계된 것임을 알려준다.
2. 해당 인터페이스가 추상 메서드를 오직 하나만 가지고 있어야 컴파일되게 해준다.
3. 유지보수 과정에서 누군가 실수로 메서드를 추가하지 못하게 막아준다.   
 
### 함수형 인터페이스를 사용할 때의 주의점
서로 다른 함수형 인터페이스를 같은 위치의 인수로 받는 메서드들을 다중정의(Overloading)해서는 안된다.   
클라이언트에게 불필요한 모호함만 안겨줄 뿐이며, 이 모호함으로 인해 실제로 문제가 일어나기도 한다.   
ExecutorService의 submit 메서드는 Callable<T>를 받는 것과 Runnable을 받는 것을 다중정의하고 있어서, submit을 적절히 사용하기 위해 형변환을 해줘야 하는 경우가 생긴다.

<img width="703" alt="스크린샷 2024-03-17 오후 1 34 38" src="https://github.com/NoSubject-Study/effective-java-study/assets/37797830/128bc2ae-4b0c-407a-8ca1-b65d61ef92d7">
<img width="725" alt="image" src="https://github.com/NoSubject-Study/effective-java-study/assets/37797830/09cea2e1-648c-405a-b142-f434b78bef3a">

