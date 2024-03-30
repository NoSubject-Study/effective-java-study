## 스레드보다는 실행자, 태스크, 스트림을 애용하라

### 실행자
java.util.concurrent 패키지의 실행자 프레임워크(Executor Framework)  
특정 태스크가 완료되기를 대기(`get`)  
태스크 모음 중 아무것 하나(`invokeAny`) 혹은 모든 태스크(`invokeAll`)가 완료되기를 대기  
실행자 서비스의 종료 대기(`awaitTerminate`)  
완료된 태스크들의 결과를 순차적으로 리턴(`ExecutorCompletionService`)  
태스크를 특정 시간 또는 주기적 실행(`ScheduledThreadPoolExecutor`)  

<br>

### 실행자 사용
````java
ExecutorService exec = Executor.newSingleThreadExecutor();
exec.execute(runnable);
exec.shutdown();
````

<br>

### 실행자 특징
ThreadPoolExecutor: 커스텀 실행자에 적합, 스레드 풀 동작 속성 설정 가능  
CachedThreadPool: 작은 프로그램 또는 가벼운 서버, 가용한 스레드가 없을시 새로 생성  
FixedThreadPool: 무거운 서버, 스레드 개수를 고정  
ForkJoinPool: ForkJoinTask 인스턴스 실행에 적합  

<br>

### 태스크
작업 단위를 나타내는 핵심 추상 개념  
`Runnable`: 반환값 없음  
`Callable`: 반환값 존재, 예외 발생 가능  

<br>
