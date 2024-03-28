# 매개변수 검사를 제대로 하지 못하면 생기는 문제

매개변수의 값 제약은 문서화해야 하며, 메서드 몸체가 시작되기 전에 검사해야 한다.

매개변수 검사를 제대로 하지 못하면 몇 가지 문제가 생길 수 있다.

1. 메서드가 수행되는 중간에 모호한 예외를 던지며 실패할 수 있다.
2. 메서드가 잘 수행되지만 잘못된 결과를 반환할 수 있다.
3. 메서드는 문제없이 수행됐지만, 어떤 객체를 이상한 상태로 만들어놓아서 미래의 알 수 없는 시점에 이 메서드와는 관련 없는 오류를 낼 수 있다.

---

# 문서화하기

public과 protected 메서드는 매개변수 값이 잘못됐을 때 던지는 예외를 문서화해야 한다.

보통 `IllegalArgumentException`, `IndexOutOfBoundsException`, `NullPointerException` 중 하나가 된다.

매개변수의 제약을 문서화한다면 그 제약을 어겼을 때 발생하는 예외도 함께 기술해야 한다.

``` java
/**
 * (현재 값 mod m) 값을 반환한다. 이 메서드는
 * 항상 음이 아닌 BigInteger를 반환한다는 점에서 remainder 메서드와 다르다.
 *
 * @param m 계수(양수여야 한다.)
 * @return 현재 값 mod m
 * @throws ArithmeticException m이 0보다 작거나 같으면 발생한다.
 */
public BigInteger mod(BigInteger m) {
  if (m.signum() <= 0)
    throw new ArithmeticException("계수(m)는 양수여야 합니다. " + m);
  ... // 계산 수행
}
```

m이 null일 경우 m.signum() 호출 때 NullPointerException을 던진다.

하지만 메서드 설명에 기술되지 않았다.

-> BigInteger 클래스 수준에서 기술했기 때문

클래스 수준 주석은 그 클래스의 모든 public 메서드에 적용되므로 각 메서드에 기술하는 것보다 깔끔하다.

`@Nullable`과 같은 애너테이션을 사용할 수 있지만 표준적인 방법이 아니다.

---

# Objects

자바 7에 추가된 java.util.Objects.requireNonNull로 인해 null 검사를 수동으로 하지 않아도 된다.

원하는 예외 메시지도 지정할 수 있고, 입력을 그대로 반환하므로 값을 사용하는 동시에 null 검사를 수행할 수 있다.

``` java
this.strategy = Objects.requireNonNull(strategy, "전략");
```

자바 9에서는 범위 검사 기능도 추가됐다.

`checkFromIndexSize`, `checkFromToIndex`, `checkIndex`

null 검사 메서드만큼 유연하지는 않다.

예외 메시지를 지정할 수 없고, 리스트와 배열 전용으로 설계됐다.

---

# 단언문(assert)

공개되지 않은 메서드라면 오직 유효한 값만이 메서드에 넘겨질 것을 보증할 수 있고, 그렇게 해야 한다.

public이 아닌 메서드라면 단언문(assert)을 사용해 유효성을 검증할 수 있다.

``` java
private static void sort(long a[], int offset, int length) {
  assert a != null;
  assert offset >= 0 && offset <= a.length;
  assert length >= 0 && length <= a.length - offset;
  ... // 계산 수행
}
```

일반적인 유효성 검사와 다르다.

실패하면 `AssertionError`를 던진다.

런타임에 아무런 효과도, 아무런 성능 저하도 없다. (단, -ea 혹은 --enableassertions 플래그를 설정하면 영향을 준다)

---

# 나중에 사용하는 매개변수

``` java
static List<Integer> intArrayAsList(int[] a) {
  Objects.requireNonNull(a);
  return new AbstractList<>() {
    ...
  };
}
```

입력받은 int 배열의 List 뷰를 반환하는 메서드이다.

`Objects.requireNonNull(a)`을 이용해 null 검사를 수행한다.

이 검사를 생략했다면 클라이언트가 돌려받은 List를 사용할 때 `NullPointerException`이 발생한다.

-> 디버깅이 어려워진다.

생성자 매개변수의 유효성 검사는 클래스 불변식을 어기는 객체가 만들어지지 않게 하는 데 꼭 필요하다.

---

# 예외

메서드 몸체 실행 전에 매개변수 유효성 검사를 해야 한다는 규칙에 예외가 있다.

검사 비용이 지나치게 높거나 실용적이지 않을 경우, 계산 과정에서 암묵적으로 검사가 수행될 때다.

`Collections.sort(List)` 메서드의 경우, 리스트 안의 객체들은 비교될 수 있어야 하며, 정렬 과정에서 비교가 이뤄진다.

비교될 수 없는 객체가 주어질 경우 `ClassCastException`을 던지게 된다.

따라서 비교될 수 있는지 검사해봐야 별다른 실익이 없다.

계산 과정에서 필요한 유효성 검사가 이뤄지지만 실패했을 때 잘못된 예외를 던지기도 한다.

즉, 발생한 예외와 API 문서에서 던지기로 한 예외가 다를 수 있다는 뜻이다.

이럴 경우 예외 번역(exception translate) 관용구를 사용하여 API 문서에 기재된 예외로 번역해줘야 한다.

---

이번 아이템을 "매개변수에 제약을 두는 게 좋다"고 해석해서는 안 된다.

메서드는 최대한 범용적으로 설계해야 하므로, 매개변수로 받은 값으로 무언가 제대로 된 일을 할 수 있다면 제약은 적을수록 좋다.
