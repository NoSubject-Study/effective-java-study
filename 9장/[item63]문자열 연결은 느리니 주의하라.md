# 문자열을 연결할 경우 String 대신 StringBuilder를 사용하자.

문자열 연결 연산자로 문자열 n개를 잇는 시간은 n^2에 비례한다.

문자열은 불변이므로 문자열을 연결할 경우 양쪽의 문자열을 복사해야 한다.
String 대신 StringBuilder를 사용하면 성능을 개선할 수 있다.

``` java
public static void main(String args) {
  String result = "";
  long startTime = System.currnetTimeMillis();
  for (int i = 0; i < 10_000; i++) {
    result += i;
  }

  System.out.println("Result Time: " + (System.currentTimeMillis() - startTime));
}
```

```
Result Time: 223
```

``` java
public static void main(String args[]) {
  StringBuilder result = new StringBuilder();
  long startTime = System.currentTimeMillis();
  for (int i = 0; i < 10_000; i++) {
    result.append(i);
  }

  System.out.println("Result Time: " + (System.currentTimeMillis() - startTime));
}
```

```
Result Time: 6
```
