## 인스턴스 수를 통제해야 한다면 readResolve보다는 열거 타입을 사용하라

### 싱글톤 보장 클래스
````java
public class Elvis {
  public static final Elvis INSTANCE = new Elvis();
  private Elvis() { ... }

  public void leaveTheBuilding() { ... }
}
````

<br>

implements Serializable 선언을 하는 순간 싱글톤 기능 상실  
명시적인 readObject 메서드를 사용해도 초기화될 때 만들어진 인스턴스와는 별개인 인스턴스 반환  
`readResolve` 메서드를 사용하면 `readObject` 메서드가 생성한 인스턴스를 다른 것으로 대체 가능  

<br>

### 인스턴스 통제를 위한 readResolve 메서드 - 개선 가능
````java
private Object readResolve() {
  // 싱글톤 인스턴스 반환, 새로 생성된 인스턴스는 가비지 컬렉터가 처리
  return INSTANCE;
}
````

<br>

인스턴스 통제 목적으로 readResolve 메서드 사용시 객체 참조 타입 인스턴스 필드는 모두 `transient` 선언 필수  
그렇지 않으면 readResolve 메서드 수행전에 역직렬화 객체 참조 공격 가능성 존재  
> 싱글톤이 transient 선언이 아닌 참조 필드인 경우, readResolve 메서드 전에 역직렬화
> 조작된 스트림을 사용해서 해당 참조 필드의 내용이 역직렬화된 시점에 인스턴스 참조를 훔침

<br>

### 잘못된 싱글톤 - 도둑 클래스
````java
public class Elvis implements Serializable {
  public static final Elvis INSTANCE = new Elvis();
  private Elvis() {}

  private String[] favoriteSongs = { "Hound Dog", "Heartbreak Hotel" };
  public void printFavorites() {
    System.out.println(Arrays.toString(favoriteSongs));
  }

  private Object readResolve() {
    return INSTANCE;
  }
}

public class ElvisStealer implements Serializable {
  static Elvis impersonator;
  private Elvis payload;

  private Object readResolve() {
    // resolve 전의 Elvis 인스턴스 참조 저장
    impersonator = payload;

    // favoriteSongs 필드에 맞는 타입 객체 반환
    return new String[] { "A Fool Such as I" };
  }
  private static final long serialVersionUID = 0L;
}


public class ElvisImpersonator {
  
  private static final byte[] serializedForm = {
    (byte)0xac, (byte)0xed, 0x00, 0x05, 0x73, 0x72, 0x00, 0x05,
    0x45, 0x6c, 0x76, 0x69, 0x73, (byte)0x84, (byte)0xe6,
    (byte)0x93, 0x33, (byte)0xc3, (byte)0xf4, (byte)0x8b,
    0x32, 0x02, 0x00, 0x01, 0x4c, 0x00, 0x0d, 0x66, 0x61, 0x76,
    0x6f, 0x72, 0x69, 0x74, 0x65, 0x53, 0x6f, 0x6e, 0x67, 0x73,
    0x74, 0x00, 0x12, 0x4c, 0x6a, 0x61, 0x76, 0x61, 0x2f, 0x6c,
    0x61, 0x6e, 0x67, 0x2f, 0x4f, 0x62, 0x6a, 0x65, 0x63, 0x74,
    0x3b, 0x78, 0x70, 0x73, 0x72, 0x00, 0x0c, 0x45, 0x6c, 0x76,
    0x69, 0x73, 0x53, 0x74, 0x65, 0x61, 0x6c, 0x65, 0x72, 0x00,
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x02, 0x00, 0x01,
    0x4c, 0x00, 0x07, 0x70, 0x61, 0x79, 0x6c, 0x6f, 0x61, 0x64,
    0x74, 0x00, 0x07, 0x4c, 0x45, 0x6c, 0x76, 0x69, 0x73, 0x3b,
    0x78, 0x70, 0x71, 0x00, 0x7e, 0x00, 0x02
  };

  public static void main(String[] args) {
    // ElvisStealer.impersonator 초기화
    // Elvis 인스턴스를 반환
    Elvis elvis = (Elvis) deserialize(serializedForm);
    Elvis impersonator = ElvisStealer.impersonator;

    // [Hound Dog, Heartbreak Hotel]
    elvis.printFavorites();

    // [A Fool Such as I]
    impersonator.printFavorites();
  }
}
````

<br>

### 열거 타입 싱글톤
````java
public enum Elvis {
  INSTANCE;

  private String[] favoriteSongs = { "Hound Dog", "Heartbreak Hotel" };
  public void printFavorites() { System.out.println(Arrays.toString(favoriteSongs)) };
}
````

