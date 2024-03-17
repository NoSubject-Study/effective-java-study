## 공유 중인 가변 데이터는 동기화해 사용하라

### 동기화
`synchronized` 키워드는 해당 메서드나 블록을 한번에 한 스레드씩 수행하도록 보장  
배타적 동기화 실행, 즉 한 스레드가 변경하는 중에 상태 관리를 위해 다른 스레드의 접근을 막는 용도로만 생각  
동기화 없이는 한 스레드가 만든 변화를 다른 스레드에서 확인하지 못 할 가능성 존재  
언어 명세상 `long`과 `double` 외의 변수를 읽고 쓰는 동작은 원자적(`atomic`)  
하지만 원자적 데이터를 읽고 쓸 때는 동기화하지 않는 것은 위험한 발상  
동기화는 배타적 실행뿐 아니라 스레드 사이의 안정적인 통신에 필수적(자바 메모리 모델에 영향받음)  
가변 데이터는 공유하지 않고 단일 스레드에서만 쓰는 것 권장  

<br>

### 잘못된 코드
````java
public class StopThread {
  private static boolean stopRequested;

  public static void main(String[] args) throws InterruptedException {
    Thread backgroundThread = new Thread(() -> {
      int i = 0;
      while (!stopRequested)
        i++;
    });
    backgroundThread.start();

    TimeUnit.SECONDS.sleep(1);
    stopRequested = true;
  }
}
````

<br>

### OpenJDK 서버 VM 끌어올리기(hosting) 최적화 기법
````java
// 원래 코드
while(!stopReqeusted)
  i++;

// 최적화 코드
if (!stopRequested)
  while (true)
    i++;
````

<br>

### 적절한 동기화 - 쓰기/읽기 모두 동기화
````java
public class StopThread {
  private static boolean stopRequested;

  private static synchronized void requestStop() {
    stopRequested = true;
  }

  private static synchronized boolean stopRequested() {
    return stopRequested;
  }

  public static void main(String[] args) throws InterruptedException {
    Thread backgroundThread = new Thread(() -> {
      int i = 0;
      while (!stopRequested())
        i++;
    });
    backgroundThread.start();

    TimeUnit.SECONDS.sleep(1);
    requestStop();
  }
}
````

<br>

### volatile 한정자 선언
````java
public class StopThread {
  private static volatile boolean stopRequested;

  public static void main(String[] args) throws InterruptedException {
    Thread backgroundThread = new Thread(() -> {
      int i = 0;
      while (!stopRequested())
        i++;
    });
    backgroundThread.start();

    TimeUnit.SECONDS.sleep(1);
    requestStop();
  }
}
````

<br>

### Atmoic
java.util.concurrent.atomic 패키지  
락 없이도(`lock-free`) 스레드 안전한 프로그래밍 지원  
`volatile`은 동기화의 통신만 지원하지만, 이 패키지는 원자성(배타적 실행)까지 지원  

<br>

### 락-프리 동기화
````java
private static final AtomicLong nextSerialNum = new AtomicLong();

public static long generateSerialNumber() {
  return nextSerialNum.getAndIncrement();
}
````

<br>


