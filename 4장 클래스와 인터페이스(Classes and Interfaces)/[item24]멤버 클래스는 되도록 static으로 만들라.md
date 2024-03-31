# **Item24: Favor static member classes over nonstatic**

*멤버 클래스는 되도록 static으로 만들라*

---

중첩 클래스(nested class): 다른 클래스 안에 정의된 클래스

→ 자신을 감깐 바깥 클래스에서만 쓰여야 하며, 그 외에 쓰임새가 있다면 톱레벨 클래스로 만들어야 함

종류: 정적 멤버 클래스, (비정적) 멤버 클래스, 익명 클래스, 지역 클래스

정적 멤버 클래스를 제외한 나머지는 내부 클래스(inner class)에 해당

> 이 아이템에서는 각각의 중첩 클래스를 언제 그리고 왜 사용해야 하는지 설명한다.
> 

## 정적 멤버 클래스(static member class)

- 다른 클래스 내부에 선언되고, 바깥 클래스의 `private` 멤버에도 접근 가능한 클래스(이 외에는 일반 클래스와 동일)
- 정적 멤버 클래스는 다른 정적 멤버와 똑같은 접근 규칙을 적용 받음(Ex. `private`으로 선언하면 바깥클래스에서만 접근 가능)
- 바깥 클래스와 논리적으로 밀접한 관계에 있지만, 인스턴스 간의 연결이 필요하지 않을 때 사용
- 흔히 바깥 클래스와 함께 쓰일 때만 유용한 `public` 도우미 클래스로 사용됨

```java
public class OuterClass {
    private static String staticOuterVariable = "바깥 클래스의 정적 변수";
    private String instanceOuterVariable = "바깥 클래스의 인스턴스 변수";
    
    // 정적 멤버 클래스
    public static class StaticNestedClass {
        public void display() {
            // 바깥 클래스의 정적 변수에 접근 가능
            System.out.println(staticOuterVariable);
            
            // 바깥 클래스의 인스턴스 변수에는 접근할 수 없음
            // System.out.println(instanceOuterVariable); // 컴파일 에러 발생
            
            System.out.println("정적 멤버 클래스의 display 메서드 호출");
        }
    }
    
    public void accessMembersFromOuter() {
        // 바깥 클래스에서 정적 멤버 클래스의 인스턴스를 생성하고 메소드 호출
        StaticNestedClass nestedObject = new StaticNestedClass();
        nestedObject.display();
    }
    
    public static void main(String[] args) {
        // 바깥 클래스의 인스턴스를 통해서도 접근 가능
        OuterClass outerObject = new OuterClass();
        outerObject.accessMembersFromOuter();
        
        // 바깥 클래스의 인스턴스 없이도 정적 멤버 클래스의 인스턴스를 생성 가능
        OuterClass.StaticNestedClass staticNestedObject = new OuterClass.StaticNestedClass();
        staticNestedObject.display();
    }
}

```

Ex) 계산기가 지원하는 연산 종류를 정의하는 열거 타입(Item 34)

```java
// Enum type with constant-specific class bodies and data
public enum Operation {
    PLUS("+") {
        public double apply(double x, double y) {
            return x + y;
        }
    },
    MINUS("-") {
        public double apply(double x, double y) {
            return x - y;
        }
    },
    TIMES("*") {
        public double apply(double x, double y) {
            return x * y;
        }
    },
    DIVIDE("/") {
        public double apply(double x, double y) {
            return x / y;
        }
    };
    private final String symbol;

    Operation(String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() {
        return symbol;
    }

    public abstract double apply(double x, double y);
}
```

*Operation* 열거 타입은 *Calculator* 클래스의 `public` 정적 멤버 → *Calculator*의 클라이언트에서 `Calculator.Operation.PLUS`나 `Calculator.Operation.MINUS` 같은 형태로 원하는 연산을 참조할 수 있음

## 비정적 멤버 클래스(nonstatic  member class)

- `static`이 붙어있지 않은 클래스
- 비정적 멤버 클래스의 인스턴스는 바깥 클래스의 인스턴스와 암묵적으로 연결 → 비정적 멤버 클래스의 인스턴스 메서드에서 정규화된 `this`를 사용해 바깥 인스턴스의 메서드를 호출하거나 바깥 인스턴스의 참조를 가져올 수 있음
    - 정규화된 this(qualified this): `클래스명.this` 형태로 바깥 클래스의 이름을 명시하는 용법[JLS, 15.8.4](https://docs.oracle.com/javase/specs/jls/se8/html/jls-15.html#jls-15.8.4)
- 비정적 멤버 클래스는 바깥 인스턴스 없이 생성할 수 없음 → 개념상 중첩 클래스의 인스턴스가 바깥 인스턴스와 독립적으로 존재할 수 있다면 정적 멤버 클래스로 만들어야 함
- 비정적 멤버 클래스의 인스턴스와 바깥 인스턴스 사이의 관계는 멤버 클래스가 인스턴스화될 때 확립되며, 더 이상 변경할 수 없음(바깥 클래스의 인스턴스 메서드에서 비정적 멤버 클래스의 생성자를 호출할 때 자동으로 만들어 짐)
- 직접 `바깥 인스턴스의 클래스.new MemberClass(args)`를 호출에 수동으로 만들 수 있음(명시적 연결)

```java
public class OuterClass {
    private int outerValue = 100;

    // 비정적 멤버 클래스
    public class InnerClass {
        private int innerValue = 200;

        public void displayValues() {
            // 바깥 클래스의 인스턴스 변수에 직접 접근
            System.out.println("Outer Value: " + outerValue);
            // 정규화된 this를 사용하여 바깥 클래스의 인스턴스 변수에 접근
            System.out.println("Outer Value with qualified this: " + OuterClass.this.outerValue);
            // 내부 클래스의 인스턴스 변수에 접근
            System.out.println("Inner Value: " + innerValue);
        }
    }

    public void createInner() {
        // 바깥 클래스의 인스턴스를 통해 비정적 멤버 클래스의 인스턴스 생성
        InnerClass inner = new InnerClass();
        inner.displayValues();
    }

    public static void main(String[] args) {
        // 자동으로 InnerClass 인스턴스 생성 및 사용
        OuterClass outer = new OuterClass();
        outer.createInner();

        // 명시적 연결을 이용해 바깥 클래스와 InnerClass 인스턴스 생성 및 사용
        OuterClass anotherOuter = new OuterClass();
        OuterClass.InnerClass anotherInner = anotherOuter.new InnerClass();
        anotherInner.displayValues();
    }
}
```

→ 이 관계 정보는 비정적 멤버 클래스의 인스턴스 안에 만들어져 메모리 공간을 차지하며, 생성 시간도 더 걸린다.

### 어댑터 패턴으로 사용

- 어댑터 패턴(Adapter Pattern): 서로 다른 인터페이스를 가진 클래스들이 함께 작동할 수 있도록 해주는 구조적 디자인 패턴
- 비정적 멤버 클래스는 어댑터를 정의할 때 자주 사용
- 어떤 클래스의 인스턴스를 감싸 마치 다른 클래스의 인스턴스처럼 보이게 하는 뷰로 사용
    - *Map* 인터페이스의 구현체들은 보통 (*keySet*, *entrySet*, *values* 메서드가 반환하는) 자신의 컬렉션 뷰를 구현할 때 비정적 멤버 클래스를 사용
    
    ```java
    public class SimpleMap<K, V> implements Map<K, V> {
        private Set<Map.Entry<K, V>> entries = new HashSet<>();
    
        @Override
        public Set<Map.Entry<K, V>> entrySet() {
            return new EntrySetView();
        }
    
        private class EntrySetView extends AbstractSet<Map.Entry<K, V>> {
            @Override
            public Iterator<Map.Entry<K, V>> iterator() {
                return entries.iterator();
            }
    
            @Override
            public int size() {
                return entries.size();
            }
        }
    }
    ```
    
    - *Set*과 *List* 같은 다른 컬렉션 인터페이스 구현들도 자신의 반복자를 구현할 때 비정적 멤버 클래스를 주로 사용
    
    ```java
    // Typical use of a nonstatic member class
    public class MySet<E> extends AbstractSet<E> {
        ... // Bulk of the class omitted
    
        @Override
        public Iterator<E> iterator() {
            return new MyIterator();
        }
    
        private class MyIterator implements Iterator<E> { 
            ...
        }
    }
    ```
    

→ **멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 static을 붙여서 정적 멤버 클래스로 만들자.**

- `static`을 생략하면 바깥 인스턴스로의 숨은 외부 참조를 갖게 됨 → 이 참조를 저장하려면 시간과 공간을 소비해야 함
- 가비지 컬렉션이 바깥 클래스의 인스턴스를 수거하지 못하는 메모리 누수가 생길 수 있음(Item 7)

### `private` 정적 멤버 클래스

주로 바깥 클래스가 표현하는 객체의 한 부분(구성 요소)을 나타낼 때 사용되며, 바깥 클래스와 독립적으로 존재할 수 있음

Ex) 키와 값을 매핑시키는 Map 인스턴스

- 많은 Map 구현체는 각각의 키-값 쌍을 표현하는 엔트리(Entry) 객체들을 가지고 있다.
- 모든 엔트리가 맵과 연관되어 있지만, 엔트리의 메서드들(getKey, getValue, setValue)은 맵을 직접 사용하지 않는다.

```java
public class CustomMap<K, V> {
    private List<MapEntry<K, V>> entries = new LinkedList<>();

    private static class MapEntry<K, V> {  // 바깥 클래스와 독립적
        // 바깥 클래스가 표현하는 객체의 한 부분(구성 요소)을 나타냄
        private K key;
        private V value;

        MapEntry(K key, V value) {
            this.key = key;
            this.value = value;
        }

        // 엔트리의 메서드들은 맵을 직접 사용하지 않음
        public K getKey() { return key; }
        public V getValue() { return value; }
        public void setValue(V value) { this.value = value; }
    }
}
```

→ 엔트리를 비정적 멤버 클래스로 표현하는 것은 낭비고, `private` 정적 멤버 클래스가 가장 알맞음. `static`을 빠뜨려도 맵은 여전히 동작하겠지만, 모든 엔트리가 바깥 맵으로 참조를 갖게 되어 공간과 시간을 낭비하게 됨

→ 멤버 클래스가 공개된 클래스의 `public`이나 `protected` 멤버라면, 멤버 클래스 역시 공개 API가 되어 만약 향후 릴리스에서 `static`을 붙이게 되면 하위 호환성이 깨진다.

## 익명 클래스(Anonymous Class)

- 바깥 클래스의 멤버가 아님(멤버와 달리 쓰이는 시점에 선언과 동시에 인스턴스가 만들어짐)
- 오직 비정적인 문맥에서 사용될 때만 바깥 클래스의 인스턴스를 참조할 수 있음(상수 표현[JLS, 4.12.4](https://docs.oracle.com/javase/specs/jls/se8/html/jls-4.html#jls-4.12.4)을 위한 초기회된 `final` 기본 타입과 문자열 필드만 가질 수 있음)

### 제약

- 선언한 지점에서만 인스턴스를 만들 수 있음
- `instanceof` 검사, 클래스의 이름이 필요한 작업을 수행할 수 없음
- 여러 인터페이스를 구현할 수 없음
- 인터페이스를 구현하는 동시에 다른 클래스를 상속할 수 없음
- 익명 클래스를 사용하는 클라이언트는 그 익명 클래스가 상위 타입에서 상속한 멤버 외에는 호출할 수 없음
- 표현식이 중간에 등장해 짧지 않으면(10줄 이하) 가독성이 나빠짐

### 사용

1. 자바가 람다(7장)을 지원하기 전에는 즉석에서 작은 함수 객체나 처리 객체(process object)를 만드는데 주로 사용됨 → 현재는 람다 사용
    
    Ex) 익명 클래스 vs 람다
    
    ```java
    List<Integer> list = Arrays.asList(10, 5, 6, 7, 1, 3, 4);
    
    // 익명 클래스 사용
    Collections.sort(list, new Comparator<Integer>() {
        @Override
        public int compare(Integer o1, Integer o2) {
            return Integer.compare(o1, o2);
        }
    });
    
    // 람다 사용
    Collections.sort(list, Comparator.comparingInt(o -> o));
    ```
    
2. 정적 팩터리 메서드를 구현할 때 사용

## 지역 클래스(Local Class)

- 지역변수를 선언할 수 있는 곳이면 실질적으로 어디서든 선언 가능하며, 유효 범위도 지역 변수와 같음
- 멤버 클래스처럼 이름이 있고 반복해서 사용할 수 있음
- 비정적 문맥에서 사용될 때만 바깥 인스턴스를 참조할 수 있음
- 정적 멤버를 가질 수 없음
- 가독성을 위해 짧게 작성해야 함

Ex) 지역 클래스를 사용하여 숫자 배열의 합계를 계산하는 메서드

```java
public class LocalClassExample {
    public static void main(String[] args) {
        LocalClassExample example = new LocalClassExample();
        example.processNumbers(new int[]{1, 2, 3, 4, 5});
    }

    public void processNumbers(int[] numbers) {
        // 지역 클래스 정의
        class NumberProcessor {
            public int sum() {
                int sum = 0;
                for (int number : numbers) {
                    sum += number;
                }
                return sum;
            }
        }

        // 지역 클래스의 인스턴스 생성 및 사용
        NumberProcessor processor = new NumberProcessor();
        int sum = processor.sum();
        System.out.println("Numbers sum: " + sum);
    }
}

```

## 정리

- 메서드 밖에서도 사용해야 하거나 메서드 안에 정의하기엔 너무 길다면 멤버 클래스로 만들기
- 멤버 클래스의 인스턴스 각각이 바깥 클래스를 참조한다면 비정적으로, 그렇지 않으면 정적으로 만들기
- 중첩 클래스가 한 메서드 안에서만 쓰이면서 그 인스턴스를 생성하는 지점이 단 한 곳이고 해당 타입으로 쓰기에 적합한 클래스나 인터페이스가 이미 있다면 익명 클래스로 만들고, 그렇지 않으면 지역 클래스로 만들자.
