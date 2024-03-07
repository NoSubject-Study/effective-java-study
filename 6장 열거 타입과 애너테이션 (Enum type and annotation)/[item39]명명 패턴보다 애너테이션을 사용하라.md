# JUnit으로 알아보는 명명 패턴의 단점

전통적으로 도구나 프레임워크가 특별히 다뤄야 할 프로그램 요소에는 딱 구분되는 명명 패턴을 적용해왔다.

테스트 프레임워크인 JUnit은 버전 3까지 테스트 메서드 이름을 test로 시작하게끔 했다.

효과적이지만 단점도 크다.

1. __오타가 나면 안된다.__

   실수로 이름을 `tsetSafetyOverride`로 지으면 JUnit은 이 메서드를 무시하고 지나간다.

   개발자는 테스트가 통과되었다고 오해할 수 있다.
3. __올바른 프로그램 요소에서만 사용되리라 보증할 방법이 없다.__

   메서드가 아닌 클래스 이름을 `TestSafetyMechanisms`로 지어 테스트를 한다고 해보자.

   해당 클래스에 존재하는 테스트 메서드들을 수행해주길 기대하지만 JUnit은 클래스 이름에는 관심이 없기 때문에 테스트가 수행되지 않는다.
4. __프로그램 요소를 매개변수로 전달할 마땅한 방법이 없다.__

   예외가 발생해야 하는 테스트에 기대하는 예외 타입을 테스트에 매개변수로 전달해야 하는 상황에서,

   예외의 이름을 테스트 메서드 이름에 덧붙이는 방법도 있지만, 보기도 나쁘고 깨지기도 쉽다.

   컴파일러는 메서드 이름에 덧붙인 문자열이 예외를 가리키는지 알 방법이 없다.

   또 테스트를 실행하기 전에는 그런 이름의 클래스가 존재하는지 혹은 예외가 맞는지조차 알 수 없다.

---

# 애너테이션으로의 전환

애너테이션은 위에서 말한 모든 문제를 해결해주며, JUnit도 버전 4부터 도입하였다.

작은 테스트 프레임워크를 제작하여 애너테이션의 동작 방식을 알아보자.

## 정적 테스트 메서드 수행

`Test`라는 이름의 애너테이션을 정의하자.

자동으로 수행되는 간단한 테스트용 애너테이션이다.

``` java
/**
 * 테스트 메서드임을 선언하는 애너테이션이다.
 * 매개변수 없는 정적 메서드 전용이다.
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {
}
```

위의 `@Test` 애너테이션을 보면 `@Retention`, `@Target`이 있는 것을 볼 수 있는데,

이처럼 애너테이션 선언에 다는 애너테이션을 *메타애너테이션*이라고 한다.

`@Retention(RetentionPolicy.RUNTIME)` 메타애너테이션은 `@Test`가 런타임에도 유지되어야 한다는 표시다.

`@Target(ElementType.METHOD)` 메타애너테이션은 `@Test`가 반드시 메서드 선언에서만 사용돼야 한다고 알려준다.

주석에 "매개변수 없는 정적 메서드 전용이다."라고 쓰여 있는데, 이 제약을 컴파일러가 강제할 수 있으면 좋겠지만, 그렇게 하려면 애너테이션 처리기를 직접 구현해야 한다.

관련 방법은 javax.annotation.processing API 문서를 참고하면 된다.

``` java
public class Sample {
    
    @Test
    public static void m1() {} // 성공해야 한다.
    
    public static void m2() {}
    
    @Test
    public static void m3() { // 실패해야 한다.
        throw new RuntimeException("실패");
    }
    
    public static void m4() {}
    
    @Test
    public void m5() {} // 잘못 사용한 예: 정적 메서드가 아니다.
    
    public static void m6() {}
    
    @Test
    public static void m7() { // 실패해야 한다.
        throw new RuntimeException("실패");
    }
    
    public static void m8() {}
}

```

위의 코드는 `@Test` 애너테이션을 실제 적용한 모습이다.

이런 애너테이션을 "아무 매개변수 없이 단순히 대상에 마킹(marking)한다"는 뜻에서 마커(marker) 애너테이션이라고 한다.

위의 코드에서 홀수번의 메서드들에만 `@Test` 애너테이션이 있고,

3, 7번 메서드에서는 예외가 발생하고,

5번 메서드는 인스턴스 메서드이므로 잘못 사용되었다.

즉, 1개의 테스트 메서드만 성공, 2개는 실패, 1개는 잘못 사용되었다.

그리고 `@Test` 애너테이션이 없는 4개의 메서드는 무시되어야 한다.

``` java
public class RunTests {

    public static void main(String[] args) {
        int tests  = 0;
        int passed = 0;
        for (Method m : Sample.class.getDeclaredMethods()) {
            if (m.isAnnotationPresent(Test.class)) {
                tests++;
                try {
                    m.invoke(null);
                    passed++;
                } catch (InvocationTargetException wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    System.out.println(m + " 실패: " + exc);
                } catch (Exception exc) {
                    System.out.println("잘못 사용한 @Test: " + m);
                }
            }
        }
        System.out.printf("성공: %d, 실패: %d%n", passed, tests - passed);
    }
}

```

위의 코드를 보면, `Sample` 클래스에서 `@Test` 애너테이션이 있는 모든 메서드를 차례로 호출한다.

테스트 메서드가 예외를 던지면 리플렉션 메커니즘이 `InvocationTargetException`으로 감싸서 다시 던진다.

`InvocationTargetException`을 잡아 원래 예외에 담긴 실패 정보를 추출해 출력한다.

`InvocationTargetException`외의 예외가 발생하면 `@Test` 애너테이션을 잘못 사용했다는 뜻이므로 해당 예외를 잡아 적절한 오류 메시지를 출력한다.

`RunTests`를 실행했을 때의 출력 메시지

```
잘못 사용한 @Test: public void Sample.m5()
public static void Sample.m7() 실패: java.lang.RuntimeException: 실패
public static void Sample.m3() 실패: java.lang.RuntimeException: 실패
성공: 1, 실패: 3
```

## 예외를 던져야만 성공하는 테스트

특정 예외를 던져야만 성공하는 테스트를 지원하기위해 새로운 애너테이션을 정의하자.

``` java
/**
 * 명시한 예외를 던져야만 성공하는 테스트 메서드용 애너테이션
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}
```

이 애너테이션의 매개변수 타입은 `Class<? extends Throwable>`이다.

여기서의 와일드카드 타입은 "Throwable을 확장한 클래스의 Class 객체"라는 뜻으로 모든 예외 타입을 다 수용한다.

다음은 이 애너테이션을 실제로 활용하는 코드다.

``` java
public class Sample2 {
    
    @ExceptionTest(ArithmeticException.class)
    public static void m1() { // 성공해야 한다.
        int i = 0;
        i = i / i;
    }

    @ExceptionTest(ArithmeticException.class)
    public static void m2() { // 실패해야 한다. (다른 예외 발생)
        int[] a = new int[0];
        int i = a[1];
    }

    @ExceptionTest(ArithmeticException.class)
    public static void m3() {} // 실패해야 한다. (예외가 발생하지 않음)
}
```

이 애너테이션을 다룰 수 있도록 테스트 도구를 수정해보면,

``` java
public static void main(String[] args) {
    int tests = 0;
    int passed = 0;
    for (Method m : Sample2.class.getDeclaredMethods()) {
        if (m.isAnnotationPresent(ExceptionTest.class)) {
            tests++;
            try {
                m.invoke(null);
                System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
            } catch (InvocationTargetException wrappedEx) {
                Throwable exc = wrappedEx.getCause();
                Class<? extends Throwable> excType = m.getAnnotation(ExceptionTest.class).value();
                if (excType.isInstance(exc)) {
                    passed++;
                } else {
                    System.out.printf("테스트 %s 실패: 기대한 예외 %s, 발생한 예외 %s%n", m, excType.getName(), exc);
                }
            } catch (Exception exc) {
                System.out.println("잘못 사용한 @ExceptionTest: " + m);
            }
        }
    }
    System.out.printf("성공: %d, 실패: %d%n", passed, tests - passed);
}
```

`@Test`애너테이션용 코드와 비슷해보이지만, 이 코드는 애너테이션 매개변수의 값을 추출하여 테스트 메서드가 올바른 예외를 던지는지 확인하는 데 사용한다.

### 배열을 이용한 여러 예외 테스트

예외를 여러 개 명시하고 그중 하나가 발생하면 성공하게 만들 수도 있다.

``` java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable>[] value();
}
```

위처럼 `@ExceptionTest` 애너테이션의 매개변수 타입을 Class 객체의 배열로 수정하자.

배열 매개변수를 받는 애너테이션용 문법은 유연하다.

위처럼 `@ExceptionTest` 애너테이션을 수정해도 앞에서 작성한 테스트 메서드들도 수정없이 사용 가능하다.

원소가 여럿인 배열을 지정할 때 중괄호로 감싸고 쉼표로 구분해주면된다.

``` java
@ExceptionTest({IndexOutOfBoundsException.class, NullPointerException.class})
public static void doublyBad() { // 성공해야 한다.
    List<String> list = new ArrayList<>();
    
    // 자바 API 명세에 따르면 다음 메서드는 IndexOutOfBoundsException이나 NullPointerException을 던질 수 있다.
    list.addAll(5, null);
}
```

위의 새로운 `@ExceptionTest`를 지원하도록 코드를 수정하면,

``` java
if (m.isAnnotationPresent(ExceptionTest.class)) {
    tests++;
    try {
        m.invoke(null);
        System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
    } catch (Throwable wrappedEx) {
        Throwable exc = wrappedEx.getCause();
        int oldPassed = passed;
        Class<? extends Throwable>[] excTypes = m.getAnnotation(ExceptionTest.class).value();
        for (Class<? extends Throwable> excType : excTypes) {
            if (excType.isInstance(exc)) {
                passed++;
                break;
            }
        }
        if (passed == oldPassed) {
            System.out.printf("테스트 %s 실패: %s %n", m, exc);
        }
    }
}
```

### @Repeatable을 이용한 여러 예외 테스트

자바 8에서는 배열 매개변수를 사용하는 대신 애너테이션에 `@Repeatable` 메타애너테이션을 다는 방식을 이용할 수 있다.

단, 주의할 점이 있다.

1. @Repeatable이 있는 애너테이션을 반환하는 '컨테이너 애너테이션'을 하나 더 정의하고, @Repeatable에 이 컨테이너 애너테이션의 class 객체를 매개변수로 전달해야 한다.
2. 컨테이너 애너테이션은 내부 애너테이션 타입의 배열을 반환하는 value 메서드를 정의해야 한다.
3. 컨테이너 애너테이션 타입에는 적절한 보존 정책(@Retention)과 적용 대상(@Target)을 명시해야 한다. 그렇지 않으면 컴파일되지 않는다.

``` java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}

// 컨테이너 애너테이션
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
    ExceptionTest[] value();
}
```

앞의 배열 방식 대신 반복 가능 애너테이션을 적용하면,

``` java
@ExceptionTest(IndexOutOfBoundsException.class)
@ExceptionTest(NullPointerException.class)
public static void doublyBad() { ... }
```

반복 가능 애너테이션을 여러 개 달면 하나만 달았을 때와 구분하기 위해 해당 '컨테이너' 애너테이션 타입이 적용된다.

`isAnnotationPresent` 메서드에서 컨테이너 애너테이션과 반복 가능 애너테이션이 있는지 모두 확인해야 하므로 각각 사용해야 한다.

반복 가능 버전으로 테스트 수행 메서드를 수정하면,

``` java
if (m.isAnnotationPresent(ExceptionTest.class) || m.isAnnotationPresent(ExceptionTestContainer.class)) {
    tests++;
    try {
        m.invoke(null);
        System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
    } catch (Throwable wrappedEx) {
        Throwable exc = wrappedEx.getCause();
        int oldPassed = passed;
        ExceptionTest[] excTests = m.getAnnotationsByType(ExceptionTest.class);
        for (ExceptionTest excTest : excTests) {
            if (excTest.value().isInstance(exc)) {
                passed++;
                break;
            }
        }
        if (passed == oldPassed) {
            System.out.printf("테스트 %s 실패: %s %n", m, exc);
        }
    }
}
```

---

위의 만든 애너테이션을 이용한 테스트 프레임워크에서 애너테이션이 명명패턴보다 낫나는 점을 확실히 보여준다.

애너테이션으로 할 수 있는 일을 명명 패턴으로 처리할 이유는 없다.

자바 프로그래머라면 예외 없이 자바가 제공하는 애너테이션 타입들은 사용해야 한다.

IDE나 정적 분석 도구가 제공하는 애너테이션을 사용하면 해당 도구가 제공하는 진단 정보의 품질을 높여줄 것이다.
