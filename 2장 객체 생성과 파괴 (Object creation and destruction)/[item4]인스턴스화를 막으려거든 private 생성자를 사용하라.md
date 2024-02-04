# item-04


'정적 메서드, 정적 필드만을 담은 클래스는 객체지향적으로 사고하지 않는 것이다.'
→
1. 인스턴스 변수와 인스턴스 메서드를 가지지 않아서 재사용성과 유연성에 제한이 생기므로. (정적 메서드는 오버라이딩 될 수 없다. 하위클래스에서 다르게 동작하도록 만드는 것이 불가능하다.)

<br>
2. 그럼에도 정적 메서드, 정적 필드만을 담은 클래스가 쓰이는 이유는?<br>
→ 반복적으로 사용되는 유틸리티 함수, 상태를 저장할 필요 없는 메서드의 경우에는 정적메서드가 적합하다. 
메모리를 아낄 수 있기 때문이다. 정적 메서드와 정적 필드는 스태틱 영역에 클래스가 로드 될 때 한 번 담기기 때문에 메모리를 아낄 수 있다.
<br>
<br>

‘추상클래스로 만드는 것으로는 인스턴스화를 막을 수 없다.’

→ 하위 클래스에서 생성자를 만들면 super()로 호출됨 (super()를 안 써도 호출됨)

→ 추상클래스로 구현되어있으므로, ‘상속받아서 써야하는구나’라고 오해할 수 있음. 추상클래스는 하위클래스에서 메소드 구현을 해줘야 하니까.

```java
package item04;

abstract class SuperClass {

    public int number;

    public SuperClass() {
        System.out.println("make SuperClass");
//        throw new AssertionError();
    }
}

class SubClass extends SuperClass {
    public int number_2;

    public SubClass() {
        System.out.println("make SubClass");
    }
}

public class PrivateTest {
    public static void main(String[] args) {
        SubClass subClass = new SubClass();
    }
}
```

출력결과

```java
make SuperClass
make SubClass
```
<br>
<br>
<br>
<br>
<br>


```java
package item04;

class SuperClass {

    public int number;

    public SuperClass() {
        System.out.println("make SuperClass");
    }
}

class SubClass extends SuperClass {
    public int number_2;

    public SubClass() {
        System.out.println("make SubClass");
    }
}

public class PrivateTest {
    public static void main(String[] args) {
        SubClass subClass = new SubClass();
    }
}
```

위처럼 SuperClass 의 생성자를 private으로 하면 SubClass의 생성자에서 불러올 수 없다.