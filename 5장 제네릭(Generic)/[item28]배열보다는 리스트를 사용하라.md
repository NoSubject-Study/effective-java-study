## 배열보다는 리스트를 사용하라  

### 배열과 제네릭 타입 비교
1. 배열은 공변(covariant)  
공변이란 함쎄 변한다는 뜻이다.  
sub이 super의 하위타입이라면 배열 sub[]는 배열 super[]의 하위타입이 된다.  

```` java
Object[] objectArray = new Long[1];
objectArray[0] = "타입이 달라 넣을 수 없다.";	//ArrayStoreException

List<Object> objectList = new ArrayList<Long>(); //컴파일 오류
objectList.add("타입이 달라 넣을 수 없다.");
````

2. 배열은 실체화(reify)  
배열은 런타임에도 자신이 담기로 한 원소의 타입을 확인한다.  
제네릭은 런타임에 원소 타입 정보가 소거된다.(컴파일 시점에 모두 확인)  
  
### 제네릭 배열 생성
배열과 제네릭은 해당 차이로 인해 어울리기 쉽지 않다.  
아래와 같이 코드 작성시 컴파일할 때 제네릭 배열 생성 오류가 발생한다.  
    new List&lt;E&gt;[], new List&lt;String&gt;[], new E[]  
    
제네릭 배열 생성을 막은 이유는 타입 안전하지 않기 때문이다.  
제네릭 타입 시스템의 취지는 기본적으로 런타임에 ClassCastException 발생을 막는 것이다.  

### 만약 제네릭 배열 생성을 허용한다면
```` java
List<String>[] stringLists = new List<String>[1];
List<Integer> intList = List.of(42);
Object[] objects = stringLists;
objects[0] = intList;
String s = stringLitsts[0].get(0);
````

### 실체화 불가 타입(non-reifiable type)
실체화되지 않아서 런타임에는 컴파일타임보다 타입 정보를 적게 가지는 타입이다.  
대표적으로 제네릭이 해당된다.(E, List&lt;E&gt;, List&lt;String&gt;)  
예외적으로 소거 메커니즘에 의해 런타임에 타입을 알 수 있는 비한정적 와일드카드 타입은 실체화 가능 타입이다.(List&lt;?&gt;, Map&lt;?,?&gt;)  

</br>

제네릭 타입과 가변인수 메서드(varargs method)를 함께 쓰면 해석하기 어려운 경고가 발생한다.  
    가변인수 메서드를 호출할 때마다 가변인수를 담을 배열이 하나 생성된다.  
    이때 생성된 배열이 실체화 불가 타입이라면 경고가 발생하는 것이다.  
    @SafeVarargs 어노테이션으로 대처 가능하다.  

</br>

배열 형변환시 제네릭 배열 생성 오류나 비검사 형변환 경고가 발생하는 경우  
    대부분 E[] 배열 대신 List&lt;E&gt; 사용으로 해결 가능하다.  
		성능이 살짝 나빠질 가능성이 있으나 그 대신 타입 안전성과 상호운용성에 좋다.  

### 제네릭을 사용하지 않은 경우
```` java
public class Chooser {
  private final Object[] choiceArray;

  public Chooser(Collection choices) {
    choiceArray = choices.toArray();
  }

  public Object choose() {
    Random rnd = ThreadLocalRandom.current();
    return choiceArray[rnd.nextInt(choiceArray.length)];
  }

}
````

### 제네릭 변환 - 형변환 경고
```` java
public class Chooser<T> {
  private final T[] choiceArray;

  public Chooser(Collection<T> choices) {
    choiceArray = (T[]) choices.toArray();
  }

  ...
}
````

### 제네릭 사용
```` java
public class Chooser<T> {
  privaate final List<T> choiceList;

  public Chooser(Collection<T> choices) {
  	choiceList = new ArrayList<>(choices);
  }

  public T choose() {
  	Random rnd = ThreadLocalRandom.current();
  	return choiceList.get(rnd.nextInt(choiceList.size()));
  }

}
````


