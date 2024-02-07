# Item17: Minimize mutability

*변경 가능성을 최소화하라*

---

## 불변 클래스

불변 클래스: 인스턴스 내부 값을 수정할 수 없는 클래스

Ex) String, boxed primitive 클래스, BigInteger, BigDecimal …

가변 클래스보다 설계, 구현, 사용이 쉬우며 오류가 생길 여지가 적고 안전함

### 불변 클래스를 만드는 다섯 가지 규칙

- **객체의 상태를 변경하는 메서드(변경자)를 제공하지 않는다.**
- **클래스를 확장할 수 없도록 한다.**
    - 상속을 할 수 없도록 클래스를 `final`로 선언
- **모든 필드를 `final`로 선언한다.**
    - 새로 생성된 인스턴스를 동기화 없이 다른 스레드로 건네도 문제없이 동작하게끔 보장하는 데 필요[JLS, 17.5; Geotz06, 16](https://docs.oracle.com/javase/specs/jls/se8/html/jls-17.html#jls-17.5)
- **모든 필드를 `private`으로 선언한다.**
    - 필드가 참조하는 가변 객체를 클라이언트에서 직접 접근해 수정하는 일을 막아줌
    - `public final`로 선언해도 불변 객체가 되지만, 다음 릴리스에서 내부 표현을 바꾸는 것이 어려움(Item 15, 16)
- **자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다.**
    - 클래스에 가변 객체를 참조하는 필드가 하나라도 있다면 클라이언트에서 그 객체의 참조를 얻을 수 없도록 해야 함
    - 클라이언트가 제공한 객체 참조를 가리키게 하거나 접근자 메서드가 그 필드를 그대로 리턴하면 안됨
    - 생성자, 접근자, `readObject` 메서드(Item 88) 모두에서 방어적 복사로 수행

예시 코드

```java
// 클래스를 final로 선언
public final class ImmutableClass {
    // 모든 필드를 final, private으로 선언
    private final int field1;
    private final String field2;
    private final MutableClass mutableField;

    public ImmutableClass(int field1, String field2, MutableClass mutableField) {
        this.field1 = field1;
        this.field2 = field2;

        // 가변 컴포넌트에 대한 독점적인 접근을 보장(방어적 복사)
        this.mutableField = new MutableClass(mutableField.getValue());
    }

    // 변경자 메소드를 제공하지 않음
    // 대신, 필드의 값을 반환하는 접근자 메소드만 제공

    public int getField1() { return field1; }
    public String getField2() { return field2; }

    // 가변 컴포넌트에 대한 독점적인 접근을 보장(방어적 복사)
    public MutableClass getMutableField() {
        return new MutableClass(mutableField.getValue());
    }
}

// 가변 클래스 예시
class MutableClass {
    private int value;
    public MutableClass(int value) {
        this.value = value;
    }
    public void setValue(int value) {
        this.value = value;
    }
    public int getValue() {
        return value;
    }
}
```

### 불변 복소수 클래스

```java
// Immutable complex number class
public final class Complex {
    private final double re;
    private final double im;

    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public double realPart() { return re; }

    public double imaginaryPart() { return im; }

    public Complex plus(Complex c) { 
        return new Complex(re + c.re, im + c.im);
    }

    public Complex minus(Complex c) {
        return new Complex(re - c.re, im - c.im);
    }

    public Complex times(Complex c) {
        return new Complex(re * c.re - im * c.im,
                re * c.im + im * c.re);
    }

    public Complex dividedBy(Complex c) {
        double tmp = c.re * c.re + c.im * c.im;
        return new Complex((re * c.re + im * c.im) / tmp,
                (im * c.re - re * c.im) / tmp);
    }

    @Override
    public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof Complex))
            return false;
        Complex c = (Complex) o;
        // See page 47 to find out why we use compare instead of ==
        return Double.compare(c.re, re) == 0
                && Double.compare(c.im, im) == 0;
    }

    @Override
    public int hashCode() {
        return 31 * Double.hashCode(re) + Double.hashCode(im);
    }

    @Override
    public String toString() {
        return "(" + re + " + " + im + "i)";
    }
}
```

- 사칙연산 메서드(`plus`, `minus`, `times`, `dividedBy`)들이 인스턴스 자신은 수정하지 않고 새로운 `Complex` 인스턴스를 만들어 리턴 → 불변성 유지
- 피연산자에 함수를 적용해 그 결과를 리턴하지만, 피연산자 자체는 그대로 → 함수형 프로그래밍
- 메서드 이름으로 동사(add) 대신 전치사(plus)를 사용 → 해당 메서드가 객체의 값을 변경하지 않는 다는 사실을 강조

## 불변 클래스의 장점

### **불변 객체는 단순하다.**

- 생성된 시점의 상태를 파괴될 때까지 그대로 간직
- 모든 생성자가 클래스 불변식(class invariant)을 보장한다면 다른 노력을 들이지 않아도 영원히 불변으로 남음

### **불변 객체는 근본적으로 스레드 안전하여 따로 동기화할 필요 없다.**

- 여러 스레드가 동시에 사용해도 절대 훼손되지 않음 → **불변 객체는 안심하고 공유할 수 있다.**
- 불변 클래스라면 한번 만든 인스턴스를 최대한 재활용하는 것이 좋음
    - 가장 쉬운 방법은 자주 쓰이는 값들을 상수(`public static final`)로 제공

```java
public static final Complex ZERO = new Complex(0, 0);
public static final Complex ONE = new Complex(1, 0);
public static final Complex I = new Complex(0, 1);
```

Complex 클래스에서 `ZERO`, `ONE`, `I`를 상수로 제공하여 재활용을 할 수 있음 

### 자주 사용되는 인스턴스를 캐싱하는 정적 팩터리(Item 1) 메서드를 제공할 수 있다.

- 기존의 인스턴스를 재사용하여 새 인스턴스를 생성하는 것을 피하게 해줌
- 여러 클라이언트는 새 인스턴스를 생성하는 대신 기존 인스턴스를 공유하게 됨 → 메모리 사용량과 가비지 컬렉션 비용이 줄어듦
- 새 클래스를 설계할 때 `public` 생성자 대신 정적 팩토리 메소드를 만들어두면, 클라이언트 수정 없이 필요에 따라 캐시 기능을 나중에 덧붙일 수 있음

Ex) 모든 박싱된 기본 타입 클래스와 BigInteger 클래스가 이렇게 동작 

```java
Integer a = Integer.valueOf(100);
Integer b = Integer.valueOf(100);

System.out.println(a == b); // 출력 결과: true

Integer c = Integer.valueOf(200);
Integer d = Integer.valueOf(200);

System.out.println(c == d); // 출력 결과: false
```

`Integer` 클래스의 인스턴스(-128~127 사이 정수)를 메서드로 생성하여 캐싱됨을 확인

### 불변 객체를 자유롭게 공유할 수 있어 방어적 복사(Item 50)도 할 필요가 없다.

- 복사본은 원본과 항상 동일, 복사 자체가 의미 없음
- 불변 클래스는 clone 메서드나 복사 생성자(Item 13)을 제공하지 않는 게 좋음

Ex) `String` 클래스의 복사 생성자는 되도록 사용하지 말아야 함

```java
String s1 = "hello";
String s2 = s1;  // 복사 생성자를 사용하지 않음
```

### **불변 객체는 자유롭게 공유할 수 있음은 물론, 불변 객체끼리는 내부 데이터를 공유할 수 있다.**

- Ex) *BigInteger* 클래스
    - sign-magnitude 표현을 이용해, 부호는 `int`형, 크기는 `int` 배열로 표현
    - `negate` 메서드는 부호만 바꾼 새 `BigInteger` 객체를 생성할 때, 크기를 표현하는 `int` 배열은 복사하지 않음 → 기존 객체의 같은 크기를 나타내는 `int` 배열을 참조

### **객체를 만들 때 다른 불변 객체들을 구성요소로 사용하면 이점이 많다.**

- 불변 구성요소들로 이루어진 객체는 그 구조가 아무리 복잡하더라도 불변식을 유지하기 수월함
- Ex) Map의 key, Set의 원소를 불변 객체로 사용하면 불변식을 유지할 수 있음

### **불변 객체는 그 자체로 실패 원자성을 제공한다(Item 76).**

- 실패 원자성(failure atomicity): 메서드에서 예외가 발생한 후에도 그 객체는 여전히 유효한 상태여야 한다는 성질 → 연산이 실패하더라도 객체의 상태가 연산 실행 전과 동일하게 유지
- 상태가 절대 변하지 않으므로 잠깐이라도 불일치 상태가 되지 않음
- Ex) `BigInteger` 객체의 `add` 메서드는 새로운 객체를 리턴하는데, 실패하더라도 원래 `BigInteger` 객체의 상태는 변하지 않음

## 불변 클래스의 단점

**값이 다르면 반드시 독립된 객체로 만들어야 한다.**

→ 값의 가짓수가 많다면 이를 만드는 데 큰 비용을 치러야 함(메모리 사용량 증가, 가비지 컬렉션 부담 증가)

Ex) `BigInteger`에서 비트 하나를 바꿀 때

```java
BigInteger moby = ...;
moby = moby.flipBit(0);
```

`flipBit` 메서드는 단지 한 비트만 다른 백만 비트짜리 인스턴스를 생성하기 위해 새로운 `BigInteger` 인스턴스를 생성 → 크기에 비례해 시간, 공간을 잡아 먹음

Ex) `BitSet`에서 비트 하나를 바꿀 때

```java
BitSet moby = ...;
moby.flip(0);
```

`BitSet`은 가변 클래스로, `flip` 메서드는 원하는 비트 하나만 상수 시간 안에 바꿔줌

### 해결 방법

1. 다단계 연산(multistep operation)들을 예측하여 기본 기능으로 제공
    - 자주 필요한 다단계 작업을 예측하고 이를 기본 기능으로 제공
    - 각 단계마다 객체를 생성하지 않아도 됨

```java
BigInteger bigInteger = ...;
BigInteger newBigInteger = bigInteger.flipMultipleBits(0, 1, 2);  // 다단계 작업을 기본 기능으로 제공
```

2. `package-private`의 가변 동반 클래스(mutable companion class)를 제공
    - 불변 클래스를 사용하는 클라이언트가 수행하려는 복잡한 작업을 정확히 예측할 수 없는 경우 유용
    - Ex) 문자열을 연결할 때
        - `String` 클래스는 불변 객체이므로, 문자열을 더하는 연산을 수행할 때마다 새로운 `String` 객체가 생성되는 문제를 해결하기 위해 `StringBuilder`(가변 동반 클래스)를 제공

```java
// BigInteger.java
package java.math;

public class BigInteger {
    // ... other codes ...

    public BigInteger add(BigInteger val) {
        MutableBigInteger result = new MutableBigInteger(this);  // 1. 내부적으로 MutableBigInteger 객체를 생성
        result.add(val);  // 2. BigInteger 값을 MutableBigInteger의 add 메서드로 더하기 연산
        return result.toBigInteger();  // 3. BigInteger 타입으로 리턴
    }
}

// MutableBigInteger.java
package java.math;

class MutableBigInteger {  // package-private 클래스로 불변식을 유지
    // ... other codes ...

    void add(BigInteger val) {  
        // ... other codes ...
    }
}
```

## 불변 클래스를 만드는 설계 방법

자신을 상속하지 못하게 해야 함 →

1. `final` 클래스로 선언
2. 모든 생성자를 `private` 혹은 `package-private`으로 만들고 `public` 정적 팩터리를 제공(Item 1) → 더 유연한 방법

Ex) 생성자 대신 정적 팩터리를 사용한 불변 클래스

```java
// Immutable class with static factories instead of constructors
public class Complex {
    private final double re;
    private final double im;

    private Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public static Complex valueOf(double re, double im) {
        return new Complex(re, im);
    }
    ... // Remainder unchanged
}
```

다수의 구현 클래스를 활용한 유연성을 제공하고, 다음 릴리스에서 객체 캐싱 기능을 추가해 성능 개선 가능

### 불변 클래스 사용시 주의 사항

*BigInteger*와 *BigDecimal*을 설계할 당시엔 불변 객체가 `final`이어야 한다는 생각이 널리 퍼지지 않았음 → 두 클래스의 메서드들은 모두 재정의가 가능하도록 설계되어 지금까지도 하위 호환성 문제가 있음

신뢰할 수 없는 클라이언트에게 *BigInteger*와 *BigDecimal*의 인스턴스를 인수로 받아오는 것을 주의해야 함

신뢰할 수 없는 하위 클래스의 인스턴스라고 확인되면, 인수들은 가변이라 가정하고 방어적으로 복사해 사용해야 함(Item 50)

```java
public static BigInteger safeInstance(BigInteger val) {
    return val.getClass() == BigInteger.class ?
    val : new BigInteger(val.toByteArray());
}
```

`val`이 `BigInteger` 클래스의 인스턴스인지 확인 후 그렇지 않은 경우 새 객체를 생성 후 리턴

### 완화한 불변 클래스 규칙

> 원칙: 모든 필드가 final이고 어떤 메서드도 그 객체를 수정할 수 없어야 한다.
→ 완화한 규칙: 어떤 메서드도 객체의 상태 중 외부에 비치는 값을 변경할 수 없다.
> 
- 계산 비용이 큰 값을 나중에 계산하여 `final`이 아닌 필드에 캐시해두어 계산 비용을 절감하는 것이 가능
- 게터(`getter`)가 있다고 해서 반드시 세터(`setter`)를 만들 필요는 없음
- **클래스는 꼭 필요한 경우가 아니라면 불변이어야 한다. →** 불변 클래스의 장점이 더 많으며, 단점이라곤 특정 상황에서의 성능 저하일 뿐
- 단순한 값 객체는 항상 불변으로 만들어야 함
- 무거운 값 객체도 불변으로 만들 수 있는지 고심해야 함 → 성능 때문에 어쩔 수 없다면(Item 67) 불변 클래스와 쌍을 이루는 가변 동반 클래스를 `public` 클래스로 제공하는 것을 고려

## 불변으로 만들 수 없는 클래스에 대한 처리

- **불변으로 만들 수 없는 클래스라도 가변 부분을 최소한으로 줄이자**
    - 객체 예측이 쉬워지고 오류 가능성이 줄어듦
    - 꼭 변경해야 할 필드를 제외한 나머지를 `final`로 선언
- **다른 합당한 이유가 없다면 모든 필드는 `private final`이어야 한다.**
- **생성자는 불변식 설정이 모두 완료된, 초기화가 완벽히 끝난 상태의 객체를 생성해야 한다.**
- 확실한 이유가 없다면 생성자와 정적 팩터리 외에는 그 어떤 초기화 메서드도 public으로 제공해서는 안됨 → 복잡성만 커지고 성능 이점은 거의 없음

Ex) *java.util.concurrent* 패키지의 *CountDownLatch* 클래스

- 가변 클래스지만 가질수 있는 상태의 수가 많지 않음
- 인스턴스를 생성해 한 번 사용하고 끝
- 카운트가 0에 도달하면 재사용이 불가능

→ 한 번 사용하고 버리는 일회용 객체
