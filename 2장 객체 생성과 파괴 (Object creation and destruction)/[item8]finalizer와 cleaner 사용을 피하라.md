# item-07 finalizer와 cleaner 사용을 피하라
(finalizer는 자바9부터 deprecated 되었음 ->대안:cleaner-> 대안:try-with-resources)


이번 장에서는 finalizer와 cleaner 사용시의 단점을 중점적으로 살펴본다.

단점은 아래와 같다.


## 실행 되기 까지 얼마나 걸릴 지 알 수 없다.(즉시 수행된다는 보장이 없다.)
finalizer나 cleaner를 얼마나 신속히 수행할지는 가비지 컬렉터의 알고리즘에 달렸으며 이는 가비지 컬렉터를 어떻게 구현했느냐(JVM 종류)에 따라 천차만별이고 프로그램이 finalizer나 cleaner의 수행 시점에 의존할 경우의 프로그램 동작 또한 그렇다.

또한 자바 언어 명세에서는 finalizer를 어떤 스레드에서 수행할 지 명시되어 있지 않아 이 문제를 예방할 방법이 없다. -> 사용하지 않는 방법 밖에 없다.

이런 문제로 인하여 finalizer 대기열에서 객체가 회수되기만을 기다리다가 OutOfMemoryError가 날 수 있다.

cleaner가 finalizer보다 나은 점은 수행할 스레드를 제어할 수 있다는 점이지만, 가비지 컬렉터의 통제하에 있어서 즉각 수행의 보장은 여전히 없다.


## 실행 여부를 보장하지 않는다.
```text
System.exit을 호출할때의 cleaner의 동작은 구현하기 나름이다
```
(cleaner의 명세)

System.gc, System.runFinalization 메서드도 실행 가능성을 높여줄뿐, finalizer와 cleaner 모두 실행을 보장하지는 않는다. -> 그러므로 반드시 실행해야 하는, 제 때 실행되어야 하는 작업은 finalizer와 cleaner에서는 할 수 없다.

finalizer나 cleaner 실행을  강제하는 함수(System.runFinalizersOnExit와Runtime.runFinalizersOnExit)가 있었으나 심각한 결함으로 이용되지 않는다.

## 예외 발생 시 무시된다.
finalizer 동작 중 발생한 예외는 무시 되고, 그 순간 종료되는데 이 때 스택 추적이 출력되지 않으며 경고조차 출력되지 않는다. -> 이로 인해 객체가 불완전한 상태로 남게 될 수 있으며, 다른 스데르가 이 객체를 사용한다면 예측 불가능한 결과를 가져올 수 있다.
cleaner는 그나마 자신의 스레드를 통제하므로 이런 문제가 발생하지 않는다.

## 심각한 성능 저하가 발생된다.
저자의 경우
AutoCloseable 객체를 생성하고 가비지 컬렉터가 수거하는데 걸리는 시간: 12ns (try-with-resources)
finalizer 사용: 550 ns
로 비교하면 50배 가량이 느렸다. finalizer가 가비지 컬렉터의 효율을 떨어뜨린다는 뜻이다. cleaner와 비교해도 비슷하지만 단, 안전망 방식의 경우에는 66ns였다)


## 심각한 보안 문제

생성자나 직렬화 과정에서 악의적으로 작성해둔 하위 클래스의 finalizer가 수행되는 보안 문제가 있다.


## 앞선 문제들의 해결법은?
AutoCloseable을 구현하고, 클라이언트에서 인스턴스를 모두 사용하면 close 메소드를 호출하는 것.
(일반적으로는 예외 발생시에도 종료되도록 try-with-resources를 사용해야 한다.)


## 그럼에도 불구하고, cleaner와 finalizer 사용 예시

첫번째로는 클라이언트가 회수 하지 않는 것 보다는 늦게라도 회수하는 것이 나으므로, finalizer를 작성해줄 때이다. 이런 안전망 역할로 FileInputStrema, FileOutputStream, ThreadpoolExecutor에서도 finalizer가 작성되어 있다.

두 번째로 네이티브 피어 객체를 해제할 시에 사용된다. 네이티브 피어 객체는 GC에 의해 탐지되지 않으므로, 이럴 경우에는 cleaner나 finalizer를 사용하여서 처리해야 한다. (단, 성능 저하를 감당해도 될 경우, 네이티브 피어가 심각한 자원을 가지고 있지 않을 경우). -> 이 단서에 해당하지 않으면 close 메서드를 사용해야 한다.


```java
public class Adult {
    public static void main(String[] args) {
        try (Room myRoom = new Room(7)) {
            System.out.println("안녕~");
        }
        //  "방 청소"가 자동으로 이루어짐
    }
}
```

```java
public class Teenager {
    public static void main(String[] args) {
        new Room(99);
        System.out.println("아무렴");
        // "방 청소"가 이루어지지 않을 수 있음
    }
}
```
Adult 클래스의 경우는 try-with-resource 구문으로 청소가 진행되지만, Teenager 클래스의 경우 청소가 진행 될 수도, 안 될 수도 있다.
 
 
