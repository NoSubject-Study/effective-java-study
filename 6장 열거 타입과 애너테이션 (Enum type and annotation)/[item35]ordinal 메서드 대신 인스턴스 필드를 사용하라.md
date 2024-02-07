# Ordinal 메서드

열거 타입은 해당 상수가 그 열거 타입에서 몇 번째 위치인지를 반환하는 ordinal 이라는 메서드를 제공한다.

열거 타입 상수와 연결된 정숫값이 필요하면 ordinal 메서드를 이용할 수 있다.

---

# 단점

다음 코드는 합주단의 종류를 연주자가 1명인 솔로(solo)부터 디텍트(dectet)까지 정의한 열거 타입이다.

``` java
// ordinal을 잘못 사용한 예 - 따라 하지 말 것!
public enum Ensemble {
  SOLO, DUET, TRIO, QUARTET, QUINTET,
  SEXTET, SEPTET, OCTET, NONET, DECTET;

  public int numberOfMusicians() {
    return ordinal() + 1;
  }
}
```

- 상수 선언 순서를 바꾸면 numberOfMusicians가 오동작한다.
- 이미 사용 중인 정수와 값이 같은 상수는 추가할 방법이 없다.
  
  8중주(octet) 상수가 이미 있으니 복4중주(double quartet)는 추가할 수 없다.
- 값을 중간에 비워둘 수도 없다.
  
  12명이 연주하는 3중 4중주(triple quartet)를 추가할 때 중간에 11명으로 구성된 연주를 일컫는 이름이 없다.
  
  추가하려면 더미 상수를 같이 추가해야만 한다.

---

# 해결책

열거 타입 상수에 연결된 값은 ordinal 메서드로 얻지 말고, 인스턴스 필드에 저장하자.

``` java
public enum Ensemble {
  SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
  SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8),
  NONET(9), DECTET(10), TRIPLE_QUARTET(12);

  private final int numberOfMusicians;

  Ensemble(int size) {
    this.numberOfMusicians = size;
  }

  public int numberOfMusicians() {
    return numberOfMusicians;
  }
}
```

## ordinal 메서드의 용도

Enum의 API 문서

> 대부분 프로그래머는 이 메서드를 쓸 일이 없다.
> 이 메서드는 EnumSet과 EnumMap 같이 열거 타입 기반의 범용 자료구조에 쓸 목적으로 설계되었다.

<img width="2886" alt="Screen Shot 2024-02-07 at 8 45 44 AM" src="https://github.com/NoSubject-Study/effective-java-study/assets/103320798/32301b56-2347-4158-86c6-d3fbf36fb743">
