## 스레드 안전성 수준을 문서화하라

### API 동기화
메서드 선언에 `synchronized` 한정자를 선언할지는 구현 이슈  
API에는 속하지 않는 문제  
멀티스레드 환경에서도 API를 안전하게 사용하게 하려면 클래스가 지원하는 스레드 안정성 수준을 정확히 명시  

<br>

### 스레드 안전성
1. 불변(immutable)  
이 클래스의 인스턴스는 마치 상수와 같아서 외부 동기화 필요없음(String, Long, BigInteger 등)  

2. 무조건적 스레드 안전(unconditionally thread-safe)  
이 클래스의 인스턴스는 수정 가능, 내부에서 충분히 동기화(AtomicLong, ConcurrentHashMap 등)  

3. 조건부 스레드 안전(conditionally thread-safe)  
무조건적 스레드 안전과 같으나 일부 메서드는 외부 동기화 필요(Collections.synchronized 래퍼 메서드 반환 컨렉션)  

4. 스레드 안전하지 않음(not thread-safe)  
이 클래스의 인스턴스는 수정 가능  
동시 사용을 위해 외부 동기화 매커니즘 필수(ArrayList, HashMap 등)  

5. 스레드 적대적(thread-hostile)  
이 클래스는 모든 메서드 호출을 외부 동기화로 감싸더라도 멀티스레드 환경에서 안전하지 않음  
이 수준의 클래스는 일반적으로 정적 데이터를 아무 동기화 없이 수정  

<br>

### 스레드 안전성 어노테이션
위의 분류는 대략 일치(스레드 적대적 제외)  
@Immutable, @ThreadSafe, @NotThreadSafe  

<br>

### Collections.synchronizedMap API 명세
````java
synchronizedMap이 반환한 맵의 컬렉션 뷰를 순회하려면 반드시 그 맵을 락으로 사용해 수동으로 동기화하라.

Map<K, V> m = Collections.synchronizedMap(new HashMap<>());
Set<K> s = m.keySet();  // 동기화 블록 밖에 있어도 된다.
...
synchronized(m) { // s가 아닌 m을 사용해 동기화해야 한다!
  for (K key : s)
    key.f();
}

이대로 따르지 않으면 동작을 예측할 수 없다.
````

<br>

### 공개락
클래스가 외부에서 사용할 수 있는 락을 제공  
클라이언트에서 일련의 메서드 호출을 원자적으로 수행 가능  
내부에서 처리하는 고성능 동시성 제어 메커니즘과 혼용 불가  
공개된 락을 계속 보유하여 서비스 거부 공격(`denial-of-service attack`) 가능  

<br>

### 비공개락
클래스 바깥에서 그 객체의 동기화에 관여 불가  
락 객체를 동기화 대상 객체 안으로 캡슐화한 것  
무조건적 스레드 안전 클래스에서만 사용 가능  
상속용 클래스에 적합  

<br>

### 비공개 락 객체 관용구 - 서비스 거부 공격 방지
````java
private final Object lock = new Object();

public void foo() {
  synchronized(lock) {
    ...
  }
}
````

<br>
