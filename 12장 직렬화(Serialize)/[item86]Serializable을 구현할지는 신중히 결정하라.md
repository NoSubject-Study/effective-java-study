## Serializable을 구현할지는 신중히 결정하라

### 직렬화 적용
어떤 클래스 인스턴스를 직렬화하려면 implements Serializable만 덧붙이면 된다.  
너무 쉽게 보이지만 길게 보면 훨씬 복잡하다.  

<br>

### 1. Serializable을 구현하면 릴리스한 뒤에 수정이 어렵다.
직렬화된 바이트 스트림 인코딩도 하나의 공개 API  
기본 직렬화 형태에서는 클래스의 private 필드마저 공개되는 꼴  

* 직렬 버전(UID - serial version UID)  
> 직렬화가 클래스 개선을 방해하는 대표적인 예  
> 모든 직렬화된 클래스는 고유 식별 번호를 부여  
> 만약 명시하지 않은 경우 런타임에 해시 함수(`SHA-1`)를 적용해 자동 부여  
> 이 값은 클래스 이름, 구현한 인터페이스 등 대부분의 클래스 멤버에 따라 결정  
> 즉, 클래스 멤버들 중 하나라도 수정하면 직렬 버전 UID 값도 변경  

```` java
static final long SerialVersionUID;
````

<br>

### 2. Serializable은 버그와 보안 구멍이 생길 위험이 높아진다. 
객체 생성은 생성자를 사용하는 것이 기본  
결국 직렬화는 언어의 기본 매커니즘을 우회하는 객체 생성 기법  

<br>

### 3. Serializable은 해당 클래스의 신버전을 릴리스할 때 테스트할 것이 늘어난다.
신버전 인스턴스와 구버전 인스턴스의 양방향 직렬화/역직렬화 테스트 필수

<br>

### 4. Serializable 구현 여부는 가볍게 결정할 사안이 아니다.
단, 객체를 전송하거나 저장할 때 자바 직렬화 프레임워크용으로 만든 클래스라면 선택의 여지가 없다.  
역사적으로 BigInteger, Instant 같은 `값` 클래스와 컬렉션 클래스들은 Serializable 구현  
스레드 풀처럼 `동작`하는 객체를 표현하는 클래스들은 대부분 Serializable 구현하지 않음  
상속용 클래스는 대부분 Serializable 구현 금지  
인터페이스도 대부분 Serializable 확장 금지  

<br>

상속용 클래스 중 Throwable, Component 클래스는 Serializable 구현  
`Throwable`은 서버가 RMI를 통해 클라이언트로 예외 전송을 위해 구현  
`Component`는 GUI를 전송하고 저장하고 복원하기 위해 구현  

<br>

### 만약 사용하는 클래스의 인스턴스 필드가 직렬화 및 확장 가능인 경우
finalizer 공격에 취약  
인스턴스 필드 값 중 불변식을 보장해야 할 경우 하위 클래스에서 `finalize 메서드` 재정의 금지  
즉, finalize 메서드를 자신이 `final`로 재정의 선언  

<br>

### 상태 존재, 확장 가능, 직렬화 가능 클래스를 위한 readObjectNoData 메서드
```` java
private void readObjectNoData() throws InvalidObjectException {
  throw new InvalidObjectException("Required stream data");
}
````

<br>

### 내부 클래스는 직렬화 구현 금지
내부 클래스는 바깥 인스턴스의 참조와 유효 범위 안의 지역변수 값들을 저장하기 위해 컴파일러가 생성한 필드들이 자동으로 추가된다.  
이 필드들이 클래스 정의에 어떻게 추가되는지 정의되지 않아 직렬화 형태가 분명하지 않음  
단, 정적 멤버 클래스는 Serializable 구현 가능  

<br>


