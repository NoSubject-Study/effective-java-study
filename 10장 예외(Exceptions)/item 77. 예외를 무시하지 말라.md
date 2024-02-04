# 예외를 무시하지 말라
Don't ignore exceptions

### 이런식으로 예외를 무시하지 말자! 
``` Java
// Empty catch block ignores exception - Highly suspect!
try {
...
} catch (SomeException e){
}
```
- 📌 API 설계자가 예외를 발생시키는 까닭은, 그 메서드를 사용할 때 적절한 조치를 취해달라고 하는 것! 
- 💡 위와 같이 적절한 예외처리를 하지 않고 무시한다면, 예외의 존재 이유가 없다!
- ⛔️ 빈 catch 블록으로 못 본척 지나가면 그 프로그램은 오류를 내재한 채 동적하게 되므로, 어느 순간 문제의 원인과 아무 상관 없는 곳에서 죽어버릴 수 있다. 

### 간혹 예외를 무시해도 되는 케이스?
``` java
        String fileName = "example.txt"; // 읽을 파일의 경로와 이름

        try (FileInputStream fileInputStream = new FileInputStream(fileName)) {
            int data;
            while ((data = fileInputStream.read()) != -1) {
                // 파일에서 읽은 데이터 처리
                System.out.print((char) data);
            }
        } catch (IOException ignored) {
            // System.err.println("파일 읽기 오류: " + e.getMessage());
        }
```
- 파일을 읽어들이기만 했으므로 파일의 상태를 변경하지 않았다. 👉 <b>복구할 것이 없다</b>
- 스트림을 닫을 땐 필요한 정보가 다 끝났다는 것이므로 남은 작업을 강제 중단할 필요도 없다.
- 혹시 읽어들이는 코드에서 예외가 자주 발생한다면 로그로 남겨도 좋을 것.    

### 예외를 무시하기로 결정했을 때!
``` Java
Future<Integer> f = exec.submit(planarMap::chromaticNumber);
int numColors = 4; // Default; guaranteed sufficient for any map
try {
  numColors = f.get(1L, TimeUnit.SECONDS);
} catch (TimeoutException | ExecutionException ignored) {
  // 기본값을 사용한다 ( 색상 수를 최소화하면 좋지만, 필수는 아니다. )
}
```
- 📌 catch block내에 예외를 무시하기로 결정한 이유를 주석으로 남겨라
- 📌 예외 변수의 이름도 ignroed로 바꿔라
