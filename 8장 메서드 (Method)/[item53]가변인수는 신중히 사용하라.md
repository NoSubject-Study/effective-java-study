# Item53: Use varargs judiciously

*가변인수는 신중히 사용하라*

---

가변인수(varargs) 메서드는 명시한 타입의 인수를 0개 이상 받을 수 있다.

→ 인수의 개수와 길이가 같은 배열을 만들어 인수들을 이 배열에 저장한 후 , 가변인수 메서드에 전달

Ex. 입력받은 `int`인수들의 합을 계산해주는 가변인수 메서드

```java
// Simple use of varargs
static int sum(int... args) {
    int sum = 0;
    for (int arg : args)
        sum += arg;
    return sum;
}
```

Ex. 최솟값을 찾는 메서드

```java
// The WRONG way to use varargs to pass one or more arguments!
static int min(int... args) {
    if (args.length == 0)
        throw new IllegalArgumentException("Too few arguments");
    int min = args[0];
    for (int i = 1; i < args.length; i++)
        if (args[i] < min)
            min = args[i];
    return min;
}
```

- 최솟값을 찾으려면 인수가 1개 이상이어야 하지만, 인수가 0개도 받을 수 있음
    - 인수를 0개만 넣어 호출할 때, 컴파일타임이 아닌 런타임에서 실패
    - `args` 유효성 검사를 명시적으로 해야하고, `min`의 초깃값을 `Integer.MAX_VALUE`로 설정하지 않고는 `for-each`문도 사용할 수 없음

Ex. 인수가 1개 이상이어야 할 때 가변인수를 제대로 사용하는 방법

```java
// The right way to use varargs to pass one or more arguments
static int min(int firstArg, int... remainingArgs) {
    int min = firstArg;
    for (int arg : remainingArgs)
        if (arg < min)
            min = arg;
    return min;
}
```

- 첫 번째로 평범한 매개변수를 받고, 가변인수를 두 번째로 받으면 앞서의 문제 해결 가능

⇒ 가변인수는 인수 개수가 정해지지 않았을 때 아주 유용하다. (printf, 리플렉션…)

Ex. 가변인수와 같이 도입된 `printf`

```java
public PrintStream printf(String format, Object ... args) {
    return format(format, args);
}
    
public PrintStream printf(Locale l, String format, Object ... args) {
    return format(l, format, args);
}
```

⇒ 성능에 민감한 상황이면 가변인수가 걸림돌이 될 수 있다. (호출될 때마다 배열을 새로 하나 할당하고 초기화 하기 때문)

Ex. 다중정의 메서드를 활용한 패턴

해당 메서드 호출의 95%가 인수를 3개 이하로 사용한다고 가정

```java
public void foo() { }
public void foo(int a1) { }
public void foo(int a1, int a2) { }
public void foo(int a1, int a2, int a3) { }
public void foo(int a1, int a2, int a3, int... rest) { }
```

- 인수가 0개부터 4개인 것까지, 총 5개를 다중정의
- 마지막 다중정의 메서드가 인수 4개 이상인 5%의 호출을 담당
- 보통 때는 별 이득이 없지만, 가변인수의 유연성이 필요할 때 사용
- `EnumSet`의 정적 팩터리도 이 기법을 사용해 열거 타입 집합 생성 비용을 최소화(비트 필드)
