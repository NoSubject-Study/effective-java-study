## wait와 notify보다는 동시성 유틸리티를 애용하라

### 고수준 동시성 유틸리티
wait와 notify는 올바르게 사용하기가 아주 까다로우니 고수준 동시성 유틸리티를 사용  
고수준 유틸리티는 실행자 프레임워크, 동시성 컬렉션(concurrent collection), 동기화 장치(synchronizer) 세 범주로 분할 가능  

<br>

### 동시성 컬렉션
List, Queue, Map 같은 표준 컬렉션 인터페이스에 동시성을 가미해 구현한 고성능 컬렉션  
동시성을 무력화하는 것 불가능  
외부에서 락을 추가로 사용하면 오히려 성능 저하  
여러 기본 동작을 하나의 원자적 동작으로 묶는 `상태 의존적 수정` 메서드들 추가  

<br>

### ConcurrentMap 동시성 정규화 맵
````java
private static final ConcurrentMap<String, String> map = new ConcurrentHashMap<>();

public static String intern(String s) {
  String previousValue = map.putIfAbsent(s, s);
  return previousValue == null ? s : previousValue;
}
````

<br>

### ConcurrentMap 동시성 정규화 맵 최적화
````java
public static String intern(String s) {
  String result = map.get(s);
  if (result == null) {
    result = map.putIfAbsent(s, s);
    if (result == null) {
      result = s;
    }
  }
  return result;
}
````

<br>

동시성 컬렉션은 동기화한 컬렉션을 더 이상 사용하지 않도록 대체  
Collections.synchronizedMap보다는 ConcurrentHashMap 사용 권장  

<br>

### 실행자
컬렉션 인터페이스 중 일부는 작업이 완료될 때까지 대기하도록 확장  
Queue를 확장한 `BlockingQueue`에 take 메서드는 첫 원소 반환, 비어있는 경우 대기  
이런 특성 덕에 작업큐(생산자-소비자 큐)로 쓰기에 적합  
ThreadPoolExecutor를 포함한 대부분의 실행자 서비스 구현체로 사용  

<br>

### 동기화 장치
스레드가 다른 스레드를 대기할 수 있도록 하여, 서로 작업을 조율  
CountDownLatch, Semaphore, Phaser 존재(CyclicBarrier, Exchanger는 덜 쓰임)  

<br>

### CountDownLatch
일회성 장벽, 하나 이상의 스레드가 또 다른 하나 이상의 스레드 작업이 끝날 때까지 대기  
유일한 생성자는 정수값을 파라미터로, 이 값이 몇 번 호출해야 대기 중인 스레드를 깨우는지 결정  

<br>

### 동시 실행 타이머
````java
public static long time(Executor executor, int concurrency, Runnable action) throws InterruptedException {
  CountDownLatch ready = new CountDownLatch(concurrency);
  CountDownLatch start = new CountDownLatch(1);
  CountDownLatch done = new CountDownLatch(concurrency);

  for (int i = 0; i < concurrency; i++) {
    executor.execute(() -> {
      ready.countDown();
      try {
        start.await();
        action.run();
      } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
      } finally {
        done.countDown();
      }
    });
  }

  ready.await();
  long startNanos = System.nanoTime();
  start.countDown();
  done.await();
  return System.nonoTime() - startNanos;
}
````

<br>

### wait 메서드를 사용하는 표준 방식
````java
synchronized (obj) {
  // 락을 놓고, 깨어나면 다시 잡는다.
  while (condition) {
    obj.wait();
  }

  // 조건 충족된 경우 동작 수행
  ...
}
````

<br>

### 동기화 불변식 깨짐 사례
스레드가 notify 메서드를 호출한 다음 대기 중이던 스레드가 깨어나는 사이에 다른 스레드가 락을 얻어 그 락이 보호하는 상태 변경  
조건이 만족되지 않았음에도 다른 스레드가 실수로 혹은 악의적으로 notify 메서드를 호출  
깨우는 스레드는 지나치게 관대해서, 대기 중인 스레드 중 일부만 조건이 충족되어도 notifyAll() 메서드 호출  
대기 중인 스레드가 notify 메서드 없이도 깨어나는 `허위 각성`(spurious wakeup)  

<br>




