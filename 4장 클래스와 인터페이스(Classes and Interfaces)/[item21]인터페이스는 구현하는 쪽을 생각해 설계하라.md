# **Item21: Design interfaces for posterity**

*인터페이스는 구현하는 쪽을 생각해 설계하라*

---

Java 8 전에는 메서드를 추가하는 것이 기존 구현을 깨뜨리는 결과를 야기했음(컴파일 에러)

→ 기존 인터페이스에 메서드를 추가할 수 있도록 디폴트 메서드가 도입되었지만, 모든 구현체들과 매끄럽게 연동되리라는 보장은 없음

- 인터페이스에 디폴트 메서드가 선언되어 있고, 그 인터페이스를 구현한 클래스가 디폴트 메서드를 재정의하지 않았다면, 디폴트 구현이 사용됨
- 자바 라이브러리의 디폴트 메서드는 코드 품질이 높고 범용적이라 대부분 상황에서 잘 작동하지만, **생각할 수 있는 모든 상황에서 불변식을 해치지 않는 디폴트 메서드를 작성하는 것은 어려움**

## Java 8의 *Collection* 인터페이스에 추가된 `removeIf` 메서드

```java
// Default method added to the Collection interface in Java 8
default boolean removeIf(Predicate<? super E> filter) {
    Objects.requireNonNull(filter);
    boolean result = false;
    for (Iterator<E> it = iterator(); it.hasNext(); ) {
        if (filter.test(it.next())) {
            it.remove();
            result = true;
        }
    }
    return result;
}
```

- `removeIf` 메서드는 디폴트 메서드로 구현
- 불리언 함수(predicate)가 `true`를 반환하는 모든 원소를 제거함
- 반복자를 이용해 순회하면서 각 원소를 인수로 넣어 프레디키트를 호출하고, 프레디키트가 `true`를 반환하면 반복자의 `remove` 메서드를 호출해 그 원소를 제거

→ 이 코드보다 더 범용적으로 구현하기도 어렵겠지만, 현존하는 모든 *Collection* 구현체와 잘 어우러지지 않음

### *Apache Commons* 라이브러리의 `SynchronizedCollection` 클래스

- 동기화된 컬렉션을 구현한 것으로, 모든 메서드 호출 시 자동으로 동기화를 진행
- 모든 메서드에서 주어진 락(lock) 객체로 동기화한 후 내부 컬렉션 객체에 기능을 위임하는 래퍼클래스(Item 18)

```java
// 기존 SynchronizedCollection 클래스의 일부
class SynchronizedCollection<E> implements Collection<E> {
    private final Object lock;
    private final Collection<E> c;

    SynchronizedCollection(Collection<E> collection, Object lock) {
        this.c = collection;
        this.lock = lock;
    }

    // 기타 메서드들은 lock으로 동기화
    @Override
    public boolean add(E e) {
        synchronized (lock) {
            return c.add(e);
        }
    }

    // removeIf 메서드는 재정의하지 않았기 때문에, 디폴트 구현을 사용
}
```

- `removeIf` 메서드를 재정의하고 있지 않아 Java 8과 함께 사용하면 `removeIf` 메서드의 디폴트 구현을 물려받게 되어, 자신의 약속을 지키지 못하게 됨 → 모든 메서드 호출을 알아서 동기화해주지 못함
- 디폴트 구현에서는 동기화를 수행하지 않으므로, 여러 스레드가 동시에 이 메서드를 호출하면 문제가 발생할 수 있음

*→ SynchronizedCollection* 인스턴스를 여러 스레드가 공유하는 환경에서 한 스레드가 `removeIf`를 호출하면 `ConcurrentModificationException`이 발생하거나 다른 예기치 못한 결과를 일으킴

자바 플랫폼 라이브러리에서 이런 문제를 예방하기 위해 구현한 인터페이스의 디폴트 메서드를 재정의하고, 다른 메서드에서는 디폴트 메서드를 호출하기 전에 필요한 작업을 수행하도록 함

```java
@Override
public boolean removeIf(Predicate<? super E> filter) {
    synchronized (lock) {
        return c.removeIf(filter);
    }
}
```

하지만 자바 플랫폼에 속하지 않은 제3의 기존 컬렉션 구현체들은 이런 언어 차원의 인터페이스 변화에 발맞춰 수정될 기회가 없었고, 일부는 여전히 수정되지 않고 있다.

## 디폴트 메서드 사용시 주의할 점

- 디폴트 메서드는 (컴파일에 성공하더라도) 기존 구현체에 런타임 오류를 일으킬 수 있음 → Java 8은 컬렉션 인터페이스에 꽤 많은 디폴트 메서드를 추가했고, 그 결과 기존에 짜여진 많은 자바 코드가 영향을 받았음
- 기존 인터페이스에 디폴트 메서드로 새 메서드를 추가하는 일은 꼭 필요한 경우가 아니라면 피해야 함
- 추가하려는 디폴트 메서드가 기존 구현체들과 충돌하지는 않을지 신중히 고려

## 디폴트 메서드를 사용한 인터페이스 설계

- 새로운 인터페이스를 만드는 경우, 디폴트 메서드는 표준적인 메서드 구현을 제공하는 데 아주 유용한 수단 → 인터페이스를 더 쉽게 구현해 활용할 수 있게끔 해줌(Item 20)
- 디폴트 메서드라는 도구가 생겼더라도 **인터페이스를 설계할 때는 여전히 세심한 주의를 기울여야 한다. →** 잘못된 인터페이스라면 이를 포함한 API에 어떤 재앙을 몰고 올지 알 수 없음
- 디폴트 메서드는 인터페이스로부터 메서드를 제거하거나 기존 메서드의 시그니처를 수정하는 용도가 아님을 명심해야 함 → 이런 형태로 인터페이스를 변경하면 기존 클라이언트를 망가뜨리게 된다.
- 새로운 인터페이스라면 릴리스 전에 반드시 테스트를 거쳐야 함 → 서로 다른 방식으로 최소한 세 가지는 구현해봐야 한다.
- **인터페이스를 릴리스한 후라도 결함을 수정하는 게 가능한 경우도 있겠지만, 절대 그 가능성에 기대서는 안된다.**
