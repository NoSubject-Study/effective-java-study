# 표준예외를 사용해라
Favor The Use of Standard Exceptions 

## 표준 예외 재사용의 장점
- 내가 개발한 API를 다른 사람이 파악하고 사용하기 쉬워짐
- 내 API를 사용한 다른 프로그램도 낯선 예외를 사용하지 않게 되어 파악이 쉬워짐
- 예외 클래스 수가 적을 수록 메모리 사용량도 줄고 클래스 적재시간도 줄어듬

## 가장 많이 재사용되는 예외
1. IllegalArgumentException
  - 호출자가 인수로 부적절한 값을 넘길때 던지는 예외
  ``` java
  // java.lang.Object 클래스에 정의된 wait(long timeout, int nanos) 메소드로, 다중 스레드 환경에서 스레드 간 동기화를 제어하기 위해 사용
    public final void wait(long timeout, int nanos) throws InterruptedException {
        if (timeout < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }

        if (nanos < 0 || nanos > 999999) {
            throw new IllegalArgumentException(
                                "nanosecond timeout value out of range");
        }

        if (nanos > 0) {
            timeout++;
        }

        wait(timeout);
    }
```


2. IllegalStateException
   - 대상 객체의 상태가 호출된 메서드를 수행하기에 적합하지 않을 때 던지는 예외
  ``` java
private ThreadSafeLazyLoadedIvoryTower() {
 // protect against instantiation via reflection
 if (instance == null) {
  instance = this;
 } else {
  throw new IllegalStateException("Already initialized.");
 }
}
```
3. NullPointException
   - Null을 허용하지 않는 메서드에 null을 건네는 경우
   - IllegalArgumentException도 아예 다른 상황은 아니지만, 이런 특수한 경우엔 구분된 예외를 사용한다
``` java
    public String toLowerCase(Locale locale) {
        if (locale == null) {
            throw new NullPointerException();
        }
```
4. IndexOutOfBoundsException
   - 어떤 시퀀스의 허용범위가 넘는 값을 보낼 때
``` java
    public char charAt(int index) {
        if ((index < 0) || (index >= value.length)) {
            throw new StringIndexOutOfBoundsException(index);
        }
        return value[index];
    }
```  

5. ConcurrentModificationException
    - 단일 스레드에서 사용하려고 설계한 객체를 여러 스레드가 동시에 수정하려고 할 때
    - 동시 수정을 확실히 검출할 수 있는 안정된 방법은 없어, 문제가 생길 가능성을 알려주는 정도의 역할로 쓰임
``` java
    public final void remove() {
        Node<K,V> p = current;
        if (p == null)
            throw new IllegalStateException();
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
        current = null;
        K key = p.key;
        removeNode(hash(key), key, null, false, false);
        expectedModCount = modCount;
    }
```
6. UnsupportedOperationException
    - 클라이언트가 요청한 동작을 대상 객체가 지원하지 않을 때 던진다.

## Exception, RuntimeException, Throwable, Error는 직접 재사용하지 말자. 
이 클래스들은 추상클래스라고 생각하라. 여러 성격의 예외들을 포괄하는 상위 클래스이므로 안정적으로 테스트하기 어렵다. 

## 상황에 부합한다면 항상 표준 예외를 재사용하자.
- API 문서를 참고해 그 예외가 어떤 상황에서 던져지는지, 예외가 던져지는 맥락에 부합하는지 확인하고 사용하라.

