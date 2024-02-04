# 가능한 한 실패 원자적으로 만들라
Strive for failure atomicity

## 실패 원자적 (failure-atomic)
### 호출된 메서드가 실패하더라도 해당 객체는 메서드 호출 전 상태를 유지해야 한다. 
작업 도중 예외가 발생해도, 그 객체는 여전히 정상적으로 사용할 수 있는 상태가 되도록

## 실패 원자적으로 설계하는 방법들
### 1) 불변 객체 (immutable objects) 로 만든다 (item 17 참고)
불변 객체의 상태는 생성 시점에 고정되어 절대 변하지 않기 때문에, 태생적으로 실패 원자적임.

### 2) 가변 객체 -  작업 수행에 앞서 매개변수의 유효성을 검사
매개변수의 유효성을 검사해서 객체의 내부 상태를 변경하기 전에 잠재적 예외의 가능성을 대부분 걸러낼 수 있는 방법. 가장 흔하다
``` java
public Object pop() {
    if (size == 0)
        throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null; //다 쓴 참조 해제
    return result;
}
```

### 3) 실패할 가능성이 있는 모든 코드를 객체 상태를 바꾸는 코드 전에 배치
``` java
    public V put(K key, V value) {
        Entry<K,V> t = root;
        if (t == null) {
            compare(key, key); // type (and possibly null) check

            root = new Entry<>(key, value, null);
            size = 1;
            modCount++;
            return null;
        }
        ...
    }
```
TreeMap을 예시로 들면, 원소를 추가할 때 비교하기 위해서는 추가하려는 원소가 트리에 있는 원소와 비교할 수 있는 타입이어야 한다. 
원소를 넣기 전에 미리 체크를 하면 객테의 상태 변경을 막을 수 있다.

### 4) 객체의 임시 복사번에서 작업을 수행한 다음, 작업이 성공적으로 완료되면 원래 객체와 교체한다
``` java
    default void sort(Comparator<? super E> c) {
        Object[] a = this.toArray(); // 입력 리스트의 원소들을 배열로 옮겨 담음
        Arrays.sort(a, (Comparator) c);
        ListIterator<E> i = this.listIterator();
        for (Object e : a) {
            i.next();
            i.set((E) e);
        }
    }
```
정렬을 수행하기 전, 입력 리스트의 원소들을 배열로 옮겨 담는다.    
복제본에 옮기는 것이 성능을 위해서 (배열 사용시 반복문에서 원소들에 훨씬 빠르게 접근 가능) 만든 결정이지만, 혹시나 정렬에 실패해도 입력리스트는 변하지 않는 효과를 얻을 수 있음.


### 5) 작업 도중 발생하는 실패를 가로채는 복구 코드를 작성하여 작업 전 상태로 되돌린다
주로 디스크 기반의 내구성을 보장해야 하는 자료구조에 쓰이지만, 자주 쓰이는 방법은 아니다. 


## 실패 원자성은 항상 달성할 수 있는 것은 아니다. 
실패 원자성은 권장되지만, 항상 달성할 수 있는 것은 아니다. 

- 예를 들어 두 쓰레드가 동기화 없이 같은 객체에 접근하여 수정한다면 그 객체의 일관성이 깨질 수 있다. 따라서 ConcurrentModificationException을 잡아냈다고 해서 그 객체가 여전히 쓸 수 있는 상태라고 가정해서는 안 된다.
- Error가 발생하면 복구할 수 없으므로, 실패 원자적으로 만드려는 시도도 하지 않아도 된다.
- 실패 원자성을 달성하기 위한 비용이나 복잡도가 큰 경우엔 트레이드오프를 잘 고려해서 설계한다.

📌 메서드 명세에 기술한 예외라면, 예외가 발생해도 객체의 상태는 메서드 호출 전과 똑같ㅌ이 유지해줘야 하는 것이 기본 규칙이다!    
이 규칙을 지키지 못한다면 문서에 실패시 객체 상태를 명시해주어야 한다.
