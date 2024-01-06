## 이왕이면 제네릭 메서드로 만들라  

### 로타입 사용 - 수용불가  
```` java
public static Set union(Set s1, Set s2) {
  Set result = new HashSet(s1);
  result.addAll(s2);
  return result;
}
````

### 제네릭 메서드  
```` java
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
  Set<E> result = new HashSet<>(s1);
  result.addAll(s2);
  return result;
}
````

### 제네릭 메서드 활용  
```` java
public static void main(String[] args) {
  Set<String> guys = Set.of("tom", "dik", "harry");
  Set<String> stooges = Set.of("lary", "moe", "curly");
  Set<String> aflCio = union(guys, stooges);
}
````

해당 예시는 한정적 와일드카드 타입을 사용해 더 유연하게 개선할 수 있다.  
때때로 불변 객체를 여러 타입으로 활용 가능해야 할 때가 있다.  
매번 그 객체의 타입을 바꿔주는 정적 팩터리를 만들어야 하며, 이를 제네릭 싱글턴 팩터리라고 한다.  
Collections.reverseOrder 같은 함수 객체나 Collections.emptySet 같은 컬렉션용으로 사용한다.  

</br>

### 제네릭 싱글턴 팩터리 패턴  
항등함수(identity function)를 담은 클래스를 만들고 싶은 경우  
자바 라이브러리 function.identity 사용하면 쉽게 구현가능하다.  
항등함수 객체는 상태가 없으니 요청마다 새로 생성하는 것은 낭비이다.  
실체화되지 않는 제네릭을 사용해서 소거 방식으로 제네릭 싱글턴 하나로 사용한다.  

### 제네릭 싱글턴 활용  
```` java
private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

@SuppressWarnings("unchecked")
public static <T> UnaryOperator<T> identityFunction() {
  return (UnaryOperator<T>) IDENTITY_FN;
}

public static void main(String[] args) {
  String[] strings = { "a", "b", "c" };
  UnaryOperator<String> sameString = identityFunction();
  for (String s : strings)
    System.out.println(sameString.apply(s));

  Number[] numbers = { 1, 2.0, 3L };
  UnaryOperator<Number> sameNumber = identityFunction();
  for (Number n : numbers)
  	System.out.println(sameNumber.apply(n));
}
````
</br>

### 재귀적 타입 한정(recursive type bound)
상대적으로 드물긴 하지만, 자기 자신이 들어간 표현식을 사용해서 타입 매개변수의 허용 범위를 한정할 수 있다.  
재귀적 타입 한정은 주로 자연적 순서를 정하는 Comparable 인터페이스와 같이 사용한다.  
Comparable 구현한 원소의 컬렉션을 입력받는 메서드들은 주로 그 원소들을 정렬, 검색, 비교하는 식으로 사용한다.  
이 기능을 수행하려면 컬렉션의 모든 원소가 상호 비교 가능하다는 제약이 있어야 한다.  

### 재귀적 타입 한정을 이용한 상호 비교 가능 표현  
```` java
public interface Comparable<T> {
  int compareTo(T o);
}

public static <E extends Comparable<E>> E max(Collection<E> c) {
  if (c.isEmpty())
    throw new IllegalArgumentException("컬렉션이 비어 있습니다.");

  E result = null;
  for (E e : c)
    if (result == null || e.compareTo(result) > 0)
      result = Objects.requireNonNull(e);

  return result;
}
````



