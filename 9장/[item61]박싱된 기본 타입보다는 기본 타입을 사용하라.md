# 기본 타입과 박싱된 기본 타입의 차이 3가지

## 1. 식별성

박싱된 기본 타입은 객체로 저장되기 때문에 식별성을 갖는다.

``` java
public void main(String[] args) {
  Integer a = Integer.valueOf(1000);
  Integer b = Integer.valueOf(1000);
  System.out.println(a == b);
  System.out.println(a.equals(b));
}
```

```
false
true
```

### 캐시 사용

대부분의 박싱된 기본 타입은 캐시를 갖고 있다. (-127 ~ 128)

``` java
public void main(String[] args) {
  Integer a = Integer.valueOf(100);
  Integer b = Integer.valueOf(100);
  System.out.println(a == b);
  System.out.println(a.equals(b));
}
```

```
true
true
```

---

## 2. 초기화

기본 타입은 초기화를 하지 않아도 유효한 값으로 초기화가 되지만 박싱된 기본 타입은 그렇지 않다.

``` java
public class Test {

  int a;
  Integer b;

  public void main(String[] args) {
    Test t = new Test();
    System.out.println(t.a);
    System.out.println(t.b);
  }
}
```

```
0
null
```

---

## 3. 성능

박싱된 기본 타입으로 연산을 수행했을 경우 박싱과 언박싱을 반복적으로 하므로 성능에 저하가 생긴다.

``` java
public void main(String[] args) {
  Long a = Long.valueOf(0);
  long startTime = System.currentTimeMillis();
  for (long i = 0; i < 100000000; i++) { // 10억까지
    a += i;
  }
  System.out.println("result Time: " + (System.currentTimeMillis() - startTime));
}
```

```
result Time: 602
```

반복된 박싱과 언박싱을 하여 연산을 하였을 경우 602ms,

``` java
public void main(String[] args) {
  long a = 0;
  long startTime = System.currentTimeMillis();
  for (long i = 0; i < 100000000; i++) {
    a += i;
  }
  System.out.println("result Time: " + (System.currentTimeMillis() - startTime));
}
```

```
result Time: 54
```

기본 타입을 사용하였을 경우 54ms가 소요된다.

---

# 박싱된 기본 타입이 사용되는 경우

## 컬렉션

``` java
List<Integer> list = new ArrayList<>();
List<int> list = new ArrayList<>(); // 컴파일 에러
```

컬렉션의 원소, 키, 값으로 사용된다.

## 제네릭

위의 컬렉션과 같은 원리이다.

제네릭 타입 매개변수로 기본 타입을 사용할 순 없다.

---

# 박싱과 언박싱

## 박싱

기본 타입인 int가 박싱을 하게될 때 다음과 같은 메서드를 호출한다.

``` java
@HotSpotIntrinsicCandidate
public static Integer valueOf(int i) {
  if (i >= IntegerCache.low && i <= IntegerCache.high)
    return IntegerCache.cache[i + (-IntegerCache.low)];
  return new Integer(i);
}
```

## 언박싱

박싱된 기본 타입을 언박싱하게 될 때 다음과 같은 메서드를 호출한다.

``` java
@HotSpotIntrinsicCandidate
public int intValue() {
  return value;
}
```
