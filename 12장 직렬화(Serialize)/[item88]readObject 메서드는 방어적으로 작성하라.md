## readObject 메서드는 방어적으로 작성하라

### 방어적 복사를 사용하는 불변 클래스
````java
public final class Period {
  private final Date start;
  private final Date end;

  /**
   * @param start 시작 시각
   * @param end 종료 시각, 시작 시각보다 뒤여야 한다.
   * @throws IllegalArgumentException 시작 시각이 종료 시각보다 늦을 때 발생한다.
   * @throws NullpointException start나 end가 null이면 발생한다.
   */
  public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime());
    if (this.start.compareTo(this.end) > 0) {
      throw new IllegalArgumentException(start + "가 " + end + "보다 늦다.");
    }
  }

  public Date start() { return new Date(start.getTime()); }
  public Date end() { return new Date(end.getTime()); }
  public String toString() { return start + " - " + end; }

  ...
}
````

<br>

위 클래스를 직렬화하기 위해 implements Serializable 추가  
이 경우 클래스의 주요한 불변식 보장이 되지 않음  
readObject 메서드 또한 다른 생성자처럼 인수의 유효성 검사와 방어적 복사 필수  
> 매개변수로 바이트 스트림을 받는 생성자  
> 보통의 경우 정상적인 인스턴스를 직렬화해 바이트 스트림 생성  
> 불변식을 깨뜨릴 악의적인 의도로 생성한 바이트 스트림을 매개변수로 받는 경우 문제 발생  

<br>

### 허용되지 않는 Period 인스턴스 생성 가능
````java
public class BogusPeriod {
  // 진짜 Period 인스턴스에서는 만들어질 수 없는 바이트 스트림
  private static final byte[] serializedForm = {
    (byte)0xac, (byte)0xed, 0x00, 0x05, 0x73, 0x72, 0x00, 0x06,
    0x50, 0x65, 0x72, 0x69, 0x6f, 0x64, 0x40, 0x7e, (byte)0xf8,
    0x2b, 0x4f, 0x46, (byte)0xc0, (byte)0xf4, 0x02, 0x00, 0x02,
    0x4c, 0x00, 0x03, 0x65, 0x6e, 0x64, 0x74, 0x00, 0x10, 0x4c,
    0x6a, 0x61, 0x76, 0x61, 0x2f, 0x75, 0x74, 0x69, 0x6c, 0x2f,
    0x44, 0x61, 0x74, 0x65, 0x3b, 0x4c, 0x00, 0x05, 0x73, 0x74,
    0x61, 0x72, 0x74, 0x71, 0x00, 0x7e, 0x00, 0x01, 0x78, 0x70,
    0x73, 0x72, 0x00, 0x0e, 0x6a, 0x61, 0x76, 0x61, 0x2e, 0x75,
    0x74, 0x69, 0x6c, 0x2e, 0x44, 0x61, 0x74, 0x65, 0x68, 0x6a,
    (byte)0x81, 0x01, 0x4b, 0x59, 0x74, 0x19, 0x03, 0x00, 0x00,
    0x78, 0x70, 0x77, 0x08, 0x00, 0x00, 0x00, 0x66, (byte)0xdf,
    0x6e, 0x1e, 0x00, 0x78, 0x73, 0x71, 0x00, 0x7e, 0x00, 0x03,
    0x77, 0x08, 0x00, 0x00, 0x00, (byte)0xd5, 0x17, 0x69, 0x22,
    0x00, 0x78
  };

  public static void main(String[] args) {
    Period p = (Period) deserialize(serializedForm);
    
    // Fri Jan 01 12:00:00 PST 1999 - Sun Jan 01 12:00:00 PST 1984
    System.out.println(p);
  }

  // 주어진 직렬화 형태(바이트 스트림)로부터 객체를 만들어 반환한다.
  static Object deserialize(byte[] sf) {
    try {
      return new ObjectInputStream(new BytearrayInputStream(sf)).readObject();
    } catch (IOException | ClassNotFoundException e) {
      throw new IllegalArgumentException(e);
    }
  }
}
````

<br>

### 유효성 검사를 수행하는 readObject 메서드
````java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
  s.defaultReadObject();

  if (start.compareTo(end) > 0) {
    throw new InvalidObjectException(start + "가 " + end + "보다 늦다.");
  }
}
````

<br>

### 가변 공격
````java
public class MutablePeriod {
  public final Period period;
  public final Date start;
  public final Date end;

  public MutablePeriod() {
    try {
      ByteArrayOutputStream bos = new ByteArrayOutputStream();
      ObjectOutputStream out = new ObjectOutputStream(bos);

      out.writeObject(new Period(new Date(), new Date()));

      /*
       * 악이적인 '이전 객체 참조', 즉 내부 Date 필드로의 참조를 추가한다.
       * 상세 내용은 자바 객체 직렬화 명세의 6.4절을 참고하자.
       */
      byte[] ref = { 0x71, 0, 0x7e, 0, 5 };
      bos.write(ref);
      ref[4] = 4;
      bos.write(ref);

      // Period 역직렬화 후 Date 참조를 '훔친다'.
      ObjectInputStream in = new ObjectInputStream(new ByteArrayInputStream(bos.toByteArray()));
      period = (Period) in.readObject();
      start = (Date) in.readObject();
      end = (Date) in.readObject();
    } catch (IOException | ClassNotFoundException e) {
      throw new AssertionError(e);
    }
  }
}

public static void main(String[] args) {
  MutablePeriod mp = new MutablePeriod();
  Period p = mp.period;
  Date pEnd = mp.end;

  // Wed Nov 22 00:21:29 PST 2017 - Wed Nov 22 00:21:29 PST 1978
  pEnd.setYear(78);
  System.out.println(p);

  // Wed Nov 22 00:21:29 PST 2017 - Wed Nov 22 00:21:29 PST 1969
  pEnd.setYear(69);
  System.out.println(p);
}
````

<br>

의도적으로 내부값을 수정할 수 있는 Period 인스턴스 확보  
해당 인스턴스가 불변이라고 가정하는 클래스에 넘겨 보안 문제 유도  
객체를 역직렬화할 때는 클라이언트가 소유해서는 안되는 객체 참조를 갖는 필드를 모두 방어적 복사  

<br>

### 방어적 복사와 유효성 검사를 수행하는 readObject 메서드
````java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
  s.defaultReadObject();

  start = new Date(start.getTime());
  end = new Date(end.getTime());

  if (start.compareTo(end) > 0) {
    throw new InvalidObjectException(start + "가 " + end + "보다 늦다.");
  }
}
````

<br>

방어적 복사를 유효성 검사보다 앞서 수행  
final 한정자는 방어적 복사가 불가능  
앞의 MutablePeriod 클래스도 힘을 쓰지 못함  

<br>

### 기본 메서드 사용 판단
transient 필드를 제외한 필드를 매개변수로 받아 유효성 검사 없이 대입하는 public 생성자를 추가해도되는가?  
답이 `아니오`라면 커스텀 readObject 메서드 생성  
모든 유효성 검사와 방어적 복사 수행  

<br>

