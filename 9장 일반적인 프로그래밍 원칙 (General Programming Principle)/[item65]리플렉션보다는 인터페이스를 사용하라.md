# 리플렉션

자바에는 리플렉션이라는 기능을 이용해 알 지 못하는 클래스를 생성하거나 메서드를 호출할 수 있다. 하지만 유용해보이는 리플렉션에 많은 단점이 존재한다.

``` java
public class TestClass {

  public static void main(String[] args) {
  Class<?> clazz = Class.forName("생성하려는 클래스 이름");
  Object obj = clazz.getConstructor().newInstance(); // 객체 생성
  }
}
```

# 리플렉션의 단점

## 컴파일 타임 타입 검사가 주는 이점을 하나도 누릴 수 없다.

리플렉션을 이용하여 메서드를 호출한다고 가정해보자.
만약 호출하려는 메서드가 클래스에 존재하지 않으면 어떻게 될까?

``` java
public class TestClass {

  public static void main(String[] arags) throws Exception {
    TestClass testClass = new TestClass();
    Method testMethod = TestClass.class.getMethod("unknownMethod"); // 존재하지 않는 unknownMethod 메서드
    testMethod.invoke(testClass); // 메서드 호출
  }
}
```

위의 코드를 실행하면 아래와 같은 예외가 발생한다.

![exception](https://velog.velcdn.com/images/lucius__k/post/2a428090-43cc-42ee-8057-9a84e0f7f512/image.png)
`NoSuchMethodException`, 해당 메서드가 존재하지 않다는 것이다.
이렇듯 리플렉션을 이용하지 않았으면 컴파일 시점에 알 수 있던 문제를 런타임 시에 알 수 있다.

또 예외 검사도 마찬가지다.

``` java
public class TestClass {

  public void doSomething() throws IOException {
    ...
  }

  public static void main(String[] args) {
    try {
      TestClass testClass = new TestClass();
      testClass.doSomething();
    } catch (IOException e) {
      System.out.println("IOException 발생!!");
    }
  }
}
```

위와 같이 `doSomething` 메서드를 호출하게 되면 **Checked Exception**인 `IOException`으로 인해 예외를 다뤄야 하는데 이를 리플렉션을 이용해 호출하게 되면

``` java
public class TestClass {

  public void doSomething() throws IOException {
    ...
  }

  public static void main(String[] args) throws NoSuchMethodException {
    TestClass testClass = new TestClass();
    Method doSomething = TestClass.class.getMethod("doSomething");
    doSomeThing.invoke(testClass); // 예외 검사 X
  }
}
```

위와 같이 예외 검사를 할 수 없게 된다.

## 성능이 저하된다.

``` java
public class TestClass {

  public int testMethod(int value) {
    return value;
  }
}
```

위에 int 타입의 정수를 받아 그대로 반환하는 메서드가 있고, 이를 10000번 실행한다고 가정해보자.

``` java
public class Main {

  public static void main(String[] args) {
    long startTime = System.currentTimeMillis();
    TestClass testClass = new TestClass();
    for (int i = 0; i < 10_000; i++) {
      testClass.testMethod(i);
    }
    System.out.println(System.currentTimeMillis() - startTime); // 1
  }
}
```
우선 생성자를 이용해 객체를 생성하여 실행하였을 경우 1ms의 시간이 소요되는 것을 볼 수 있다.

``` java
public class Main {

  public static void main(String[] args) {
    long startTime = System.currentTimeMillis();
    TestClass testClass = new TestClass();
    Method testMethod = TestClass.class.getMethod("testMethod", int.class);
    for (int i = 0; i < 10_000; i++) {
      testMethod.invoke(testClass, i);
    }
    System.out.println(System.currentTimeMillis() - startTime); // 11
  }
}
```

위처럼 리플렉션을 이용해 메서드를 호출하게 되면 11ms이 소요된다. 생성자를 이용해 호출했을 경우에 비해 11배나 느린 속도이다.

## 코드가 지저분하고 장황해진다.

리플렉션을 이용해 객체를 생성하게 되면 코드가 길어지고 가독성이 떨어진다.
먼저, 생성자를 이용하여 객체를 생성하게 되면 코드 한 줄로 작성할 수 있다.

``` java
TestClass testClass = new TestClass();
```

하지만 위의 `TestClass`를 리플렉션을 이용해 생성하게 되면

``` java

Class<?> clazz = null;
try {
  clazz = Class.forName("package.TestClass");
} catch (ClassNotFoundException e) {
	e.printStackTrace();
}

Constructor<?> constructor = null;
try {
	constructor = clazz.getConstructor();
} catch (NoSuchMethodException e) {
	e.printStackTrace();
}

try {
	Object o = constructor.newInstance();
} catch (InstantiationException e) {
	e.printStackTrace();
} catch (IllegalAccessException e) {
	e.printStackTrace();
} catch (InvocationTargetException e) {
	e.printStackTrace();
}
```

위 처럼 23줄에 걸쳐 생성을 하게 된다. 그리고 가독성도 물론 좋지 않다.

6가지의 예외가 발생할 수 있기 때문에 코드가 길어졌는데 이는 리플렉션을 이용했기 때문에 발생할 수 있게 된다.

자바 7부터 리플렉션 예외들이 `ReflectiveOperationException`을 상위 클래스로 두기 때문에 아래와 같이 더 간단하게 작성할 수 있다.

``` java
try {
  Class<?> clazz = Class.forName("package.testClass");
  Constructor<?> constructor = clazz.getConstructor();
  Object o = constructor.newInstance();
} catch (ReflectiveOperationException e) {
  e.printStackTrace();
}
```

# 리플렉션 활용

코드 분석 도구나 의존관계 주입 프레임워크처럼 리플렉션을 써야 하는 복잡한 애플리케이션이 몇 가지 있다. 이런 도구들마저 리플렉션 사용을 줄이고 있다. 만약 애플리케이션을 제작하는데 반드시 리플렉션을 사용해야만 하는 경우가 아니라면 사용을 피해야 한다.
만일 리플렉션을 사용한다면 아주 제한된 형태로만 사용하여 리플렉션으로 인한 성능감소를 최소화해야 한다. 되도록 객체 생성에만 사용하고, 생성된 객체를 이용할 때는 적절한 인터페이스나 컴파일타임에 알 수 있는 상위 클래스로 형변환해 사용해야 한다.
