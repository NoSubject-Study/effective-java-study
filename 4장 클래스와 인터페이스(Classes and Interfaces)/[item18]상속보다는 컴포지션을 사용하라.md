# Item18: Favor composition over inheritance

*상속보다는 컴포지션을 사용하라*

---

*이번 아이템에서 말하는 상속은 구현 상속(클래스가 다른 클래스를 확장하는)을 뜻함. 인터페이스 상속과는 무관함.*

### 상속이 안전한 경우

- 상위 클래스와 하위 클래스 모두 같은 프로그래머가 통제하는 패키지 안에 있는 경우
- 확장할 목적으로 설계되었고 문서화도 잘 된 클래스(Item 19)

→ 일반적인 구체 클래스를 패키지 경계를 넘어, 즉 다른 패키지의 구체 클래스를 상속하는 일은 위험

### **메서드 호출과 달리 상속은 캡슐화를 깨뜨린다.**

- 상위 클래스에 따라 하위 클래스의 동작에 이상이 생길 수 있음
- 상위 클래스 설계자가 확장을 충분히 고려하지 않고 문서화도 제대로 해두지 않으면 하위 클래스는 상위 클래스의 변화에 발맞춰 수정해야 함

Ex) *HashSet*을 상속받아 구현한 *InstrumentedHashSet* 클래스

```java
// Broken - Inappropriate use of inheritance!
public class InstrumentedHashSet<E> extends HashSet<E> {
    // The number of attempted element insertions
    private int addCount = 0;
    public InstrumentedHashSet() {
    }

    public InstrumentedHashSet(int initCap, float loadFactor) {
        super(initCap, loadFactor);
    }

    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }
}
```

- `HashSet`의 기능에 `addCount` 변수를 추가하여, 원소가 추가될 때마다 카운트를 증가시키는 기능을 추가
- `addCount`: 원소 추가 시도 횟수를 카운트하는 변수
- `add`, `addAll` : `HashSet`의 메서드를 오버라이딩한 메서드

Ex) `addAll` 메서드를 사용하여 3개 원소 추가

```java
InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
s.addAll(List.of("Snap", "Crackle", "Pop"));
```

`getAddCount` 메서드가 3을 리턴할 것으로 예상되지만 실제로는 6을 리턴

→ `HashSet`의 `addAll` 메서드가 내부적으로 `add` 메서드를 사용하여 구현되어 있기 때문

1. `InstrumentedHashSet`의 `addAll` 메서드 호출: `addCount`에 추가할 원소의 수만큼 더함
2. `super.addAll(c)`를 통해 `HashSet`의 `addAll` 메서드 호출: `HashSet`의 `addAll` 메서드는 내부적으로 `add` 메서드를 사용
3. `add` 메소드는 `InstrumentedHashSet`에서 오버라이드 되었으므로, `HashSet`의 `add` 메서드 대신 `InstrumentedHashSet`의 `add` 메서드 호출: `addCount`가 다시 증가

### 해결하기 위해 시도해볼 방법?

1. 하위 클래스에서 `addAll` 메서드를 오버라이드 하지 않기
    - `HashSet`의 `addAll` 메서드가 `add` 메서드를 이용해 구현했음을 가정한 해법이라는 한계를 지님
    - 자기 사용(self-use) 여부는 내부 구현 방식으로, 모든 Java 플랫폼의 구현에서 보장되지 않으며 릴리스마다 변경될 수 있음
    - 이러한 가정에 기대는 `InstrumentedHashSet` 클래스는 깨지기 쉬움
2. `addAll` 메서드를 다른 식으로 오버라이드 하기
    - 주어진 컬렉션을 순회하며 원소 하나당 `add` 메서드를 한 번만 호출
    - 상위 클래스의 메소드를 다시 구현하는 이 방식은 어렵고, 시간이 많이 걸리며, 오류를 내거나 성능을 저하시킬 수 있음
    - 일부 메소드는 하위 클래스에서 접근할 수 없는 `private` 필드에 접근해야만 구현할 수 있음
3. 하위 클래스의 취약성과 새로운 메소드 추가
    - 하위 클래스는 상위 클래스가 후속 릴리즈에서 새로운 메소드를 얻을 수 있어 취약
    - Ex) 모든 요소가 특정 조건을 만족해야 한다는 보안 요구사항이 있는 프로그램
        - 상위 클래스를 상속받아 원소를 추가하는 모든 메서드를 오버라이드하여 원소 추가 전에 해당 조건을 확인하도록 구현하도록 시도해볼 수 있음
        - 상위 클래스에 새로운 요소를 삽입할 수 있는 메서드가 추가되면, 하위 클래스에서 오버라이드하지 않은 새로운 메서드를 호출하여 "허용되지 않은" 원소를 추가하는 것이 가능해짐
        - 실제로 *Hashtable*과 *Vector*가 Collection Framework에 참여하도록 수정되었을 때 이러한 성격의 여러 보안 문제가 수정되어야 했음
4. 메소드 오버라이딩의 문제점과 새로운 메소드 추가
    - 클래스를 확장하더라도 기존 메서드를 오버라이드하지 않고 새로운 메서드를 추가하여 것이 안전할 것이라고 생각할 수 있음 → 훨씬 안전하지만 위험을 완전히 배제할 수 없음
    - Ex) 후속 릴리즈에서 상위 클래스에 새 메서드가 추가됐는데, 하위 클래스에 같은 시그니처를 가지고 다른 리턴 타입의 메서드가 있다면, 하위 클래스는 더 이상 컴파일되지 않음
    - 추가된 새로운 메서드가 같은 리턴 타입을 갖는다면, 하위 클래스는 그 메서드를 오버라이드하게 되어 앞서 설명한 문제에 직면하게 됨
    - 직접 만든 메서드는 상위 클래스가 요구하는 규약을 만족하지 못할 가능성이 큼

### 컴포지션과 전달을 이용한 해결 방법

- 컴포지션(composition): 기존 클래스를 확장하는 대신, 새 클래스에 기존 클래스의 인스턴스를 참조하는 `private` 필드 만들기 → 기존 클래스가 새 클래스의 구성요소가 됨
- 전달(forwarding): 새 클래스의 각 인스턴스 메서드는 기존 클래스의 인스턴스에 대응하는 메서드를 호출하고 그 결과를 리턴 → 새 클래스의 메소드들을 전달 메소드(forwarding method)라고 함

→ 결과적으로 새 클래스는 기존 클래스의 내부 구현에 대한 의존성이 없고, 기존 클래스에 새로운 메서드를 추가해도 새 클래스에는 영향을 미치지 않음

Ex) 컴포지션과 전달 방식을 사용하여 구현한 *InstrumentedHashSet* 클래스:

클래스 자체와 전달 메서드만으로 이루어진 재사용 가능한 전달 클래스 두 개로 구현

```java
// Wrapper class - uses composition in place of inheritance
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;
    public InstrumentedSet(Set<E> s) {
        super(s);
    }
    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }
    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
    public int getAddCount() {
        return addCount;
    }
}

// Reusable forwarding class
public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;
    public ForwardingSet(Set<E> s) { this.s = s; }
    public void clear() { s.clear(); }
    public boolean contains(Object o) { return s.contains(o); }
    public boolean isEmpty() { return s.isEmpty(); }
    public int size() { return s.size(); }
    public Iterator<E> iterator() { return s.iterator(); }
    public boolean add(E e) { return s.add(e); }
    public boolean remove(Object o) { return s.remove(o); }
    public boolean containsAll(Collection<?> c) { return s.containsAll(c); }
    public boolean addAll(Collection<? extends E> c) { return s.addAll(c); }
    public boolean removeAll(Collection<?> c) { return s.removeAll(c); }
    public boolean retainAll(Collection<?> c) { return s.retainAll(c); }
    public Object[] toArray() { return s.toArray(); }
    public <T> T[] toArray(T[] a) { return s.toArray(a); }

    @Override
    public boolean equals(Object o) { return s.equals(o); }
    @Override
    public int hashCode() { return s.hashCode(); }
    @Override
    public String toString() { return s.toString(); }
}
```

*InstrumentedSet*클래스

- 컴포지션
    - `ForwardingSet`을 통해 `Set` 인터페이스의 모든 메서드를 사용할 수 있지만, `Set<E>` 인스턴스를 직접 상속받지 않음
    - `Set<E>` 인스턴스는 `ForwardingSet` 내부에 포함되어 있고(`private final Set<E> s;`), 이를 통해 `Set`의 메서드를 사용
- 전달
    - `InstrumentedSet`은 `add`와 `addAll` 메서드를 오버라이드하여 추가적인 기능(요소가 추가될 때마다 카운트 증가)을 제공
    - 기존의 `Set` 인터페이스의 행동을 변경하지 않으면서 새로운 기능을 추가

상속 방식은 구체 클래스(concrete class) 각각을 따로 확장해야 하고, 상위 클래스의 생성자 각각에 대응하는 생성자를 별도로 정의해야 함. 컴포지션은 어떠한 `Set` 구현체(Ex. *TreeSet*, *HashSet*…)라도 계측할 수 있으며, 기존 생성자들과 함께 사용할 수 있음.

```java
Set<Instant> times = new InstrumentedSet<>(new TreeSet<>(cmp));
Set<E> s = new InstrumentedSet<>(new HashSet<>(INIT_CAPACITY));
```

- `cmp`는 `TreeSet`의 비교자(comparator), `INIT_CAPACITY`는 `HashSet`의 초기 용량

*InstrumentedSet*을 이용하면 대상 `Set` 인스턴스를 특정 조건하에서만 임시로 계측 가능

```java
static void walk(Set<Dog> dogs) {
    InstrumentedSet<Dog> iDogs = new InstrumentedSet<>(dogs);
    *// Within this method use iDogs instead of dogs*
}
```

- `dogs`를 `InstrumentedSet`의 생성자에 전달하여 새로운 `InstrumentedSet` 인스턴스 `iDogs`를 생성
- `iDogs`는 `dogs`와 동일한 `Set`을 참조하지만, 추가적인 계측 기능을 제공

### `InstrumentedSet` 클래스의 특징과 디자인 패턴

- **래퍼 클래스(Wrapper Class)**
    - `InstrumentedSet` 클래스는 각 인스턴스가 다른 `Set` 인스턴스를 '포함(wrap)'하고 있음
    - `InstrumentedSet`이 기본적으로 다른 `Set` 객체를 '감싸고(wrap)' 있는 형태를 가짐
- **데코레이터 패턴(Decorator Pattern)**
    - 기존 클래스를 수정하지 않고 그 기능을 확장하거나 변경하려는 경우에 사용
    - `InstrumentedSet` 클래스는 `Set`에 '계측'이라는 기능을 추가함으로써 `Set`을 '장식(decorate)'함
- **위임(Delegation)**
    - 때때로 컴포지션과 전달의 조합을 넓은 의미로 위임이라고 부름
    - 래퍼 객체가 래핑된 객체에게 자신을 전달하는 경우에만 정확히 위임이라고 함
    - `InstrumentedSet` 클래스는 자신이 감싸고 있는 `Set` 인스턴스에게 자신을 전달하지 않으므로, 이는 정확히 위임이 아님

`InstrumentedSet` 클래스는 래퍼 클래스의 특징을 가지며, 데코레이터 패턴을 따라 `Set`의 기능을 확장 → 상속보다 더 유연한 방법으로 코드의 재사용성을 높이고 기능을 확장할 수 있음

## 래퍼 클래스의 단점

*콜백(callback) 프레임워크와 어울리지 않음*

1. **SELF 문제**: 래핑된 객체가 자신(`this`)에 대한 참조를 외부에 전달할 때, 래퍼는 그 과정을 감지하지 못하고, 외부에서는 래퍼가 아닌 감싸진 객체를 직접 참조하게 되는 현상
    - 객체가 자기 자신의 참조(`this`)를 콜백 함수로 전달할 때 발생. 래퍼에 의해 감싸져 있는 객체는 래퍼를 무시하고 자신의 메서드가 직접 호출되어 래퍼에서 추가된 동작이 적용되지 않게 됨.

```java
interface Printer {
    void print();
}

class RealPrinter implements Printer {	// 래핑될 객체
    void print() {
        System.out.println("The delegate");
    }
}

class PrinterWrapper implements Printer {	// 래퍼 클래스
    private final RealPrinter realPrinter;

    PrinterWrapper(RealPrinter realPrinter) {
        this.realPrinter = realPrinter;
    }

    void print() {
        System.out.println("The wrapper");
        realPrinter.print();
    }
}

public class Main {
    public static void printSomething(Printer printer) {
        printer.print();
    }

    public static void main(String[] args) {
        RealPrinter realPrinter = new RealPrinter();
        PrinterWrapper printerWrapper = new PrinterWrapper(realPrinter);

        printSomething(printerWrapper); // Output: "The wrapper" "The delegate"
        printSomething(realPrinter); // Output: "The delegate"
    }
}
```

- `printSomething(printerWrapper)`를 호출하면, 먼저 래퍼의 `print` 메서드가 호출되고, 그 안에서 래핑된 `RealPrinter` 객체의 `print` 메서드가 호출됨
- `printSomething(realPrinter)`를 호출하면, 래퍼를 거치지 않고 바로 `RealPrinter`의 `print` 메서드가 호출되어 래퍼에서 추가한 동작(`System.out.println("The wrapper")`)이 적용되지 않음
- 래핑된 객체는 래퍼를 알지 못하므로, 자신(this)에 대한 참조를 전달하고 콜백 때 래퍼가 아닌 내부 객체를 호출하게 됨

2. 성능 및 메모리 영향: 메서드 호출의 전달 또는 래퍼 객체의 메모리 사용량에 대한 성능 영향 → 실제로 이러한 요소가 큰 영향을 미치지 않음
3. 전달 메소드 작성의 번거로움: 전달 메서드를 작성하는 것은 지루할 수 있음 → 재사용 가능한 전달 클래스를 각 인터페이스당 하나씩 만들어 두면 원하는 기능을 덧씌우는 전달 클래스들을 손쉽게 구현할 수 있음(Ex. Guava 라이브러리는 모든 컬렉션 인터페이스에 대한 전달 클래스를 제공)

## 상속과 컴포지션의 적절한 사용

- 상속은 하위 클래스가 상위 클래스의 "진짜" 하위 타입인 경우에만 쓰여야 함
- 클래스 *B*가 클래스 *A*를 상속할 때, "모든 B는 A인가?"라는 질문에 '예'라고 답할 수 있어야 함
- 대답이 “아니다”일 경우, *A*를 `private` 인스턴스로 두고 *A*와 다른 API를 제공

Ex) 스택은 벡터가 아니지만 *Stack*은 *Vector*를 확장했고, 속성 목록도 해시테이블이 아니지만 *Properties*도 *HashTable*을 확장한 것은 좋지 않은 예시

- 컴포지션을 써야할 상황에 상속을 사용하는 건 내부 구현을 불필요하게 노출하는 꼴 → API가 내부 구현에 묶이고 그 클래스의 성능도 영원히 제한됨
- Ex) `Properties`의 인스턴스 `p`가 있을 때, `p.getProperty(key)`와 `p.get(key)`는 결과가 다를 수 있음
    - 전자는 기본 동작이고, 후자는 상위 클래스인 `HashTable`로부터 물려받은 메서드이기 때문
    - `Properties`는 키와 값으로 문자열만 허용하도록 설계하려 했으나, 상위 클래스 `HashTable`의 메서드를 직접 호출하면 이 불변식이 깨져버릴 수 있음

- 확장하려는 클래스의 API에 결함이 있는 경우
    - 상속은 상위 클래스의 API를 ‘그 결함까지도’ 그대로 승계
    - 컴포지션을 이용하여 이런 결함을 숨기는 새로운 API를 설계할 수 있음
