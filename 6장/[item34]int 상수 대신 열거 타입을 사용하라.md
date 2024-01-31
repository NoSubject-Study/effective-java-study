# 열거 타입이란

열거 타입은 일정 개수의 상수 값을 정의한 다음, 그 외의 값은 허용하지 않는 타입이다.

사계절, 태양계의 행성, 카드게임의 카드 종류 등이 좋은 예다.

---

# 정수 열거 패턴

열거 타입을 지원하기 전에는 다음 코드처럼 정수 상수를 한 묶음 선언해서 사용하곤 했다.

``` java
// 정수 열거 패턴 - 상당히 취약하다.
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD = 2;
```

---

## 정수 열거 패턴의 단점

정수 열거 기법(int enum pattern)에는 단점이 많다.

---

### 타입 안전과 표현력

타입 안전을 보장할 방법이 없으며 표현력도 좋지 않다.

오렌지를 건네야 할 메서드에 사과를 보내고 동등 연산자(`==`)로 비교하더라도 컴파일러는 아무런 경고 메시지를 출력하지 않는다.

``` java
int i = (APPLE_FUJI - ORANGE_TEMPLE) / APPLE_PIPPIN;
```

자바가 정수 열거 패턴을 위한 별도 이름공간(namespace)을 지원하지 않기 때문에 어쩔 수 없이 접두어를 써서 이름 충돌을 방지해야 한다.

---

### 깨지기 쉬운 프로그램

정수 열거 패턴을 사용한 프로그램은 깨지기 쉽다.

평범한 상수를 나열한 것뿐이라 컴파일하면 그 값이 클라이언트 파일에 그대로 새겨진다.

따라서 상수의 값이 바뀌면 클라이언트도 반드시 다시 컴파일해야 한다.

---

### 문자열로의 출력

정수 상수는 문자열로 출력하기가 다소 까다롭다.

그 값은 출력하거나 디버거로 살펴보면 단지 숫자로만 보여서 썩 도움이 되지 않는다.

같은 정수 열거 그룹에 속한 모든 상수를 한 바퀴 순회하는 방법도 마땅치 않다.

---

## 문자열 열거 패턴

정수 대신 문자열 상수를 사용하는 변형 패턴도 있다.

문자열 열거 패턴(string enum pattern)이라 하는 이 변형은 더 나쁘다.

상수의 의미를 출력할 수 있다는 점은 좋지만, 경험이 부족한 프로그래머가 문자열 상수의 이름 대신 문자열 값을 그대로 하드코딩하게 만들기 때문이다.

---

# 자바의 열거 타입

자바는 열거 패턴의 단점을 말끔히 씻어주는 동시에 여러 장점을 안겨주는 대안을 제시했다.

바로 열거 타입(enum type)이다.

``` java
// 가장 단순한 열거 타입
public enum Apple = {FUJI, PIPPIN, GRANNY_SMITH}
public enum Orange = {NAVEL, TEMPLE, BLOOD}
```

자바의 열거 타입은 완전한 형태의 클래스라서 다른 언어의 열거 타입보다 훨씬 강력하다.

열거 타입 자체는 클래스이며, 상수 하나당 자신의 인스턴스를 하나씩 만들어 `public static final` 필드로 공개한다.

열거 타입은 밖에서 접근할 수 있는 생성자를 제공하지 않으므로 사실상 `final`이다.

따라서 클라이언트가 인스턴스를 직접 생성하거나 확장할 수 없으니 열거 타입 선언으로 만들어진 인스턴스들은 딱 하나씩만 존재함이 보장된다.

싱글턴은 원소가 하나뿐인 열거 타입이라 할 수 있고, 거꾸로 열거 타입은 싱글턴을 일반화한 형태라고 볼 수 있다.

<br>

위의 코드에서 `Apple` 열거 타입을 매개변수로 받는 메서드를 선언했다면, 건네받은 참조는 (`null`이 아니라면) `Apple`의 세 가지 값 중 하나임이 확실하다.

<br>

열거 타입에는 각자의 이름 공간이 있어서 이름이 같은 상수도 평화롭게 공존한다.

열거 타입에 새로운 상수를 추가하거나 순서를 바꿔도 다시 컴파일하지 않아도 된다.

공개되는 것이 오직 필드의 이름뿐이라, 정수 열거 패턴과 달리 상수 값이 클라이언트로 컴파일되어 각인되지 않기 때문이다.

마지막으로 열거 타입의 `toString` 메서드는 출력하기에 적합한 문자열을 내어준다.

<br>

열거 타입에는 임의의 메서드나 필드를 추가할 수 있고 임의의 인터페이스를 구현하게 할 수도 있다.

`Object` 메서드들을 높은 품질로 구현해놨고, `Comparable`과 `Serializable`을 구현했으며, 그 직렬화 형태도 웬만큼 변형을 가해도 문제없이 동작하게끔 구현해놨다.

---

## 열거 타입에 메서드나 필드를 추가할 경우

각 상수와 연관된 데이터를 해당 상수 자체에 내재시키고 싶다고 해보자.

`Apple`과 `Orange`를 예로 들면, 과일의 색을 알려주거나 과일 이미지를 반환하는 메서드를 추가하고 싶을 수 있다.

<br>

태양계의 여덟 행성은 거대한 열거 타입을 설명하기에 좋은 예다.

각 행성에는 질량과 반지름이 있고, 이 두 속성을 이용해 표면중력을 계산할 수 있다.

따라서 어떤 객체의 질량이 주어지면 그 객체가 행성 표면에 있을 때의 무게도 계산할 수 있다.

``` java
public enum Planet {
  MERCURY (3.302e+23, 2.439e6),
  VENUS   (4.869e+24, 6.052e6),
  EARTH   (5.975e+24, 6.378e6),
          ...

  private final double mass;           // 질량 (단위: 킬로그램)
  private final double radius;         // 반지름 (단위: 미터)
  private final double surfaceGravity; // 표면중력 (단위: m / s^2)

  // 중력상수 (단위: m^3 / kg s^2)
  private static final double G = 6.67300E-11;

  // 생성자
  Planet(double mass, double radius) {
    this.mass = mass;
    this.radius = radius;
    surfaceGravity = G * mass / (radius * radius);
  }

  public double mass() {
    return mass;
  }

  public double radius() {
    return radius;
  }

  public double surfaceGravity() {
    return surfaceGravity;
  }

  public double surfaceWeight(double mass) {
    return mass * surfaceGravity; // F = ma
  }
}
```

열거 타입 상수 오른쪽 괄호 안 숫자는 생성자에 넘겨지는 매개변수로, 이 예에서는 행성의 질량과 반지름을 뜻한다.

열거 타입 상수 각각을 특정 데이터와 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장하면 된다.

열거 타입은 **근본적으로 불변**이라 모든 필드는 `final` 이어야 한다.

필드를 `public` 으로 선언해도 되지만, `private`으로 두고 별도의 `public` 접근자 메서드를 두는 게 낫다.

<br>

어떤 객체의 지구에서의 무게를 입력받아 여덟 행성에서의 무게를 출력하는 일을 다음처럼 짧은 코드로 작성할 수 있다.

```java
public class WeightTable {
  public static void main(String[] args) {
    double earthWeight = Double.parseDouble(args[0]);
    double mass = earthWeight / Planet.EARTH.surfaceGravity();
    for (Planet p : Planet.values())
      System.out.printf("%s에서의 무게는 %f이다.%n", p, p.surfaceWeight(mass));
  }
}
```

열거 타입은 자신 안에 정의된 상수들의 값을 배열에 담아 반환하는 정적 메서드인 `values`를 제공한다.

값들은 선언된 순서로 저장된다.

각 열거 타입 값의 `toString` 메서드는 상수 이름을 문자열로 반환하므로 `println`과 `printf`로 출력하기에 좋다.

```
명령줄 인수로 185를 주었다.
결과:

MERCURY에서의 무게는 69.912739이다.
VENUS에서의 무게는 167.434436이다.
EARTH에서의 무게는 185.000000이다.
...
```

**열거 타입에서 상수를 하나 제거해도 제거한 상수를 참조하지 않는 클라이언트에는 아무 영향이 없다.**

위의 결과에서는 출력되는 줄 수가 하나 줄어들 뿐이다.

만약 제거된 상수를 참조하는 클라이언트가 있다면 컴파일 오류가 발생할 것이다.

정수 열거 패턴에서는 기대할 수 없는 이점을 누릴 수 있다.

<br>

열거 타입을 선언한 클래스 혹은 그 패키지에서만 유용한 기능은 `private`이나 `package-private` 메서드로 구현한다.

일반 클래스와 마찬가지로, 그 기능을 클라이언트에 노출해야 할 합당한 이유가 없다면 `private`으로, 혹은 `package-private`으로 선언하라.

<br>

널리 쓰이는 열거 타입은 톱레벨 클래스로 만들고, 특정 톱레벨 클래스에서만 쓰인다면 해당 클래스의 멤버 클래스로 만든다.

예를 들어 소수 자릿수의 반올림 모드를 뜻하는 열거 타입인 `java.math.RoundingMode`는 `BigDecimal`이 사용한다.

그런데 반올림 모드는 `BigDecimal`과 관련 없는 영역에서도 유용한 개념이라 자바 라이브러리 설계자는 `RoundingMode`를 톱레벨로 올렸다.

이 개념을 많은 곳에서 사용하여 다양한 API가 더 일관된 모습을 갖출 수 있도록 장려한 것이다.

<br>

사칙 연산 계산기의 연산 종류를 열거 타입으로 선언하고, 실제 연산까지 열거 타입 상수가 직접 수행했으면 한다고 해보자.

먼저 switch 문을 이용해 상수의 값에 따라 분기하는 방법을 시도해보자.

``` java
public enum Operation {
  PLUS, MINUS, TIMES, DIVIDE;

  // 상수가 뜻하는 연산을 수행한다.
  public double apply(double x, double y) {
    switch(this) {
      case PLUS: return x + y;
      case MINUS: return x - y;
      case TIMES: return x * y;
      case DIVIDE: return x / y;
    }
    throw new AssertionError("알 수 없는 연산: " + this);
  }
}
```

동작은 하지만 코드가 아름답지 않다.

마지막의 `throw` 문은 실제로는 도달할 일이 없지만 기술적으로는 도달할 수 있기 때문에 생략하면 컴파일 오류가 발생한다.

새로운 상수를 추가하면 해당 `case` 문도 추가해야 한다.

만일 추가하지 않으면 “알 수 없는 연산”이라는 런타임 오류를 내며 프로그램이 종류된다.

<br>

열거 타입에 `apply`라는 추상 메서드를 선언하고 각 상수별 클래스 몸체(constant-specific class body), 즉 각 상수에서 자신에 맞게 재정의하는 방법이다.

이를 상수별 메서드 구현(constant-specific method implementation)이라 한다.

``` java
// 상수별 메서드 구현을 활용한 열거 타입
public enum Operation {
  PLUS {public double apply(double x, double y) {return x + y;}},
  MINUS {public double apply(double x, double y) {return x - y;}},
  TIMES {public double apply(double x, double y) {return x * y;}},
  DIVIDE {public double apply(double x, double y) {return x / y;}};

  public abstract double apply(double x, double y);
}
```

새로운 상수를 추가할 때 `apply`를 재정의 하지 않으면 컴파일 오류가 발생하기 때문에 깜빡하고 구현하지 않는 일은 없을 것이다.

<br>

상수별 메서드 구현을 상수별 데이터와 결합할 수도 있다. 예컨대 다음은 `Operation`의 `toString`을 재정의해 해당 연산을 뜻하는 기호를 반환하도록 한 예다.

``` java
// 상수별 클래스 몸체(class body)와 데이터를 사용한 열거 타입
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

  @Override public String toString() {
    return symbol;
  }

  public abstract double apply(double x, double y);
}
```

``` java
public static void main(String[] args) {
  double x = Double.parseDouble(args[0]);
  double y = Double.parseDouble(args[1]);
  for (Operation op : Operation.values())
    System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
}
```

`toString` 메서드를 재정의하였기 때문에 출력이 편해졌음을 알 수 있다.

```
명령줄 인수에 2와 4를 주어 실행하면
결과:
2.000000 + 4.000000 = 6.000000
2.000000 - 4.000000 = -2.000000
2.000000 * 4.000000 = 8.000000
2.000000 / 4.000000 = 0.500000
```

<br>

열거 타입에는 상수 이름을 입력받아 그 이름에 해당하는 상수를 반환해주는 `valueOf(String)` 메서드가 자동 생성된다.

열거 타입의 `toString` 메서드를 재정의하려거든, `toString`이 반환하는 문자열을 해당 열거 타입 상수로 변환해주는 `fromString` 메서드도 함께 제공하는 걸 고려해보자.

``` java
// 열거 타입용 fromString 메서드 구현하기

private static final Map<String, Operation> stringToEnum =
                  Stream.of(values()).collect(
                        toMap(Object::toString, e -> e));

// 지정한 문자열에 해당하는 Operation을 (존재한다면) 반환한다.
public static Optional<Operation> fromString(String symbol) {
  return Optional.ofNullable(stringToEnum.get(symbol));
}
```

<br>

`Operation` 상수가 `stringToEnum` 맵에 추가되는 시점은 열거 타입 상수 생성 후 정적 필드가 초기화될 때다.

앞의 코드는 `values` 메서드가 반환하는 배열 대신 스트림을 사용했다.

자바 8 이전에는 빈 해시맵을 만든 다음 `values`가 반환한 배열을 순회하면 {문자열, 열거 타입 상수} 쌍을 맵에 추가했을 것이다.

하지만 열거 타입 상수는 생성자에서 자신의 인스턴스를 맵에 추가할 수 없다.

열거 타입의 정적 필드 중 열거 타입의 생성자에서 접근할 수 있는 것은 상수 변수뿐이다.

열거 타입 생성자가 실행되는 시점에는 정적 필드들이 아직 초기화되기 전이라, 자기 자신을 추가하지 못하게 하는 제약이 꼭 필요하다.

열거 타입 생성자에게서 같은 열거 타입의 다른 상수에도 접근할 수 없다.

<br>

`fromString`이 `Optional<Operation>`을 반환하는 점도 주의하자.

클라이언트에게 주어진 연산이 존재하지 않을 수 있음을 알리고 그 상황을 클라이언트에서 대처하도록 한 것이다.

<br>

상수별 메서드 구현에는 열거 타입 상수끼리 코드를 공유하기 어렵다는 단점이 있다.

급여명세서에서 쓸 요일을 표현하는 열거 타입을 예로 생각해보자.

이 열거 타입은 직원의 (시간당) 기본 임금과 그날 일한 시간(분 단위)이 주어지면 일당을 계산해주는 메서드를 갖고 있다.

주중에 오버타임이 발생하면 잔업수당이 주어지고, 주말에는 무조건 잔업수당이 주어진다.

`switch` 문을 이용하면 `case` 문을 날짜별로 두어 이 계산을 쉽게 수행할 수 있다.

``` java
enum PayrollDay {
  MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;

  private static final int MINS_PER_SHIFT = 8 * 60;

  int pay(int minutesWorked, int payRate) {
    int basePay = minutesWorked * payRate;

    int overtimePay;
    switch(this) {
      case SATURDAY: // 주말
      case SUNDAY: // 주말
        overtimePay = basePay / 2;
        break;
      default: // 주중
        overtimePay = minutesWorked <= MINS_PER_SHIFT ?
        0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
    }

    return basePay + overtimePay;
  }
}
```

간결하지만 관리 관점에서는 위험한 코드다.

휴가와 같은 새로운 값을 열거 타입에 추가하려면 그 값을 처리하는 `case` 문을 잊지 말고 쌍으로 넣어줘야 하는 것이다.

<br>

상수별 메서드 구현으로 급여를 정확히 계상하는 방법은 두 가지다.

첫째, 잔업수당을 계산하는 코드를 모든 상수에 중복해서 넣으면 된다.

둘째, 계산 코드를 평일용과 주말용으로 나눠 각각을 도우미 메서드로 작성한 다음 각 상수가 자신에게 필요한 메서드를 적절히 호출하면 된다.

두 방식 모두 코드가 장황해져 가독성이 크게 떨어지고 오류 발생 가능성이 높아진다.

<br>

`PayrollDay`에 평일 잔업수당 계산용 메서드인 `overtimePay`를 구현해놓고, 주말 상수에서만 재정의해 쓰면 장황한 부분은 줄일 수 있다.

하지만 `switch` 문을 썼을 때와 똑같은 단점이 나타난다.

즉, 새로운 상수를 추가하면서 `overtimePay` 메서드를 재정의하지 않으면 평일용 코드를 그대로 물려받게 되는 것이다.

<br>

가장 깔끔한 방법은 새로운 상수를 추가할 때 잔업수당 ‘전략’을 선택하도록 하는 것이다.

잔업수당 계산을 `private` 중첩 열거 타입(다음 코드의 `PayType`)으로 옮기고 `PayrollDay` 열거 타입의 생성자에서 이 중 적당한 것을 선택한다.

그러면 `PayrollDay` 열거 타입은 잔업수당 계산을 그 전략 열거 타입에 위임하여, `switch` 문이나 상수별 메서드 구현이 필요 없게 된다.

이 패턴은 `switch` 문보다 복잡하지만 더 안전하고 유연하다.

``` java
enum PayrollDay {
  MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY),
  THURSDAY(WEEKDAY), FRIDAY(WEEKDAY),
  SATURDAY(WEEKEND), SUNDAY(WEEKEND);

  private final PayType payType;

  PayrollDay(PayType payType) {
    this.payType = payType;
  }

  int pay(int minutesWorked, int payRate) {
    return payType.pay(minutesWorked, payRate);
  }

  // 전략 열거 타입
  enum PayType {
    WEEKDAY {
      int overtimePay(int minsWorked, int payRate) {
        return minsWorked <= MINS_PER_SHIFT ?
          0 : (minsWorked - MINS_PER_SHIFT) * payRate / 2;
      }
    },
    WEEKEND {
      int overtimePay(int minsWorked, int payRate) {
        return minsWorked * payRate / 2;
      }
    };

    abstract int overtimePay(int mins, int payRate);
    private static final int MINS_PER_SHIFT = 8 * 60;

    int pay(int minsWorked, int payRate) {
      int basePay = minsWorked * payRate;
      return basePay + overtimePay(minsWorked, payRate);
    }
  }
}
```

`switch` 문은 열거 타입의 상수별 동작을 구현하는 데 적합하지 않다.

하지만 기존 열거 타입에 상수별 동작을 혼합해 넣을 때는 `switch` 문이 좋은 선택이 될 수 있다.

예를 들어, 서드파티에서 가져온 `Operation` 열거 타입이 있는데, 각 연산의 반대 연산을 반환하는 메서드가 필요하다고 해보자.

<br>

``` java
// switch 문을 이용해 원래 열거 타입에 없는 기능을 수행한다.
public static Operation inverse(Operation op) {
  switch(op) {
    case PLUS: return Operation.MINUS;
    case MINUS: return Operation.PLUS;
    case TIMES: return Operation.DIVIDE;
    case DIVIDE: return Operation.TIMES;

    default: throw enw AssertionError("알 수 없는 연산: " + op);
  }
}
```

추가하려는 메서드가 의미상 열거 타입에 속하지 않는다면 직접 만든 열거 타입이라도 이 방식을 적용하는 게 좋다.

종종 쓰이지만 열거 타입 안에 포함할만큼 유용하지는 않은 경우도 마찬가지다.

<br>

대부분의 경우 열거 타입의 성능은 정수 상수와 별반 다르지 않다.

열거 타입을 메모리에 올리는 공간과 초기화하는 시간이 들긴 하지만 체감될 정도는 아니다.

---

# 열거 타입을 사용하는 경우

필요한 원소를 컴파일타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자.

태양계 행성, 한 주의 요일, 체스 말처럼 본질적으로 열거 타입인 타입은 당연히 포함된다.

그리고 메뉴 아이템, 연산 코드, 명령줄 플래그 등 허용하는 값 모두를 컴파일타임에 이미 알고 있을 때도 쓸 수 있다.

열거 타입에 정의된 상수 개수가 영원히 고정 불변일 필요는 없다.

열거 타입은 나중에 상수가 추가돼도 바이너리 수준에서 호환되도록 설계되었다.
