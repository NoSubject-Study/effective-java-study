이펙티브 자바 # item-09
try-finally보다는 try-with-resources를 사용하라.

try-with-resources는 자원을 회수하는 최선책이다.

앞선 item-08에서 finalizer와 closer 사용을 피하자고 했고 대안으로 try-finally -> try-with-resources가 소개 되었다.

try-finally는 왜 피해야할까?

예시 코드를 살펴보면서 확인하자.


```java
    static String firstLineOfFile(String path) throws IOException {
       
        BufferedReader br = new BufferedReader(new FileReader(path));
        try {
            return br.readLine();
        } finally {
            br.close();
        }
    }
```
해당 코드의 문제점은? try 블록에 진입을 우선했다면, close 메소드는 실행된다. 그러나 진입하지 않았다면 close 메소드가 실행되지 못한다. (BufferedReader br = new BufferedReader(new FileReader(path));에서 종료 될 경우)

또한 아래의 코드를 보자.
```java
package item09;

import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
import java.net.URL;

public class FileTest {

    static class CustomBufferedReader extends BufferedReader {
        public CustomBufferedReader(FileReader fileReader) {
            super(fileReader);
        }

        @Override
        public String readLine() throws IOException {
//            URL fileUrl = FileTest.class.getResource("/item09/forTest.txt");
//            if (fileUrl == null) {
//                throw new IOException("파일을 찾을 수 없습니다 in readLine.");
//            }
            //주석처리 된 부분을 주석처리 해제하고 실행시켜서, IOException을 유도해도 try 블록 자체에는 진입하므로 firstLineOfFile메소드에서 close()메소드는 실행된다.
            System.out.println("readLine 메서드 호출");
            return super.readLine();
        }

        @Override
        public void close() throws IOException {
//            URL fileUrl = FileTest.class.getResource("/item09/forTest.txt");
//            if (fileUrl == null) {
//                throw new IOException("파일을 찾을 수 없습니다 in close.");
//            }
            //주석처리 된 부분을 주석처리 해제하고 실행시켜서, IOException을 유도하여 readLine과 close 메소드 모두 예외를 던지면(즉 try,finally 블록 모두)  디버깅이 힘들어진다.

            System.out.println("close 메서드 호출");
            super.close();
        }
    }

    static String firstLineOfFile(String path) throws IOException {
        URL fileUrl = FileTest.class.getResource(path);
        if (fileUrl == null) {
            throw new IOException("파일을 찾을 수 없습니다.");
        }
        String filePath = fileUrl.getPath();
        CustomBufferedReader br = new CustomBufferedReader(new FileReader(filePath));
        try {
            return br.readLine();
        } finally {
            br.close();
        }
    }

    public static void main(String[] args) {
        try {
            String firstLine = firstLineOfFile("/item09/test.txt");
            System.out.println("파일의 첫 줄: " + firstLine);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```

주석을 해제처리 하고 실행하면, try, finally 블록 모두에서 예외가 발생하고 두번째의 예외가 첫번째 예외를 집어삼킨다. 따라서 디버깅이 힘들어진다.

```text
java.io.IOException: 파일을 찾을 수 없습니다 in close.
	at item09.FileTest$CustomBufferedReader.close(FileTest.java:30)
	at item09.FileTest.firstLineOfFile(FileTest.java:49)
	at item09.FileTest.main(FileTest.java:55)
```

두 번째로는 자원을 두 개 이용할 시 코드가 복잡해지는 이유가 있다.
```java
   static void copy(String src, String dst) throws IOException {
       
  	InputStream in = new FileInputStream(src);
       
        try {
            OutputStream out = new FileOutputStream(dst);
            try {
                byte[] buf = new byte[BUFFER_SIZE];
                int n;
                while ((n = in.read(buf)) >= 0)
                    out.write(buf, 0, n);
            } finally {
                out.close();
            }
        } finally {
            in.close();
        }
    }
```
(이 코드도 첫번째 이유의 단점을 포함하고 있기도 하다.)


개선된 버전은 이렇다. try-with-resources를 적용해야 하는 것이다.


```java
    static String firstLineOfFile(String path) throws IOException {
        try (BufferedReader br = new BufferedReader(
                new FileReader(path))) {
            return br.readLine();
        }
    }
```

```java
    static void copy(String src, String dst) throws IOException {
        try (InputStream in = new FileInputStream(src);
             OutputStream out = new FileOutputStream(dst)) {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        }
    }
```

코드가 깔끔해지고, 자원 회수도 필수적으로 이루어진다.
또한 try, finally 모두에서 예외가 발생할 시에 두번째 예외가 첫번째 예외를 덮었는데, 이렇게 숨겨진 예외도 그냥 버려지는 것이 아니라 (suppressed)라는 꼬리표를 달고 출력된다.

또한 아래의 코드처럼 try-with-resources를 catch 문과 함께 써서 기본값을 반환하는 것도 가능하다.

```java
    static String firstLineOfFile(String path, String defaultVal) {
    try (BufferedReader br = new BufferedReader(
                new FileReader(path))) {
            return br.readLine();
        } catch (IOException e) {
            return defaultVal;
        }
    }
```