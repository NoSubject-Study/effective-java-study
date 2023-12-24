## 복구할 수 있는 상황에는 검사 예외를, 프로그래밍 오류에는 런타임 예외를 사용하라
Use Checked Exceptions for Recoverable Conditions

<img src="https://github.com/NoSubject-Study/effective-java-study/assets/37797830/f73e08b3-dfe9-4bd6-b53a-6663211de34c" width="600"></img>

### 검사 예외 vs 비검사 예외
<img src="https://github.com/NoSubject-Study/effective-java-study/assets/37797830/f73e08b3-dfe9-4bd6-b53a-6663211de34c" width="600"></img>
- 검사 예외(Checked Exception) :
  - 상속 : Exception 
  - 컴파일러에 의해 강제로 처리해야 하는 예외. 즉, 코드에서 이 예외를 처리하거나 던지도록 요구됨.
  - try-catch 블록을 사용하여 검사 예외를 처리하거나, 예외를 더 상위 메서드로 전달하기 위해 throws 절을 사용
  - 예외처리 필수 !
  - <b>개발자의 실수 감소</b>
  - <b>레이어간의 의존성이 높아지고, 스트림 내에서 사용이 어려움</b>
- 비검사 예외(Unchecked Exception) :
  - RunTimeException 
  - 컴파일러에서 예외 처리를 강제하지 않는 예외. 따라서 개발자의 주의로 처리하거나, 처리하지 않아도 컴파일 오류가 발생하지 않음
  - 예외처리 필수 아님
  - <b>개발자의 실수가 많아질 수 있음<b>
  - <b>레이어간의 의존성이 낮아짐</b>
 

### 호출하는 쪽에서 복구하리라 여겨지는 상황이라면 검사 예외 (Checked Exceptions) 를 사용하라
``` java
private static void checkedExceptionWithThrows() throws FileNotFoundException {
    File file = new File("not_existing_file.txt");
    FileInputStream stream = new FileInputStream(file);
}
```
``` java
private static void checkedExceptionWithTryCatch() {
    File file = new File("not_existing_file.txt");
    try {
        FileInputStream stream = new FileInputStream(file);
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    }
}
```

### 프로그래밍 오류를 나타낼 때는 런타임 예외를 사용하자. (Unchecked Exceptions)
- (API 혹은 메소드를 사용하는) 클라이언트가 실수 했거나 개발자의 프로그래밍 오류일 때
- RunTimeException의 하위 클래스 :
  - IllegalArgumentException
  - IlleagalStateException
  - NullPointerException
  - IndexOutOfBoundsException


### 예외처리할 때 주의점
- ⛔️ Error는 보통 JVM의 자원 부족 등 외부 상황으로 인해 프로그램이 계속 수행되지 못할때 사용되니, 커스텀 에러 클래스를 만드는 일은 없어야 한다.
- ⛔️ 예외 메시지를 API Response 처럼 사용하지 말자?
  - 예외를 일으킨 상황에 대한 정보를 전달하는데, JVM이나 릴리즈에 따라 포맷이 달라질 수 있다.
  - 메시지 문자열을 파싱해 얻은 코드는 깨지기 쉽고 다른 환경에서 동작하지 않을 수 있다.
- ❗️ 검사예외를 사용했을 땐 예외 상황에서 벗어나는데 필요한 정보를 알려주는 메소드를 함께 제공하자    
    
``` java
public void getCookie() {
    goCookieStore();
    try {
        Stirng cookie = pickCookie();
        buyCookie(cookie);
    } catch (CookieSoldOutException e) {
        log.error("선택한 쿠키 {}의 재고가 없습니다. 에러메시지 : {}", cookie, e.getMessage());
    }
}
```
