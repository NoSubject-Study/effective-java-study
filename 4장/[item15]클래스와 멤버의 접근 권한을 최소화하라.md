# Item15: Minimize the accessibility of classes and members

*클래스와 멤버의 접근 권한을 최소화하라*

---

> 잘 설계된 컴포넌트는 모든 내부 구현을 완벽히 숨겨, 구현과 API를 깔끔히 분리함 → *정보은닉(캡슐화)*
> 

## 정보 은닉(Information Hiding)과 캡슐화(Encapsulation)

- **정보 은닉(Information Hiding)**: 클래스의 내부 데이터와 구현 세부 사항을 외부에서 알 수 없도록 숨기는 것
- **캡슐화(Encapsulation)**: 객체의 상태(state)와 행동(behavior)을 하나의 캡슐로 묶는 것

```java
public class EncapsulatedObject {
    private int hiddenData;

    public int getHiddenData() {
        return hiddenData;
    }
}
```

## 정보 은닉의 장점

- 시스템 개발 속도 향상: 여러 컴포넌트를 병렬로 개발 가능
- 시스템 관리 비용 절감: 각 컴포넌트를 더 빨리 이해하여 디버깅이나 교체 부담 감소
- 성능 최적화 도움: 다른 컴포넌트에 영향을 주지 않고 해당 컴포넌트만 최적화 가능
- 소프트웨어 재사용성 증가: 독립적으로 동작하는 컴포넌트는 다른 환경에서도 재사용 가능
- 큰 시스템 제작 난이도 감소: 시스템 전체가 완성되지 않아도 개별 컴포넌트의 동작 검증 가능

## 접근 제어 메커니즘(Access Control Mechanism)

- Java에서 정보 은닉을 위해 사용되는 다양한 장치 중 하나[JLS, 6.6](https://docs.oracle.com/javase/specs/jls/se7/html/jls-6.html#jls-6.6)
- 각 요소의 접근성은 그 요소가 선언된 위치와 접근 제한자(`private`, `protected`, `public`)로 정해짐→이 접근 제한자를 제대로 활용하는 것이 정보 은닉의 핵심

## 어떻게 설계해야 하는가?

> 기본 원칙: **모든 클래스와 멤버의 접근성을 가능한 한 좁혀야 한다.
→** 소프트웨어가 올바로 동작하는 한 항상 가장 낮은 접근 수준을 부여해야 함
> 

### 톱레벨 클래스(Non-nested Class), 인터페이스의 접근 제한

- 접근 수준: `package-private`, `public`
    - `package-private`: 해당 패키지 안에서만 이용 가능(접근 제한자를 명시하지 않았을 때 기본 적용되는 수준)
    - `public`: 공개 API로 선언
    
- 패키지 외부에서 사용할 일이 없다면 `package-private`으로 설정 → 내부 구현이 되어 클라이언트에 영향 없이 다음 릴리스에서 수정, 교체, 제거 가능
- `package-private` 수준의 톱레벨 클래스나 인터페이스가 오직 한 개의 클래스에서만 사용되면, 이를 사용하는 클래스 안에 `private static`으로 중첩 가능(Item 24)

### 멤버(필드, 메서드, 중첩 클래스, 중첩 인터페이스)의 접근성

- 접근 수준: `private`, `package-private`, `protected`, `public`
    - `private`: 멤버를 선언한 톱레벨 클래스에서만 접근 가능
    - `package-private`: 멤버가 소속된 패키지 안의 모든 클래스에서 접근 가능(접근 제한자를 명시하지 않았을 때 기본 적용되는 수준)
    - `protected`: `package-private`의 범위를 포함하며, 이 멤버를 선언한 클래스의 하위 클래스에서도 접근 가능
    - `public`: 모든 곳에서 접근 가능

- 클래스의 공개 API를 세심히 설계한 후, 그 외 모든 멤버를 `private`으로 설정
- 같은 패키지의 다른 클래스가 멤버에 접근해야 하는 경우에만 `private` 제어자를 `package-private`으로 풀어줌(자주 하게 된다면 컴포넌트 분해를 다시 고려)
- `private`과 `package-private` 멤버는 해당 클래스의 구현에 해당하므로, 보통 공개 API에 영향을 주지 않음(예외: *Serializable*을 구현한 클래스, Item 86, 87)
- 가능한 `protected` 멤버의 수는 적게(공개 API가 되어 영원히 지원돼야 함, Item 19)

### 멤버 접근성을 좁히지 못하게 방해하는 제약

- 상위 클래스의 메서드를 오버라이드할 때, 그 접근 수준을 상위 클래스에서보다 좁게 설정할 수 없음[JLS, 8.4.8.3](https://docs.oracle.com/javase/specs/jls/se8/html/jls-8.html#jls-8.4.8.3)
- 상위 클래스의 인스턴스는 하위 클래스의 인스턴스로 대체해 사용할 수 있어야 함(*리스코프 치환 원칙(Liskov Substitution Principle)*, Item 10) → 지켜지지 않으면 컴파일 에러 발생

```java
public class SuperClass {
    public void someMethod() { }
}

public class SubClass extends SuperClass {
    // COMPILE ERROR !!!: 상위 클래스의 메서드보다 더 제한적인 접근 수준을 가짐
    private void someMethod() { }

}
```

- 클래스가 인터페이스를 구현하는 경우, 클래스는 인터페이스가 정의한 모든 메서드를 `public`으로 선언해야 함(인터페이스의 메서드가 기본적으로 `public`이기 때문)

```java
public interface SomeInterface {
    void someMethod();
}

public class SomeClass implements SomeInterface {
    // COMPILE ERROR !!!: public이 아님
    private void someMethod() { }
}
```

## 테스트 목적으로 접근 범위를 넓히지 않기

- 적당한 수준까지는 괜찮음(`public` 클래스의 멤버를 `private` → `package-private`)
- 이외에 접근성을 더욱 넓히는 것은 허용되지 않음(클래스, 인터페이스, 멤버를 패키지의 공개 API의 일부로 만드는 것)
- 테스트 코드를 테스트 대상과 같은 패키지에 두어서 `package-private` 요소에 접근 가능하므로, 불필요하게 접근 범위를 넓힐 필요가 없음

## public 클래스의 인스턴스 필드 접근 제한

> **public 클래스의 인스턴스 필드는 되도록 public이 아니어야 한다**(Item 16)**.**
> 

public 클래스의 인스턴스 필드가 `public`이라면?

- 필드에 담을 수 있는 값을 제한할 수 없음: 불변식을 보장할 수 없음(외부에서 필드 값 변경 가능)
    - 불변식(Invariant): 객체가 생성된 후에도 그 값이 변하지 않는 특성(Ex: 사람은 태어나고 출생 일자가 바뀌지 않는다…)
- **일반적으로 스레드 안전하지 않음(not thread-safe)**: 여러 개의 스레드가 동시에 필드를 변경하면, 어떤 값이 저장될지 예측할 수 없음
- 필드가 `final`이고 불변 객체를 참조하는 방법을 사용해도 리팩터링이 어려워지는 문제 발생
- 정적(`static`) 필드에도 똑같이 적용
    - 예외: 추상화에 꼭 필요한 구성요소로써의 상수라면 `public static final`로 사용해도 괜찮음
        - 이름은 관례상 대문자 알파벳으로, 각 단어 사이에 밑줄(_)을 넣음(Item 68)
        - 반드시 기본 타입 값이나 불변 객체를 참조해야 함(Item 17)

## public 배열의 보안상 허점

> **클래스에서 `public static final` 배열 필드를 두거나 이 필드를 반환하는 접근자 메서드를 제공해서는 안 된다. →** 클라이언트에서 배열의 내용을 수정할 수 있음
> 
- 길이가 0이 아닌 배열은 항상 가변적(mutable)
- `final` 키워드를 사용하더라도 인스턴스가 참조하는 주소 값만 변경되지 않고, 참조하는 객체의 내용은 변경 가능

```java
// Potential security hole!
public static final Thing[] VALUES = { ... };
```

### 해결책

1. 배열을 `private`으로 만들고 `public` 불변 리스트를 추가

```java
private static final Thing[] PRIVATE_VALUES = { ... };
public static final List<Thing> VALUES =
	Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
```

1. 배열을 `private`으로 만들고 그 복사본을 반환하는 `public` 메서드를 추가(방어적 복사)

```java
private static final Thing[] PRIVATE_VALUES = { ... };
public static final Thing[] values() {
	return PRIVATE_VALUES.clone();
}
```

## 모듈 시스템

- 모듈(module): 패키지들의 묶음
- Java 9에서 도입되면서 두 가지 암묵적 접근 수준이 추가됨
    - 모듈은 자신에 속하는 패키지 중 공개(export)할 것들을 (관례상 *module-info.java* 파일에) 선언함
    - 모듈 내 접근성: 모듈 내부에서 모든 패키지가 서로 접근 가능
    - 모듈 간 접근성: 모듈 외부에서 `exports`로 선언된 패키지의 `public` 멤버만 접근 가능
- 아직 모듈 개념이 널리 사용되지 않아, 꼭 필요한 경우가 아니라면 사용 자제
