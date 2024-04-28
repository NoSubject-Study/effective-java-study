## 프로그램의 동작을 스레드 스케줄러에 기대지 말라

정확성이나 성능이 스레드 스케줄러에 따라 달라지는 프로그램은 다른 플랫폼 이식이 어려움  
실행 가능한 스레드의 평균적인 수를 프로세서 수보다 지나치게 많아지지 않도록 구현  
스레드는 당장 처리할 작업이 없는 경우 실행돼서는 안된다(busy waiting)  

<br>

### 바쁜 대기
````java
public class SlowCountDownLatch {
  private int count;

  public SlowCountDownLatch(int count) {
    if (count < 0) {
      throw new IllegalArgumentException(count + " < 0");
    }
    this.count = count;
  }

  public void await() {
    while (true) {
      synchronized(this) {
        if (count == 0) {
          return;
        }
      }
    }
  }

  public synchronized void countDown() {
    if (count != 0) {
      count--;
    }
  }
}
````

<br>


