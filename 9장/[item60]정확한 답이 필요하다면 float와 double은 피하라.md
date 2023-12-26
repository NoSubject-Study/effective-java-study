# float와 double 타입을 피해야하는 이유

float와 double 타입은 과학과 공학 계산용으로 설계되었다. 이진 부동소수점 연산에 쓰이기 때문에 정확한 값을 계산할 수 없다.

``` java
System.out.println(0.1 + 0.2);
```

``` text
결과: 0.30000000000000004
```

---

## 부동소수점

![부동소수점](https://codetorial.net/articles/_images/floating_point_04.png)

부동소수점을 표현하는 방식도 정하는 방식에 따라 다를 수 있지만 일반적으로 사용하고 있는 방식은 IEEE에서 표준으로 제안한 방식이다.(IEEE - 754)

### -9.625를 부동소수점으로 표현하게 되면

부호가 음수이므로 1이 된다.
```
1 | 00000000 | 0000000 00000000 00000000
```

9.625를 2진수로 표현하게 되면 1001.101이 된다.

1001.101을 소수점 왼쪽에 1만 남도록 소수점을 이동하게 되면 1.001101이 된다.

아래와 같이 표현할 수 있다.

```
정규화된 부동소수점
1001.101 = 1.001101 X 2^3
```

여기서 소수점의 오른쪽 부분을 가수에 채워주도록 한다.

```
1 | 00000000 | 00110100 00000000 00000000
```

127과 정규화된 부동소수점의 2의 지수인 3을 더한 값인 130을 2진수로 나타내어 지수 부분에 채워주도록 한다.

```
1 | 10000010 | 0011010 00000000 00000000
```

---

### 무한 소수

위처럼 9.625와 같은 숫자가 아닌 0.1을 부동소수점으로 나타낸다고 가정해 보자.

먼저 2진수를 나타내면

0.000110011... 과 같이 끝이 없는 소수가 되는데 이를 무한 소수라고 한다.

그리고 이것을 부동소수점으로 표현하기 위해서 반올림하여 가수 부분을 채우게 된다.

그렇기 때문에 대부분의 실수 계산에 오차가 생기는 것이다.

---

# 정확한 계산을 위한 대안

## BigDecimal

정확한 계산을 하기 위해 BigDecimal을 사용할 수 있다.

``` java
BigDecimal decimal1 = new BigDecimal(".1");
BigDecimal decimal2 = new BigDecimal(".2");
System.out.println(decimal1.add(decimal2));
```

```
결과: 0.3
```

하지만 BigDecimal은 기본 타입보다 느리다는 단점을 갖고 있다.

아래는 BigDecimal을 이용해 1부터 10000000까지 모두 더했을 경우에 걸린 시간을 계산한 결과이다.

``` java
BigDecimal result = new BigDecimal("0");
long startTime = System.currentTimeMillis();
for (int i = 1; i <= 10000000; i++) {
	result = result.add(new BigDecimal(i));
}
System.out.println("ResultTime = " + (System.currentTimeMillis() - startTime));
System.out.println("Result = " + result);
```

```
ResultTime = 144
result = 50000005000000
```

아래는 long 타입을 이용해 계산한 결과이다.

``` java
long result = 0;
long startTime = System.currentTimeMillis();
for (int i = 1; i <= 10000000; i++){
	result += i;
}
System.out.println("ResultTime = " + (System.currentTimeMillis() - startTime));
System.out.println("Result = " + result);
```

```
ResultTime = 8
result = 50000005000000
```


long 타입을 이용한 계산이 훨씬 빠른 것을 볼 수 있다.

일회성 계산을 하는 경우에는 BigDecimal이 좋은 방법일 수 있다.

---

# 결론

성능 저하를 신경 쓰지 않겠다면 BigDecimal을 사용하면 된다.

하지만 성능이 중요하고 소수점을 직접 추적할 수 있고 숫자가 너무 크지 않다면 int나 long을 사용하여 소수점을 직접 관리하여 계산하면 된다.
