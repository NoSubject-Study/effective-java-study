# **Item22: Use interfaces only to define types**

인터페이스는 타입을 정의하는 용도로만 사용하라

---

## 올바른 인터페이스 사용 용도

인터페이스: 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할

→ 클래스가 어떤 인터페이스를 구현한다는 것은 자신의 인스턴스로 무엇을 할 수 있는지를 클라이언트에 얘기해주는 것(오직 이 용도로만 사용해야 함!)

상수 인터페이스(constant interface): 메서드 없이 상수를 뜻하는 static final 필드로만 가득 찬 인터페이스

```java
// Constant interface antipattern - do not use!
public interface PhysicalConstants {
    // Avogadro's number (1/mol)
    static final double AVOGADROS_NUMBER = 6.022_140_857e23;
    // Boltzmann constant (J/K)
    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
    // Mass of the electron (kg)
    static final double ELECTRON_MASS = 9.109_383_56e-31;
}
```

## 상수 인터페이스 안티 패턴(Constant Interface Anti-Pattern)

**상수 인터페이스 안티 패턴은 인터페이스를 잘못 사용한 예다.**

1. 인터페이스의 사용 목적과 부합하지 않음
    - 인터페이스는 주로 타입을 정의하고, 해당 타입의 여러 구현체를 다루기 위한 목적
2. 클래스와의 결합도 증가
    - 클래스 내부에서 사용하는 상수는 외부 인터페이스가 아닌 내부구현에 해당 → 내부 구현을 클래스의 API로 노출하는 행위
    - 클라이언트 코드가 내부 구현에 해당하는 상수들에 종속되게 함 → 구현한 클래스가 인터페이스의 상수에 의존하게 되어 인터페이스 변경이 어려워지므로, 코드 유지보수가 어려워짐
    - 다음 릴리스에서 이 상수들을 더 이상 사용하지 않더라도 바이너리 호환성을 위해 여전히 상수 인터페이스를 구현하고 있어야 함
        - 바이너리 호환성(binary compatibility): 이미 컴파일된 프로그램이나 라이브러리의 코드를 변경한 후에도, 변경 이전에 컴파일된 클라이언트 코드가 재컴파일 없이 기존과 동일하게 동작하는 것
        - 상수 변경, 삭제 시 런타임/컴파일 에러가 발생하거나 예상치 못한 동작을 일으킬 수 있음
3. 이름 공간(Name Space) 오염
    - `final`이 아닌 클래스가 상수 인터페이스를 구현한다면 모든 하위 클래스의 이름공간이 그 인터페이스가 정의한 상수들로 오염됨
    - 클래스와 하위 클래스의 이름 공간이 인터페이스에 정의된 모든 상수로 채워지게 되어, 클래스 내에서 정의한 멤버들과 구분하기 어려워짐 → 코드의 가독성과 유지보수성 저하

자바 플랫폼 라이브러리(*java.io.ObjectStreamConstants* …)에도 상수 인터페이스가 몇 개 있으나, 인터페이스를 잘못 활용한 예로 따라하지 않아야 함

## 상수를 공개하는 방법

1. 특정 클래스나 인터페이스와 강하게 연관된 상수라면 그 클래스나 인터페이스 자체에 추가(Ex. *Integer*와 *Double*에 선언된 *MIN_VALUE*, *MAX_VALUE*) → 상수들이 해당 데이터 타입과 긴밀하게 연관되어 있음을 강조

```java
public final class Integer extends Number implements Comparable<Integer> {
	  ...
    @Native
    public static final int MIN_VALUE = 0x80000000;

    @Native
    public static final int MAX_VALUE = 0x7fffffff;
    ...
}
```

2. 열거 타입(`Enum`)으로 나타내기 적합한 상수면 열거 타입으로 만들어 공개(Item 34) → 타입 안정성을 보장하고, 내부적으로 추가적인 기능과 메서드를 제공 가능

```java
public enum Season {
    SPRING, SUMMER, FALL, WINTER
}
```

3. 인스턴스화할 수 없는 유틸리티 클래스(Item 4)에 담아 공개 → 

```java
// Constant utility class
package com.effectivejava.science;

public class PhysicalConstants {
    private PhysicalConstants() { }  // Prevents instantiation
    
    // Avogadro's number (1/mol)
    public static final double AVOGADROS_NUMBER = 6.022_140_857e23;
    // Boltzmann constant (J/K)
    public static final double BOLTZMANN_CONST = 1.380_648_52e-23;
    // Mass of the electron (kg)
    public static final double ELECTRON_MASS = 9.109_383_56e-31;
}
```

> Java 7부터 숫자 리터럴 사이에 밑줄(_)을 사용하는 것이 허용되어 숫자 리터럴의 값에는 아무런 영향을 주지 않으면서, 읽기는 훨씬 편하게 해준다.
> 

- 유틸리티 클래스에 정의된 상수를 클라이언트에서 사용하려면 클래스 이름까지 함께 명시해야 함(Ex. *PhysicalConstants.AVOGADROS_NUMBER*)
- 유틸리티 클래스의 상수를 빈번히 사용한다면 정적 임포트(static import)하여 클래스 이름 생략 가능

```java
// Use of static import to avoid qualifying constants
import static com.effectivejava.science.PhysicalConstants .*;

public class Test {
    double atoms(double mols) {
        return AVOGADROS_NUMBER * mols;
    }
    ...
    // Many more uses of PhysicalConstants justify static import 
}
```

→ *인터페이스는 타입을 정의하는 용도로만 사용해야 한다. 상수 공개용 수단으로 사용하지 말자.*
