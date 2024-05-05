## 공개된 API 요소에는 항상 문서화 주석을 작성하라

### 자바독(javadoc)
소스코드 파일에서 문서화 주석(doc comment)이라는 특수한 형태로 기술된 설명을 추려 API 문서로 변환  
문서화 주석 작성 규칙의 표준은 없지만 업계 표준인 [문서화 주석 작성법](https://www.oracle.com/kr/technical-resources/articles/java/javadoc-tool.html) 웹페이지에 기술  
공개된 모든 클래스, 인터페이스, 메서드, 필드 선언에 문서화 주석 필수  
자바독 유틸리티는 문서화 주석을 HTML로 변환하므로 주석 안에 HTML 태그 사용 가능  

<br>

### 메서드용 문서화 주석
해당 메서드와 클라이언트 사이의 규약을 명료하게 기술  
클라이언트가 해당 메서드를 호출하기 위한 전제조건(`precondition`), 수행된 후 만족해야 하는 사후조건(`postcondition`), 부작용 기술  
`@throws` 태그로 비검사 예외를 선언하여 암시적으로 기술  
`@param` 태그로 매개변수가 뜻하는 값 기술  
`@return` 태그로 void를 제외한 반환값 기술  

<br>

````java
/**
 * Returns the element at the specified position in this list.
 *
 * <p>This method is <i>not</i> guaranteed to run in constant
 * time. In some implementations it may run in time proportional
 * to the element position.
 *
 * @param index index of element to return; must be
 *        non-negative and less than the size of this list
 * @return the element at the specified position in this list
 * @throws IndexOutOfBoundException if the index is out of range
 *         ({@code index < 0 || index >= this.size()})
 */
E get(int index);

````

<br>

### 상속용 클래스
자기사용 패턴(self-use pattern)에 대해서도 기술  
`@implSpec` 태그로 해당 메서드와 하위 클래스 사이의 계약 기술  
자바 11까지는 자바독 명령줄에 `-tag "implSpec:Implementation Requirements:"` 추가해야 동작  

<br>

````java
/**
 * Returns true if this collection is empty.
 *
 * @implSpec
 * This implementation returns {@code this.size() == 0}.
 *
 * @return true if this collection is empty
 */
public boolean isEmpty() { ... }

````

<br>

### HTML 메타문자
HTML 마크업이나 자바독 태그를 무시하는 기능  
`@literal` 태그로 감싸서 &lt;, &gt;, & 등의 메타문자 포함 가능  
`@code` 태그와 유사하지만 코드 폰트로 렌더링하지 않음  

<br>

````java
/**
 * A geometric series converges if {@literal |r| < 1}.
 *
 * ...
 */
...

````

<br>

### 요약 설명(summary description)
각 문서화 주석의 첫 번째 문장은 해당 요소의 요약 설명으로 간주  
반드시 대상의 기능을 고유하게 기술  
한 클래스/인터페이스 안에서 요약 설명이 똑같은 멤버/생성자가 둘 이상 금지  
요약 설명 끝의 기준은 {&lt;**마침표**&gt; &lt;**공백**&gt; &lt;**소문자가 아닌 다음 문장 시작**&gt;} 패턴의 &lt;**마침표**&gt;  
자바 10부터 요약 설명 전용 태그인 `@summary` 태그 추가  

<br>

````java
/**
 * A suspect, such as Colonel Mustard or {@literal Mrs. Peacock}.
 */
public class Suspect { ... }

/**
 * {@summary A suspect, such as Colonel Mustard or Mrs. Peacock.}
 */
public class Suspect { ... }

````

<br>

메서드/생성자의 요약 설명은 해당 메서드와 생성자의 동작을 설명하는 주어 없는 동사구(3인칭)  
> * **ArrayList(int initialCapacity)**: Constructs an empty list with specified initial capacity.  
> * **Collection.size()**: Returns the number of elements in this collection.  

<br>

클래스/인터페이스/필드의 요약 설명은 대상을 설명하는 명사절  
> * **Instant**: An instantaneous point on the time-line.  
> * **Math.PI**: The **double** value that is closer than any other to pi, the ratio of the circumference of a circle to its diameter.  

<br>

### 색인
자바 9부터 검색/색인 기능 추가  
클래스, 메서드, 필드 같은 API 요소는 자동 색인  
`@index` 태그로 사용자 지정 색인 가능  

<br>

````java
/**
 * This method complies with the {@index IEEE 754} standard.
 * ... 
 */
...

````

<br>

### 제네릭
제네릭 타입이나 제네릭 메서드는 모든 타입 매개변수에 주석 필수  

<br>

````java
/**
 * An object that maps keys to values. A map cannot contain
 * duplicate keys; each key can map to at most one value.
 *
 * (Remainder omitted)
 *
 * @param <K> the type of keys maintained by this map
 * @param <V> the type of mapped values
 */
public interface Map<K, V> { ... }

````

<br>

### 열거 타입
열거 타입은 상수들에도 주석 필수

<br>

````java
/**
 * An instrument section of a symphony orchestra.
 */
public enum OrchestraSection {
  /** Woodwinds, such as flute, clarinet, and oboe. */
  WOODWIND,

  /** Brass instruments, such as french horn and trumpet. */
  BRASS,

  /** Percussion instruments, such as timpani and symbals. */
  PRECUSSION,

  /** Stringed instruments, such as violin and cello. */
  STRING;
}
````

<br>

### 어노테이션
어노테이션 타입은 멤버들에도 모두 주석 필수

<br>

````java
/**
 * Indicates that the annotated method is a test method that
 * must throw the designated exception to pass.
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
  /**
   * The exception that the annotated test method must throw
   * in order to pass. (The test is permitted to throw any)
   * subtype of the type described by this class object.)
   */
  Class<? extends Throwable> value();
}
````

<br>

### 패키지/모듈
패키지를 설명하는 문서화 주석은 `package-info.java` 파일에 작성  
모듈 시스템을 사용한다면 `module-info.java` 파일에 작성  
이 파일은 패키지 선언을 반드시 포함, 패키지 선언 관련 어노테이션을 추가로 포함 가능  

<br>

### 스레드 안정성 & 직렬화 가능성
API 문서화에서 가장 많이 누락되는 설명  
스레드 안정/직렬화 가능 여부에 상관없이 스레드 안전 수준/직렬화 가능성은 반드시 API 설명에 포함  

<br>

### 메서드 주석 상속
자바독은 메서드 주석을 `상속` 가능  
문서화 주석이 없는 API 요소 발견시 자바독이 가장 가까운 문서화 주석 탐색(인터페이스 -&gt; 상위 클래스 순)  
`@inheritDoc` 태그로 상위 타입의 문서화 주석 일부를 상속 가능  

<br>


