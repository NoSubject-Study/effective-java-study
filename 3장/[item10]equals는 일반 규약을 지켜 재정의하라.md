
# item-09 equals는 일반 규약을 지켜 재정의하라

## equals 메소드 오버라이딩
Object 클래스는 모든 클래스의 최상위 부모로, equals, hashCode, toString, clone, finalize 같은 nonfinal 메소드들을 갖고 있다,
이 메소드들은 다른 클래스들이 올바르게 동작할 수 있도록 설계된 명시적인 규약(general contracts)을 가지고 있으며, 이 메소드들을 오버라이딩 할 때는 해당 규약들을 지켜야만 올바르게 작동한다.

## equals 메소드 오버라이딩 시 고려사항
저자는 equals 메소드를 오버라이딩하지 않는 것이 가장 안전한 방법일 수 있으며, 특정 조건들 하에 이 방식이 적합하다고 한다. 아래는 오버라이딩이 필요하지 않은 경우다.(재정의 안하는게 최선)

- 클래스의 각 인스턴스가 고유하게 유일한 경우 (예: Thread 클래스, 값의 표현이 아니라 동작하는 개체이다. 이미 Object의 eqauls에 딱 맞게 구현되어 있음.)
- 클래스가 논리적 동치성을 따질 필요가 없는 경우(정규표현식의 설계 경우)
- 상위 클래스가 이미 equals를 오버라이딩했고, 하위 클래스에도 적합한 경우
- 클래스가 private 또는 package-private이며, equals 메소드가 절대 호출되지 않을 것이라고 확신하는 경우

## equals 메소드의 규약
이번 규약을 읽으면서 논리학을 다시 배우는 것 같았다.

equals 메소드를 오버라이드할 때는 다음과 같은 규약을 준수해야 한다(Object 클래스 명세에 그대로 적혀있음)

- 반사성(Reflexivity): 모든 비 null 참조 값 x에 대해, x.equals(x)는 true를 반환해야 한다.

- 대칭성(Symmetry): 모든 비 null 참조 값 x와 y에 대해, x.equals(y)가 true를 반환하면 y.equals(x)도 true를 반환해야 한다.

- 추이성(Transitivity): 모든 비 null 참조 값 x, y, z에 대해, x.equals(y)가 true이고 y.equals(z)가 true이면, x.equals(z)도 true여야 한다.

- 일관성(Consistency): 모든 비 null 참조 값 x와 y에 대해, equals 비교에 사용된 정보가 수정되지 않는 한 x.equals(y)의 여러 호출은 일관되게 true 또는 false를 반환해야 한다.

- null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false를 반환해야 한다.

## 규약을 어기는 경우

대소문자 구별을 하지 않은 CaseInsensitiveString 클래스의 예시를 보자

```java
public final class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }

    // 대칭성 위반하는 equals 구현
    @Override 
    public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString)
            return s.equalsIgnoreCase(((CaseInsensitiveString) o).s);
        if (o instanceof String)
            return s.equalsIgnoreCase((String) o);
        return false;
    }
}
```
이 코드에서 CaseInsensitiveString 인스턴스는 String과 equals 비교를 시도할 때 대칭성을 위반한다. CaseInsensitiveString 인스턴스는 String을 "동등하다"고 간주할 수 있지만, String의 equals 메소드는 CaseInsensitiveString을 인식하지 못해 false를 반환한다.

해결법은? CaseInsensitiveString 인스턴스들만 비교하도록 equals 메소드를 단순화한다.

```java
@Override public boolean equals(Object o) {
    return o instanceof CaseInsensitiveString &&
           ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
}
```
이 방식으로 equals 메소드를 구현하면 대칭성, 전이성, 반사성을 모두 만족시키며, 논리적 동치성을 따질 수 있다.

## 상속 대신 컴포지션을 사용하여 equals 규약 지키기
ColorPoint가 Point를 extends하는 대신에, ColorPoint 내에 Point 필드를 private으로 가지고, 해당 위치의 Point 뷰를 반환하는 public  메소드를 제공하는 방식이다.

```java

public class ColorPoint {
    private final Point point;
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        this.point = new Point(x, y);
        this.color = Objects.requireNonNull(color);
    }
//뷰 메소드
    public Point asPoint() {
        return this.point;
    }

    @Override 
    public boolean equals(Object o) {
        if (!(o instanceof ColorPoint)) return false;
        ColorPoint cp = (ColorPoint) o;
        return cp.point.equals(this.point) && cp.color.equals(this.color);
    }
}
```
위의 코드는 ColorPoint 인스턴스가 같은 위치에 있는 Point와 동일한 색상을 가지고 있는지 비교함으로써 equals 규약들을 만족시키는 것을 보여준다.상속 대신 컴포지션을 사용하는 방식(item-18)은 equals 계약의 대칭성, 추이성, 일관성 요구 사항을 위반하지 않으면서 값을 추가할 수 있게 한다.



## equals 메소드 오버라이딩이 잘 이루어진 예시
- 자기 참조 확인: == 연산자를 사용하여 입력된 객체가 현재 객체와 동일한 참조인지 확인한다.
- 인스턴스 타입 확인: instanceof 연산자를 사용하여 입력된 객체가 올바른 타입인지 확인한다.
- 필드별 값 비교: 클래스 내의 "중요한" 필드들이 입력된 객체의 해당 필드와 일치하는지 비교한다.
- hashCode 오버라이딩: equals를 오버라이딩할 때는 반드시 hashCode도 오버라이딩해야 한다. 이는 equals가 true를 반환하는 두 객체가 동일한 해시 코드를 반환해야 하는 규약 때문이다.

잘 구현된 예시는 아래와 같다.
```java
public final class PhoneNumber {
    private final short areaCode, prefix, lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = rangeCheck(areaCode, 999, "area code");
        this.prefix = rangeCheck(prefix, 999, "prefix");
        this.lineNum = rangeCheck(lineNum, 9999, "line num");
    }

    private static short rangeCheck(int val, int max, String arg) {
        if (val < 0 || val > max)
            throw new IllegalArgumentException(arg + ": " + val);
        return (short) val;
    }

    @Override
    public boolean equals(Object o) {
        if (o == this) return true;
        if (!(o instanceof PhoneNumber)) return false;
        PhoneNumber pn = (PhoneNumber) o;
        return pn.lineNum == lineNum && pn.prefix == prefix && pn.areaCode == areaCode;
    }
}
```
이 코드는 PhoneNumber 클래스의 인스턴스가 논리적으로 같은지 비교하기 위해 equals 메소드를 오버라이딩하는 방법을 보여준다. 대칭성, 추이성, 일관성을 잘 지키는 것을 알 수 있다.(hashCode는 생략)