Cloneable 인터페이스는 원래는 mixin 인터페이스로 의도되었으나, 이에 실패하였다. 
그럼에도 불구하고 편의성 덕분에 널리 쓰이는 clone 메소드의 사용을 위해서
이번 아이템에서는 잘 동작하는 clone 메소드를 구현하는 방법을 알아본다.


## Cloneable 인터페이스

우선 Cloneable 인터페이스를 알아보자
- Cloneable 인터페이스에는 메소드가 없지만(Object 클래스에 있음), 이 인터페이스의 역할은 클래스가 clone 가능함을 알려주고, 또 clone() 메소드가 어떻게 동작할지를 결정하게 된다. 

- Cloneable을 구현하는 클래스라면 객체의 각각의 필드별로 복사본을 리턴하게 된다.(Object 에 있는 clone()의 동작)

- Cloneable 인터페이스를 구현하지 않으면서 clone() 메소드를 호출할 경우 `CloneNotSupportedException` 예외를 던진다.


## clone() 메소드의 규약

- x.clone() != x                            ->true

- x.clone().getClass() == x.getClass()       ->true

- x.clone().eqauls(x)       				->true, 필수는 아님.


## Cloneable 인터페이스가 필수가 아닌 경우

final 클래스에 `super.clone()`을 호출하지 않는 clone 메소드의 경우, Cloneable 인터페이스를 구현할 필요가 없음,

그러나 final 클래스가 아닐 경우에는 그 하위 클래스가 super.clone() 호출시 clone 메소드가 올바르게 동작하지 않음.

## clone() 메소드 예시

```java
package java_lab;

public class PhoneNumber implements Cloneable {
    private int areaCode;
    private int prefix;
    private int lineNumber;

    public PhoneNumber(int areaCode, int prefix, int lineNumber) {
        this.areaCode = areaCode;
        this.prefix = prefix;
        this.lineNumber = lineNumber;
    }

    @Override
    public PhoneNumber clone() {
        try {
            return (PhoneNumber) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError("클로닝 실패");
        }
    }

    @Override
    public String toString() {
        return String.format("(%03d) %03d-%04d", areaCode, prefix, lineNumber);
    }

    public static void main(String[] args) {
        PhoneNumber original = new PhoneNumber(123, 456, 7890);
        PhoneNumber cloned = original.clone();

        System.out.println("원본: " + original);
        System.out.println("복제: " + cloned);
    }
}

```
해당 예시는 PhoneNumber 클래스의 모든 필드가 기본 타입이거나 불변 객체를 참조하므로 얕은 복사여도 문제가 되지 않는다.
그리고 특히 `(PhoneNumber) super.clone();` 에서 
Object 타입을 PhoneNumber 타입으로 캐스팅하여 리턴해주는 것을 알 수 있다. -> API 사용성 향상

## Stack 클래스에서의 예시

```java
public class Stack {
	private Object[] elements;
	private int size = 0;
	private static final int DEFAULT_INITIAL_CAPACITY = 16;
    
	public Stack() {
		this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
	}
    
	public void push(Object e) {
		ensureCapacity();
	elements[size++] = e;
	}
    
	public Object pop() {
		if (size == 0)
			throw new EmptyStackException();
		Object result = elements[--size];
		elements[size] = null; 
        return result;
	}

	private void ensureCapacity() {
		if (elements.length == size)
			elements = Arrays.copyOf(elements, 2 * size + 1);
	}
}
```

이 클래스를 Cloneable 하게 하려면, cloneable 인터페이스를 구현하고 clone 메소드에서 super.clone()을 리턴해주면 되는데 elements 필드가 원래의 Stack 인스턴스와 같은 배열을 참조하게 되는 문제가 발생한다.

이를 해결하는 코드는 아래와 같다.
```java
	@Override 
	public Stack clone() {
		try {
			Stack result = (Stack) super.clone();
			result.elements = elements.clone();
			return result;
		} catch (CloneNotSupportedException e) {
			throw new AssertionError();
		}
}

```
가변 상태를 참조하는 경우에는 이렇게 stack의 내부를 복사하는 방식을 이용해야 한다.(재귀적)


## HashTable의 예시

```java
public class HashTable implements Cloneable {
    private Entry[] buckets; 

    private static class Entry {
        final Object key;
        Object value;
        Entry next;

        Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }

   
        Entry deepCopy() {
            return new Entry(key, value, next == null ? null : next.deepCopy());
        }
    }

    @Override
    public HashTable clone() {
        try {
            HashTable result = (HashTable) super.clone();
            result.buckets = new Entry[buckets.length];
            for (int i = 0; i < buckets.length; i++) {
                if (buckets[i] != null)
                    result.buckets[i] = buckets[i].deepCopy();
            }
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}

```

여기서는 
```java
        Entry deepCopy() {
            return new Entry(key, value, next == null ? null : next.deepCopy());
```
의 코드를 통해서 재귀적으로 깊은 복사를 한다. 이 때 재귀적 방법은 스택 오버플로우를 일으킬 수 있어서 반복문의 사용으로 이를 대체할 수 있다.

```java
Entry deepCopy() {
    Entry result = new Entry(key, value, next);
    for (Entry p = result; p.next != null; p = p.next) {
        p.next = new Entry(p.next.key, p.next.value, p.next.next);
    }
    return result;
}
```


## Cloneable 구현을 피하는 경우

- 상속을 고려한 클래스일 경우 피한다. 상속 받은 클래스가 clone() 메소드를 오버라이딩할 경우 clone()의 동작을 예측하기 어렵다.
- final 필드와의 충돌 (clone() 메소드에서는 final 필드의 값을 변경하지 못함)

## 스레드 안전성

스레드 안전한 클래스에서 Cloneable을 구현할 때, Object의 clone 메소드는 동기화되어 있지 않아서 clone 메소드를 오버라이딩 해야한다.
```java
@Override
public synchronized Object clone() throws CloneNotSupportedException {
    return super.clone();
}
```


## clone() 메소드 대신, 복사 생성자 / 복사 팩토리 메소드


앞서 살펴본 바와 같이 clone() 메소드를 사용하는데에는 큰 복잡성이 따르기 때문에 그 대안으로 복사 생성자, 복사 팩토리 메소드가 제시된다. 

- clone() 메소드에 비한 장점
	- 외부 언어의 객체 생성 매커니즘에 의존하지 않음.
	- 엄격한 규약을 요구하지 않음
	- final 필드와 충돌하지 않음
	- chcked exceptions를 던지지 않음
	- 캐스팅이 필요하지 않음
