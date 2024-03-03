# 람다보다는 메서드 참조를 사용하라
Prefer method references to lambdas

## 메소드 참조(Method Reference)란?
이미 있는 메소드들을 참조해서 람다식을 단순하게 만드는데 사용되는 `람다식의 축약형`
> Method references are a special type of lambda expressions. They’re often used to create simple lambda expressions by referencing existing methods.

### 장점
- (+) 가독성을 높일 수 있다.
- (+) 의도를 더 명확하게 보여줄수 있다 (때에 따라서)

### 예시       
``` java
// 람다 사용
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
```

``` java
// java.util.Comparator.comparing 메소드 참조
inventory.sort(comparing(Apple::getWeight));
```

    
|Lambda expression|Method Reference|
|---|---|
|(Apple apple) -> apple.getWeight()|Apple::getWeight()|
|Thread.currentThread().dumpStack()|Thread.currentThread()::dumpStack()|
|(str, i) -> str.substring(i)|String::substring|
|(String s) -> System.out.println(s)| System.out::println(s)|
|(String s) -> this.isValidName(s)| this::isValidName(s)|


## 메서드 참조의 유형
1. 정적 메서드 참조
정적 메서드를 가리키는 메서드 참조
``` java
public static int parseInt(String s) throws NumberFormatException {
    return parseInt(s,10);
}

str -> Integer.parseInt(str);
Integer::parseInt();
```
2. 인스턴스 메서드 참조 
- (A) 다양한 형식의 인스턴스 메서드 참조 VS 기존 객체의 인스턴스 메서드 참조
  - 다양한 형식의 인스턴스 메서드 참조     
   String::length같은 메서드 참조를 활용해서 람다 표현식의 파라미터로 전달
   ``` java
   strings.stream()
       .sorted(Comparator.comparing(String::length)) // String::length 메서드 참조를 Comparator에 전달
       .forEach(System.out::println); // 정렬된 문자열 출력
   ```
  - 람다 표현식에서 현존하는 외부 객체의 메서드 호출할때 사용     
    private 헬퍼 메소드를 정의한 상황에서 유용하게 활용 가능
  ``` java
  private boolean isValidName(String string) {
      return Character.isUpperCase(string.charAt(0));
  }

  filter(words, this::isValidName)
  ```
- (B) 한정적(bound) 인스턴스 메서드 참조 VS 비한정적(unbound) 인스턴스 메서드 참조
   - 한정적 인스턴스 메서드 참조 :
     - 한정적 인스턴스 메서드 참조는 특정 객체에 한정 됨.
     - 객체명::메서드명 형태로 사용.
     - 해당 메서드를 특정 객체에 대해 호출
     - ``` java
       // 람다
       Instant then = Instant.now();
       t -> then.isAfter(t)

       // 메서드 참조
       Instant.now()::isAfter
       ```
   - 비한정적 인스턴스 메서드 참조 :
     - 보통 스트림에서 map과 filter 함수에 자주 쓰임
     - 클래스명::메서드명 형태로 사용
     - 해당 메서드를 특정 객체에 종속되지 않고 호출할 수 있음
     - ``` java
       str -> str.toLowerCase()
       String::toLowerCase
       ```
4. 클래스 생성자 메서드 참조    
ClassName::new처럼 클래스명과 new 키워드를 이용해서 기존 생성자의 참조를 만들 수 있는 방식
5. 배열 생성자 메서드 참조.
``` java
  // 람다
  len -> new int[len]

  // 메서드 참조
  int[]::new
```
## 메서드 참조를 사용할 때 유의할 점
IDE들은 대개 람다를 메서드 참조로 대체하라고 권고하지만, 간혹 람다가 메서드 참조보다 짧고 명확한 경우가 있다.    
무조건 메서드 참조가 가독성이 좋은 것은 아니니 잘 선택해서 사용하기

### 예시       
GoshThisClassNameIsHumongous라는 클래스 안에 있는 메서드를 활용할 때
``` java
// 람다 사용
service.execute(() -> action());
```
``` java
// 메서드 참조 사용
service.execute(GoshThisClassNameIsHumongous::action);
```


