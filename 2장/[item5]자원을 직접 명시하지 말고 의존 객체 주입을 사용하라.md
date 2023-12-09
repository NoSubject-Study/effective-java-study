# Prefer dependency injection to hardwiring resource 
### 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

Dependency Injection(의존성 주입이란?)
- 소프트웨어 개발에서 사용되는 디자인 패턴 중 하나로, 객체 간의 의존성을 외부에서 주입하는 방식
- 객체지향 프로그래밍에서 객체 간의 결합도를 낮추고 코드의 재사용성과 테스트 용이성을 증가

### ASIS 
유연하지 않고 테스트 하기 어려운 코드의 예시

```` java
public class SpellChecker {
  private static final Lexicon dictionary = ...;

  private SpellChecker() {}

  public static boolean isValid(String word) { ... }
  public static List<String> suggestions(String typo) { ... }
}
````
맞춤법 검사기의 코드가 위와 같다면 맞춤법검사기 클래스를 사용하는 코드에서는...?
```` java
    SpellChecker spellChecker = new SpellChecker();
    boolean isCorrect = spellChecker.isValid("맞춤법");
````
- 다른 언어의 사전을 추가하고 샆다면?
- 다른 출판사의 사전을 추가하고 싶다면?
- 특수 어휘용 사전이 별도로 필요하다면?

       
👉 사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스 혹은 싱글턴 방식이 적합하지 않음    
👉 이런 자원들을 클래스가 직접 만들게 하지 말자
💡 <b>인스턴스를 생성할 때 생성자(혹은 정적팩토리/빌더)에 필요한 자원을 넘겨주는 방식을 사용하자 = 의존 객체를 주입해준다</b>
### 의존 객체 주입
인스턴스를 생성할 때, 생성자에 필요한 자원을 넘겨주는 방식   
➕ 장점 : 클래스의 유연성, 재사용성, 테스트 용이성    
➖ 단점 : 의존성이 많아지면 코드가 복잡해지고 관리가 어려움    

### 생성자에 자원 넘겨주기
```` java
public class SpellChecker {

		private final Lexicon dictionary;

		public SpellChecker(Lexicon dictionary) {
			this.dictionary = Objects.requireNonNull(dictionary);
		}

		public boolean isValid(String word) { ... }
		public List<String> suggestions(String typo) { ... }
}
````
```` java
// 의존 객체 생성 (dependencies)
Lexicon lexicon = new Lexicon();

// 클래스 생성시 주입 (Inject)
SpellChecker spellChecker = new SpellChecker(lexicon);
boolean isCorrect = spellChecker.isValid("맞춤법");
````

### 자원을 생성하는 팩터리를 넘겨주기
```` java
class Tile {
    ...
}

// Mosaic 클래스
class Mosaic {
    private List<Tile> tiles;

    public Mosaic(List<Tile> tiles) {
        this.tiles = tiles;
    }
    ...
}

public class MosaicFactory {
    public static Mosaic create(Supplier<? extends Tile> tileFactory, int mosaicSize) {
        List<Tile> tiles = new ArrayList<>();

        for (int i = 0; i < mosaicSize; i++) {
            Tile tile = tileFactory.get();
            tiles.add(tile);
        }

        return new Mosaic(tiles);
    }
}
````
```` java
// Tile을 생성하는 Supplier를 정의
Supplier<Tile> tileSupplier = Tile::new;

// Mosaic를 생성 (예를 들어, 크기가 10인 Mosaic)
Mosaic mosaic = create(tileSupplier, 10);
````
### 나라면?
```` java
public class Korean implements Dictionary {
    @Override
    public boolean isValid(String word) {
        return false;
    }
}

public class Oxford implements Dictionary {
    @Override
    public boolean isValid(String word) {
        return false;
    }
}

public class SpellChecker {

    private final Dictionary dictionary;

    public SpellChecker(Dictionary dictionary) {
        this.dictionary = dictionary;
    }

    public boolean isValid(String word) {
        return true;
    }

    public List<String> suggestions(String typo) {
        return null;
    }
}
````
