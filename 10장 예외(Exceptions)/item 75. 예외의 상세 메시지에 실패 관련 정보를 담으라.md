# 예외의 상세 메시지에 실패 관련 정보를 담으라
Include failure-capture information in detail messages

- 📌 예외 메시지에는 실패순간을 포착할 수 있도록 <b>발생한 예외의 모든 매개변수와 필드 값을 실패 메시지에</b> 담아야 한다. 📌
- 👉 만약, 예외 메시지에 실패 순간의 매개변수 혹은 필드값이 없다면 오류를 파악해야 하는 SRE나 개발자들은 Throwable의 stacktrace로만 유추해야한다. 
- 💡 보통 예외를 분석하는 이들은 소스코드를 함께 분석하기 때문에, 소스코드에서 알 수 있는 정보보단 외부에서 전달받은 정보 위주로 메시지에 담을 수 있도록 한다!

### 예외 메시지에 담기
``` java
    public boolean isStudentInClass(int studentId, String className) throws CustomException {
        for (Student student : studentList) {
            if (student.getId() == studentId && student.getClassName().equals(className)) {
                return true;
            }
        }

        // 해당 학생이 해당 클래스에 없는 경우 예외를 발생시킵니다.
        String errorMessage = "학생 ID " + studentId + "는 " + className + " 수업에 속하지 않습니다.";
        throw new CustomException(errorMessage, CustomExceptionType.NOT_STUDENT_FOR_CLASS);
    }
```



### 예외클래스의 생성자 안에 담기
``` java
/** Constructs an IndexOutOfBoundsException.
 * @param lowerBound the lowest legal index value
 * @param upperBound the highest legal index value plus one
 * @param index the actual index value
 */
public IndexOutOfBoundsException(int lowerBound, int upperBound, int index) {
  // Generate a detail message that captures the failure
  super(String.format("Lower bound: %d, Upper bound: %d, Index: %d", lowerBound, upperBound, index));

  // Save failure information for programmatic access
  this.lowerBound = lowerBound;
  this.upperBound = upperBound;
  this.index = index;
}
```
자바 9에서 부터 IndexOutOfBoundsException에 정수 인덱스 값을 포함하는 생성자가 추가됨.     
이렇게 예외 클래스의 생성자 안에서 상세 메시지를 생성해두면, 예외 상세 메시지를 만드는 코드를 예외 클래스안에서 처리하기 때문에     
클래스 사용자가 메시지를 만드는 작업을 하지 않아도 된다


## 주의할 점
- ⛔️ 비밀번호, 암호 키값과 같은 민감한 정보들은 노출되지 않도록 한다!
- ⛔️ 소스코드에서 분석할 수 있는 지나친 정보들을 예외 메시지에 담지 않는다
- ⛔️ 유저 레벨 메시지와 혼동하지 말자! 
  - 우리가 지금 이야기 하는 메시지는 개발자 혹은 SRE가 프로그램의 문제 상황을 파악하기 위한 정보이기 때문에 친절하거나 감싼 정보를 줄 필요가 없음
  - 가독성 필요없이, 상태를 파악할 수 있는 정보를 전달
    
