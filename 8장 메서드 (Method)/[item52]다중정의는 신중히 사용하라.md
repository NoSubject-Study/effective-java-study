## 다중정의로 인한 문제상황

``` java
public class CollectionClassifier {
  public static String classify(Set<?> s) {
    return "집합";
  }

  public static String classify(List<?> lst) {
    return "리스트";
  }

  public static String classify(Collection<?> c) {
    return "그 외";
  }

  public static void main(String[] args) {
    Collection<?>[] collections = {
      new HashSet<String>(),
      new ArrayList<BigInteger>(),
      new HashMap<String, String>().values()
    };

    for (Collection<?> c : collections) {
      System.out.println(classify(c));
    }
  }
}
```

결과로 "집합", "리스트", "그 외"를 차례로 출력할 것을 예상하지만, "그 외"만 세 번 출력한다.

다중정의는 컴파일타임에 정해지며, for 문 안의 c는 항상 Collection<?> 타입이다.

그러므로 `classify(Collection<?> c)` 메서드가 3번 호출된다.

다중정의가 혼동을 일으키는 상황을 피해야 한다.

## 다중정의가 혼동을 일으키는 상황을 피해야 한다.

안전하고 보수적으로 가려면 매개변수 수가 같은 다중정의를 만들면 안된다.

가변인수를 사용하는 메서드라면 다중정의를 아예 하지 말아야 한다.

다중정의하는 대신 메서드 이름을 다르게 지어주자.

`ObjectOutputStream` 클래스의 write 메서드는 모든 기본 타입과 일부 참조 타입용 변형을 가지고 있다.

하지만 다중정의가 아닌, 모든 메서드에 다른 이름을 지어주는 길을 택했다.

writeBoolean(boolean), writeInt(int), writeLong(long)

## 매개변수 수가 같은 다중정의 메서드

생성자는 이름을 다르게 지을 수 없으니 두 번째 생성자부터는 무조건 다중정의가 된다.

정적 팩터리라는 대안을 활용할 수 있는 경우가 많다.

그래도 여러 생성자가 같은 수의 매개변수를 받아야 하는 경우를 완전히 피해갈 수 없다.

매개변수 수가 같은 다중정의 메서드가 많더라도, 매개변수가 명확히 구분된다면 괜찮다.

즉, 매개변수를 어느 쪽으로든 형변환할 수 없다면 된다.

이 조건만 충족되면 어느 다중정의 메서드를 호출할지가 매개변수들의 런타임 타입만으로 결정된다.

예를 들어, ArrayList에는 int를 받는 생성자와 Collection을 받는 생성자가 있는데, 어떤 상황에서든 두 생성자 중 어느 것이 호출될지 헷갈릴 일은 없을 것이다.

자바 4까지는 모든 기본 타입이 모든 참조 타입과 근본적으로 달랐지만, 오토박싱이 도입되면서 상황이 바뀌었다.

``` java
public class SetList {
  public static void main(String[] args) {
    Set<Integer> set = new TreeSet<>();
    List<Integer> list = new ArrayList<>();

    for (int i = -3; i < 3; i++) {
      set.add(i);
      list.add(i);
    }

    for (int i = 0; i < 3; i++) {
      set.remove(i);
      list.remove(i);
    }

    System.out.println(set + " " + list);
  }
}
```

예상하는 출력 값은
> [-3, -2, -1] [-3, -2, -1]

이지만 실제로 출력되는 값은 아래와 같다.
> [-3, -2, -1] [-2, 0, 2]

set의 경우에는 예상하는 대로 출력이 되지만 list의 경우에는 다르다.

list.remove(i)는 다중정의된 remove(Object)와 remove(int index) 중 remove(int index)를 선택하여 지정된 위치의 원소를 제거하는 기능을 수행한다.

이 문제는 아래와 같이 list.remove의 인수를 Integer로 형변환하여 사용하면 해결된다.

``` java
for (int i = 0; i < 3; i++) {
  set.remove(i);
  list.remove((Integer) i); // 혹은 remove(Integer.valueOf(i))
}
```

자바 8에서 도입한 람다와 메서드 참조 역시 마찬가지다.

``` java
// 1번, Thread의 생성자 호출
new Thread(System.out::println).start();

// 2번, ExecutorService의 submit 메서드 호출
ExecutorService exec = Executors.newCachedThreadPool();
exec.submit(System.out::println);
```

1번과 2번의 모습이 비슷하지만, 2번의 경우 컴파일 오류가 발생한다.

원인은 submit 다중정의 메서드 중에 Callable\<T>를 받는 메서드도 있기 때문이다.

모든 println 메서드가 void를 반환하니 문제가 없을 것이라 생각할 수 있지만, 이렇게 동작하지 않는다.

println 메서드가 다중정의 없이 단 하나만 존재했다면 제대로 컴파일되겠지만, println과 submit 두 메서드 모두 다중정의가 되어 있기 때문에 다중정의 해소 알고리즘이 제대로 동작하지 않는 것이다.

다중정의된 메서드들이 함수형 인터페이스를 인수로 받을 때, 비록 서로 다른 함수형 인터페이스라도 인수 위치가 같으면 혼란이 생긴다.

따라서 메서드를 다중정의할 때, 서로 다른 함수형 인터페이스라도 같은 위치의 인수로 받아서는 안 된다.

---

String은 자바 4 시절부터 contentEquals(StringBuffer) 메서드를 가지고 있었다.

그런데 자바 5에서 StringBuffer, StringBuilder, CharBuffer 등의 비슷한 부류의 타입을 위한 공통 인터페이스로 CharSequence가 등장하였고, String에도 CharSequence를 받은 contentEquals가 다중정의되었다.

그 결과 이번 아이템의 내용을 어기는 모습이 되었다.

하지만 두 메서드에 같은 객체를 입력해도 완전히 같은 작업을 수행해 주므로 문제가 되지 않는다.

이처럼 어떤 다중정의 메서드가 호출되는지 몰라도 기능이 똑같다면 신경을 쓰지 않아도 된다.

상대적으로 더 특수한 다중정의 메서드에서 덜 특수한 다중정의 메서드로 일을 넘기는 것이다.

``` java
public boolean contentEquals(StringBuffer sb) {
  return contentEquals((CharSequence) sb);
}
```
