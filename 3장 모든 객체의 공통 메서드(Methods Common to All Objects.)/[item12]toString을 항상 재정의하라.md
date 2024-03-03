## item-12 toString을 항상 재정의하라


- 간결하면서 사람이 읽기 쉬운 형태의 유익한 정보를 반환해야 한다.
- 모든 하위 클래스에서 이 메서드를 재정의하라

재정의 하지 않는다면?
아래와 같은 반환 값을 준다.

```
내 전화번호: item12.PhoneNumber@b501c
```
단순한 `클래스_이름@16진수로_표시한_해시코드`의 형태를 반환한다.

이보다는 클래스의 목적에 맞게 전화번호를 직접 알려주는 형태를 보여주는 것이 훨씬 유익할 것이다.

왜냐하면 println, printf, 문자열 연결 연산자(+), assert 구문에 넘길 때, 디버거가 객체 출력할 때 자동으로 불리기 때 자동으로 불리기 때문에 toString을 잘 구현해 놓아야 훨씬 사용하기도 즐겁고 디버깅하기 쉽다.


따라서 toString을 아래와 같이 오버라이딩하여 사용한다.
```java
package item12;

import java.util.Objects;

class PhoneNumber {
    private final short areaCode;
    private final short prefix;
    private final short lineNumber;

    public PhoneNumber(int areaCode, int prefix, int lineNumber) {
        this.areaCode = (short) areaCode;
        this.prefix = (short) prefix;
        this.lineNumber = (short) lineNumber;
    }

    @Override
    public String toString() {
        return String.format("%03d-%03d-%04d", areaCode, prefix, lineNumber);
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        PhoneNumber that = (PhoneNumber) o;
        return areaCode == that.areaCode &&
                prefix == that.prefix &&
                lineNumber == that.lineNumber;
    }

    @Override
    public int hashCode() {
        return Objects.hash(areaCode, prefix, lineNumber);
    }

}

public class PhoneNumberExample {
    public static void main(String[] args) {
        PhoneNumber myNumber = new PhoneNumber(707, 867, 5309);
        System.out.println("내 전화번호: " + myNumber);
    }
}
```

```
내 전화번호: 707-867-5309
```
결과는 위와 같다.

## 객체가 거대하거나 문자열로 표현하기 어려울 경우

객체가 가진 주요 정보를 모두 반환하는 것이 좋지만 이런 경우라면 요약 정보를 담는 것이 좋다
- 예시: `맨해튼 거주자 전화번호부(총 1487536개)`

## toString 반환값의 포맷 문서화

포맷을 할 시에는 장점으로 명확해지고 쉽게 읽을 수 있으므로 그대로 입출력에 사용하거나
데이터 객체로 저장하여 사용 할 수 있다는 점이 있다.
단점으로는 한 번 명시 후 개발자들이 그 포맷에 맞추어 개발하다가
포맷을 바꿀 시에 그 간의 코드나 데이터를 수정해줘야 한다는데에 있다.
포맷을 명시하지 않는다면 릴리즈마다 정보를 더 넣거나 포맷을 개선해나가는 유연성을 얻을 수 있는 장점이 있다.

## toString이 반환한 값을 가져오는 API 제공

포맷 명시 여부와 관계 없이 toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공해야 한다.
예시는 아래와 같다.
```java
package item12;

public class PhoneNumberWithGetter {
    private final short areaCode;
    private final short prefix;
    private final short lineNumber;

    public PhoneNumberWithGetter(int areaCode, int prefix, int lineNumber) {
        this.areaCode = (short) areaCode;
        this.prefix = (short) prefix;
        this.lineNumber = (short) lineNumber;
    }

    public short getAreaCode() {
        return areaCode;
    }

    public short getPrefix() {
        return prefix;
    }

    public short getLineNumber() {
        return lineNumber;
    }

    @Override
    public String toString() {
        return String.format("%03d-%03d-%04d", areaCode, prefix, lineNumber);
    }

    public static void main(String[] args) {
        PhoneNumberWithGetter myNumber = new PhoneNumberWithGetter(707, 867, 5309);
        System.out.println("전화번호: " + myNumber);

        System.out.println("지역 코드: " + myNumber.getAreaCode());
        System.out.println("국번: " + myNumber.getPrefix());
        System.out.println("전화번호: " + myNumber.getLineNumber());
    }
}

```

전화번호 각 부분의 getter메소드를 제공하였다.
이렇게 하지 않으면 개발자는 toString의 값을 파싱하여 사용해야하므로 성능상 문제도 생기고 애초에 필요하지 않은 작업을 더 하게 된다.

