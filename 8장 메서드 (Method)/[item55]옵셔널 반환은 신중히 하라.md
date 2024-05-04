# Item55: Return optionals judiciously

*옵셔널 반환은 신중히 하라*

---

## 메서드가 특정 조건에서 값을 반환할 수 없을 때

자바 8 이전: 예외를 던지거나, (반환 타입이 객체 참조라면) `null`을 반환

- 예외는 진짜 예외적인 상황에 사용해야 하며(Item 69) 예외를 생성할 때 스택 추적 전체를 캡처하는 비용도 높음
- `null` 반환 시 처리 코드를 추가해야 함

→ 자바 8 이후: `Optional<T>` 클래스를 이용

## `Optional<T>` 클래스

- 원소를 최대 1개 가질 수 있는 ‘불변’ 컬렉션
- `null`이 아닌 `T` 타입 참조를 담거나(*not empty*) 아무것도 담지 않을 수 있음(*empty*)
- 일반적인 상황에서 메서드는 `T`를 반환하고, 아무것도 반환하지 않아야 할 때 `Optional<T>`를 반환 
→ 유효한 반환값이 없을 때 빈 결과를 반환하는 메서드가 생성됨

→ `Optional<T>`를 반환하는 메서드는 예외를 던지는 것보다 더 유연하고 사용하기 쉬우며, `null`을 반환하는 것보다 오류 가능성이 적다.

Ex. 컬렉션에서 최댓값을 반환하는 메서드

```java
// Returns maximum value in collection - throws exception if empty
public static <E extends Comparable<E>> E max(Collection<E> c) {
	if (c.isEmpty())
		throw new IllegalArgumentException("Empty collection");
	E result = null;
	for (E e : c)
		if (result == null || e.compareTo(result) > 0)
			result = Objects.requireNonNull(e);
		return result;
}
```

이 메서드에 빈 컬렉션을 건네면 `IllegalArgumentException`을 던진다.

Ex. `Optional<E>`로 반환하는 메서드

```java
// Returns maximum value in collection as an Optional<E>
	public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
		if (c.isEmpty())
			return **Optional.empty()**;
			
		E result = null;
		for (E e : c)
			if (result == null || e.compareTo(result) > 0)
				result = Objects.requireNonNull(e);
		return **Optional.of(result)**;
 } 
```

정적 팩터리 메서드들을 사용해 옵셔널을 생성하여 반환한다.

- `Optional.empty()`: 빈 옵셔널을 반환
- `Onptional.of(value)`: 주어진 값을 감싼 옵셔널을 반환(주어진 값이 `null`이 아니어야 함 → `NullPointerException` 발생)
- `Optional.ofNullable(value)`: 주어진 값이 `null`이 아니면 해당 값을 감싼 옵셔널을 반환하고, `null`인 경우 빈 옵셔널을 반환

**옵셔널을 반환하는 메서드에서는 절대 `null`을 반환하지 말자.** → NPE를 발생시켜 옵셔널을 도입한 취지를 완전히 무시하는 행위

Ex. 스트림 버전 메서드

```java
// Returns max val in collection as Optional<E> - uses stream
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
	return c.stream().max(Comparator.naturalOrder());
}
```

스트림의 종단 연산 중 많은 것들이 옵셔널을 반환한다.

## 옵셔널 값의 유무에 따른 처리 방법

**옵셔널은 검사 예외와 취지가 비슷하다**(Item 71). → 반환값이 없을 수도 있음을 API 사용자에게 명확히 알려준다.

⇒ 메서드가 옵셔널을 반환한다면 클라이언트는 값을 받지 못했을 때 취할 행동을 선택해야 함

1. 기본 값 설정: `orElse()`
    
    ```java
    String lastWordInLexicon = max(words).orElse("No words...");
    ```
    
2. 예외 던지기: `orElseThrow()`
    
    ```java
    Toy myToy = max(toys).orElseThrow(TemperTantrumException::new);
    ```
    
    실제 예외가 아닌 팩터리를 건넴
    
3. 값 가져오기: `get()`
    
    ```java
    Element lastNobleGas = max(Elements.NOBLE_GASES).get();
    ```
    
    옵셔널에 항상 값이 채워져 있다고 확신하면 곧바로 값을 꺼내 사용(잘못 판단시 `NoSuchElementException`이 발생)
    

이 외 방법:

- `Supplier<T>`를 인수로 받는 `orElseGet`을 사용하면, `Supplier<T>`을 사용해 값을 처음 생성하므로 초기 비용을 낮출 수 있음
- `filter`, `map`, `flatMap`, `ifPresent`등의 메서드를 사용하여 다양한 연산 가능
- `isPresent` 메서드를 사용해 옵셔널이 값을 포함하고 있는지 확인(`true` / `false`) ← 대부분 위의 메서드들로 대체 가능하니, 신중히 사용

Ex. 부모 프로세스의 PID를 출력하는 코드

```java
// Java 8
Optional<ProcessHandle> parentProcess = ph.parent();
System.out.println("Parent PID: " + (parentProcess.isPresent() ?
	String.valueOf(parentProcess.get().pid()) : "N/A"));

// Java 9 or later
System.out.println("Parent PID: " +
	ph.parent()**.map(h -> String.valueOf(h.pid()))**.orElse("N/A"));
```

Ex. 스트림을 사용할 때 옵셔널을 이용

```java
// Java 8
streamOfOptionals
	.filter(Optional::isPresent)
	.map(Optional::get)
	
// Java 9 or later
streamOfOptionals
	.flatMap(Optional::stream)
```

## 적절한 옵셔널 사용

- **컬렉션, 스트림, 배열, 옵셔널 같은 컨테이너 타입은 옵셔널로 감싸면 안된다.**
→ 빈 `Optional<List<T>>`를 반환하기보다 빈 `List<t>`를 반환하는 게 좋음(Item 54)
- **기본규칙: 결과가 없을 수 있으며, 클라이언트가 이 상황을 특별하게 처리해야 한다면 `Optional<T>`를 반환한다.**
    - `Optional`도 엄연히 새로 할당하고 초기화해야 하는 객체이고, 그 안에서 값을 꺼내려면 메서드를 호출해야 함
    - 성능이 중요한 상황에선 적절하지 않을 수 있음
    - 어떤 메서드가 이 상황에 처하는지 알아내려면 세심히 측정해보는 수밖에 없음(Item 67)
- 박싱된 기본 타입을 담는 옵셔널은 기본 타입 자체보다 무겁다.
    - 이에 따라 자바 API 설계자는 `int`, `long`, `doubale` 전용 옵셔널 클래스들을 제공(`Optional<T>`가 제공하는 메서드를 거의 다 제공함)
    - **박싱된 기본 타입을 담은 옵셔널을 반환하는 일은 없도록 하자.** (단, ‘덜 중요한 기본 타입’용인 `Boolean`, `Byte`, `Character`, `Short`, `Float`은 예외)
- **옵셔널을 컬렉션의 키, 값, 원소나 배열의 원소로 사용하는 게 적절한 상황은 거의 없다.**
Ex. 맵 안에 키가 없다는 사실을 나타내는 방법이 두 가지가 됨(키 자체가 없는 경우, 키가 속이 빈 옵셔널인 경우)
- 옵셔널을 인스턴스 필드에 저장하는 것은 필수 필드를 갖는 클래스와, 이를 확장해 선택적 필드를 추가한 하위 클래스를 따로 만들어야 함을 암시하는 ‘나쁜 냄새’다.
예외: `NutritionFacts` 클래스 → 인스턴스의 필드 상당수가 필수가 아님

## 정리

- 값을 반환하지 못할 가능성이 있고, 호출할 때마다 반환값이 없을 가능성을 염두에 둬야 하는 메서드인 경우 옵셔널을 반환해야 할 상황일 수 있음
- 옵셔널 반환에는 성능 저하가 뒤따르니, 성능에 민감한 메서드라면 `null`을 반환하거나 예외를 던지는 편이 나을 수 있음
- 옵셔널을 반환값 이외의 용도로 쓰는 경우는 매우 드묾
