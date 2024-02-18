# **Item20: Prefer interfaces to abstract classes**

*추상 클래스보다는 인터페이스를 우선하라*

---

자바의 다중 구현 메커니즘: 인터페이스, 추상 클래스

추상 클래스가 정의한 타입을 구현하는 클래스는 반드시 추상 클래스의 하위 클래스가 되어야 함 → 단일 상속만 가능하므로 새로운 타입을 정의하는데 큰 제약

인터페이스가 선언한 메서드를 모두 정의하고 그 일반 규약을 잘 지킨 클래스라면 다른 어떤 클래스를 상속했든 같은 타입으로 취급

## **기존 클래스에도 손쉽게 새로운 인터페이스를 구현해 넣을 수 있다.**

인터페이스가 요구하는 메서드를 추가하고, 클래스 선언에 `implements` 구문 추가

```java
// 인터페이스를 정의
public interface Flyable {
    void fly();
}

// 기존 클래스
public class Bird {
    public void eat() {
        System.out.println("The bird is eating.");
    }
}

// 기존 클래스에 인터페이스를 구현
public class BirdThatFlies extends Bird implements Flyable {
    @Override
    public void fly() {
        System.out.println("The bird is flying.");
    }
}
```

반면 일반적으로 기존 클래스 위에 새로운 추상 클래스를 끼워넣기 어려움(공통 조상을 가져야 함)

```java
// 기존 클래스
public class Animal {
    public void eat() {
        System.out.println("The animal is eating.");
    }
}

// 추상 클래스
public abstract class AbstractFlyable {
    public abstract void fly();
}

// Animal 클래스를 이미 상속받은 Bird 클래스
public class Bird extends Animal {
    public void tweet() {
        System.out.println("The bird is tweeting.");
    }
}

// Bird 클래스에 AbstractFlyable 추상 클래스를 추가로 상속하는 것은 불가능
public class BirdThatFlies extends Bird, AbstractFlyable { // 컴파일 에러
    @Override
    public void fly() {
        System.out.println("The bird is flying.");
    }
}
```

## **인터페이스는 믹스인 정의에 안성맞춤이다.**

- 믹스인(mixin): 클래스가 구현할 수 있는 타입으로, 원래의 ‘주된 타입’ 외에도 특정 선택적 행위를 제공한다는 선언하는 효과를 제공
- 추상 클래스로 믹스인을 정의할 수 없음

Ex. *Comparable* 인터페이스를 구현한 *Integer* 클래스

```java
public final class Integer extends Number implements Comparable<Integer> {
    // ...
}
```

`Integer` 클래스가 `Comparable` 인터페이스를 구현함으로써, `Integer` 인스턴스들은 서로 비교할 수 있는 비교 가능성을 추가로 가지게 됨

## **인터페이스로는 계층구조가 없는 타입 프레임워크를 만들 수 있다.**

현실적으로 타입을 계층적으로 엄격히 구분하기 어려운 경우가 있음

```java
public interface Singer {
       AudioClip sing(Song s);
}

public interface Songwriter {
       Song compose(int chartPosition);
}

public interface SingerSongwriter extends Singer, Songwriter {
       AudioClip strum();
       void actSensitive();
}
```

- `Singer`와 `Songwriter`  모두를 확장하고 새로운 메서드까지 추가한 제3의 인터페이스를 정의할 수도 있음
- 이 정도의 유연성이 항상 필요하지는 않지만, 같은 구조를 클래스로 만들려면 가능한 조합 전부를 각각의 클래스로 정의한 고도비만 계층구조가 만들어짐 → 조합 폭발(combinatorial explosioin) 현상

## 래퍼 클래스와 함께 사용한 인터페이스

래퍼클래스 관용구(Item 18)와 함께 사용하면 **인터페이스는 기능을 향상시키는 안전하고 강력한 수단이 된다.**

Ex. *java.util.Collections* 클래스

```java
List<E> list = new ArrayList<>();
List<E> synchronizedList = Collections.synchronizedList(list);
List<E> unmodifiableList = Collections.unmodifiableList(list); 
```

- 정적 팩터리 메서드 `synchronizedList`, `unmodifiableList` 는 `List` 인터페이스를 구현하는 래퍼 클래스를 리턴
- `synchronizedList`는 동기화를 추가한 List를, `unmodifiableList`는 변경이 불가능한`List` 를 리턴 → 기존 클래스를 변경하지 않고도 필요한 기능을 추가
- 타입을 추상 클래스로 정의하면 상속을 이용해 타입을 추가해야 하는데, 활용도가 떨어지고 깨지기 쉬움

## 디폴트 메서드

- 인터페이스 메서드 중 구현 방법이 명백한 것이 있다면, 그 구현을 디폴트 메서드로 제공해 프로그래머들의 일감을 덜어줄 수 있다.
- 상속하려는 사람을 위한 설명을 `@implSpec` 자바독 태그를 붙여 문서화 해야함(Item 19)

```java
public interface Animal {
    void eat();

    default void walk() {
        System.out.println("The animal is walking.");
    }
}
```

### 디폴트 메서드의 제약

- *Object* 클래스의 메서드들(`equals`, `hashCode` 등)에 대한 디폴트 메서드를 제공하면 안됨 → *Object* 클래스의 메서드들이 모든 클래스에 공통적으로 제공되는 기본 메서드이기 때문
- 인터페이스는 인스턴스 필드를 가질 수 없고 `public`이 아닌 정적 멤버를 가질 수 없음( `private static` 메서드는 예외) → 인터페이스의 주 목적이 메서드의 명세(specification)를 제공하는 것이며, 구현 세부사항은 구현 클래스에게 맡기기 위함
- 자신이 컨트롤하지 않는 인터페이스에 디폴트 메서드를 추가할 수 없음

## 템플릿 메서드 패턴

인터페이스와 추상 골격 구현(skeletal implementation) 클래스를 함께 제공하는 식으로 인터페이스와 추상 클래스의 장점을 모두 취하는 방법

- 인터페이스로 타입을 정의하고, 필요하면 디폴트 메서드 몇 개도 함께 제공
- 골격 구현 클래스의 나머지 메서드들까지 구현

```java
public abstract class Game {
    abstract void initialize();
    abstract void startPlay();
    abstract void endPlay();

    // 템플릿 메서드
    public final void play(){
        initialize();
        startPlay();
        endPlay();
    }
}

public class Chess extends Game {
    @Override
    void initialize() { /* 구현 */ }

    @Override
    void startPlay() { /* 구현 */ }

    @Override
    void endPlay() { /* 구현 */ }
}
```

- `play()` (템플릿 메서드) 메서드에서 게임을 진행하는 전체적인 흐름을 정의,  `initialize()`, `startPlay()`, `endPlay()`  메서드는 하위 클래스에서 구현

관례상 인터페이스의 이름이 *Interface*라면, 그 골격 구현 클래스의 이름은  *AbstractInterface*로 지음(Ex. *AbstractCollection*, *AbstractSet*, *AbstractList*, *AbstractMap*…)

Ex. *AbstractList* 골격 구현을 활용해 *List* 구현체를 반환하는 정적 팩터리 메서드

```java
// Concrete implementation built atop skeletal implementation
static List<Integer> intArrayAsList(int[] a) {
    Objects.requireNonNull(a);
    // The diamond operator is only legal here in Java 9 and later
    // If you're using an earlier release, specify <Integer>
    return new AbstractList<>() {
        @Override
        public Integer get(int i) {
            return a[i];  // Autoboxing (Item 6)
        }

        @Override
        public Integer set(int i, Integer val) {
            int oldVal = a[i];
            a[i] = val;     // Auto-unboxing
            return oldVal;  // Autoboxing
        }

        @Override
        public int size() {
            return a.length;
        }
    };
}
```

- `AbstractList`를 확장하는 클래스는 `get`, `set`, `size` 등의 메서드만 구현하면 되며, 나머지 `List` 인터페이스의 메서드들은 이미 `AbstractList`에서 구현되어 있음
- `intArrayAsList` 메서드는 정수 배열 `int[]`에 대한 `List<Integer>` 뷰를 제공 → `int` 배열을 받아 `Integer` 인스턴스의 리스트 형태로 보여주는 어댑터(Adapter) 역할을 함

## 골격 구현 클래스의 장점

- 추상 클래스처럼 구현을 도와주는 동시에, 타입을 정의할 때 추상 클래스가 갖는 제약에서 자유로움
- 구조상 골격 구현을 확장할 수 없으면 인터페이스를 직접 구현해야 함 → 이 경우에도 디폴트 메서드의 이점을 누릴 수 있음
- 골격 구현 클래스를 우회적으로 이용할 수도 있음
인터페이스를 구현한 클래스에서 해당 골격 구현을 확장한 `private` 내부 클래스를 정의하고, 각 메서드 호출을 내부 클래스의 인스턴스에 전달 → 시뮬레이트한 다중 상속(Simulated Multiple Inheritance)

Ex. 골격 구현 클래스와 시뮬레이트한 다중 상속을 이용한  Car 클래스 구현

```java
// 인터페이스 정의
interface Drivable {
    void drive();
}

interface AudioSystem {
    void playMusic();
}

// 골격 구현 정의
abstract class AbstractDrivable implements Drivable {
    public void drive() {
        // 기본적인 운전 기능 구현
    }
}

abstract class AbstractAudioSystem implements AudioSystem {
    public void playMusic() {
        // 기본적인 음악 재생 기능 구현
    }
}

// Car 클래스에서 시뮬레이트한 다중 상속 사용
public class Car implements Drivable, AudioSystem {
    private final Drivable drivable = new DrivableImpl();
    private final AudioSystem audioSystem = new AudioSystemImpl();

    @Override
    public void drive() {
        drivable.drive();
    }

    @Override
    public void playMusic() {
        audioSystem.playMusic();
    }

    // 내부 클래스를 통한 골격 구현 활용
    private class DrivableImpl extends AbstractDrivable { /* 필요한 경우 추가 구현 */ }

    private class AudioSystemImpl extends AbstractAudioSystem { /* 필요한 경우 추가 구현 */ }
}
```

## 골격 구현 클래스 작성 방법

1. 인터페이스를 잘 살펴 다른 메서드들을 구현하는데 기본이 되는 메서드를 선정(이 기본 메서드들이 골격 구현에서의 추상 메서드가 됨)
2. 기반 메서드들을 사용해 직접 구현할 수 있는 메서드를 모두 디폴트 메서드로 제공(단,  `equals`, `hashCode`와 같은 `Object` 메서드는 X)
3. 기반 메서드나 디폴트 메서드로 만들지 못한 메서드가 남아있다면, 이 인터페이스를 구현하는 골격 구현 클래스를 하나 만들어 나머지 메서드들을 작성
- 필요하면 `public`이 아닌 필드와 메서드를 추가해도 됨

### 기반 메서드(base methods)와 디폴트 메서드(default methods)의 차이

- 기반 메서드(base methods): 인터페이스에 선언된 메서드 중, 다른 메서드의 구현에 필요한 핵심적인 메서드로, 골격 구현 클래스에서는 이 메서드들을 추상 메서드로 선언하여 구현을 강제함 → 서브클래스가 이 메서드들을 반드시 구현해야 함
- 디폴트 메서드(default methods): 인터페이스에서 제공하는 메서드 중, 기본 구현을 가지는 메서드 → 인터페이스를 구현하는 클래스에서 필요에 따라 재정의할 수 있지만, 재정의를 하지 않아도 기본 구현을 사용할 수 있음

```java
interface Drawable {
    void draw(); // 기반 메서드

    default void rotate(int degree) { // 디폴트 메서드
        System.out.println("Rotate " + degree + " degrees.");
    }

    default void scale(double factor) { // 디폴트 메서드
        System.out.println("Scale by a factor of " + factor + ".");
    }
}
```

Ex. 골격 구현 클래스

```java
// Skeletal implementation class
public abstract class AbstractMapEntry<K, V> implements Map.Entry<K, V> {
    // Entries in a modifiable map must override this method
    @Override
    public V setValue(V value) {
        throw new UnsupportedOperationException();
    }

    // Implements the general contract of Map.Entry.equals
    @Override
    public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof Map.Entry))
            return false;
        Map.Entry<?, ?> e = (Map.Entry) o;
        return Objects.equals(e.getKey(), getKey()) && Objects.equals(e.getValue(), getValue());
    }

    // Implements the general contract of Map.Entry.hashCode
    @Override
    public int hashCode() {
        return Objects.hashCode(getKey()) ^ Objects.hashCode(getValue());
    }

    @Override
    public String toString() {
        return getKey() + "=" + getValue();
    }
}
```

`Object` 메서드들은 디폴트 메서드로 제공하면 안 되므로, 골격 구현 클래스에서 구현

> `Map.Entry` 인터페이스나 그 하위 인터페이스로는 이 골격 구현을 제공할 수 없다. 디폴트 메서드는 `equals`, `hashCode`, `toString` 같은 `Object` 메서드를 재정의할 수 없기 때문이다.
> 

- 골격 구현은 기본적으로 상속해서 사용하는 걸 가정하므로 Item 19에서 이야기한 설계 및 문서화 지침을 모두 따라야 함 → 동작 방식을 잘 정리해 문서로 남겨야 함
- 단순 구현(simple implementation)은 골격 구현의 작은 변종(Ex. *AbstractMap.SimpleEntry)*
- 단순 구현도 상속을 위해 인터페이스를 구현한 것이지만, 추상 클래스가 아님
