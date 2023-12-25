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
