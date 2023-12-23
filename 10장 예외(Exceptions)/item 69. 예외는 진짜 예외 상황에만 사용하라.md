## 예외는 진짜 예외 상황에만 사용해라 (Use Exceptions for Only Exceptional Circumstances)

### 에러 VS 예외
- 에러(error) : 컴퓨터 하드웨어의 오동작 또는 고장으로 인해 응용프로그램 실행 오류가 발생하는 것
- 예외(exception) : 사용자의 잘못된 조작 또는 잘못된 코딩으로 인해 발생하는 프로그램 오류     

💡 둘 다 발생될 시 프로그램이 종료된다는 점에서 같으나, <b>예외는 예외처리(Exception Handling)을 통해 프로그램을 종료하지 않고 정상 실행 상태가 유지</b>되도록 할 수 있다는 점에서 차이

### 검사 예외 vs 비검사 예외
<img src="https://github.com/NoSubject-Study/effective-java-study/assets/37797830/f73e08b3-dfe9-4bd6-b53a-6663211de34c" width="600"></img>
- 검사 예외(Checked Exception) :
  - 컴파일러에 의해 강제로 처리해야 하는 예외. 즉, 코드에서 이 예외를 처리하거나 던지도록 요구됨.
  - try-catch 블록을 사용하여 검사 예외를 처리하거나, 예외를 더 상위 메서드로 전달하기 위해 throws 절을 사용
- 비검사 예외(Unchecked Exception) :
  - 컴파일러에서 예외 처리를 강제하지 않는 예외. 따라서 개발자의 주의로 처리하거나, 처리하지 않아도 컴파일 오류가 발생하지 않음
 
### 예외를 처리할 때 주의할 점
- 오직 예외 상황에서만 사용한다. 절대로 일상적인 제어 흐름용으로 쓰여선 안 된다.
- 잘 설계된 API라면 클라이언트가 정상적인 제어흐름에서 예외를 사용할 일이 없게 해야 한다.

### 일반적인 제어 흐름에서 예외를 사용하지 않고 처리하는 방법
1. 옵셔널이나 특정 값을 사용하기
``` java
    Optional<List<String>> result = getList();

    List<String> defaultList = new ArrayList<>();
    defaultList.add("기본 항목 1");
    defaultList.add("기본 항목 2");

    return result.orElseGet(() -> defaultList);
```
``` java
    public static Optional<Integer> getPositiveNumber(int number) {
        if (number > 0) {
            return Optional.of(number);
        } else {
            return Optional.empty();
        }
    }
```
- 상태 검사 메서드와 상태 의존적 메서드 호출 사이에 객체의 상태가 변할 때
- 성능이 중요한 상황에서 지속적인 상태 검사 메소드가 수행되는 것을 제외하고 싶을때


3. 상태 검사 메서드를 사용하기
``` java
public String findNameById(String id) {
    return id == null ? null : "example-name";
}
```
- 가독성이 좋고, 잘못 사용했을 때 발견하기가 쉬움

