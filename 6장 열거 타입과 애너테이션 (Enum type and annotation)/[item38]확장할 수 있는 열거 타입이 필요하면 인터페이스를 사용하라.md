# 열거 타입의 확장

거의 모든 상황에서 열거 타입이 타입 안전 열거 패턴보다 우수하다.

하지만 타입 안전 열거 패턴은 확장할 수 있으나 열거 타입은 그럴 수 없다는 단점이 있다.

열거 타입이 임의의 인터페이스를 구현할 수 있다는 사실을 이용하면 비슷한 효과를 볼 수 있다.

---

# 인터페이스를 이용한 열거 타입의 확장

``` java
public interface Operation {
  double apply(double x, double y);
}

public enum BasicOperation implements Operation {
    PLUS("+") {
        public double apply(final double x, final double y) {return x + y;}
    },
    MINUS("-") {
        public double apply(final double x, final double y) {return x - y;}
    },
    TIMES("*") {
        public double apply(final double x, final double y) {return x * y;}
    },
    DIVIDE("/") {
        @Override
        public double apply(final double x, final double y) {return x / y;}
    };

    private final String symbol;

    BasicOperation(final String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() {
        return symbol;
    }
}
```

열거 타입인 `BasicOperation`은 확장할 수 없지만, `Operation`은 확장할 수 있다.

-> 인터페이스를 연산의 타입으로 사용

지수 연산(EXP)과 나머지 연산(REMAINDER)을 추가하면,

``` java
public enum ExtendedOperation implements Operation {
    EXP("^") {
        public double apply(final double x, final double y) {return Math.pow(x, y);}
    },
    REMAINDER("%") {
        public double apply(final double x, final double y) {return x % y;}
    };
    
    private final String symbol;

    ExtendedOperation(final String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() {
        return symbol;
    }
}
```

`Operation` 인터페이스를 사용하도록 작성되어 있기만 하면 언제든 새로운 연산을 적용할 수 있다.

개별 인스턴스 수준이 아닌 타입 수준에서도 사용할 수 있다.

``` java
public static void main(String[] args) {
  double x = 4;
  double y = 2;
  test(ExtendedOperation.class, x, y);
}

private static <T extends Enum<T> & Operation> void test(Class<T> opEnumType, double x, double y) {
  for (Operation op : opEnumType.getEnumConstants()) {
    System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
  }
}
```

`test`메서드에 `ExtendedOperation`의 class 리터럴을 넘겨 확장된 연산들이 무엇인지 알려준다.

여기서 class 리터럴은 한정적 타입 토큰 역할을 한다.

`onEnumType` 매개변수의 선언 `<T extends Enum<T> & Operation> Class<T>`

-> Class 객체가 열거 타입인 동시에 `Operation`의 하위 타입이어야 한다.

두 번째 대안은 Class 객체 대신 한정적 와일드 카드 타입인 `Collection<? extends Operation>`을 넘기는 방법이다.

``` java
public static void main(String[] args) {
  double x = 4;
  double y = 2;
  test(Arrays.asList(ExtendedOperation.values()),x , y);
}

private static void test(Collection<? extends Operation> opSet, double x, double y) {
  for (Operation op : opSet) {
    System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
  }
}
```

이 방안의 test 메서드는 여러 구현 타입의 연산을 조합해 호출할 수 있다.

## 문제점

열거 타입끼리 구현을 상속할 수 없다.

아무 상태에도 의존하지 않는 경우에는 디폴트 구현을 이용해 인터페이스에 추가하는 방법이 있다.

하지만 위의 예에서는 연산 기호를 저장하고 찾는 로직이 `BasicOperation`과 `ExtendedOperation` 모두에 들어가야 한다.

중복량이 적으면 문제되지 않지만, 많다면 그 부분을 별도의 도우미 클래스나 정적 도우미 메서드로 분리하는 방식으로 코드 중복을 없앨 수 있다.

---

# LinkOption

자바 라이브러리도 이러한 패턴을 사용한다.

`java.nio.file.LinkOption` 열거 타입은 `CopyOption`과 `OpenOption` 인터페이스를 구현했다.

![LinkOption](https://github.com/NoSubject-Study/effective-java-study/assets/103320798/a57126c2-02fc-4e4e-8c9c-acd716ed25f8)

뿐만 아니라, `CopyOption`의 구현체로 `ExtendedCopyOption`, `StandardCopyOption`가 있고,

`OpenOption`의 구현체로 `ExtendedOpenOption`, `StandardOpenOption`가 있다.
