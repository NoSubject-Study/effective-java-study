# 제네릭

제네릭은 자바 5부터 사용가능하다.  
자바 5 이전에는 컬렉션에서 객체를 꺼낼 때마다 형변환이 필요했다.  
제네릭을 사용하면 엉뚱한 타입의 객체를 넣는 시도를 컴파일 시점에서 차단한다.

<br/>

### 로 타입을 사용하지 마라

**용어 정리**
1. 매개변수화 타입(parameterized type)  
    List&lt;String&gt;  

2. 실제 타입 매개변수(actual type parameter)  
    String  

3. 제네릭 타입(generic type)  
    List&lt;E&gt;
   
4. 정규 타입 매개변수(formal type parameter)  
    E
   
5. 비한정적 와일드 카드 타입(unbounded wildcard type)  
    List&lt;?&gt;
    
6. 로 타입(raw type)  
    List
    
7. 한정적 타입 매개변수(bounded type)  
    &lt;E extends Number&gt;
    
8. 재귀적 타입 한정(recursive type bound)  
    &lt;T extends Comparable&lt;T&gt;&gt;
    
9. 한정적 와일드카드 타입(bounded wildcard type)  
    List&lt;? extends Number&gt;  

10. 제네릭 메서드(generic method)  
    static &lt;E&gt; List&lt;E&gt; asList(E[] a)  

11. 타입 토큰(type token)  
    String.class  

<br/>

### 타입 안전성  

로 타입 사용을 언어 차원에서 막지 않았다.  
하지만 로 타입을 사용하면 제네릭이 주는 안전성과 표현력을 모두 상실한다.  
로 타입을 지원하는 이유는 제네릭 이전의 기존 코드의 호환성을 위해 지원한다.

#### 로 타입 사용으로 인한 타입 안전성 상실  
```` java
private final Collection stamps = ...;
stamps.add(new Coin(...));

for (Iterator i = stamps.iterator(); i.hasNext(); ) {
  Stamp stamp = (Stamp) i.next();
  stamp.cancel();
}
````

#### 매개변수화된 컬렉션 타입 사용으로 타입 안전성 확보(형변환 보장)  
```` java
private final Collection<Stamp> stamps = ...;
stamps.add(new Coin(...));
````  

<br/>

### 로 타입과 제네릭 비교  
1. _List_  
로 타입으로 선언한 리스트  
타입 안전성을 보장할 수 없다.  

2. _List&lt;Object&gt;_   
제네릭 타입으로 선언한 리스트  
모든 타입을 허용한다는 의사를 컴파일러에 명확히 전달한다.  

**제네릭 하위 타입 규칙**  
매개변수로 List(로 타입)를 받는 메서드에 List&lt;String&gt; 객체를 넘기는 것이 가능하다.  
매개변수로 List&lt;Object&gt;를 받는 메서드에 List&lt;String&gt; 객체를 넘기는 것이 불가능하다.  
-> List&lt;String&gt;은 List(로 타입)의 하위 타입이지만 List&lt;Object&gt;의 하위 타입이 아니다.  
-> 매개변수화 타입을 사용할 때와 달리 로 타입을 사용하면 타입 안전성을 잃게 된다.  

#### 제네릭 하위 타입 규칙의 타입 안정성  
```` java
public static void main(String[] args) {
  List<String> strings = new ArrayList<>();
  unsafeAdd(strings, Integer.valueOf(42));
  String s = strings.get(0);
}

//런타임 오류 발생
private static void unsafeAdd(List list, Object o) {
  list.add(o);
}

//컴파일 오류 발생
private static void safeAdd(List<Object> list, Object o) {
  list.add(o);
}
````  

<br/>

### 로 타입과 비한정적 와일드카드 타입 비교  
1. _Set_  
로 타입 형식으로 선언한 셋  
아무 원소나 넣을 수 있으니 타입 불변식을 훼손하기 쉽다.  

2. _Set&lt;?&gt;_  
비한정적 와일드카드 타입  
모종의 타입 객체만 저장할 수 있다.  
null 원소를 제외한 어떤 원소도 넣을 수 없다.  
원소를 넣는 것을 방지하고 꺼낼 수 있는 객체의 타입도 전혀 알 수 없다.  

#### 모르는 타입의 원소를 받기 위해 로 타입 사용  
```` java
static int numElementsInCommon(Set s1, Set s2) {
  int result = 0;
  for (Object o1 : s1)
    if (s2.contains(o1))
      result++;
  return result;
}
````

#### 비한정적 와일드카드 타입(unbounded wildcard type)  
```` java
static int numElementsInCommon(Set<?> s1, Set<?> s2) { ... }
````  

<br/>

### 로 타입을 사용해야 하는 경우  
1. _class_ 리터럴  
class 리터럴은 매개변수화 타입 사용이 불가능하다.  
-> List.class, String[].class, int.class 허용  
-> List&lt;String&gt;.class, List&lt;?&gt;.class 불가  

2. _instanceof_ 연산자  
비한정적 와일드카드 타입 이외의 매개변수화 타입 적용이 불가능하다.  
로 타입이든 비한정적 와일드 카드 타입이든 똑같이 동작하기 때문에 가독성을 위해 로 타입을 사용한다.  

```` java
if (o instanceof Set) {
  Set<?> s = (Set<?>) o;
  ...
}
````


