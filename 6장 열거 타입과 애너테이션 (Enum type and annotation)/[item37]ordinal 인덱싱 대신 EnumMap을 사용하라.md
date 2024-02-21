# ordinal 인덱싱을 활용했을 경우

식물을 간단히 나타낸 클래스가 있다.

``` java
public class Plant {
    enum LifeCycle {ANNUAL, PERENNIAL, BIENNIAL}

    final String name;
    final LifeCycle lifeCycle;

    public Plant(final String name, final LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }

    @Override
    public String toString() {
        return name;
    }
}

```

정원에 심은 식물들을 배열 하나로 관리하고, 이들을 생애주기별로 총 3개의 집합을 만들고 정원에 있는 식물들을 해당하는 집합에 넣는다고 가정해보면.

``` java
Set<Plant>[] plantsByLifeCycle = (Set<Plant>[]) new Set[Plant.LifeCycle.values().length]; // 비검사 형변환
for (int i = 0; i < plantsByLifeCycle.length; i++) {
    plantsByLifeCycle[i] = new HashSet<>();
}

for (Plant plant : plantList) {
    plantsByLifeCycle[plant.lifeCycle.ordinal()].add(plant);
}

for (int i = 0; i < plantsByLifeCycle.length; i++) {
    System.out.printf("%s: %s%n", Plant.LifeCycle.values()[i], plantsByLifeCycle[i]); // 인덱스의 의미를 모르기 때문에 직접 의미를 달아야 한다.
}
```

위의 같이 열거 타입의 `ordinal` 인덱싱을 활용해 코드를 작성할 수 있다.

동작은 하지만 문제점들이 존재한다.

1. 배열은 제네릭과 호환되지 않으니 비검사 형변환을 수행해야 한다.
2. 배열은 각 인덱스의 의미를 모르니 출력 결과에 직접 의미를 달아야 한다.
3. 정확한 정숫값을 사용한다는 것을 직접 보증해야 한다.

---

# 해결책

배열은 각 열거 타입 상수를 값으로 매핑하는 일을 한다.

`Map`으로 매핑하여 사용할 수 있다.

열거 타입을 키로 사용하도록 설계한 `EnumMap`이 있다.

``` java
EnumMap<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);

for (Plant.LifeCycle lifeCycle : Plant.LifeCycle.values()) {
    plantsByLifeCycle.put(lifeCycle, new HashSet<>());
}
for (Plant plant : plantList) {
    plantsByLifeCycle.get(plant.lifeCycle).add(plant);
}
System.out.println(plantsByLifeCycle);
```

1. 성능이 배열을 사용했을 경우와 비슷하다.
2. 안전하지 않은 형변환을 하지 않는다.
3. 맵의 키인 열거 타입이 그 자체로 출력용 문자열을 제공하여 출력 결과에 직접 의미를 달 필요가 없다.
4. 배열 인덱스를 계산하는 과정에서 오류가 날 가능성이 없다.

---

# EnumMap

`EnumMap`의 성능이 `ordinal`을 활용한 배열을 사용했을 경우와 비슷한 이유는 내부에서 배열을 사용하기 때문이다.

![EnumMapConstructor](https://github.com/NoSubject-Study/effective-java-study/assets/103320798/3be32d88-66b3-4665-9418-c3badb65532f)

생성자를 보면 `vals = new Object[keyUniverse.length];` 코드를 볼 수 있는데, 매개변수로 들어온 열거 타입의 모든 상수를 가져온 뒤, 개수만큼의 크기를 가진 `Object` 배열을 생성한다.

![EnumMap](https://github.com/NoSubject-Study/effective-java-study/assets/103320798/8a992e88-2f25-412a-a5ef-0895b0480e77)

그 후 값을 넣거나 뺄 때, 키로 들어온 열거 타입 상수의 `ordinal` 메서드를 호출한 뒤 반환된 정숫값을 인덱스로 활용한다.

![EnumMapGet](https://github.com/NoSubject-Study/effective-java-study/assets/103320798/2678c1dc-f527-40b6-8407-fe48e4758688)

![EnumMapPut](https://github.com/NoSubject-Study/effective-java-study/assets/103320798/2d3d28e9-645a-4e60-91b3-495424cb3c48)

---

# Stream 활용

스트림을 활용하면 코드를 더 줄일 수 있다.

``` java
System.out.println(plantList.stream()
                .collect(groupingBy(plant -> plant.lifeCycle)));
```

``` java
Map<Plant.LifeCycle, List<Plant>> map = plantList.stream()
                .collect(groupingBy(plant -> plant.lifeCycle)); // HashMap 구현체를 사용
```

위에서는 `EnumMap`이 아닌 맵 구현체를 사용했기 때문에 `EnumMap`을 사용했을 경우의 이점이 사라진다는 문제가 있다.

``` java
groupingBy(Function<? super T, ? extends K> classifier,
                                  Supplier<M> mapFactory,
                                  Collector<? super T, A, D> downstream);
```

매개변수 3개를 가진 `Collectors.groupingBy` 메서드는 `mapFactory` 매개변수에 원하는 맵 구현체를 명시해 호출할 수 있다.

``` java
System.out.println(plantList.stream()
                .collect(groupingBy(plant -> plant.lifeCycle, 
                        () -> new EnumMap<>(Plant.LifeCycle.class), toSet())));
```

``` java
EnumMap<Plant.LifeCycle, Set<Plant>> map = plantList.stream()
                .collect(groupingBy(plant -> plant.lifeCycle,
                        () -> new EnumMap<>(Plant.LifeCycle.class), toSet()));
```

## EnumMap을 활용했을 경우와의 차이

Stream을 활용하게 되면 특정 열거 타입에 해당하는 객체가 존재하지 않으면 해당 키를 만들지 않게 된다.

``` java
Plant annual = new Plant("annual", Plant.LifeCycle.ANNUAL);
Plant perennial = new Plant("perennial", Plant.LifeCycle.PERENNIAL);
List<Plant> plantList = List.of(annual, perennial);
System.out.println(plantList.stream()
                .collect(groupingBy(plant -> plant.lifeCycle,
                        () -> new EnumMap<>(Plant.LifeCycle.class), toSet())));
```

위의 코드를 실행하면 아래와 같이 출력된다.

```
{ANNUAL=[annual], PERENNIAL=[perennial]}
```

하지만 위의 `EnumMap`을 활용할 경우의 코드에서는 특정 열거 타입에 해당하는 객체가 존재하지 않아도 해당 열거 타입의 키를 만들게 된다.

``` java
Plant annual = new Plant("annual", Plant.LifeCycle.ANNUAL);
Plant perennial = new Plant("perennial", Plant.LifeCycle.PERENNIAL);
List<Plant> plantList = List.of(annual, perennial);
EnumMap<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);

for (Plant.LifeCycle lifeCycle : Plant.LifeCycle.values()) {
    plantsByLifeCycle.put(lifeCycle, new HashSet<>());
}
for (Plant plant : plantList) {
    plantsByLifeCycle.get(plant.lifeCycle).add(plant);
}
System.out.println(plantsByLifeCycle);
```

위의 코드를 실행하면 아래와 같이 출력된다.

```
{ANNUAL=[annual], PERENNIAL=[perennial], BIENNIAL=[]}
```

---

# 중첩 배열의 경우

아래에 상태가 열거 타입으로 존재하고 두 상태와 전이를 매핑한 코드가 있다.

``` java
public enum Phase {
    SOLID, LIQUID, GAS;

    public enum Transition {
        MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;

        // 행은 from의 ordinal을, 열은 to의 ordinal을 인덱스로 사용한다.
        private static final Transition[][] TRANSITIONS = {
                {null, MELT, SUBLIME},
                {FREEZE, null, BOIL},
                {DEPOSIT, CONDENSE, null}
        };

        // 한 상태에서 다른 상태로의 전이를 반환한다.
        public static Transition from(Phase from, Phase to) {
            return TRANSITIONS[from.ordinal()][to.ordinal()];
        }
    }
}
```

위의 코드도 `ordinal`을 이용하고 있으므로 마찬가지로 아래와 같은 문제가 존재한다.

컴파일러는 `ordinal`과 배열 인덱스의 관계를 알지 못하므로 `Phase`, `Trasition` 열거 타입을 수정하게 될 경우 `TRANSITIONS`를 함께 수정하지 않거나 잘못 수정하면 런타임 오류가 일어날 수 있다.
(`ArrayIndexOutOfBoundsException`, `NullPointerException` 예외가 발생할 수 있고 혹은 이상하게 동작할 수도 있다.)

상태의 가짓수가 늘어나면 `TRANSITIONS`의 크기도 제곱해서 커지며 `null`로 채워지는 칸도 늘어날 것이다.

이 경우에도 `EnumMap`을 활용하는 편이 좋다.

``` java
public enum Phase {
    SOLID, LIQUID, GAS;

    public enum Transition {
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID), BOIL(LIQUID, GAS),
        CONDENSE(GAS, LIQUID), SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);

        private final Phase from;
        private final Phase to;

        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }

        // 맵을 초기화한다.
        private static final Map<Phase, Map<Phase, Transition>> map =
                Stream.of(values()).collect(groupingBy(
                        t -> t.from,
                        () -> new EnumMap<>(Phase.class),
                        toMap(
                                t -> t.to,
                                t -> t,
                                (x, y) -> y,
                                () -> new EnumMap<>(Phase.class)
                        )
                ));

        public static Transition from(Phase from, Phase to) {
            return map.get(from).get(to);
        }
    }
}
```

여기에 새로운 상태인 플라즈마(PLASMA)를 추가한다고 가정해보자.

이 상태와 연결된 전이는 두 가지인데,

기체에서 플라즈마로 변하는 이온화(IONIZE),

플라즈마에서 기체로 변하는 탈이온화(DEIONIZE)다.

만일 배열로 만든 코드에 추가한다면, `Phase`에 1개, `Phase.Transition`에 2개를 추가하고, 원소 9개를 가진 배열들의 배열을 원소 16개로 교체해야 한다.

하지만 `EnumMap`을 활용한 코드에서는 아래와 같이 상태 목록에 `PLASMA`를 추가하고, 전이 목록에 `IONIZE(GAS, PLASMA)`와 `DEIONIZE(PLASMA, GAS)`만 추가하면 된다.

``` java
public enum Phase {
    SOLID, LIQUID, GAS, PLASMA;

    public enum Transition {
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID), BOIL(LIQUID, GAS),
        CONDENSE(GAS, LIQUID), SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID),
        IONIZE(GAS, PLASMA), DEIONIZE(PLASMA, GAS);

        ... // 나머지 코드는 그대로다.
    }
}
```
