# **Item23: Prefer class hierarchies to tagged classes**

*태그 달린 클래스보다는 클래스 계층구조를 활용하라*

---

## 태그 달린 클래스(Tagged Class)

→ 두 가지 이상의 의미를 표현할 수 있으며, 그중 현재 표현하는 의미를 태그 값으로 알려주는 클래스

Ex. 원과 사각형을 표현할 수 있는 클래스

```java
// Tagged class - vastly inferior to a class hierarchy!
class Figure {
    enum Shape {RECTANGLE, CIRCLE};

    // Tag field - the shape of this figure
    final Shape shape;
    
    // These fields are used only if shape is RECTANGLE
    double length;
    double width;
    
    // This field is used only if shape is CIRCLE
    double radius;

    // Constructor for circle
    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    // Constructor for rectangle
    Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    double area() {
        switch (shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
```

### **태그 달린 클래스는 장황하고, 오류를 내기 쉽고, 비효율적이다.**

1. 과도한 코드와 복잡성
    - 열거 타입 선언, 태그 필드, `switch` 문 등 쓸데없는 코드가 많음
    - 여러 구현이 한 클래스에 혼합돼 있어서 가독성을 저해
2. 메모리의 비효율적 사용
    - 다른 의미를 위한 코드도 언제나 함께 하니 메모리도 많이 사용
3. 초기화의 복잡성
    - 필드들을 `final`로 선언하려면 해당 의미에 쓰이는 데이터 필드들가지 생성자에서 초기화해야 함
    - 쓰지 않는 필드를 초기화하는 불필요한 코드가 늘어남
    - 생성자가 태그 필드를 설정하고 해당 의미에 쓰이는 데이터 필드들을 초기화하는 데 컴파일러가 도와줄 수 있는 건 별로 없다.
    - 엉뚱한 필드를 초기화해도 런타임에야 문제가 드러날 뿐이다.
4. 확장성의 문제
    - 또 다른 의미를 추가하려면 코드를 수정해야 함
    - Ex) 새로운 의미를 추가할 때마다 모든 `switch` 문을 찾아 새 의미를 처리하는 코드를 추가해야 하는데, 하나라도 빠뜨리면 역시 런타임에 문제가 불거져 나올 것이다.
5. 타입의 불명확성
    - 인스턴스의 타입만으로는 현재 나타내는 의미를 알 길이 전혀 없다.

→ 클래스 계층구조를 활용하는 서브타이핑(subtyping)을 사용(**태그 달린 클래스는 클래스 계층구조를 어설프게 흉내낸 아류일 뿐이다.)**

## 태그 달린 클래스를 클래스 계층구조로 바꾸기

1. 계층 구조의 루트(root)가 될 추상 클래스를 정의하고, 태그 값에 따라 동작이 달라지는 메서드들을 루트 클래스의 추상 메서드로 선언
2. 태그 값에 상관 없이 동작이 일정한 메서드들을 루트 클래스에 일반 메서드로 추가
3. 모든 하위 클래스에서 공통으로 사용하는 데이터 필드들도 전부 루트 클래스로 올리기
4. 루트 클래스를 확장한 구체 클래스를 의미별로 하나씩 정의
5. 각 하위 클래스에 각자 의미에 해당하는 데이터 필드들을 넣기
6. 루트 클래스가 정의한 추상 메서드를 각자의 의미에 맞게 구현

Ex. 태그 달린 클래스를 클래스 계층구조로 변환

```java
// Class hierarchy replacement for a tagged class
abstract class Figure {
    abstract double area();
}

class Circle extends Figure {
    final double radius;

    Circle(double radius) { this.radius = radius; }

    @Override double area() { return Math.PI * (radius * radius); }
}

class Rectangle extends Figure {
    final double length;
    final double width;

    Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }

    @Override
    double area() { return length * width; }
}
```

## 클래스 계층구조의 장점

→ 태그 달린 클래스의 단점을 모두 제거

- 쓸데없는 코드를 정리하고 관련 없는 데이터 필드를 제거함(남은 필드들은 모두 `final`로 선언됨)
- 각 클래스의 생성자가 모든 필드를 남김없이 초기화하며 추상 메서드를 구현했는지 컴파일러가 확인해줌
- 루트 클래스의 코드를 변경하지 않고도 다른 프로그래머들이 독립적으로 계층 구조를 확장하고 사용할 수 있음
- 타입이 의미별로 존재하니 변수의 의미를 명시하거나 제한할 수 있으며, 특정 의미만을 매개변수로 받을 수 있음
- 타입 사이의 계층 관계를 반영할 수 있어 유연성과 컴파일타임 타입 검사 능력을 향상시킴
- 새로운 타입을 추가할 경우, 기존 코드를 변경할 필요 없이 새로운 클래스를 만들어 구현할 수 있음

Ex. 클래스 계층구조에서 정사각형 클래스 추가

```java
class Square extends Rectangle {
    Square(double side) {
        super(side, side);
    }
}
```

> 이번 아이템의 예에서는 접근자 메서드 없이 필드를 직접 노출했다. 이는 코드를 단순하게 할 의도로 만약 공개할 클래스라면, 이렇게 설계하는 것은 좋지 않다(Item 16).
> 

## 정리

- 태그 달린 클래스를 써야할 상황은 거의 없음
- 새로운 클래스를 작성하는 데 태그 필드가 등장한다면 태그를 없애고 계층구조로 대체하는 방법을 생각해야 함
- 기존 클래스가 태그 필드를 사용하고 있다면 계층구조로 리팩터링하는 걸 고민해야 함
