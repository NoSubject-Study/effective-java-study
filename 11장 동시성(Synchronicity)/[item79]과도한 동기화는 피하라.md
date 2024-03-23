## 과도한 동기화는 피하라

### 외계인 메서드(alien method)
응답 불가와 안전 실패를 피하려면 동기화 메서드나 동기화 블록 안에서는 제어를 절대로 클라이언트에 양도하면 안 된다.  
동기화된 영역 안에서는 `재정의 가능 메서드`나 `클라이언트가 넘겨준 함수 객체`를 호출하지 않는다.  

<br>

### 동기화 관찰자 코드
````java
public class ObservalbeSet<E> extends ForwardingSet<E> {
  public ObservableSet(Set<E> set) { super(set); }

  private final List<SetObserver<E>> observers = new ArrayList<>();

  public void addObserver(SetObserver<E> observer) {
    synchronized(observers) {
      observers.add(oberser);
    }
  }

  public boolean removeObserver(SetObserver<E> observer) {
    synchronized(observers) {
      return observers.remove(observer);
    }
  }

  private void notifyElementAdded(E element) {
    synchronized(observers) {
      for (SetObserver<E> observer : observers) {
        observer.added(this, element);
      }
    }
  }

  @Override 
  public boolean add(E element) {
    boolean added = super.add(element);
    if (added) {
      notifyElementAdded(element);
    }
    return added;
  }

  @Override
  public boolean addAll(Collection<? extends E> c) {
    boolean result = false;
    for (E element : c) {
      result |= add(element);
    }
    return result;
  }

}

@FunctionalInterface
public interface SetObserver<E> {
  void added(ObservableSet<E> set, E element);
}

````
<br>

### 동기화 코드 속 외계인 메서드 호출
````java
public static void main(String[] args) {
  ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());
  set.addObserver(new SetObserver<>() {
    public void added(ObservableSet<Integer> s, Integer e) {
      System.out.println(e);

      // ConcurrentModificationException 오류 발생
      if (e == 23) {
        s.removeObserver(this);
      }
    }
  });
  
  for (int i = 0; i < 100; i++) {
    set.add(i);
  }
}
````

<br>

### 교착상태 - 백그라운드 스레드 사용 관찰자
````java
set.addObserver(new SetObserver<>() {
  public void added(ObservableSet<Integer> s, Integer e) {
    System.out.println(e);
    if (e == 23) {
      ExecutorService exec = Executors.newSingleThreadExecutor();

      try {
        exec.submit(() -> s.removeObserver(this)).get();
      } catch (ExecutionException | InterruptedException ex) {
        throw new AssertionError(ex);
      } finally {
        exec.shutdown();
      }
    }
  }
});
````

<br>

위 예제에서 동기화 영역이 보호하는 자원(관찰자)은 외계인 메서드가 호출될 때 일관된 상태  
자바 언어의 락은 `재집입`(reentrant)을 허용하므로 교착상태에 빠지지는 않는다.  
재진입 가능 락은 객체 지향 멀티스레드 프로그램을 쉽게 구현 가능  
하지만 `응답 불가`(교착 상태)가 될 상황을 `안전 실패`(데이터 훼손)로 변모시킬 가능성 존재  

<br>

### 열린 호출 - 동기화 블록 위치 변경
````java
private void notifyElementAdded(E element) {
  List<SetObserver<E>> snapshot = null;
  synchronized(observers) {
    snapshot = new ArrayList<>(observers);
  }
  for (SetObserver<E> observer : snapshot) {
    observer.added(this, element);
  }
}
````

<br>

### 동시성 컬렉션 라이브러리 사용
````java
private final List<SetObserver<E>> observers = new CopyOnWriteArrayList<>();

public void addObserver(SetObserver<E> observer) {
  observers.add(observer);
}

public boolean removeObserver(SetObserver<E> observer) {
  return observers.remove(observer);
}

private void notifyElementAdded(E element) {
  for (SetObserver<E> observer : observers) {
    observer.added(this, element);
  }
}
````
<br>

열린 호출은 실패 방지 효과 외에도 동시성 효율을 크게 개선  
기본 규칙은 동기화 영역에서는 가능한 일을 적게 하는 것  

<br>

### 성능
동기화가 초래하는 진짜 비용은 락을 얻는데 드는 CPU 시간이 아님  
병렬로 실행할 기회를 잃고 모든 코어가 메모리를 일관되게 보기 위한 지연시간  
가상 머신의 최적화를 제한한다는 점 또한 비용  

<br>

### 가변 클래스 사용 규칙

1. 동기화를 전혀 하지 않고, 그 클래스를 동시에 사용해야하는 클래스가 외부에서 알아서 동기화하게 강요  
2. 동기화를 내부에서 수행해 스레드 안전한 클래스로 생성  
  
java.util은 첫 번째 방식을, java.util.concurrent는 두 번째 방식을 취함  
StringBuffer 인스턴스는 항상 단일 스레드에 사용됐지만 내부적으로 동기화  
뒤늦게 `StringBuilder` 등장(동기화하지 않은 `StringBuffer`)  
비슷한 이유로 `Random`과 `ThreadLocalRandom` 객체 존재  

<br>

### 동기화 기법
클래스 내부를 동기화하기로 결정했다면 다양한 기법 중 차선책 사용  
락 분할(lock splitting), 락 스트라이핑(lock striping), 비차단 동시성 제어(nonblocking concurrency control) 등  

<br>


