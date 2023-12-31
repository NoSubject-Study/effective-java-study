# 생성자 대신 정적 팩터리 메서드를 고려하라

생성자란
- 객체가 생성될때 자동으로 호출
- 객체 초기화를 위한 메서드

정적 팩터리 메서드란
- 객체 생성의 역할을 하는 클래스 메서드
- 불변 객체 생성 및 함수형 프로그래밍에 특화


### 정적 팩터리 메서드 사용한 불린값 박싱
```` java
public static Boolean valueOf(boolean b) {
		return b ? Boolean.TRUE : Boolean.FALSE;
}
````


### 정적 팩터리 메서드의 장점
#### 1. 이름을 가질 수 있다.
생성자를 사용한 경우 매개변수와 생성자 자체 이름만으로는 반환될 객체의 특성을 제대로 설명하지 못한다.

#### 2. 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.
생성자와는 다르게 호출할 때마다 새롭게 인스턴스를 생성하지 않고 생성한 인스턴스를 재활용할 수 있다.

#### 3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.
반환할 객체의 클래스(해당 객체의 하위 클래스)를 자유롭게 선택 가능하다.
45개의 유틸리티 구현체를 제공하는 java.util.Collections 클래스 또한 정적 팩터리 메서드를 사용한다.

#### 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
EnumSet 클래스의 경우 매개변수의 원소 갯수에 따라(65개 미만, 이상) 서로 다른 구현체 반환한다.(RegularEnumSet, JumboEnumSet)

#### 5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.
이런 유연함을 이용해 구현된 프레임워크가 많이 존재한다.
JDBC의 경우 데이터베이스 드라이버 구현체가 등장하기 전에 작성되었다.


### 정적 팩터리 메서드의 단점
#### 1. 상속을 하려면 public, protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.
컬렉션 프레임워크 유틸리티 구현 클래스는 상속이 불가하다.
이는 불변 객체를 만들어 사용하도록 유도하는 장점이 존재한다.

#### 2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.
생성자와 같이 API 명세에 정확히 드러나지 않는다.
여러 정적 팩토리 메서드가 존재하는 경우, 클래스를 인스턴스화할 수 있는 여러 방법을 스스로 학습해야 사용가능하다.


