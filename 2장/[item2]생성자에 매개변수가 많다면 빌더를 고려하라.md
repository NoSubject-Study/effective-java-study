※ Effective Java 3/E

생성자에 매개변수가 많을 경우 사용할 수 있는 3가지 패턴이 있다.

# 점층적 생성자 패턴

생성자에 매개변수가 많이 있을 경우에 점층적 생성자 패턴(Telescoping Constructor Pattern)을 사용하여 다음과 같이 구현할 수 있다.

``` java
public class NutritionFacts {

	private final int servingSize; // (ml, 1회 제공량) 필수
	private final int servings; // (회, 총 n회 제공량) 필수
	private final int calories; // (1회 제공량당) 선택
	private final int fat; // (g/1회 제공량) 선택
	private final int sodium; // (mg/1회 제공량) 선택
	private final int carbohydrate; // (g/1회 제공량) 선택

	public NutritionFacts(int servingSize, int servings) {
		this(servingSize, servings, 0);
	}

	public NutritionFacts(int servingSize, int servings, int calories) {
		this(servingSize, servings, calories, 0);
	}

	public NutritionFacts(int servingSize, int servings, int calories, int fat) {
		this(servingSize, servings, calories, fat, 0);
	}

	...

	public NutritionFacts(int servingSize, 
                              int servings, 
                              int calories, 
                              int fat, 
                              int sodium, 
                              int carbohydrate) {

		this.servingSize = servingSize;
		this.servings = servings;
		this.calories = calories;
		this.fat = fat;
		this.sodium = sodium;
		this.carbohydrate = carbohydrate;
	}
}
```

위와 같이 매개변수 개수만큼 생성자를 늘리는 방식을 점층적 생성자 패턴이라고 한다.

클라이언트는 원하는 생성자를 호출해 인스턴스를 생성할 수 있다.

하지만 매개변수가 많을 경우 아래와 같은 단점이 존재한다.

## 점층적 생성자 패턴의 단점

``` java
NutritionFacts cocaCola = new NutritionFacts(240, 8, 100, 0, 35, 27);
```

해당 코드를 보았을 때 영향정보를 한눈에 알아보기 쉽지않다.

따라서 해당 코드를 작성하는 사람도 실수를 하기 쉬워진다.

위와 같이 같은 타입의 매개변수가 연달아 있으면 찾기 어려운 버그로 이어질 수 있다.

만일 매개변수의 순서를 잘못 입력할 경우 컴파일러에러가 아닌 런타임에러로 이어지게 된다.

# 자바빈즈 패턴

다른 대안으로는 자바빈즈 패턴(JavaBeans Pattern)이 있다.

``` java
public class NutritionFacts {

	// 기본값이 있다면 기본값으로 초기화한다.
	private int servingSize = -1; // 필수, 기본값 없음
	private int servings = -1; // 필수, 기본값 없음
	private int calories = 0;
	private int fat = 0;
	private int sodium = 0;
	private int carbohydrate = 0;

	public NutritionFacts() {} // 생략 가능

	// 세터 메서드들
	public void setServingSize(int servingSize) { this.servingSize = servingSize;}
	public void setServings(int servings) { this.servings = servings;}
	public void setCalories(int calories) { this.calories = calories;}
	public void setFat(int fat) { this.fat = fat;}
	public void setSodium(int sodium) { this.sodium = sodium;}
	public void setCarbohydrate(int carbohydrate) { this.carbohydrate = carbohydrate;}
}
```

위와 같이 기본 생성자만 구현을 한 후, setter 메서드를 이용해 값들을 설정하는 방식을 자바빈즈 패턴이라고 한다.

자바빈즈 패턴을 사용할 경우 점층적 생성자 패턴의 단점이 보이지 않는 것을 알 수 있다.

클라이언트는 코드를 작성할 경우 실수를 할 확률이 줄어들고 가독성이 좋아진다.

하지만 마찬가지로 자바빈즈 패턴에도 단점이 존재한다.

## 자바빈즈 패턴의 단점

``` java
NutritionFacts cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
cocaCola.setCarbohydrate(27);
```

우선 객체를 하나 생성할 경우 생성자 이외의 많은 메서드들을 호출해야한다.

그리고 객체에 값들을 설정하기 전까지 일관성이 무너진 상태에 놓이게 된다.

일관성이 깨진 객체를 사용할 경우 런타임에러로 디버깅하기 쉽지 않다.

그리고 불변 클래스로 만들 수 없게 된다.

이는 스레드 안전성을 위해서는 추가적인 작업이 필요하다는 것을 의미한다.

# 빌더 패턴

위의 두 패턴들의 장점들을 모두 가지고 있는 빌더 패턴이 있다.

``` java
public class NutritionFacts {

	private final int servingSize;
	private final int servings;
	private final int calories;
	private final int fat;
	private final int sodium;
	private final int carbohydrate;

	public static class Builder {

		// 필수 매개변수
		private final int servingSize;
		private final int servings;

		// 선택 매개변수 - 기본값으로 초기화
		private int calories = 0;
		private int fat = 0;
		private int sodium = 0;
		private int carbohydrate = 0;

		public Builder(int servingSize, int servings) {
			this.servingSize = servingSize;
			this.servings = servings;
		}

		public Builder calories(int calories) {
			this.calories = calories;
			return this;
		}

		public Builder fat(int fat) {
			this.fat = fat;
			return this;
		}

		public Builder sodium(int sodium) {
			this.sodium = sodium;
			return this;
		}

		public Builder carbohydrate(int carbohydrate) {
			this.carbohydrate = carbohydrate;
			return this;
		}

		public NutritionFacts build() {
			return new NutritionFacts(this);
		}
	}

	private NutritionFacts(Builder builder) {
		servingSize = builder.servingSize;
		servings = builder.servings;
		calories = builder.calories;
		fat = builder.fat;
		sodium = builder.sodium;
		carbohydrate = builder.carbohydrate;
	}
}
```

클라이언트는 생성자를 직접 호출하는 대신, 필수 매개변수만으로 빌더 객체를 생성한다.

그 후 빌더 객체의 setter 메서드들을 호출해 값을 설정한 후 build 메서드를 호출해 원하는 객체를 생성할 수 있다.

빌더 패턴을 사용하면 불변 클래스로 만들 수 있고, 가독성도 좋아진다.

다음은 클라이언트가 빌더 패턴으로 객체를 생성하였을 경우의 코드이다.

``` java
NutritionFacts cocaCola = new NutritionFacts.builder(240, 8)
                                            .calories(100)
                                            .sodium(35)
                                            .carbohydrate(27)
                                            .build();
```

이처럼 값을 설정할 수 있는데 builder 객체 자신을 반환하기 때문에 메서드를 연쇄적으로 사용할 수 있다.

이런 방식을 플루언트 API(fluent API) 혹은 메서드 연쇄(method chaining)라고 한다.

## 계층적으로 설계된 클래스와 함께 사용할 경우

빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기에 좋다.

아래 추상 클래스인 피자가 있다고 가정해보자.

``` java
public abstract class Pizza {

	public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
	final Set<Topping> toppings;

	abstract static class Builder<T extends Builder<T>> {

		EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);

		public T addTopping(Topping topping) {
			toppings.add(Objects.requireNonNull(topping));
			return self();
		}

		abstract Pizza build();

		// 하위 클래스는 이 메서드를 재정의(overriding)하여
		// "this"를 반환하도록 해야 한다.
		protected abstract T self();
	}

	Pizza(Builder<?> builder) {
		toppings = builder.toppings.clone();
	}
}
```

Pizza 추상 클래스안에 Builder 추상 클래스가 있고 구체 클래스에서는 해당 Builder 구체 클래스를 갖게 한다.

``` java
// 뉴욕 피자 - 사이즈 매개변수가 필수
public class NyPizza extends Pizza {

	public enum Size { SMALL, MEDIUM, LARGE }
	private final Size size;

	public static class Builder extends Pizza.Builder<Builder> {

		private final Size size;

		public Builder(Size size) {
			this.size = Objects.requireNonNull(size);
		}

		@Override
		public NyPizza build() {
			return new NyPizza(this);
		}

		@Override
		protected Builder self() {
			return this;
		}
	}

	private NyPizza(Builder builder) {
		super(builder);
		size = builder.size;
	}
}
```

``` java
// 칼초네 피자 - 소스를 넣을지 선택하는 매개변수가 필수
public class Calzone extends Pizza {

	private final boolean sauceInside;

	public static class Builder extends Pizza.Builder<Builder> {

		private boolean sauceInside = false; // 기본값

		public Builder sauceInside() {
			sauceInside = true;
			return this;
		}

		@Override
		public Calzone build() {
			return new Calzone(this);
		}

		@Override
		protected Builder self() {
			return this;
		}
	}

	private Calzone(Builder builder) {
		super(builder);
		sauceInside = builder.sauceInside;
	}
}
```

위에 Pizza 추상 클래스를 상속받은 하위 클래스 2개가 있다.

각각의 클래스는 Pizza 추상 클래스를 상속받고, Builder 클래스는 추상 클래스의 Builder 클래스를 상속받는다.

build() 메서드에서 Pizza가 아닌 상속 받은 클래스를 반환하도록 하는데 이를 공변 반환 타이핑(covariant return typing)이라고 한다.

<details>
<summary>공변 반환 타이핑</summary>
<div markdown="1">
  <hr/>
  <p>JDK 1.5에서 추가된 기능이다.</p>
  <p>부모 클래스의 메서드를 Overriding하는 경우 반환 타입을 하위 타입으로 변경할 수 있다.</p>
  <hr/>
</div>
</details>

self() 메서드는 addTopping() 메서드에서 메서드 연쇄를 지원하기 위해 this를 반환하도록 오버라이딩해야한다.

아래는 클라이언트에서의 코드이다.

``` java
NyPizza nyPizza = new NyPizza.Builder(SMALL)
                              .addTopping(SAUSAGE)
                              .addTopping(ONION)
                              .build();

Calzone czPizza = new Calzone.Builder()
                              .addTopping(HAM)
                              .sauceInside()
                              .build();
```

빌더 패턴을 사용하였을 경우에 생성자로는 누릴 수 없는 가변인수 메서드를 여러 개 사용할 수 있다.

- 가변인수 메서드 사용 예
    
``` java
NyPizza nyPizza = new NyPizza.Builder(SMALL)
                              .addToppings(SAUSAGE, ONION, ...)
                              .build();
```

## 빌더 패턴와 맞지 않은 경우

멤버 변수가 적은 클래스의 경우에는 빌더 패턴은 작성해야 하는 코드량만 늘릴 수 있다.

멤버 변수의 개수를 확인하고, 미래에 멤버 변수가 추가될 수 있는지를 염두에 두고 사용해야 한다.

__※ 하지만 Lombok 라이브러리를 사용하게 되면 간단하게 빌더 패턴을 구현할 수 있다.__

그리고 또한 가변 객체를 생성할 경우이다. 가변 객체를 생성하게 될 경우 setter 메서드를 사용할 수 있는데 빌더 패턴을 사용하게 되면 작성해야 하는 코드가 많아진다.


## 빌더 패턴 사용 예

빌더 패턴을 Java API에서 찾을 수 있다.

``` java
Locale locale = new Locale.Builder()
                          .setLanguage("sr")
                          .setScript("Latn")
                          .setRegion("RS")
                          .build();

Calendar calendar = new Calendar.Builder()
                                .setCalendarType("iso8601")
                                .setWeekDate(2012, 1, MONDAY)
                                .build();
```
이외에도 다음과 같은 사용 예시가 있다.

- Stream.Builder
- IntStream.Builder
- LongStream.Builder
- DoubleStream.Builder
- StringBuilder
- Connection.prepareStatement(...);

# References

https://amenable.tistory.com/54

https://stackoverflow.com/questions/2169190/example-of-builder-pattern-in-java-api
