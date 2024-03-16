# @Override

![@Override](https://github.com/NoSubject-Study/effective-java-study/assets/103320798/9496609f-2fe6-433f-a942-75f44fdd7c8f)

`@Override`는 메서드 선언에만 달 수 있다.

상위 타입의 메서드를 재정의했음을 뜻한다.

``` java
public class Bigram {
    
    private final char first;
    private final char second;
    
    public Bigram(char first, char second) {
        this.first = first;
        this.second = second;
    }
    
    public boolean equals(Bigram b) {
        return b.first == first && b.second == second;
    }
    
    public int hashCode() {
        return 31 * first + second;
    }
    
    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for (int i = 0; i < 10; i++) {
            for (char ch = 'a'; ch <= 'z'; ch++) {
                s.add(new Bigram(ch, ch));
            }
        }
        System.out.println(s.size());
    }
}
```

`Bigram` 클래스는 영어 알파벳 2개로 구성된 문자열을 표현한다.

`main` 메서드를 보면 총 26개의 문자로 이루어진 `Bigram` 객체를 10번 반복해서 `Set`에 추가하고 있다.

중복된 문자를 10번 추가하기 때문에 26이 출력될 것을 예상하지만, 260이 출력된다.

``` java
public boolean equals(Bigram b) {
    return b.first == first && b.second == second;
}

public boolean equals(Object obj) { // Object의 equals
    return (this == obj);
}
```

메서드의 시그니처가 잘못됐기 때문에 Override가 되지 않았다.

`@Override` 애너테이션을 사용하면 이와 같은 실수를 방지할 수 있다.

``` java
@Override
public boolean equals(Object o) {
    if (!(o instanceof Bigram)) {
        return false;
    }
    Bigram b = (Bigram) o;
    return b.first == first  && b.second == second;
}
```

구체 클래스에서 상위 클래스의 추상 메서드를 재정의할 때는 굳이 `@Override`를 달지 않아도 된다.

컴파일러가 알려주기 때문이다.

---

인터페이스의 메서드를 재정의할 때도 사용할 수 있다.

인터페이스가 디폴트 메서드를 지원하는 경우 `@Override`를 다는 습관을 들이면 시그니처가 올바른지 확신할 수 있다.

디폴트 메서드가 존재하지 않는 인터페이스를 구현하는 경우에는 생략하여 코드를 깔끔히 유지해도 좋다.

추상 클래스나 인터페이스에서 상위 클래스나 상위 인터페이스의 메서드를 재정의하는 모든 메서드에 `@Override`를 다는 것이 좋다.

예로 `Set` 인터페이스는 `Collection` 인터페이스를 확장했지만 새로 추가한 메서드는 없다.

따라서 모든 메서드 선언에 `@Override`를 달아 실수로 추가한 메서드가 없음을 보장한다.
