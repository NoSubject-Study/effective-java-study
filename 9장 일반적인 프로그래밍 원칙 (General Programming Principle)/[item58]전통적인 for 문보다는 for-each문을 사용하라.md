# for 문과 for-each 문의 비교

아래는 for 문으로 컬렉션을 순회하는 코드다.

``` java
for (Iterator<Element> i = c.iterator(); i.hasNext();) {
  Element e = i.next();
  ...
}
```

아래는 for 문으로 배열을 순회하는 코드다.

``` java
for (int i = 0; i < a.length; i++) {
  int n = a[i];
  ...
}
```

위의 for 문들은 while 문을 사용하는 것보다는 안전하지만 가장 좋은 방법은 아니다.
반복자와 인덱스가 사용되어 코드를 지저분하게 하며 추가적인 요소들로 인한 오류가 발생할 가능성이 높아진다.
컬렉션을 순회할 경우 반복자는 3번 등장하며, 배열을 순회할 경우 인덱스는 4번 등장하여 변수를 잘못 사용할 가능성이 생기게 된다.

위의 문제들은 for-each 문을 사용하면 모두 해결된다.

for-each 문의 정식 명칭은 '향상된 for 문(enhanced for statement)'이다.

컬렉션을 순회하는 for 문을 for-each 문으로 표현하면 다음과 같다.

``` java
for (Element e : elements) {
  ...
}
```

반복자가 없어 코드의 가독성이 좋아지고 오류가 생길 확률이 줄어들게 된다.

``` java
enum Suit { CLUB, DIAMOND, HEART, SPADE }
enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT, NINE, TEN, JACK, QUEEN, KING }

...

static Collection<Suit> suits = Arrays.asList(Suit.values());
static Collection<Rank> ranks = Arrays.asList(Rank.values());

List<Card> deck = new ArrayList<>();
for (Iterator<Suit> i = suits.iterator(); i.hasNext();) {
  for (Iterator<Rank> j = ranks.iterator(); j.hasNext;) {
    deck.add(new Card(i.next(), j.next())); // Error !!
  }
}
```

위의 코드에는 오류가 있다.
언뜻 보기에 각 Rank 마다 Suit와 매핑하여 Deck에 추가하는 것 처럼 보이지만 i.next()를 계속 사용하기 때문에 i에 Suit가 존재하지 않게 되면 NoSuchElementException을 던지게 될 것이다.

이는 for-each 문을 사용하게 되면 간단하게 오류를 피할 수 있다.

``` java
for (Suit suit : suits)
  for (Rank rank : ranks)
    deck.add(new Card(suit, rank));
```

---

# for-each 문을 사용할 수 없는 상황

- __파괴적인 필터링(destructive filtering)__ - 컬렉션을 순회하면서 원소를 제거해야 한다면 반복자의 remove 메서드를 호출해야 한다. 자바 8부터는 Collection의 removeIf 메서드를 사용해 컬렉션을 순회하는 일을 피할 수 있다.

<details>
<summary>Collection의 removeIf</summary>
<div>
	<img src="https://velog.velcdn.com/images/lucius__k/post/8daaa5b4-19d2-455f-9b41-6d22b9ccaf05/image.png" alt="removeIf" />
  <p>removeIf() 매개변수로 필터링 조건을 전달하면 해당하는 원소들이 제거된다.</p>
  <hr/>
</div>
</details>

- __변형(transforming)__ - 리스트나 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 교체해야 한다면 리스트의 반복자나 배열의 인덱스를 사용해야 한다.

- __병렬 반복(parallel iteration)__ - 여러 컬렉션을 병렬로 순회해야 한다면 각각의 반복자와 인덱스 변수를 사용해 엄격하고 명시적으로 제어해야 한다.

위의 세 상황에서는 for 문을 사용해야 한다.

for-each 문은 Iterable 인터페이스를 구현한 객체라면 무엇이든 순회할 수 있다.

``` java
public interface Iterable<E> {
  Iterator<E> iterator();
}
```

Iterable 인터페이스를 구현하기는 까다롭지만 원소들을 묶음으로 관리하는 타입을 작성한다면 Iterable을 구현하여 for-each 문을 사용할 수 있도록 하자.
