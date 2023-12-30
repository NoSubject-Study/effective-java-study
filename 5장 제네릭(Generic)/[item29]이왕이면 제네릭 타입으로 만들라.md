## 이왕이면 제네릭 타입으로 만들라

### 배열을 제네릭으로 변경하는 방법  
1. E[] 배열 사용  
생성할 때 한번만 형변환  
힙 오염(heap pollution) 발생 가능성  
	
2. 직접 증명  
원소를 꺼내서 사용할 때마다 형변환 필요  

### Object 기반 스택  
```` java
public class Stack {
  private Object[] elements;
  private int size = 0;
  private static final int DEFAULT_INITIAL_CAPACITY = 16;

  public Stack() {
  	elements = new Object[DEFAULT_INITIAL_CAPACITY];
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

  public boolean isEmpty() {
  	return size == 0;
  }

  private void ensureCapacity() {
  	if (elements.length == size)
  	  elements = Arrays.copyOf(elements, 2 * size + 1);
  }

}
````

### E[] 배열 사용  
```` java
public class Stack<E> {
  private E[] elements;
  private int size = 0;
  private static final int DEFAULT_INITIAL_CAPACITY = 16;

  @SuppressWarnings("unchecked")
  public Stack() {
  	elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
  }

  ...

}
````

### 직접 증명  
```` java
public class Stack<E> {
	private Object[] elements;

	...

	public E pop() {
	  if (size == 0)
	    throw new EmptyStackException();
	  @SuppressWarnings("unchecked") E result = (E) elements[--size];
	  elements[size] = null;
	  return result;
	}

}
````



