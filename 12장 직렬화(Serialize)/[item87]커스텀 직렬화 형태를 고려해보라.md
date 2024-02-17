## 커스텀 직렬화 형태를 고려해보라

### 기본 직렬화 사용
기본 직렬화 형태는 유연성, 성능, 정확성 측면에서 신중히 고민한 수 합당할 때만 사용한다.  
이상적인 직렬화 형태라면 물리적인 모습과 독립된 논리적인 모습만 표현해야한다.  
즉, 객체의 물리적 표현과 논리적 내용이 같다면 기본 직렬화 형태라도 무방하다.  
기본 직렬화 형태가 적합하다고 결정했더라도 불변식 보장과 보안을 위해 readObject 메서드 제공할 때가 많다.  
기본 직렬화 내부 필드가 모두 private라도 직렬화 형태에 포함되는 경우 모두 문서화해야한다.  
자바독을 이용할 경우 _`@serial`_ 태그를 이용한다.  

<br>

### 기본 직렬화 형태에 적합한 클래스
````java
public class Name implements Serializable {
  
  /**
   * 성. null이 아니어야 함.
   * @serial
   */
  private final String lastName;

  /**
   * 이름. null이 아니어야 함.
   * @serial
   */
  private final String firstName;

  /**
   * 중간이름. 중간이름이 없다면 null.
   * @serial
   */
  private final String middleName;

  ...
}
````

<br>

### 기본 직렬화 형태에 적합하지 않은 클래스
````java
public final class StringList implements Serializable {
  private int size = 0;
  private Entry head = null;

  private static class Entry implements Serializable {
    String data;
    Entry next;
    Entry previous;
  }

  ...
}
````

<br>

### 객체의 물리적 표현과 논리적 표현의 차이가 큰 경우
1. 공개 API가 현재의 내부 표현 방식에 영구히 묶인다.  
  StringList.Entry가 공개  
  이후 릴리스에서 StringList 내부 동작을 변경하더라도 기존의 연결리스트 표현도 제공 필수  

2. 너무 많은 공간을 차지할 수 있다.  
  StringList의 모든 엔트리와 연결 정보까지 기록  
  엔트리와 연결정보는 내부 구현에 해당하기 때문에 직렬화 형태에 포함할 가치 없음  

3. 시간이 너무 많이 걸릴 수 있다.  
  객체 그래프의 위상에 관한 정보가 없으니 그래프를 직접 순회  

4. 스택 오버플로를 일으킬 수 있다.  
  객체 그래프를 재귀 순회  

<br>

### 합리적인 커스텀 직렬화 형태
````java
public final class StringList implements Serializable {
  private transient int size = 0;
  private transient Entry head = null;

  private static class Entry {
    String data;
    Entry next;
    Entry previous;
  }

  //지정한 문자열을 이 리스트에 추가한다.
  public final void add(String s) { ... }

  /**
   * 이 {@code StringList} 인스턴스를 직렬화한ㄷ.
   *
   * @serialData 이 리스트의 크기(포함된 문자열의 개수)를 기록한 후 
   * ({@code int}), 이어서 모든 원소를(각각은 {@code String})
   * 순서대로 기록한다.
   */
  private void writeObject(ObjectOutputStream s) throws IOException {
    s.defaultWriteObject();
    s.writeInt(size);

    //모든 원소를 올바른 순서로 기록한다.
    for (Entry e = head; e != null; e = e.next) {
      s.writeObject(e.data);
    }
  }

  private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();
    int numElements = s.readInt();

    //모든 원소를 읽어 이 리스트에 삽입한다.
    for (int i = 0; i < numElements; i++) {
      add((String) s.readObject());
    }
  }

  ...
}
````

<br>

### _default_ 메서드
default 메서드 호출시 transient로 선언하지 않은 모든 인스턴스 필드가 직렬화/역직렬화  
직렬화 인스턴스의 모든 필드가 transient인 경우 default 메서드 호출이 필수는 아님  
하지만 직렬화 명세는 이 작업을 무조건 하라고 요구  
이렇게 해야 향후 릴리스에 transient 필드가 아닌 새로운 필드가 추가되더라도 상호호환 제공  
신버전 인스턴스를 직렬화한 후 구버전으로 역직렬화하면 새로운 필드들은 무시되며 StreamCorruptedException 발생  

<br>

### _transient_ 한정자
해당 객체의 논리적 상태와 무관한 필드라고 확신할 때만 transient 한정자 생략 가능  
기본 직렬화 사용시 transient 필드들은 역직렬화될 때 기본값으로 초기화  
원래 객체의 불변식이 세부 구현에 따라 달라지는 객체는 정확성마저 깨질 가능성 존재  
> 해시테이블은 물리적으로 키-값 엔트리를 담은 해시 버킷을 차례로 나열한 형태  
> 어떤 엔트리를 어떤 버킷에 담을지는 키에서 구한 해시코드가 결정  
> 계산 방식은 구현에 따라 달라질 수 있고 계산할 때마다 달라질 가능성 존재  
> 기본 직렬화 사용시 심각한 버그 발생  
> 역직렬화하면 불변식이 심각하게 훼손된 객체들 발생  

<br>

### 동기화 메커니즘 직렬화
기본 직렬화 사용 여부와 상관없이 객체의 전체 상태를 읽는 메서드에 적용해야 하는 동기화 메커니즘을 직렬화에도 적용해야 한다.  
writeObject 메서드 안에서 동기화하고 싶다면 클래스의 다른 부분에서 사용하는 락 순서를 따라야 자원 순서 교착상태(resource-ordering deadlock)에서 안전  

````java
private synchronized void writeObject(ObjectOutputStream s) throws IOException {
  s.defaultWriteObject();
}
````

<br>

### 직렬 버전 UID
어떤 직렬화 형태를 택하든 직렬화 가능 클래스 모두에 직렬 버전 UID를 명시적으로 부여  
이렇게 해야 직렬 버전 UID가 일으키는 잠재적인 호환성 문제 예방 가능  
직렬 버전 UID가 꼭 고유할 필요는 없음  
직렬 버전 UID를 이용해서 구버전과 신버전의 호환성 관리 가능  
즉, 구버전 직렬화 인스턴스들과의 호환성을 끊으려는 경우를 제외하고 직렬 버전 UID 수정 금지  

````java
private static final long serialVersionUID = <random long value>
````

<br>



