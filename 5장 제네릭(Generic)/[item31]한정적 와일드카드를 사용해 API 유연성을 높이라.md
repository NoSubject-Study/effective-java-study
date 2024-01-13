## 한정적 와일드카드를 사용해 API 유연성을 높이라  

매개변수화 타입은 불공변(invariant)이다.  
하지만 때론 리스코프 치환 원칙에 어긋나지 않고 불공변 방식보다 유연한 무언가가 필요하다.  

</br>

### 이전 Stack 추가 메서드  
```` java
public void pushAll(Iterable<E> src) {
  for (E e : src)
    push(e);
}

Stack<Number> numberStack = new Stack();
Iterable<Integer> integers = ...;
numberStack.pushAll(integers);

public void popAll(Collection<E> dst) {
  while (!isEmpty())
    dst.add(pop());
}

Stack<Number> numberStack = new Stack<>();
Collection<Object> objects = ...;
numberStack.popAll(objects);
````

### E 생산자(producer) 매개변수에 와일드카드 타입 적용  
```` java
public void pushAll(Iterable<? extends E> src) {
  for (E e : src)
    push(e);
}
````

### E 소비자(consumer) 매개변수에 와일드카드 타입 적용  
```` java
public void popAll(Collection<? super E> dst) {
  while (!isEmpty())
    dst.add(pop());
}
````

### PECS: Producer-Extends, Consumber-Super  
매개변수화 타입 T가 생산자라면 &lt;? extends T&gt;  
매개변수화 타입 T가 소비자라면 &lt;? super T&gt;  

</br>

### max 메서드 선언
```` java
public static <E extends Comparable<E>> E max(List<E> list)
public static <E extends Comparable<? super E>> E max(List<? extends E> list)
````

일반적으로 Comparable, Comparactor는 언제나 소비자이다.  
따라서 &lt;? super E&gt;를 사용하는 편이 낫다.  

```` java
List<ScheduledFuture<?>> scheduledFutures = ...;
````

수정 전 max는 해당 리스트를 처리할 수 없다.  
SchesuledFuture 인스턴스는 다른 Delayed 인스턴스와도 비교 가능하기 때문에 거부하는 것이다.  
일반화하면 Comparable(Comparator)을 직접 구현하지 않고, 직접 구현한 다른 타입을 확장한 타입을 지원하기 위해 와일드카드가 필요하다.  
  
```` java
public interface Comparable<E>;
public interface Delayed extends Comparable<Delayed>;
public interface ScheduledFuture<V> extends Delayed, Future<V>;
````

</br>

### 타입 매개변수와 와일드카드 선택  
두가지 모두 공통되는 부분이 있어서 어느 것을 사용해도 괜찮은 경우가 많다.  
기본적으로 메서드 선언에 타입 매개변수가 한번만 나오면 와일드카드로 대체하라.  

```` java
public static <E> void swap(List<E> list, int i, int j);
public static void swap(List<?> list, int i, int j);

public static void swap(List<?> list, int i, int j) {
  swapHelper(list, i, j);
}

private static <E> void swapHelper(List<E> list, int i, int j) {
  list.set(i, list.set(j, list.get(i)));
}
````



