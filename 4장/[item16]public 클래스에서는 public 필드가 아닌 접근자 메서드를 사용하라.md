# Item16: In public classes, use accessor methods, not public fields

*public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라*

---

## 인스턴스 필드만 모아두는 클래스

*데이터 필드에 직접 접근이 가능해 캡슐화의 이점을 제공하지 못함(Item 15)*

```java
// Degenerate classes like this should not be public!
class Point {
    public double x;
    public double y;
}
```

- API 변경 없이 내부 표현을 바꿀 수 없음: 필드의 자료형, 구조를 변경하려면 클래스를 사용하는 측의 코드도 변경해야 함
- 불변식이 보장되지 않음: 클래스 외부에서 필드의 값을 임의로 변경 가능
- 외부에서 필드에 접근할 때 부수 작업 수행 불가: 값을 변경하거나 조회하는 것 이상의 작업을 하기 어려움(Ex: 필드의 값이 변경될 때마다 로그 남기기)

## 접근자와 변경자(mutator) 메서드 활용

필드들을 모두 `private`으로 바꾸고 `public` 접근자(getter)와 변경자(setter) 메서드 추가 → 캡슐화

```java
// Encapsulation of data by accessor methods and mutators
class Point {
    private double x;
    private double y;
    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }
    public double getX() { return x; }
    public double getY() { return y; }
    public void setX(double x) { this.x = x; }
    public void setY(double y) { this.y = y; }
}
```

**패키지 바깥에서 접근할 수 있는 클래스**(`public`): **접근자를 제공**하여 클래스 내부 표현을 언제든 바꿀 수 있는 유연성을 얻을 수 있음

## package-private 클래스, private 중첩 클래스

`**package-private` 클래스와 `private` 중첩 클래스는 필드를 노출해도 문제 없음**

### package-private 클래스

```java
// In the same package
class PackagePrivateClass {
    // These fields are exposed within the package
    double x;
    double y;
}
```

- 같은 패키지 내에서만 접근 가능
- 클래스의 내부 표현을 변경해야 하는 경우에도 같은 패키지 내의 코드만 수정하면 됨

### private 중첩 클래스

```java
public class OuterClass {
    // Inner class is private
    private class InnerClass {
        // These fields are exposed within the OuterClass
        double x;
        double y;
    }
}
```

- 바깥 클래스에서만 접근 가능
- 중첩 클래스의 내부 표현을 변경해야 하는 경우에도 외부 클래스의 코드만 수정하면 됨

## 자바 플랫폼 라이브러리에서 필드를 노출한 사례

- 자바 플랫폼 라이브러리에도 `public` 클래스의 필드를 직접 노출한 사례가 있음(Ex. `java.awt`의 `Point`, `Dimension` 클래스) → 좋지 않은 구현
- 불변 필드를 노출한`public` 클래스: 불변식은 보장할 수 있지만 여전히 API를 변경없이 내부 구현을 바꿀 수 없고 외부에서 필드에 접근할 때 부수 작업 수행 불가

```java
// Public class with exposed immutable fields - questionable
    public final class Time {
        private static final int HOURS_PER_DAY = 24;
        private static final int MINUTES_PER_HOUR = 60;
        public final int hour;
        public final int minute;
        public Time(int hour, int minute) {
            if (hour < 0 || hour >= HOURS_PER_DAY)
                throw new IllegalArgumentException("Hour: " + hour);
            if (minute < 0 || minute >= MINUTES_PER_HOUR)
                throw new IllegalArgumentException("Min: " + minute);
            this.hour = hour;
            this.minute = minute;
        }
        ... // Remainder omitted
    }
```
