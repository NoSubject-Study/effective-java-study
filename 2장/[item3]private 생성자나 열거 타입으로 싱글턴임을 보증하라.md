## item-03 private 생성자나 열거 타입으로 싱글턴임을 보증하라.

싱글턴: 인스턴스를 오직 하나만 생성할 수 있는 클래스.
단점으로는 이를 사용하는 클라이언트를 테스트하기가 어려워질 수 있다는 것이다. 싱글턴 인스턴스를 mock 구현으로 대체할 수 없기 때문.

- mock 이란?

[모의 객체(Mock Object)란 주로 객체 지향 프로그래밍으로 개발한 프로그램을 테스트할 경우 테스트를 수행할 모듈과 연결되는 외부의 다른 서비스나 모듈들을 실제 사용하는 모듈을 사용하지 않고 실제의 모듈을 "흉내"내는 "가짜" 모듈을 작성하여 테스트의 효용성을 높이는 데 사용하는 객체이다. 사용자 인터페이스(UI)나 데이터베이스 테스트 등과 같이 자동화된 테스트를 수행하기 어려운 때 널리 사용된다. - 위키백과, Mock Object]

- 테스트 하기 어려워지는 이유는?
  싱글턴 인스턴스의 경우 애플리케이션 전역에서 상태가 공유되므로, 이로 인해서 테스트 케이스에서 싱글턴 상태를 변경하면 다른 테스트 케이스에도 영향을 줄 수 있다.

- 그렇다면 해결 방법은?
  인터페이스를 정의한 다음 그 인터페이스를 구현해서 만든 싱글턴으로 해결 가능하다.


아래는 Mockito 라이브러리를 통해 가짜 객체를 생성해 테스트하는 방법이다.
```java
public interface SingletonService {
    void performAction();
}
```

```java
public class Singleton implements SingletonService {
    private static Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }

    @Override
    public void performAction() {
               ...
    }
}

```

```java
import org.junit.jupiter.api.Test;
import static org.mockito.Mockito.*;

public class SingletonTest {
    @Test
    public void testPerformAction() {
        // 모의 객체 생성
        SingletonService mockService = mock(SingletonService.class);

        // 모의 객체의 메서드 호출 시의 동작을 정의
        doNothing().when(mockService).performAction();

        // 모의 객체의 메서드를 호출
        mockService.performAction();

        // 검증
        verify(mockService, times(1)).performAction();
    }
}
```
이렇게 하면 실제 인스턴스에 의존성을 갖지 않게 하면서도 해당 객체의 동작을 시뮬레이션 해볼 수 있다.


싱글턴 패턴을 구현하는 방식을 살펴본다.
1. public static final 필드 방식
```java
public class Elvis {
	public static final Elvis INSTANCE = new Elvis();
	private Elvis() { ... }
	public void leaveTheBuilding() { ... }
}
```
이 방식의 장점은, API를 통해서 해당 클래스가 싱글턴임을 쉽게 알 수 있다. public static final로 선언되어 있기 때문이다.
단점은 AccessibleObject.setAccessible 을 사용해 private 생성자를 호출 할 수 있단 것이다.

```java
import java.lang.reflect.Constructor;

public class SingletonReflectionAttack {
    public static void main(String[] args) {
        Singleton instance1 = Singleton.getInstance();
        Singleton instance2 = null;

        try {
            Constructor[] constructors = Singleton.class.getDeclaredConstructors();
            for (Constructor constructor : constructors) {
                // private 생성자의 접근성을 변경
                constructor.setAccessible(true);
                // 새로운 인스턴스 생성
                instance2 = (Singleton) constructor.newInstance();
                break;
            }
        } catch (Exception e) {
            e.printStackTrace();
        }

        System.out.println("Instance 1 hash:" + instance1.hashCode());
        System.out.println("Instance 2 hash:" + instance2.hashCode());
    }
}
```
접근제한자를 임의로 바꿀 수 있음을 보여주고 있다. 이로 인한 보안문제가 생긴다.


2. 정적 팩터리 방식의 싱글턴
```java
public class Elvis {
	private static final Elvis INSTANCE = new Elvis();
	private Elvis() { ... }
	public static Elvis getInstance() { return INSTANCE; }

	public void leaveTheBuilding() { ... }
}
```
이 방식의 장점은
- API를 바꾸지 않고 싱글턴이 아니게 할 수 있다. 아래와 같이.

```java
public class Elvis {
    private Elvis() {}

    public static Elvis getInstance() { return INSTANCE; }

    public void leaveTheBuilding() { ... }
}
```

- 정적 팩터리를 제네릭 싱글턴 팩터리로 만들수 있다. ->다양한 타입의 객체에 대해 동일한 싱글턴 인스턴스 로직을 적용 할 수 있다.

- 정적 팩터리의 메서드 참조를 공급자로 사용할 수 있다.-> 재사용성과 유연성을 높여줌
  Elvis::getInstance는 Supplier<Elvis> 타입의 공급자로 사용된다. get() 메서드를 호출하면 getInstance() 메서드가 호출되고, 이를 통해 Elvis 인스턴스에 접근할 수 있다.  (Supplier 인터페이스의 get() 메서드 구현을 대체)

```java
  public class Singleton {
    private static final Singleton INSTANCE = new Singleton();

    private Singleton() {}

    public static Singleton getInstance() {
        return INSTANCE;
    }
}

// 메서드 참조를 공급자로 사용
Supplier<Singleton> singletonSupplier = Singleton::getInstance;

// 공급자를 통해 싱글턴 인스턴스에 접근
Singleton singletonInstance = singletonSupplier.get();
```

싱글턴 클래스를 직렬화 할 때는, 모든 인스턴스 필드를 일시적이라고 선언하고 ReadResolve 메서드를 제공해야한다.

```java
  public class Singleton implements Serializable {
    private static final long serialVersionUID = 1L;
    private static final Singleton INSTANCE = new Singleton();
    private transient String state; // transient로 선언된 인스턴스 필드

    private Singleton() {}

    public static Singleton getInstance() {
        return INSTANCE;
    }

    // readResolve 메서드는 역직렬화 시 싱글턴 인스턴스를 유지하기 위해 필요
    private Object readResolve() {
        return INSTANCE;
    }
}

```
readResolve 메서드는 역직렬화 과정에서 자동으로 호출되어, 싱글턴 인스턴스의 유일성을 보장한다.

- Java transient?
  transient는 직렬화 과정에 제외하고 싶은 경우 선언하는 키워드이다.
- 사용 이유는?
  패스워드와 같은 보안정보를 직렬화 제외하고 싶을 때 사용한다. 즉, 데이터 전송을 하고 싶지 않을 때 선언하게 된다.
  또한 필드가 실행 환경에 따라 변할 수 있거나, 직렬화 후 역직렬화 과정에서 동일한 상태를 유지할 수 없는 경우, 이 필드를 transient로 선언하여 직렬화 과정에서 제외한다. 이렇게 하면 역직렬화된 객체에서 해당 필드는 기본값(예: null, 0)으로 설정된다.


3. 열거 타입 방식의 싱글턴 (바람직)

```java
  public enum Elvis {
  	INSTANCE;
  	
  public void leaveTheBuilding() { ...}
  }
  
```
대부분 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다.
싱글턴이 다른 클래스를 상속해야 한다면 이 방법은 사용할 수 없다(기본적으로 Enum 클래스를 상속하고 있기 때문. java.lang.Enum) 인터페이스 구현은 가능하다.


- enum은 기본적으로 serializable가 가능해서 역직렬화로 인한 새로운 인스턴스가 생성되는 것을 막을 수 있다. (serializable interface를 구현할 필요 없음)

- Reflection을 이용한 enum의 인스턴스화를 막을 수 있다.
  리플렉션을 사용하여 enum 인스턴스를 생성하려는 시도는 Java 언어의 근본적인 규칙을 위반하는 것으로 간주된다.  