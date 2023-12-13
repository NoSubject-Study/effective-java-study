지역변수의 범위를 최소화하면 코드 가독성, 유지보수성이 높아지고 오류 가능성이 낮아진다.

다음은 지역변수의 범위를 줄이는 3가지 방법이다.

## 사용시점에 선언하기

지역변수는 사용할 시점에 선언하는 것이 가장 좋다.
만약 미리 선언하게 되면 코드가 어수선해져 가독성이 떨어진다.
또한 변수를 실제로 사용하는 시점에 해당 변수의 타입, 초기값이 기억나지 않을 수 있다.

그리고 만약 사용하는 블록 바깥에서 선언하게 될 경우 모두 사용하고 나서도 살아 있게 되므로 오류를 불러 일으키기 쉽다.

``` java
public void badMethod() { // 좋지 않은 코드
    Animal d = new Dog();
    Animal c = new Cat();
    
    Information information = getInformation();
    
    String dogName = information.getDogName();
    String catName = information.getCatName();
    d.setName(dogName);
    c.setName(catName);
    
    addAnimal(d, c);
}

public void goodMethod() {
    Information information = getInformation();
    String dogName = information.getDogName();
    String catName = information.getCatName();
    
    Animal d = new Dog();
    d.setName(dogName);
    
    Animal c = new Cat();
    c.setName(catName);
    
    addAnimal(d, c);
}
```

## 선언과 동시에 초기화하기

초기화에 필요한 정보가 충분하지 않다면 충분해질 때까지 선언을 미뤄야 한다.
이는 가독성도 좋지않고, 불완전한 객체를 생성함으로써 개발자에게 실수를 유도할 수 있기 때문이다.

하지만 try-catch 문은 예외다.
변수를 초기화 하는 표현식에서 예외가 발생할 가능성이 있다면 try 블록 안에서 초기화해야 한다.
이는 메서드로 예외가 전파될 수 있기 때문이다.

변수를 정확히 초기화하지 못하더라도 try 블록 바깥에 선언해야 할 경우에는 try 블록 바로 앞에서 선언해야 한다.
아래가 그 예이다.

``` java
public Connection getConnection() {
    Connection conn = null;
    
    try {
    	Class.forName(DB_DRIVER);
        conn = DriverManager.getConnection(DB_URL, ID, PASSWORD);
    } catch (Exception e) {
    	...
    }
    
    return conn;
}
```

그리고 반복문을 사용할 경우 반복 변수의 범위는 for 키워드와 몸체 사이의 괄호, 반복문의 몸체 안으로 제한된다. 따라서 반복 변수를 반복문이 종료되고 나서도 사용해야 하는 경우가 아니라면 while 문보다 for 문을 사용하는 편이 좋다.

``` java
for (Element e : c) {
    ... // e로 무언가를 한다.
}
```

만일 반복자를 사용해야 하는 경우라면 for-each 문 대신 for 문을 사용하는 것이 좋다.

``` java
for (Iterator<Element> i = c.iterator(); i.hasNext();) {
    Element e = i.next();
    ... // e와 i로 무언가를 한다.
}
```

다음의 코드를 보면 while 문보다 for 문이 더 나은 이유를 알 수 있다.

``` java
Iterator<Element> i = c.iterator();
while (i.hasNext()) {
  doSomething(i.next());
}

...

Iterator<Element> i2 = c2.iterator();
while (i.hasNext()) { // Bug!
  doSomething(i2.next());
}
```

while 문을 사용하게 될 경우 while 블록 바깥에서도 해당 변수가 살아있기 때문에 실수를 할 가능성이 높다. 그리고 이는 컴파일 시점에는 알 수 없는 런타임 에러이므로 나중에 해당 버그를 찾아내기 어렵다.

하지만 for 문을 사용하게 되면 for 문 몸체 안으로 변수의 범위가 좁혀지기 때문에 바깥에서는 사용을 할 수 없어 실수할 가능성이 줄어들게 된다.

``` java
for (Iterator<Element> i = c.iterator(); i.hasNext();) {
    doSomething(i.next());
}

for (Iterator<Element> i = c2.iterator(); i.hasNext();) {
    doSomething(i.next());
}
```

위의 두 for 문에서 i는 각각 다른 변수이다.

또한 for 문을 사용할 경우 while 문보다 짧아서 가독성이 좋다는 장점도 있다.

``` java
for (int i = 0, n = expensiveComputation(); i < n; i++) {
    ... // i로 무언가를 한다.
}
```

다음은 while 문을 사용했을 경우이다.

``` java
int i = 0;
while (i < expensiveComputation()) {
    ... // i로 무언가를 한다.
    i++;
}
```

for 문을 사용하게 되면 변수 n에 expensiveComputation() 메서드의 반환 값을 저장하여 다시 메서드를 호출할 필요가 없게 된다. 하지만 while 문을 사용하게 되면 매 비교마다 호출해야 한다.

## 메서드를 작게 유지하기

한 메서드에서 여러가지 기능을 처리한다면 그중 한 기능과 관련된 지역변수라도 다른 기능을 수행하는 코드에서 접근할 수 있을 것이다. 메서드를 기능별로 나누면 간단히 해결할 수 있다.
