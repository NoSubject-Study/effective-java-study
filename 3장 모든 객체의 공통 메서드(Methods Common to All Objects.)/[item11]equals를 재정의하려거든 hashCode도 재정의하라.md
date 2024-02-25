## item-11 equals를 재정의하려거든 hashCode도 재정의하라
equals를 재정의한 클래스에서는 모두 hashCode도 재정의 해야한다. 그렇지 않다면  hashCode의 일반 규약을 어기게 된다.
Object 명세에서의 규약은 아래와 같다.

- equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 hashCode 메서드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다. 단, 애플리케이션을 다시 실행한다면 이 값이 달라져도 상관없다.

- equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다.

- equals(Object)가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다. 단 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

규약을 어기면 발생하는 문제는 아래의 코드에서 확인할 수 있다.

```java
package item11;

import java.util.HashMap;
import java.util.Map;
import java.util.Objects;

public class PhoneNumber {
    private final short areaCode;
    private final short prefix;
    private final short lineNumber;

    public PhoneNumber(int areaCode, int prefix, int lineNumber) {
        this.areaCode = (short) areaCode;
        this.prefix = (short) prefix;
        this.lineNumber = (short) lineNumber;
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

//    @Override
//    public int hashCode() {
//        int result = Short.hashCode(areaCode);
//        result = 31 * result + Short.hashCode(prefix);
//        result = 31 * result + Short.hashCode(lineNumber);
//        return result;
//    }

    public static void main(String[] args) {
        Map<PhoneNumber, String> m = new HashMap<>();
        m.put(new PhoneNumber(707, 867, 5309), "Jenny");

        String result = m.get(new PhoneNumber(707, 867, 5309));

        if (result != null) {
            System.out.println("검색 결과: " + result);
        } else {
            System.out.println("검색 결과가 없습니다.");
        }
    }
}



```
오버라이딩 된 hashCode 코드를 주석처리 한 후의 실행 결과는 어떨까?

```text
검색 결과가 없습니다.
```
로 나오게 된다.
```java
m.put(new PhoneNumber(707, 867, 5309), "Jenny");
String result = m.get(new PhoneNumber(707, 867, 5309));
```
두 코드에서의 new PhoneNumber(707, 867, 5309)로 생기는 두 객체가 논리적으로는 동치이지만 서로 다른 해시코드를 반환하여 두 번째 규약을 어기기 때문이다.

이 문제를 해결해주려면? hashCode 메소드를 오버라이딩 해주어야 한다.
이상적인 해시 함수는 주어진 서로 다른 인스턴스들을 32비트 정수범위에 균일하게 분배해야 하는데, 이를 정확히 구현하기는 어렵지만 비슷하게 구현하는 코드를 짜야 한다.

전형적인 코드는 아래와 같다.
```java
    @Override
    public int hashCode() {
        int result = Short.hashCode(areaCode);
        result = 31 * result + Short.hashCode(prefix);
        result = 31 * result + Short.hashCode(lineNumber);
        return result;
    }
```

```text
검색 결과: Jenny
```
로 예상한 결과가 나온다.


hashCode에 들어가는 필드들은 핵심 필드만이 들어가야 하며,구현에 곱셈이 들어간 이유는 아래와 같다.
곱셈 없이 단순하게 덧셈과 같은 방법으로 구현한다면, 아래의 결과를 보이게 된다.

```java
package item11;

public class SimpleHashCode {
    private String part1;
    private String part2;

    public SimpleHashCode(String part1, String part2) {
        this.part1 = part1;
        this.part2 = part2;
    }

    @Override
    public int hashCode() {
        return part1.hashCode() + part2.hashCode(); 
    }
//    @Override
//    public int hashCode() {
//        int result = 0;
//        result = 31 * result + part1.hashCode();
//        result = 31 * result + part2.hashCode();
//        return result;
//    }


    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        SimpleHashCode that = (SimpleHashCode) o;
        return part1.equals(that.part1) && part2.equals(that.part2);
    }

    public static void main(String[] args) {
        SimpleHashCode obj1 = new SimpleHashCode("star", "rats");
        SimpleHashCode obj2 = new SimpleHashCode("rats", "star");

        System.out.println("obj1의 해시코드: " + obj1.hashCode());
        System.out.println("obj2의 해시코드: " + obj2.hashCode());

        if (obj1.hashCode() == obj2.hashCode()) {
            System.out.println("두 객체의 해시코드가 동일합니다.");
        } else {
            System.out.println("두 객체의 해시코드가 다릅니다.");
        }
    }
}
```
실행결과
```text
obj1의 해시코드: 7033664
obj2의 해시코드: 7033664
두 객체의 해시코드가 동일합니다.
```
main 메소드에서 생성된 두 객체는 아나그램으로 철자가 같고 순서만 다른 문자열이다.

곱셈을 통한다면 서로 다른 필드에 다른 가중치가 적용되므로 해시 충돌의 가능성을 줄일 수 있다.<br>
곱하는 수가 31인 이유는, 31이 홀수이면서 소수(prime number)이기 때문이다.<br>
소수를 곱하는 이유는 명확하지 않지만 전통적으로 그리 해왔다고 저자는 말하고 있다.<br>
또한 31 * i 는 (i << 5 ) - i 로 쉬프트 연산을 하여 더 최적화를 시킬 수 있다고 하는데 요즘 VM은 이런 최적화를 자동으로 해주고 있다.


```java
@Override 
public int hashCode() {
	return Objects.hash(lineNum, prefix, areaCode);
}
```
한 줄로 hashCode 를 단순하게 작성하는 방법도 있다. Objects 클래스에서 임의의 개수만큼 객체를 받아 해시코드를 계산해주는 정적 메소드인데, 속도는 더 느리므로 성능에 민감하지 않을 때만 사용하는 것이 좋다.



또한 클래스가 불변이고 해시코드 계산 비용이 크면, 매번 새로 계산하는 것보다는 캐싱 방식을 고려해야 한다.

```java
//...
    private int hashCode; // 자동으로 0으로 초기화된다.
    
    @Override
    public int hashCode() {
        int result = hashCode;
        if (result == 0) {
            result = Short.hashCode(areaCode);
            result = 31 * result + Short.hashCode(prefix);
            result = 31 * result + Short.hashCode(lineNumber);
            hashCode = result;
        }
        return result;
    }
```

성능을 높이기 위해 핵심 필드를 해시코드 계산에서 생략하면 안된다.
해시 품질이 나빠지므로 해시테이블의 성능을 떨어뜨리기 때문이다. 