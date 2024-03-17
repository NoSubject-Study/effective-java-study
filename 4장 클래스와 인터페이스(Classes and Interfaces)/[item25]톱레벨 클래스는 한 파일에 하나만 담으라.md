*톱레벨 클래스는 한 파일에 하나만 담으라*

---

소스 파일 하나에 톱레벨 클래스를 여러 개 선언하더라도 자바 컴파일러는 불평하지 않지만, 하지만 아무런 득이 없을 뿐더러 심각한 위험을 감수해야 함 

→ 한 클래스를 여러 가지로 정의할 수 있어, 어느 것을 먼저 사용할 지는 어떤 소스 파일을 먼저 컴파일하냐에 따라 달라지기 때문

Ex. `Main` 클래스에 다른 톱레벨 클래스 2개를 참조하는 경우

```java
public class Main {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
}
```

집기(`Utensil`)와 디저트(`Dessert`) 클래스가 `Utensil.java` 한 파일에 정의

```java
// Two classes defined in one file. Don't ever do this!
class Utensil {
    static final String NAME = "pan";
}

class Dessert {
    static final String NAME = "cake";
}
```

이 상황에서 우연히 똑같은 두 클래스를 담은 `Dessert.java` 파일을 생성

```java
// Two classes defined in one file. Don't ever do this!
class Utensil {
    static final String NAME = "pot";
}

class Dessert {
    static final String NAME = "pie";
}
```

### 실행 결과

- `javac Main.java Dessert.java` 명령어로 컴파일
    - 컴파일러는 `Main.java`를 먼저 컴파일하면서 `Utensil`과 `Dessert` 클래스를 찾기 위해 `Utensil.java`를 참조
    - 이후 컴파일러가 `Dessert.java`를 처리할 때, 이미 정의된 `Utensil`과 `Dessert` 클래스를 다시 만나게 되어, 컴파일 에러가 발생
- `javac Main.java` 또는 `javac Main.java Utensil.java` 명령어로 컴파일
    - 이전에 `Dessert.java` 파일을 작성하기 이전과 동일하게 "pancake"를 출력
- `javac Dessert.java Main.java` 명령어로 컴파일
    - 컴파일러가 `Dessert.java`를 먼저 처리하고, 그 다음 `Main.java`를 처리하여 "potpie"를 출력

### 해결 방법

→ 단순히 톱레벨 클래스들(`Utensil`과 `Dessert`)을 서로 다른 소스 파일로 분리

굳이 여러 톱레벨 클래스를 한 파일에 담고 싶다면 정적 멤버 클래스(Item 24) 사용을 고려

일반적으로 다른 클래스에 딸린 부차적인 클래스의 경우 정적 멤버 클래스로 만드는 것이 더 나음 → 읽기 좋고 `private`으로 선언하면(Item 15) 접근 범위도 최소로 관리할 수 있기 때문

Ex. 톱레벨 클래스들을 정적 멤버 클래스로 바꿔본 모습

```java
// Static member classes instead of multiple top-level classes
public class Test {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }

    private static class Utensil {
        static final String NAME = "pan";
    }

    private static class Dessert {
        static final String NAME = "cake";
    }
}

```
