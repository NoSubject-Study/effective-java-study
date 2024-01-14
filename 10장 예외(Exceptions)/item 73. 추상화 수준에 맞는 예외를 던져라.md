# 추상화 수준에 맞는 예외를 던져라
Throw exceptions appropriate to the abstractions

### Bad Practice 
수행하는 일과 관련 없어 보이는 예외 발생 시키는 경우
- 상위 수준 레벨의 개발자가 당황스러울 수 있다.
- 이 예외를 처리해야 하는 상위 레벨의 API가 오염될 수 있다.


## 1. Exception Translation
- 예외 번역이란? 상위 계층에서 저수준 예외를 잡아 자신의 추상화 수준에 맞는 예외로 바꿔 던지는 것
```java
try{
    ...
} catch (LowerLevelException e) {
    throw new HigherLevelException("...");
}
```
```java
public abstract class AbstractSequentialList<E> extends AbstractList<E> {

    /**
     * Returns the element at the specified position in this list.
     *
     * @param index index of the element to return
     * @return the element at the specified position in this list
     * @throws IndexOutOfBoundsException if the index is out of range
     *         (index < 0 || index >= size())
     */
    public E get(int index) {
        try {
            return listIterator(index).next();
        } catch (NoSuchElementException exc) {
            throw new IndexOutOfBoundsException("Index: " + index);
        }
    }
}
```

👉 그런데 저수준 예외가 디버깅할때 필요하다면? 예외 연쇄(Execption Chaining)

## 2. Exception Chaining
- 예외 연쇄란? 문제의 근본 원인인 저수준예외를 고수준예외에 실어 보내서 디버깅할 때 사용할 수 있도록 하는 방식
```java
// exception chaining 예시
try{
    ...
} catch (LowerLevelException e) {
    throw new HigherLevelException(e);
}
```
```java
public class OrderProcessingException extends Exception {
    public OrderProcessingException(String message, Throwable cause) {
        super(message, cause);
    }
}

public class OrderProcessor {
    public void processOrder(Order order) throws OrderProcessingException {
        try {
            // 여기서 주문 처리 로직을 수행합니다.
            if (order.getTotalAmount() <= 0) {
                throw new IllegalArgumentException("주문 금액은 0보다 커야 합니다.");
            }
            // 주문 처리 성공
            System.out.println("주문이 성공적으로 처리되었습니다.");
        } catch (IllegalArgumentException e) {
            // 주문 처리 중 실패 시 OrderProcessingException을 던집니다.
            throw new OrderProcessingException("주문 처리 중 오류 발생", e);
        }
    }
}
```

- 📌 예외 연쇄는 문제의 원인을 (Throwable의 getCause 메서드로) 프로그램에서 접근할 수 있게 해주며, 원인과 고수준 예외의 스택 추적 정보 통합해준다.
- 📌 대부분의 표준예외는 예외 연쇄용 생성자를 갖추고 있음    
  (그렇지 않아도 Throwable의 initCause 메소드를 이용해 원인을 직접 못받을 수 있다.)
- 📌 예외를 전파하는 것보다 예외 번역하는 것이 우수한 방법이지만 남용해서는 안된다.
- 📌 가능하다면 저수준 메서드가 반드시 성공하도록하여 아래 계층에서는 예외가 발생하지 않도록 하는 것이 최선이다.
- 차선책
  - 상위 계층 메서드의 매개변수 값을 아래 계층 메서드로 건내기 전에 미리 검사하는
  - 아래 계층에서의 예외를 피할 수 없다면, 상위 계층에서 그 예외를 조용히 처리하여 문제를 API 호출자에까지 전파하지 않는 방법
    이러한 경우 java.util.logging 같은 적절한 로깅 기능을 활용하여 기록하는 것이 좋다.
    클라이언트 코드와 사용자에게 문제를 전파하지 않으면서 프로그래머가 로그를 분석해 추가 조치를 취할 수 있도록 해주기 때문이다.

## 정리
- 예외 번역 : 아래 계층의 예외를 아래 계층에서 예방하거나 처리할 수 없어서 위로 보내야하지만, 상위 계층에서 그대로 노출하기 곤란할 때
- 예외 연쇄 : 상위 계층의 맥락에 어울리는 고수준 예외를 던지면서 근본 원인도 함께 알려주어 오류를 분석할 수 있도록 하고 싶을 때



### (+)
💡 Data Exception Wrapping 
- 예외를 래핑 또는 감싸는 방법을
- 예외를 좀 더 추상화하거나 예외를 다른 예외로 변환하여 예외 처리를 보다 구조적으로 하고, 예외 정보를 보다 명확하게 전달하는 데 사용
  - 추상화: 원래 예외를 래핑함으로써 예외를 추상화. 이로써 클라이언트 코드에서는 구체적인 예외를 처리하는 대신 더 일반적인 예외 유형을 처리가능
  - 예외 정보 전달: 데이터 예외 래핑을 사용하면 원래 예외에 추가적인 정보를 포함가능. 이렇게 하면 예외 발생 시 어떤 데이터나 문제에 대한 자세한 내용을 기록하고 전달가능
  - 예외 포워딩: 다른 예외 유형으로 변환하여 예외 처리의 흐름을 변경 가능. 예를 들어, 더 구체적인 예외를 래핑하여 클라이언트 코드가 더 적절하게 처리
    
http://www.javapractices.com/topic/TopicAction.do?Id=77

