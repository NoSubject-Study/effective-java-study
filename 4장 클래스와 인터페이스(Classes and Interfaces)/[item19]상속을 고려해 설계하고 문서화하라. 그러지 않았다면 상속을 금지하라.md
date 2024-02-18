# **Item19: Design and document for inheritance or else prohibit it**

*상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라*

---

## 상속을 고려한 설계와 문서화란?

메서드를 재정의(overriding)하면 어떤 일이 일어나는지 정확히 정리하여 문서화 해야함

→ **상속용 클래스는 재정의할 수 있는 메서드들을 내부적으로 어떻게 이용하는지(자기사용) 문서로 남겨야 한다.**

- 클래스의 API로 공개된 메서드에서 클래스 자신의 또 다른 재정의 가능한 메서드를 호출할 수 있는 경우: 호출되는 메서드가 재정의 가능하다는 사실을 호출하는 메서드 API에 적시해야함
- 어떤 순서로 호출하는지, 각각의 호출 결과가 이어지는 처리에 어떤 영향을 주는지도 담아야 함(’재정의 가능’ 이란 `public`과 `protected` 메서드 중 `final`이 아닌 모든 메서드를 뜻함)

→ 재정의 가능 메서드를 호출할 수 있는 모든 사항을 문서로 남겨야 한다. (Ex. 백그라운드 스레드나 정적 초기화 과정에도 호출이 일어날 수 있음)

Ex) 같은 클래스에 있는 메서드 `m1`이 재정의 가능한 메서드 `m2`를 호출할 때, `m1`의 API 문서에는 `m2`가 재정의 가능하므로, 이 클래스를 상속하는 클래스에서 `m2`를 재정의하면 `m1`의 동작에 영향이 있을 수 있음을 포함해야 함

Implementation Requirements: 메서드의 내부 동작 방식을 설명 → 메서드 주석에 `@implSpec` 태그를 붙여주면 자바독 도구가 생성해줌

> Ex) `java.util.AbstractCollection`의 `remove` 메서드
> 
> 
> ```java
> public boolean remove(Object o)
> ```
> 
> Removes a single instance of the specified element from this collection, if it is present (optional operation). More formally, removes an element `e` such that `Objects.equals(o, e)`, if this collection contains one or more such elements. Returns true if this collection contained the specified element (or equivalently, if this collection changed as a result of the call).
> 
> **Implementation Requirements:** This implementation iterates over the collection looking for the specified element. If it finds the element, it `removes` the element from the collection using the iterator’s remove method. Note that this implementation throws an `UnsupportedOperationException` if the iterator returned by this collection’s `iterator` method does not implement the remove method and this collection contains the specified object.
> 
- `iterator` 메서드를 재정의하면 `remove` 메서드 동작에 영향을 줌을 확실히 알 수 있음
- `iterator` 메서드로 얻은 반복자의 동작이 `remove` 메서드에 동작에 주는 영향도 정확히 설명함

→ “좋은 API 문서는 ‘어떻게’가 아닌 ‘무엇’을 하는지 설명해야 한다”라는 격언과 대치됨

클래스를 안전하게 상속하기 위해 (상속을 하지 않았더라면 기술하지 않았어야 할) 내부 구현을 설명해야 함

효율적인 하위 클래스를 큰 어려움 없이 만들 수 있게 하려면 **클래스의 내부 동작 과정 중간에 끼어들 수 있는 훅(hook)을 잘 선별하여 `protected` 메서드 형태로 공개해야할 수 있다.** 드물게는 `protected` 필드로 공개해야 할 수도 있다.

- `protected` 메서드 → 하위 클래스에서 훅 메서드를 재정의(override)하여 원하는 동작을 수행
- `protected` 필드 →  하위 클래스가 해당 필드를 직접 사용하여 원하는 동작을 수행(클래스의 캡슐화를 해칠 수 있음)

> Ex) `java.util.AbstractList`의 `removeRange` 메서드
> 
> 
> ```java
> protected void removeRange(int fromIndex, int toIndex)
> ```
> 
> Removes from this list all of the elements whose index is between `fromIndex`, inclusive, and `toIndex`, exclusive. Shifts any succeeding elements to the left (reduces their `index`). This call shortens the list by (`toIndex` - `fromIndex`) elements. (`If toIndex == fromIndex`, this operation has no effect.)
> 
> This method is called by the `clear` operation on this list and its sublists. Overriding this method to take advantage of the internals of the list implementation can substantially improve the performance of the `clear` operation on this list and its sublists.
> 
> **Implementation Requirements:** This implementation gets a list iterator positioned before `fromIndex` and repeatedly calls `ListIterator.next` followed by `ListIterator.remove`, until the entire range has been removed. **Note: If `ListIterator.remove` requires linear time, this implementation requires quadratic time.**
> 
> Parameters:
>     `fromIndex`     index of first element to be removed.
>     `toIndex`         index after last element to be removed.
> 

`List` 구현체의 최종 사용자는 `removeRange` 메서드에 관심이 없지만, 하위 클래스에서 부분리스트의 `clear` 메서드를 고성능으로 만들기 쉽게 하기 위해서 이 메서드를 제공

`removeRange` 메서드가 없다면 하위 클래스에서 `clear` 메서드를 호출하면 (제거할 원소 수의) 제곱에 비례해 성능이 느려지거나 부분리스트의 매커니즘을 밑바닥부터 새로 구현했어야 함

### 어떤 메서드를 `protected`로 노출해야 할지 결정하는 방법

- 심사숙고해서 잘 예측해본 다음, 실제 하위 클래스를 만들어 시험해보는 것이 최선
- `protected` 멤버 하나하나가 내부 구현에 해당하므로 가능한 적게 노출시켜야 함
- 너무 적게 노출해서 상속으로 얻는 이점마저 없애지 않도록 주의해야 함

### 상속용 클래스를 시험하는 방법

- **직접 하위 클래스를 만들어보는 것이 ‘유일’**
- 꼭 필요한 `protected` 멤버를 놓치면 하위 클래스를 작성할 때 그 빈자리가 확연히 드러남
- 하위 클래스를 여러 개 만들 때까지 전혀 쓰이지 않는 `protected` 멤버는 `private`이었어야할 가능성이 큼
- 하위 클래스 3개 정도를 만들어 보는게 적당(이 중 하나 이상은 제3자가 작성해봐야 함)

널리 쓰일 클래스를 상속용으로 설계한다면 문서화환 내부 사용 패턴, `protected` 메서드와 필드를 구현하면서 선택한 결정에 영원이 책임져야 함 → 클래스의 성능과 기능에 영원한 족쇄가 되어 후속 릴리스에서 개선을 어렵게 하거나 불가능하게 할 수 있음

**상속용으로 설계한 클래스는 배포 전에 반드시 하위 클래스를 만들어 검증해야 함**

### 상속을 허용하는 클래스가 지켜야 할 제약

**상속용 클래스의 생성자는 직접적/간접적으로 재정의 가능 메서드를 호출해서는 안 됨**

- 상위 클래스의 생성자가 하위 클래스의 생성자보다 먼저 실행되므로 하위 클래스에서 재정의한 메서드가 하위 클래스의 생성자보다 먼저 호출됨
- 재정의한 메서드가 하위 클래스의 생성자에서 초기화하는 값에 의존한다면 의도대로 동작하지 않음

```java
public class Super {
    // Broken - constructor invokes an overridable method
    public Super() {
        overrideMe();
    }

    public void overrideMe() {
    }
}
```

```java
public final class Sub extends Super {
		// Blank final, set by constructor
    private final Instant instant;
    Sub() {
        instant = Instant.now();
    }

		// Overriding method invoked by superclass constructor
    @Override public void overrideMe() {
        System.out.println(instant);
    }

    public static void main(String[] args) {
        Sub sub = new Sub();
        sub.overrideMe();
    }
}
```

`Super` 클래스의 생성자가 `overrideMe` 메서드를 호출하고 있어 `Sub` 클래스의 생성자가 `instant`를 초기화 하기 전에 `null`을 출력하고, 모든 생성자 호출이 완료되고 나서 `sub.overrideMe()`가 정상적인 `instant`를 출력함

> `private`, `final`, `static` 메서드는 재정의가 불가능하니 생성자에서 안심하고 호출해도 된다.
> 

Ex) 상위 클래스의 생성자가 하위 클래스에서 재정의된 메서드를 호출하는 경우

```java
public class SuperClass {
    public SuperClass() {
        print();
    }

    void print() {
        System.out.println("SuperClass");
    }
}

public class SubClass extends SuperClass {
    private int value;

    public SubClass(int value) {
        this.value = value;
    }

    @Override
    void print() {
        System.out.println("SubClass value: " + value);
    }
}

public class Main {
    public static void main(String[] args) {
        SubClass sub = new SubClass(5);
    }
}
```

출력 결과: SubClass value: 0

`Cloneable`과 `Serializable` 인터페이스를 구현한 클래스를 상속할 수 있게 설계하는 것은 좋지 않음

→ 그 클래스를 확장하려는 프로그래머에게 엄청난 부담을 지우기 때문 (Item 13, 86에서 구현하는 방법 설명)

상속용 클래스에서 `Clonable`이나 `Serializable` 을 구현할지 정해야 한다면 `clone`과 `readObject` 메서드가 생성자와 비슷한 효과를 낸다는 것을 주의(새로운 객체 생성)

→ **`clone`과 `readObject` 모두 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안 됨**

- `readObject`: 하위 클래스의 상태가 미처 다 역직렬화되기 전에 재정의한 메서드부터 호출하게 됨
- `clone`: 하위 클래스의 `clone` 메서드가 복제본의 상태를 (올바른 상태로) 수정하기 전에 재정의한 메서드를 호출 → 특히 복제본뿐 아니라 원본 객체에도 피해를 줄 수 있음

`Serializable`을 구현한 상속용 클래스가 `readResolve`나 `writeReplace` 메서드를 갖는다면 이 메서드들은 `private`이 아닌 `protected`로 선언해야 함

→ `privated`으로 선언하면 하위 클래스에서 무시되기 때문

```java
public class Singleton implements Serializable {
    public static final Singleton INSTANCE = new Singleton();
    private Singleton() {
         ...
    }
    
    // 항상 같은 인스턴스를 반환하여 역직렬화 과정에서 새로운 인스턴스가 생성되는 것을 방지
    protected Object readResolve() {
        return INSTANCE;
    }
}

public class ExtendedSingleton extends Singleton {
     ...
    @Override
    protected Object readResolve() {
        ...
        return INSTANCE;
    }
}
```

**클래스를 상속용으로 설계하려면 엄청난 노력이 들고 그 클래스에 안기는 제약도 상당함**

- 추상 클래스나 인터페이스의 골격 구현(Item 20) → 상속 허용 O
- 불변 클래스(Item 17) → 상속 허용 X

### 일반적인 구체 클래스?

- 전통적으로 `final`도 아니고 상속용으로 설계되거나 문서화되지도 않음
- 구체 클래스의 내부만 수정해도 이를 확장한 클래스에서 문제가 발생하는 일이 적지 않음

→ **상속용으로 설계하지 않은 클래스는 상속을 금지**

### 상속을 금지시키는 방법

1. 클래스를 `final`로 선언
2. 모든 생성자를 `private`이나 `package-private`으로 선언하고 `public` 정적 팩터리를 만들어 주기
- 핵심 기능을 정의한 인터페이스가 있고, 클래스가 그 인터페이스를 구현했다면 상속을 금지해도 개발하는데 어려움이 없음(Ex. *Set*, *List*, *Map*)
- 래퍼 클래스 패턴도 상속 대신 사용하기 좋은 대안

### 상속을 꼭 허용해야 하는 경우에는?

- 구체 클래스가 표준 인터페이스를 구현하지 않았는데 상속을 금지하면 사용하기에 상당히 불편해짐
- 클래스 내부에서는 재정의 가능 메서드를 사용하지 않게 만들고 이 사실을 문서화
- 재정의 가능 메서드를 호출하는 자기 사용 코드를 완벽히 제거

### 클래스 동작을 유지하면서 재정의 기능 메서드를 사용하는 코드를 제거하는 방법

- 각각의 재정의 가능 메서드는 자신의 본문 코드를 `private` ’도우미 메서드(helper method)’로 옮기고, 이 도우미 메서드를 호출하도록 수정

```java
public class InheritableClass {
    public void doSomething() {
        helperMethod();
        // ...
    }
    
    protected void overrideableMethod() {
        // 재정의 가능한 메서드
    }
    
    private void helperMethod() {
        // 재정의 가능 메서드의 본문 코드를 도우미 메서드로 옮김
    }
}
```

- 재정의 가능 메서드를 호출하는 다른 코드들도 모두 이 도우미 메서드를 호출하도록 수정
