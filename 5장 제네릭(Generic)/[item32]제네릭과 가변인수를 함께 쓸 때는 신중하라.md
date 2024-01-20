## 제네릭과 가변인수를 함께 쓸 때는 신중하라  
가변인수와 제네릭은 궁합이 좋지 않다.  
가변인수는 메서드에 넘기는 인수의 개수를 클라이언트가 조절하게 해주지만, 구현 방식에 허점이 존재한다.  
가변인수를 담기 위한 배열을 내부로 감추지 않고 클라이언트에 노출하는 문제가 있다.  
그 결과 varargs 매개변수에 제네릭이나 매개변수화 타입이 포함되면 알기 어려운 컴파일 경고가 발생한다.  
제네릭 varargs 배열 매개변수에 값을 저장하는 것은 안전하지 않다.  
만약 메서드 타입 안전성을 보장한다면 @SafeVarargs 어노테이션을 사용하자.  

### 제네릭과 가변인수 동시 사용으로 인한 컴파일 경고
```` java
static void dangerous(List<String>... stringLists) {
  List<Integer> intList = List.of(42);
  Object[] objects = stringLists;
  objects[0] = intList;
  String s = stringLists[0].get(0);
}

warning: [uncheked] Possible heap pollution from
    parameterized vararg type List<String>
````

</br>

### 제네릭 배열과 제네릭 varargs 비교
제네릭 배열은 프로그래머가 직접 생성하는 것은 허용하지 않는다.  
제네릭 varargs 매개변수를 받는 메서드는 선언이 가능하다.  
-> 실무에서 너무 유용하기 때문에 언어 설계자는 이 모순을 수용하기로 했다.  


### 유용한 제네릭 가변인수 예시  
```` java
Arrays.asList(T... a);
Collections.addAll(Collection<? super T> c, T... elements);
EnumSet.of(E first, E... rest);
````

</br>

### 제네릭 varargs 타입 안전성 확인방법  
가변인수 메서드를 호출할 때 varargs 매개변수를 담는 제네릭 배열이 생성된다.  
아래의 조건을 만족한다면 가변인수 메서드가 타입 안전하다고 생각할 수 있다.  
  
1. 메서드가 가변인수 배열에 아무것도 저장하지 않는다.  
2. 가변인수 배열의 참조가 밖으로 노출되지 않는다.  
  
### 제네릭 매개변수 배열 참조 노출  
```` java
static <T> T[] toArray(T... args) {
  return args;
}

static <T> T[] pickTwo(T a, T b, T c) {
  switch(ThreadLocalRandom.current().nextInt(3)) {
    case 0: return toArray(a, b);
    case 1: return toArray(a, c);
    case 2: return toArray(b, c);
  }
  throw new AssertionError();
}

public static void main(String[] args) {
  String[] attributes = pickTwo("a", "b", "c");
}
````

</br>

### 배열 참조 노출이 가능한 경우
1. @SafeVarags 어노테이션이 제대로 된 또 다른 varargs 메서드에 넘기는 경우  
2. 그저 이 배열 내용의 일부 함수를 호출만 하는 일반 메서드에 넘기는 경우  

### 안전하게 사용하는 메서드  
```` java
@SafeVarargs
private static <T> List<T> flatten(List<? extends T>... lists) {
  List<T> result = new ArrayList<>();
  for (List<? extends T> list : lists) {
    result.addAll(list);
  }
  return result;
}
````

</br>

### @SafeVarargs 어노테이션 적용 방법  
제네릭이나 매개변수화 타입의 varargs 매개변수를 받는 모든 메서드에 어노테이션 추가하자.  
이 말은 안전하지 않은 varargs 메서드는 절대 작성하지 말라는 뜻이 된다.  
  
</br>


