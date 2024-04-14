*적시에 방어적 복사본을 만들라*

---

자바는 버퍼 오버런, 배열 오버런, 와일드 포인터 같은 메모리 충돌 오류에서 안전

자바로 작성한 클래스는 시스템의 다른 부분에서 무슨 짓을 하든 불변식이 지켜짐

하지만 다른 클래스로부터의 침범을 아무런 노력 없이 다 막을 수 있는 건 아니다.

→ **클라이언트가 여러분의 불변식을 깨뜨리려 혈안이 되어 있다고 가정하고 방어적으로 프로그래밍해야 한다.**

- 악의적인 공격이나 실수로 클래스를 오작동하게 만들 수 있음
- 적절치 않은 클라이언트로부터 클래스를 보호하는 데 충분한 시간을 투자해야 함

## 클래스 내부를 수정하도록 허락하는 경우

어떤 객체든 그 객체의 허락 없이는 외부에서 내부를 수정하는 일은 불가능하지만, 주의를 기울이지 않으면 자기도 모르게 내부를 수정하도록 허락하는 경우가 발생

Ex. 기간(period)을 표현하는  클래스 - 불변식을 지키지 못했다.

```java
// Broken "immutable" time period class
public final class Period {
    private final Date start;
    private final Date end;

    /**
     * @param start the beginning of the period
     * @param end   the end of the period; must not precede start 
     * @throws IllegalArgumentException if start is after end
     * @throws NullPointerException if start or end is null
     */
    public Period(Date start, Date end) {
        if (start.compareTo(end) > 0)
            throw new IllegalArgumentException(start + " after " + end);
        this.start = start;
        this.end = end;
    }

    public Date start() {
        return start;
    }

    public Date end() {
        return end;
    }
    ...  // Remainder omitted
}
```

→ 불변처럼 보이고, 시작 시각이 종료 시각보다 늦을 수 없다는 불변식이 무리 없지 지켜질 것 같지만, `Date`가 가변이므로 어렵지 않게 불변식을 깨질 수 있음

```java
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end); 
end.setYear(78); // Modifies internals of p!
```

- `Period` 클래스의 생성자는 `start`가 `end`보다 이후인 경우를 방지하는 로직을 포함하고 있지만, 생성자를 통과한 후에는 `start`와 `end` 객체 자체가 외부에서 변경될 수 있어 불변식이 지켜지지 않음
- 자바 8 이후 단순히 `Date` 대신 불변(Item 17)인 `Instant`를 사용(혹은 `LocalDateTime` 이나 `ZonedDateTime` 사용)하여 해결할 수 있음 
→ **`Date`는 낡은 API이니 새로운 코드를 작성할 때는 더 이상 사용하면 안된다.**
- 하지만 가변인 낡은 값 타입을 사용하던 시절이 너무 길었던 탓에 여전히 많은 API와 내부 구현에 그 잔재가 남아 있음

## 방어적 복사본으로 해결

외부 공격으로부터 `Period` 인스턴스 내부를 보호하려면 생성자에서 받은 가변 매개변수 각각을 방어적으로 복사(defensive copy)한 후, `Period` 인스턴스 안에서는 원본이 아닌 복사본을 사용

```java
// Repaired constructor - makes defensive copies of parameters
public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end   = new Date(end.getTime());
    if (this.start.compareTo(this.end) > 0)
        throw new IllegalArgumentException(this.start + " after " + this.end);
}
```

방어적 복사를 이용한 생성자를 사용하면 앞서의 공격은 더 이상 위협이 되지 않음

1. **매개 변수의 유효성을 검사(Item 49)하기 전에 방어적 복사본을 만들고, 이 복사본으로 유효성을 검사한 점에 주목하자.** 
    - 멀티스레딩 환경이라면 원본 객체의 유효성을 검사한 후 복사본을 만드는 그 찰나의 취약한 순간에 다른 스레드가 원본 객체를 수정할 위험 존재 → TOCTOU 공격(검사시점/사용시점(time-of-check-/time-of-use) 공격)
    - 방어적 복사를 매개 변수 유효성 검사 전에 수행하면 이런 위험에서 해방될 수 있음
2. 방어적 복사를 수행할 때 **매개변수가 제3자에 의해 확장될 수 있는 타입이라면 방어적 복사본을 만들 때 `clone`을 사용해서는 안 된다.**
    - `Date`는 `final`이 아니므로 `clone`이 `Date`가 정의한 게 아닐 수 있음 → `clone`이 악의를 가진 하위 클래스의 인스턴스를 반환할 위험 존재
    - 예컨대 이 하위 클래스는 start와 end 필드 참조를 private 정적 리스트에 담아뒀다가 공격자에게 이 리스트에 접근하는 길을 열어줄 수 있다.
    
    ```java
    // 악의적인 Date 하위 클래스
    public class MaliciousDate extends Date {
        // start와 end 필드 참조를 담을 private 정적 리스트
        private static final List<MaliciousDate> maliciousInstances = new ArrayList<>();
    
        public MaliciousDate() {
            super();
        }
        public MaliciousDate(long date) {
            super(date);
        }
    
        // clone 메서드 오버라이드
        @Override
        public Object clone() {
            MaliciousDate clone = new MaliciousDate(this.getTime());
            // 복사된 인스턴스를 리스트에 추가
            maliciousInstances.add(clone);
            return clone;
        }
    
        // 공격자가 액세스할 수 있는 메서드
        public static List<MaliciousDate> getMaliciousInstances() {
            return maliciousInstances;
        }
    }
    ```
    

### 가변 필드의 방어적 복사본을 반환

생성자를 수정하면 앞서의 공격은 막아낼 수 있지만, `Period` 인스턴스는 아직도 변경 가능함 ← 접근자 메서드가 내부의 가변 정보를 직접 드러내기 때문

```java
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end); 
p.end().setYear(78); // Modifies internals of p!
```

`Period` 인스턴스 생성 후, 이 인스턴스의 `end` 날짜를 가져와서 (`p.end()`) 그 연도를 변경(`setYear()`)함으로써, `Period` 인스턴스의 내부 상태를 변경

위 공격을 막아내기 위해 단순히 접근자가 가변 필드의 방어적 복사본을 반환하면 해결할 수 있음 → 완벽한 불변

```java
public Date start() {
    return new Date(start.getTime());
}
public Date end() {
    return new Date(end.getTime());
}
```

- `Period` 자신 말고는 가변 필드에 접근할 방법이 없음(모든 필드가 객체 안에 완벽하게 캡슐화)
- 생성자와 달리 접근자 메서드에서는 방어적 복사에 `clone`을 사용 가능
    - `Period`가 가지고 있는 `Date` 객체는 `java.util.Date`임이 확실하기  때문(신뢰할 수 없는 하위 클래스가 아님)
    - 그렇더라도 인스턴스를 복사하는 데는 일반적으로 생성자나 정적 팩터리를 사용하는 것이 좋음

### 매개변수 방어적 복사를 통한 객체 무결성 유지

매개변수를 방어적으로 복사하는 목적은 불변 객체를 만드는건 뿐만 아니라, 클래스 내부에서 클라이언트가 제공한 객체의 참조를 안전하게 관리하기 위함

- 메서드든 생성자든 클라이언트가 제공한 객체의 참조를 내부의 자료구조에 보관해야 할때면 항시 그 객체가 잠재적으로 변경될 수 있는지를 생각해야 함
- 변경될 수 있는 객체라면 그 객체가 클래스에 넘겨진 뒤 임의로 변경되어도 그 클래스가 동작할지를 따져야 함
- 확신할 수 없다면 복사본을 만들어 저장
- Ex. 예컨대 클라이언트가 건네준 객체를 내부의 `Set` 인스턴스에 저장하거나 `Map` 인스턴스의 키로 사용한다면, 추후 그 객체가 변경될 경우 객체를 담고 있는 `Set` 혹은 `Map` 불변식이 깨짐

내부 객체를 클라이언트에 건네주기 전에 방어적 복사본을 만드는 이유도 마찬가지

- 가변인 내부 객체를 클라이언트에 반환할 때는 반드시 심사숙고 해야함
- Ex. 길이가 1 이상인 배열은 무조건 가변 → 내부에서 사용하는 배열을 클라이언트에 배열을 반환할 때는 항상 방어적 복사를 수행해야 함

⇒ 되도록 불변 객체들을 조합해 객체를 구성해야 방어적 복사를 할 일이 줄어든다.

## 방어적 복사의 선택적 적용과 통제권 이전의 중요성

방어적 복사에는 성능 저하가 따르고 항상 쓸 수 있는 것이 아니다. 

- 호출자가 컴포넌트 내부를 수정하지 않으리라 확신하면 생략할 수 있음
- 이러한 상황이라도 호출자에서 해당 매개변수나 반환값을 수정하지 말아야 함을 문서화 하는 것이 좋음

다른 패키지에서 사용한다고 해서 넘겨받은 가변 매개변수를 항상 방어적으로 복사해 저장해야 하는 것은 아니다.

- 때로는 메서드나 생성자의 매개변수로 넘기는 행위가 그 객체의 통제권을 명백히 이전함을 뜻하기도 함.
- 통제권을 이전하는 메서드를 호출하는 클라이언트는 해당 객체를 더 이상 직접 수정하는 일이 없다고 약속해야 함
- 클라이언트가 건네주는거면 가변 객체의 통제권을 넘겨받는다고 기대하는 메서드나 생성자에게도 그 사실을 문서화 해야함

통제권을 넘겨받기로 한 메서드나 생성자를 가진 클래스들은 악의적인 클라이언트의 공격에 취약하다.

- 방어적 복사를 생략해도 되는 상황은 해당 클래스와 그 클라이언트가 상호 신뢰할 수 있을  때, 혹은 불변식이 깨지더라도 그 영향이 오직 호출한 클라이언트로 국한할 때로 한정해야 함

## 정리

- 클래스가 클라이언트로부터 받는 혹은 클라이언트로 반환하는 구성요소가 가변이라면 그 요소는 반드시 방어적으로 복사해야 한다.
- 복사 비용이 너무 크거나 클라이언트가 그 요소를 잘못 수정할 일이 없음을 신뢰한다면 방어적 복사를 수행하는 대신 해당 구성요소를 수정했을 때의 책임이 클라이언트에 있음을 문서에 명시하도록 하자.
