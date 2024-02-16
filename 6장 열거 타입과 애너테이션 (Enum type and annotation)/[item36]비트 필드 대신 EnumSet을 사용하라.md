열거한 값들이 집합으로 사용될 경우, 예전에는 각 상수에 서로 다른 2의 거듭제곱 값을 할당한 정수 열거 패턴을 사용해왔다.

# 비트 필드(bit field)

``` java
public class Text {
  public static final int STYLE_BOLD = 1 << 0; // 1 -> 0001
  public static final int STYLE_ITALIC = 1 << 1; // 2 -> 0010
  public static final int STYLE_UNDERLINE = 1 << 2; // 4 -> 0100
  public static final int STYLE_STRIKETHROUGH = 1 << 3; // 8 -> 1000

  // 매개변수 styles는 0개 이상의 STYLE_ 상수를 비트별 OR한 값이다.
  public void applyStyles(int styles) {...}
}
```

`text.applyStyles(STYLE_BOLD | STYLE_ITALIC);`

위와 같은 식으로 비트별 OR 연산을 사용해 여러 상수를 하나의 집합으로 모을 수 있으며, 이렇게 만들어진 집합을 비트 필드(bit field)라고 한다.

위의 예시에서 비트 필드

> `STYLE_BOLD | STYLE_ITALIC` -> 0011 -> 3
> 
> `STYLE_BOLD | STYLE_UNDERLINE` -> 0101 -> 5
> 
> ...

비트 필드를 사용하면 집합 연산을 효율적으로 수행할 수 있다.

---

## 비트 필드의 단점

- 해석하기 어렵다.

  3을 보고 `STYLE_BOLD | STYLE_ITALIC` 을 해석하기 어렵다.
  
- 비트 필드 하나에 녹아 있는 모든 원소를 순회하기 어렵다.

  비트 필드에 어떤 원소가 들어있는지 해석하기가 까다롭고 해석하여 어떤 원소들이 들어있는지 파악하더라고 순회하기 어렵다.
  
- 최대 몇 비트가 필요한지 미리 예측하여 적절한 타입(int, long)을 선택해야 한다.
  
  위에서는 4개의 상수만을 가졌지만 상수가 더욱 많아질 경우 자료형 선택이 까다로워진다.

---

# EnumSet

`EnumSet` 클래스를 이용하여 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현해준다.

원소가 총 64개 이하인 경우, `long` 타입 변수에 원소들을 표현하므로 비트 필드에 비견되는 성능을 보여준다.

![noneOf](https://github.com/NoSubject-Study/effective-java-study/assets/103320798/ab571303-2dcd-46a3-89d1-ccae459c20a7)

`EnumSet`의 경우 추상 클래스이며 원소의 개수에 따라 다른 구현체를 제공하는데,

원소의 개수가 64개 이하인 경우, 위의 사진 처럼 `RegularEnumSet`을 생성하는데,

![regularenumset](https://github.com/NoSubject-Study/effective-java-study/assets/103320798/144e3257-1649-4347-b4ba-3a90b949d404)

`elements` 변수가 `long` 타입으로 선언되어 있다.

---

## 열거 타입과 EnumSet을 사용할 경우

``` java
public class Text  {
  public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }

  // 어떤 Set을 넘겨도 되나, EnumSet이 가장 좋다.
  public void applyStyles(Set<Style> styles) {...}
}
```

`text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));`

위의 코드는 `EnumSet` 인스턴스를 사용하는 클라이언트 코드이다.

`EnumSet`은 다양한 기능의 정적 팩터리를 제공한다.

`EnumSet`을 사용하면 비트 필드를 사용했을 경우의 단점들을 볼 수 없다.

열거 타입의 인스턴스가 그대로 넘어오기 때문에 해석하기 편하며, 자료형을 `EnumSet`이 결정해주므로 자료형에 대해 고민할 필요가 없어진다.
