# item-06 불필요한 객체 생성을 피하라.

## 똑같은 기능의 객체를 매번 생성하기보다는 객체 하나를 재사용하는 편이 나을 때가 많다.

```java
String s = new String("bikini);

String s = "bikini";
```

두 문장의 코드에서 윗문장은 String 인스턴스를 계속해서 만든다. (메서드 실행히 빈번하다면 String 인스턴스가 계속해서 만들어질 수 있다.) 이는 String이 불변 객체임을 고려할 때 불필요한 행위이다.

아래 문장은 같은 문자열 리터럴에 대해서 동일한 String 인스턴스를 재사용한다.(동일한 참조 객체를 가리킴)
-> 메모리 사용을 줄이고 성능 향상에 도움이 된다. (문자열 리터럴은 JVM 내에서 동일한 객체로 관리됨)
<br>
<br>
```java
//생성자 사용 예시
Boolean true1 = new Boolean("true");
Boolean true2 = new Boolean("true");
System.out.println(true1 == true2); // false, 서로 다른 객체
```
- new Boolean("true")는 true1과 true2가 각각 새로운 Boolean 객체를 참조하게 한다.
  <br>
  <br>
```java
//정적 팩터리 메서드 사용 예시
Boolean true1 = Boolean.valueOf("true");
Boolean true2 = Boolean.valueOf("true");
System.out.println(true1 == true2); // true, 같은 객체 재사용
```
- Boolean.valueOf("true")는 필요에 따라 기존 객체를 재사용한다.

## 가변 객체 재사용 예시
데이터 베이스 커넥션 풀을 예시로 들 수 있다.

데이터 베이스 커넥션을 획들할때는 커넥션 조회, DB와 TCP/IP 커넥션 연결, ID/PW DB에 전달, 인증 완료, 내부에 DB 세션 생성, 커넥션 생성 완료 응답, 커넥션 객체 생성 후 클라이언트에 반환... 의 복잡한 과정을 거치기 때문에 비용이 많이 소모 된다.

따라서 커넥션 객체를 미리 pool에 생성해두는 커넥션 풀을 이용한다. (요청마다 연결 시간이 소비 되지 않는다.)

## 생성 비용이 비싼 객체
```java
static boolean isRomanNumeral(String s) {
	return s.matches("^(?=.)M*(C[MD]|D?C{0,3})" + "(X[CL]|L?X{0,3})(I[XV]|V?		I{0,3})$");
}
```
isRomanNumeral 될 때마다 정규표현식을 컴파일 하는 비용이 발생한다.
'Pattern은 입력받은 정규표현식에 해당하는 유한상태 머신을 만들기 때문에 인스턴스 생성비용이 높다'
-> 유한상태머신을 쉽게 말하면 입력에 따라 상태가 변경되는 모델이다.
유한상태머신은 주어진 문자열을 순회하면서, 조건을 체크하여 정규 표현식과 일치하는지 판단한다.
<br>
<br>

```java
public class RomanNumerals {
	private static final Pattern ROMAN = Pattern.compile(
	"^(?=.)M*(C[MD]|D?C{0,3})" + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

	static boolean isRomanNumeral(String s) {
		return ROMAN.matcher(s).matches();
	}
}
```
성능 개선을 위해 위처럼 Pattern 객체를 정적 필드로 선언 초기화한다.(클래스가 메모리에 로드될 때 한 번만 초기화됨, 해당 클래스의 모든 인스턴스에서 공유됨) 이렇게 하면 Pattern 객체가 한 번만 생성되어서 isRomanNumeral메서드에서 필요할 때마다 재사용 된다.(캐싱)

위 예시를 통해 '비싼' 객체들은 캐싱하여 재사용하는 것이 좋다는 것을 알 수 있다.
이 코드에서 isRomanNumeral 메서드가 한 번도 호출하지 않는다면 쓸데 없이 초기화 된 것이므로 지연 초기화를 생각할 수 있으나 코드 복잡성이 늘고 성능 개선은 크게 되지 않아 권장되지 않는다.
<br>
<br>

## 오토박싱으로 인한 성능 저하
```java
private static long sum() {
    Long sum = 0L; // 박싱된 타입 (Long)
    for (long i = 0; i <= Integer.MAX_VALUE; i++) {
        sum += i; // 오토박싱 발생
    }
    return sum;
}
```
위 코드에서는 따라서, 반복문이 실행될 때마다 새로운 Long 객체가 생성되어 성능 저하가 발생한다.
sum 변수를 기본 타입인 long으로 변경하면, 오토박싱이 발생하지 않으므로 성능이 향상된다.
```text
박싱된 기본 타입보다는 기본 타입을 사용하고, 의도치 않은 오토 박싱이 숨어들지 않도록 주의해야한다.
```
<br>
<br>
아이템 50의 제목과 해당 아이템의 내용이 상반되는 것으로 보이는데, 방어적 복사가 필요한 상황에서 객체를 재사용했을 때의 피해가 크므로 이럴 때는 필요 없는 객체여도 반복 생성하는 것이 낫다.(버그와 보안 구멍으로 이어짐)