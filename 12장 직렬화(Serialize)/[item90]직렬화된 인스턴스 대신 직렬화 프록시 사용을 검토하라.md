## 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라

### 직렬화 프록시 패턴(serialization proxy pattern)
바깥 클래스의 논리적 상태를 정밀하게 표현하는 중첩 클래스를 설계한 후 `private static`으로 선언  
이 중첩 클래스가 바깥 클래스의 직렬화 프록시  
중첩 클래스 생성자는 단 하나, 바깥 클래스를 매개변수로 사용  
바깥 클래스와 중첩 클래스 모두 Serializable 구현 선언  
가짜 바이트 스트림 공격, 내부 필드 탈취 공격 차단  

<br>

### Period 클래스용 직렬화 프록시
````java
public class Period implements Serializable {
  private final Date start;
  private final Date end;

  // 직렬화 시스템은 절대 바깥 클래스의 직렬화된 인스턴스 생성 불가
  private Object writeReplace() {
    return new SerializationProxy(this);
  }

  // 직렬화 프록시 패턴용
  private void readObject(ObjectInputStream stream) throws InvalidObjectException {
    throw new InvalidObjectException("프록시가 필요합니다.");
  }

  // 중첩 클래스
  private static class SerializationProxy implements Serializable {
    private final Date start;
    private final Date end;

    SerializationProxy(Period p) {
      this.start = p.start;
      this.end = p.end
    }

    private static final long serialVersionUID = 234098243823485285L;

    private Object readResolve() {
      return new Period(start, end);
    }
  }
}
````

<br>

### EnumSet 직렬화 프록시
````java
private static class SerializationProxy <E extends Enum<E>> implements Serializable {
  private final Class<E> elementType;
  private final Enum<?> eleemnts;

  SerializationProxy(EnumSet<E> set) {
    elementType = set.elementType;
    elements = set.toArray(new Enum<?>[0]);
  }

  private Obejct readResolve() {
    EnumSet<E> result = EnumSet.noneOf(elementType);
    for (Enum<?> e : elements)
      result.add((E) e);
    return result;
  }

  private static final long serialVersionUID = 362491234563181265L;

}
````

<br>

### 직렬화 프록시 패턴 한계
1. 클라이언트가 멋대로 확장할 수 있는 클래스에 적용 불가
2. 성능 이슈

<br>


