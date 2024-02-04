## 다 쓴 객체 참조를 해제하라.

C, C++ 언어의 경우 메모리를 직접 관리해줘야하는 언어여서 C,C++을 쓰다가 자바를 쓴다면, 처음에는 GC를 갖춘 언어이므로 훨씬 편안하다고 느낄 수 있다.(다 쓴 객체를 알아서 회수해감) 그러나  메모리 관리에 신경 쓰지 않는다면 메모리 누수가 일어나는 경우도 있다.

```java
public class Stack {
	private Object[] elements;
	private int size = 0;
	private static final int DEFAULT_INITIAL_CAPACITY = 10;
	
	public Stack() {
		elements = new Object[DEFAULT_INITIAL_CAPACITY];
	}
	
	public void push(Object e) {
		ensureCapcity();
		elements[size++] = e;
	}
	
	public Object pop() {
		if(size == 0) {
			throw new EmptyStackException();
		}
			return  elements[--size];	
	}
    	private void ensureCapcity() {
		if(elements.length == size) {
			elements = Arrays.copyOf(elements, 2 * size + 1);
		}
	}
}
```

위의 코드는 Stack을 구현한 코드이다. 이 코드에서의 문제점은 어디일까?

<br><br><br><br>
바로 pop() 메서드 내부에 답이 있다. 			
return  elements[--size]; 를 하여 elements의 사용가능한 '활성영역'은 줄어들게 되는데 그 다음 활성영역 바깥의 참조들이 그대로 남아있으므로, 가비지 컬렉터는 이를 알 방법이 없으므로 객체가 그대로 남아있게 된다. 이 문제가 계속해서 반복된다면 메모리 사용량이 늘어나 성능이 저하되고, 심할 때는 디스크 페이징이나 OutOfMemoryError를 일으키게 된다.(디스크 페이징은 메모리 공간이 부족할 경우 HDD나 SSD의 공간을 메모리로 활용하는 경우를 말한다. HDD, SSD가 메모리보다 느리기 때문에 성능이 떨어지게 된다.)

이를 해결하는 방법은 가비지 컬렉터가 더 이상 쓰지 않는 객체임을 알아챌 수 있게 아래와 같이 코드를 수정하는 것이다.

```java
public Object pop() {
    if (size == 0)
        throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null; // 이전 객체 참조 제거
    return result;
}
```
<br><br>


```java
elements[size] = null; 
```
이제는 객체 참조를 제거 했기 때문에 가비지 컬렉터가 동작하게 된다.

이처럼, 메모리를 직접 관리하는 클래스를 작성한다면 프로그래머가 메모리 누수에 주의를 더 주의해주어야 한다.
아래는 자바에 구현된 java.util 패키지에 구현된 Stack 클래스의 removeElementAt(int index) 메소드의 구현 내용이다.
```java
    public synchronized void removeElementAt(int index) {
        if (index >= elementCount) {
            throw new ArrayIndexOutOfBoundsException(index + " >= " +
                                                     elementCount);
        }
        else if (index < 0) {
            throw new ArrayIndexOutOfBoundsException(index);
        }
        int j = elementCount - index - 1;
        if (j > 0) {
            System.arraycopy(elementData, index + 1, elementData, index, j);
        }
        modCount++;
        elementCount--;
        elementData[elementCount] = null; /* to let gc do its work */
    }
```

사용되지 않는 객체는 null 처리를 하여, gc가 동작할 수 있도록 하는 것을 확인할 수 있다.

또한 계속해서 null처리하는 것 외에도 저자가 더 추천하는 방법이 있는데, 그건 변수의 유효한 범위를 최대한 좁혀서 정의하라는 것이다.

두 번째 메모리 누수를 일으키는 원인으로는, 객체 참조를 캐시에 넣어두고 사용하고 나서 이걸 잊고 그대로 놔두는 경우이다. 이 해결법으로는 WeakHashMap을 사용해 다 쓴 엔트리를 자동으로 제거되게 할 수 있다.

세 번째는 리스너, 콜백에서 발생하는데 콜백을 등록만하고 명확히 해지 않을 시에 콜백이 계속 쌓여가기만 한다. 이를 해결하기 위해 콜백을 약한 참조로 저장하면 된다.
```java
import java.util.Map;
import java.util.WeakHashMap;

interface EventListener {
    void onEvent(Event event);
}

class EventManager {
    // WeakHashMap을 사용하여 리스너를 관리
    private final Map<EventListener, Boolean> listeners = new WeakHashMap<>();

    public void registerListener(EventListener listener) {
        listeners.put(listener, Boolean.TRUE);
    }

    // WeakHashMap을 사용하므로 unregisterListener 메소드는 필요하지 않을 수 있음
    public void unregisterListener(EventListener listener) {
        listeners.remove(listener);
    }

    public void fireEvent(Event event) {
        for (EventListener listener : listeners.keySet()) {
            listener.onEvent(event);
        }
    }
}
```



약한 참조는 GC가 두 번 동작해야 깨끗하게 청소되므로, 신중하게 사용하여야 한다.
<br>
출처:https://blog.naver.com/kbh3983/221001751572