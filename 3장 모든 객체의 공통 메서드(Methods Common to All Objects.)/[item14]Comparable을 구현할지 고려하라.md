## 자바에서의 비교

자바에서 기본 자료형의 비교를 한다면
```java
		if(a > b) {
			System.out.println("a가 b보다 큽니다.");
		}
		else if(a == b) {
			System.out.println("a와 b는 같습니다.");
		}
		else {
			System.out.println("b가 a보다 큽니다. ");
		}
```
위와 같이 간단하게 할 수 있을 것이다. 허나 객체간의 비교를 한다면?
이 객체들이 어떤 필드를 갖고 있는지도 봐야하고 그 필드들 중 어떤 것을 기준으로 비교하고 정렬을 할 지 결정을 해야한다.



## Comparable 인터페이스
Comparable 인터페이스는 compareTo 메소드 하나만 갖고 있다.
```java
public interface Comparable<T> {

    public int compareTo(T o);
}

```
그러므로 사용해주려면 Comparable<T>를 implement한 클래스에서 이를 오버라이딩해주어야 한다.
이 메소드는 객체 자신과 다른 객체를 비교하여 정수 값을 반환하는데, 반환값들의 뜻이 아래와 같다.

```text
음수: 객체 자신이 인자로 주어진 객체보다 작을 때
0: 두 객체가 같을 때
양수: 객체 자신이 인자로 주어진 객체보다 클 때
```
## compareTo 메소드 규약

compareTo 메소드는 몇 가지 기본적인 규칙을 따라야 하는데 아래와 같다.

- x.compareTo(y)의 부호는 y.compareTo(x)의 부호와 반대여야 함.
- x.compareTo(y) > 0이고 y.compareTo(z) > 0이면, x.compareTo(z) > 0이어야 함.
- x.compareTo(y) == 0이면, 모든 z에 대해 sgn(x.compareTo(z)) == sgn(y.compareTo(z))가 성립해야 함.

- equals와의 일관성(필수는 아님.)

### equals와의 일관성
compareTo는 equals와 일관성을 유지하는 것이 권장된다. 즉, x.compareTo(y) == 0이면 x.equals(y)도 true여야 하는데. 하지만 이는 필수 사항은 아니며, 만약 두 메소드의 기준이 다르면 해당 사실을 문서화해야 한다.

이에 주의해야할 것이, 객체를 정렬된 컬렉션에 넣으면 해당 컬렉션이 구현한 인터페이스에(Collection, Set, Map) 정의된 동작에서 엇박자를 내게 된다.
왜냐하면 이 인터페이스들은 규약에서는 eqauls를 기준으로 한다고 되어있지만 정렬된 컬렉션에서는 실제 동치성 비교시에 compareTo를 이용하기 때문이다.
(따라서 필수는 아니더라도 이런 점을 고려하여 웬만하면 일관되게 하는 것이 낫다.)
- 예시: equals와 compareTo가 일관되지 않은 BigDecimal 클래스가 있다.
 HashSet 인스턴스를 생성하고 `new BigDecimal("1.0")` 과 `new BigDecimal("1.00")`를 차례로 추가하게 되면 eqauls 메소드로 비교할 시에는 서로 달라서 HashSet은 두 개의 원소를 갖지만 HashSet 대신 TreeSet을 사용하면 원소를 하나만 갖게 된다. (compareTo 메소드로 비교했기 때문)


## Comparable 인터페이스는 자바의 정렬, 검색 등에서 활용된다.
예를 들면, 아래처럼 Arrays.sort() 메소드를 사용하여 Comparable을 구현하는 객체 배열을 쉽게 정렬할 수 있다
```java
package item14;
import java.util.Arrays;

public class Person implements Comparable<Person> {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }



    @Override
    public int compareTo(Person other) {
        if (this.age < other.age) {
            return -1;  // 현재 객체가 더 어릴 때
        } else if (this.age > other.age) {
            return 1;   // 현재 객체가 더 나이가 많을 때
        } else {
            return 0;   // 두 객체의 나이가 같을 때
        }
    }
/*
      @Override
    public int compareTo(Person other) {
        return Integer.compare(this.age, other.age);
    }
*/

    public static void main(String[] args) {
        Person[] people = {new Person("김OO", 24), new Person("나OO", 30), new Person("하OO", 22)};
        Arrays.sort(people);
        for (Person p : people) {
            System.out.println(p.name + ", " + p.age);
        }
    }
}

```
위와 같이 Comparable 인터페이스를 구현해주고 compareTo 메소드를 오버라이딩해주어야 한다.
나이 기준으로 정렬되는 것을 확인할 수 있다.

(여기서 compareTo 메소드는 자신을 다른 객체와 비교하여 세 가지 가능한 결과(음수, 0, 양수)를 반환하여 정렬 순서를 정의한다.)
```text
하OO, 22
김OO, 24
나OO, 30
```
이 때 나이가 오름차순으로 정렬되는데 반대로 내림차순으로 정렬하고 싶다면 주석처리된 코드처럼
 반대의 결과를 출력해주면 된다.
```java
    @Override
    public int compareTo(Person other) {
        if (this.age < other.age) {
            return 1;
        } else if (this.age > other.age) {
            return -1;
        } else {
            return 0;
        }
    }
```
```text
나OO, 30
김OO, 24
하OO, 22
```

## 객체 참조 필드들의 비교
```java
  public final class CaseInsensitiveString implements Comparable<CaseInsensitiveString> {
    private String s;

    public int compareTo(CaseInsensitiveString cis) {
        return String.CASE_INSENSITIVE_ORDER.compare(this.s, cis.s);
    }
}
```
객체 참조 필드를 비교할 때는 해당 필드의 compareTo 메소드를 재귀적으로 호출하여 비교한다.

## 기본 자료형 필드 비교대상이 여럿이라면?
```java
package item14;

  public class PhoneNumber implements Comparable<PhoneNumber> {
    private short areaCode;
    private short prefix;
    private short lineNum;

    public int compareTo(PhoneNumber pn) {
        int result = Short.compare(areaCode, pn.areaCode);
        if (result == 0) {
            result = Short.compare(prefix, pn.prefix);
            if (result == 0) {
                result = Short.compare(lineNum, pn.lineNum);
            }
        }
        return result;
    }
}

```
이처럼 하나하나 차례대로 비교해준다.


## Comparator 인터페이스

Comparator 인터페이스에서는 앞서 살펴본  Comparable 인터페이스와 달리 두 객체를 받아서 비교한다.

## Comparator 인터페이스의 정적 메소드 활용
Java 8 이후로 Comparator 인터페이스는 객체를 비교하는 데 도움이 되는 다양한 정적 메소드들을 제공한다.
이 메소드들을 사용하면 Comparator 객체를 간단하게 생성, 메소드 연쇄 방식으로 복잡한 비교 로직을 구현할 수 있다.


```java
public class PhoneNumber implements Comparable<PhoneNumber> {
    private short areaCode;
    private short prefix;
    private short lineNum;

    private static final Comparator<PhoneNumber> COMPARATOR =
        Comparator.comparingInt((PhoneNumber pn) -> pn.areaCode)
                  .thenComparingInt(pn -> pn.prefix)
                  .thenComparingInt(pn -> pn.lineNum);

    public int compareTo(PhoneNumber pn) {
        return COMPARATOR.compare(this, pn);
    }
}
```
이 예제에서 PhoneNumber 클래스는 areaCode, prefix, lineNum 순서로 비교한다.
이렇게 구성된 Comparator는 각 필드를 순차적으로 비교하여 전화번호의 정렬 순서를 결정한다.
즉 각 필드들의 우선순위대로 비교를 한다.
이 방식을 사용하여 코드의 가독성을 높일 수 있지만 약간의 성능저하는 생기는데 저자의 경우 10퍼센트의 성능 저하가 있다고 한다.


  ## compareTo / compare 메소드 구현시 주의할점

```java
   @Override
    public int compareTo(Person other) {
        if (this.age < other.age) {
            return -1;  // 현재 객체가 더 어릴 때
        } else if (this.age > other.age) {
            return 1;   // 현재 객체가 더 나이가 많을 때
        } else {
            return 0;   // 두 객체의 나이가 같을 때
        }
    }
```
  이 코드를 아래와 같이 구현하면 더 깔끔해보일 수 있다.
  ```java
     @Override
    public int compareTo(Person other) {
		return this.age - other.age;
    }
  ```
  한 줄의 코드로 if, else-if를 모두 대체했으니 말이다. 그러나 이는 오버플로우를 일으키거나 부동소수점 계산 방식에 따른 오류를 낼 수 있으므로 사용하지 않아야 한다.

따라서 아래의 방식들을 권장한다.
  ```java
    @Override
    public int compareTo(Person other) {
        return Integer.compare(this.age, other.age);
    }
  ```
다른 방법으로는 비교자 생성메소드를 이용한 방법이 있다.
  ```java
  static Comparator<Object> hashCodeOrder = Comparator.comparingInt(o -> o.hashCode());
  ```