문자열은 다른 값 타입을 대신하기에 적합하지 않다.

기본 타입이든 참조 타입이든 적절한 타입이 있다면 그것을 사용하고 없다면 새로 작성해서 사용하는 것이 좋다.

## 문자열은 열거 타입을 대신할 수 없다.

만일 문자열 상수를 사용하게 되면 경험이 부족한 프로그래머가 문자열 상수의 이름 대신 문자열 값을 그대로 하드코딩하게 만든다.

## 문자열은 혼합 타입을 대신하기에 적합하지 않다.

``` java
String compoundKey = className + "#" + i.next();
```

만약 문자열 "#"이 두 요소(className, i.next())에서 사용되었다면 오류가 발생할 수 있다.

그리고 각 요소를 개별적으로 접근하려면 문자열을 파싱해야 하므로 느리고, 귀찮고, 오류 가능성이 커진다.

</br>

String 타입이므로 equals, toString, compareTo 메서드를 사용할 수 없고 String 클래스가 지원하는 메서드에만 의존해야 한다.

그래서 전용 클래스를 새로 만드는 것이 좋다.

이런 경우에는 private 정적 멤버 클래스로 만드는 것이 좋다. (item24 참조)

## 문자열은 권한을 표현하기에 적합하지 않다.

JDK 1.2 에 ThreadLocal 클래스가 나오기 전에 프로그래머들은 개별적으로 구현하여 사용하였다. (ThreadLocal은 스레드별로 멤버 변수를 갖게하는 클래스)

개발자들이 방법을 모색하다가 결국 같은 구현에 이르렀다.

``` java
public class ThreadLocal {

  private ThreadLocal() {}

  public static void set(String key, Object value);
  public static Object get(String key);

}
```

위와 같은 구현으로 사용하였다.

만약 클라이언트가 같은 문자열을 key로 주게 되면 여러 스레드가 같은 필드를 공유하게 되므로 제대로 작동하지 못하게 된다.

그리고 악의적인 클라이언트가 의도적으로 같은 key를 주면 다른 클라이언트의 필드에 접근할 수 있으므로 보안에 취약하다.

</br>

이를 대신해 String 대신 위조할 수 없는 키를 만들어 사용하면 해결된다.

이 키를 권한(capacity)라고 한다.

``` java
public class ThreadLocal {

  private ThreadLocal() {}

  public static class Key {

  public Key() {}

  }

  public static void set(Key key, Object value);
  public static Object get(Key key);

}
```

위와 같이 구현하게 되면 문자열을 사용하였을 경우 생기는 문제가 해결된다.

</br>

조금 더 개선할 수 있다.

set, get 메서드가 정적 메서드일 필요가 없으므로 인스턴스 메서드로 변경하자.

이렇게 되면 ThreadLocal은 더 이상 불필요하게 되므로 지우고 Key클래스를 ThreadLocal로 변경하자.

``` java
public final class ThreadLocal {

  public ThreadLocal() {}

  public void set(Object value);
  public Object get();

}
```

Object를 반환하여 형변환을 해야 하므로 타입 안전하지 않다.

``` java
public final class ThreadLocal<T> {
  public ThreadLocal() {}

  public void set(T value);
  public T get();
}
```

이제 java.lang.ThreadLocal과 흡사하다.
