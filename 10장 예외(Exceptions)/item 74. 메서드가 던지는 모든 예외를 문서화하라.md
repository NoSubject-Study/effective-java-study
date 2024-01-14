# 메서드가 던지는 모든 예외를 문서화하라
Document all exceptions thrown by each method

메서드가 던지는 예외는 메서드를 올바로 사용하는데 아주 중요한 정보이다!
따라서 각 메서드가 던지는 예외들을 문서화하는게 중요하다

## 검사 예외
- 검사 예외는 항상 따로따로 선언하고, 각 예외가 발생하는 상황을 자바독 @throws를 사용하영 정확히 문서화한다.
```java
/ **
 * 주석의 설명문
 * 
 * @throws java.io.FileNotFoundException 지정된 파일을 찾을 수 없습니다
 * /
```
```java
     * @throws NullPointerException if the window argument is null
     * @throws NullPointerException if the point argument is null
     * @throws IllegalArgumentException if the window is trusted (i.e.
     * the {@code getWarningString()} returns null
     * @throws IllegalArgumentException if the alignmentX or alignmentY
     * arguments are not within the range [0.0f ... 1.0f]
```

⛔️ 메서드가 Exception이나 Throwable 같은 공통 상위 클래스를 던진다고 선언하지 마라!    
  메서드 사용자에게 각 예외에 대처할 수 있는 힌트도 주지 못하고, 같은 맥락에서 발생할 여지가 있는 다른 예외들까지 삼켜버릴 수 있어 API 사용성을 떨어뜨린다. 
  (단, main 은 오직 JVM 만이 호출하므로 Exception 을 던지도록 선언해도 괜찮다.)

💡 만약 한 클래스에 정의된 많은 메서드가 같은 이유로 같은 예외를 던진다면, 그 예외를 각 메서드가 아닌 예외 클래스의 설명에 추가한다.
```java
/**
 * Thrown when an application attempts to use {@code null} in a
 * case where an object is required.
 * ...
 */
public
class NullPointerException extends RuntimeException {
  ...
}
```

## 비검사 예외
- 필수 요구사항은 아니지만, 비검사 예외도 문서화해두면 좋다.
  - 비검사 예외는 일반적으로 프로그래밍 오류이기 때문에, 자신이 일으킬 수 있는 오류들이 무엇인지 문서를 통해 알려주면 이를 사용하는 프로그래머들이 해당 오류가 나지 않도록 코딩할 수 있다. 
- ⛔️ 비검사 예외는 @throws 목록에 넣지 않는다!
  - 검사, 비검사에 따라 API 사용자가 할 일이 달라지기 때문에 이 둘은 확실히 구분한다!

 
## 정리
메서드가 던질 가능성이 있는 모든 예외를 문서화하라.     
검사/비검사 예외, 추상/구체 메서드 모두에 해당한다.     
검사 예외만 메서드 선언의 throws 문에 일일히 선언해서 문서화 하고, 비검사 예외는 기입하지 말자. 
