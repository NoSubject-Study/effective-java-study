자바 플랫폼의 명명 규칙은 자바 언어명세 (JLS 6.1)에 기술되어 있다.

JSL: https://docs.oracle.com/javase/specs/jls/se11/html/jls-6.html

---

# 명명 규칙을 따라야 하는 이유

이 규칙을 어긴 API는 사용하기 어렵고, 유지보수하기 어렵다.

철자 규칙이나 문법 규칙을 어기면 다른 프로그래머들이 그 코드를 읽기 번거로울 뿐 아니라 다른 뜻으로 오해할 수도 있고 그로 인해 오류까지 발생할 수 있다.

---

# 철자 규칙

## 패키지와 모듈

패키지와 모듈 이름은 각 요소를 점(.)으로 구분하여 계층적으로 짓는다.

요소들은 모두 소문자 알파벳 혹은 (드물게)숫자로 이뤄진다.

조직 바깥에서도 사용될 패키지라면 인터넷 도메인 이름을 역순으로 사용한다. (edu.cmu, com.google)

예외적으로 표준 라이브러리와 선택적 패키지들은 각각 java와 javax로 시작한다.

패키지 이름의 나머지는 해당 패키지를 설명하는 하나 이상의 요소로 이뤄진다.

각 요소는 일반적으로 8자 이하의 짧은 단어로 한다.

utilities보다는 util처럼 의미가 통하는 약어를 추천한다.

여러 단어로 구성된 이름이라면 awt(Abstract Window Toolkit)처럼 각 단어의 첫 글자만 따서 써도 좋다.

인터넷 도메인 이름 뒤에 요소 하나만 붙인 패키지가 많지만, 많은 기능을 제공하는 경우엔 계층을 나눠 더 많은 요소로 구성해도 좋다.

예를 들어, `java.util`은 `java.util.concurrent.atomic`과 같이 그 밑에 수많은 패키지를 가지고 있다.

---

## 클래스와 인터페이스

클래스와 인터페이스의 이름은 하나 이상의 단어로 이뤄지며, 각 단어는 대문자로 시작한다.

여러 단어의 첫 글자만 딴 약자나 max, min처럼 널리 통용되는 줄임말을 제외하고는 단어를 줄여 쓰지 않도록 한다.

약자의 경우 첫 글자만 대문자로 할지, 전체를 대문자로 할지는 논란이 있다.

첫 글자만 대문자로 하는 쪽이 훨씬 많다.

HttpUrl처럼 여러 약자가 혼합된 경우라도 각 약자의 시작과 끝을 명확히 알 수 있기 때문이다.(HttpUrl, HTTPURL)

---

## 메서드와 필드 이름

첫 글자를 소문자로 쓴다는 점만 빼면 클래스 명명 규칙과 같다.

### 상수 필드

‘상수 필드’는 예외이다. 상수 필드를 구성하는 단어는 모두 대문자로 쓰며 단어 사이는 밑줄로 구분한다.(VALUES, NEGATIVE_INFINITY…)

상수 필드는 값이 불변인 static final 필드를 말한다.

이름에 밑줄을 사용하는 요소로는 상수 필드가 유일하다.

### 지역 변수

지역 변수에도 다른 멤버와 비슷한 명명 규칙이 적용된다.

단, 약어를 써도 좋다.

약어를 써도 그 변수가 사용되는 문맥에서 의미를 쉽게 유추할 수 있기 때문이다.

입력 매개변수도 지역변수의 하나다.

하지만 메서드 설명 문서에까지 등장하는 만큼 일반 지역변수보다는 신경을 써야 한다.

### 타입 매개변수

타입 매개변수는 보통 한 문자로 표현한다.

대부분은 다음의 다섯 가지 중 하나다.

T: 임의의 타입

E: 컬렉션 원소의 타입

K, V: 맵의 키와 값

X: 예외

R: 메서드의 반환 타입

임의 타입의 시퀀스에는 T, U, V 혹은 T1, T2, T3를 사용한다.

| 식별자 타입 | 예 |
| --- | --- |
| 패키지와 모듈 | org.junit.jupiter.api, com.google.common.collect |
| 클래스와 인터페이스 | Stream, FutureTask, LinkedHashMap, HttpClient |
| 메서드와 필드 | remove, groupingBy, getCrc |
| 상수 필드 | MIN_VALUE, NEGATIVE_INFINITY |
| 지역 변수 | i, denom, houseNum |
| 타입 매개 변수 | T, E, K, V, X, R, U, T1, T2 |

---

# 문법 규칙

## 패키지

패키지에 대한 규칙은 따로 없다.

---

## 클래스와 인터페이스

보통 단수 명사나 명사구를 사용한다. (Thread, PriorityQueue, ChessPiece…)

객체를 생성할 수 없는 클래스의 이름은 보통 복수형 명사로 짓는다. (Collectors, Collections…)

인터페이스 이름은 클래스와 똑같이 짓거나(Collection, Comparator…), able 혹은 ible로 끝나는 형용사로 짓는다.(Runnable, Iterable, Accessible…)

애너테이션은 다양하게 활용되어 지배적인 규칙이 없이 명사, 동사, 전치사, 형용사가 두루 쓰인다.(BindingAnnotation, Inject, ImplementedBy, Singleton…)

---

## 메서드

메서드의 이름은 동사나 동사구로 짓는다.(append, drawImage…)

boolean 값을 반환하는 메서드라면 보통 is나 (드물게) has로 시작하고 명사나 명사구, 혹은 형용사로 기능하는 아무 단어나 구로 끝나도록 짓는다. (isDigit, isProbablePrime, isEmpty, isEnabled…)

반환 타입이 boolean이 아니거나 해당 인스턴스의 속성을 반환하는 메서드의 이름은 보통 명사, 명사구, 혹은 get으로 시작하는 동사구로 짓는다(size, hashCode, getTime…)

get으로 시작하는 형태만 써야 한다는 주장도 있지만,

``` java
if (car.speed() > 2 * SPEED_LIMIT) {
  generateAudibleAlert("경찰 조심하세요!");
}
```

위처럼 명사나, 명사구가 가독성이 더 좋은 경우가 있다.

get으로 시작하는 형태는 주로 자바빈즈(JavaBeans) 명세에 뿌리를 두고 있다.

자바빈즈는 재사용을 위한 컴포넌트 아키텍처의 초기 버전 중 하나로, 이 명명 규칙을 따르는 도구와 어우러지는 코드를 작성한다면 이 규칙을 따라도 상관없다. (getAttribute, setAttribute)

객체의 타입을 바꿔서, 다른 타입의 또 다른 객체를 반환하는 인스턴스 메서드의 이름은 보통 toType 형태로 짓는다. (toString, toArray…)

객체의 내용을 다른 뷰로 보여주는 메서드의 이름은 asType 형태로 짓는다. (asList….)

객체의 값을 기본 타입 값으로 반환하는 메서드의 이름은 보통 typeValue 형태로 짓는다. (intValue…)

정적 팩터리의 이름은 다양하지만 from, of, valueOf, instance, getInstance, newInstance, getType, newType을 흔히 사용한다.

---

## 필드 이름

필드 이름의 명명 규칙은 클래스, 인터페이스, 메서드 이름에 비해 덜 명확하고 덜 중요하다.

API 설계를 잘 했다면 필드가 직접 노출될 일이 거의 없기 때문이다.

boolean 타입의 필드 이름은 보통 boolean 접근자 메서드에서 앞 단어를 뺀 형태다.(initialized, composite…)

다른 타입의 필드라면 명사나 명사구를 사용한다.(height, digits, bodyStyle…)

지역 변수도 필드와 비슷하게 지으면 되나, 조금 더 느슨하다.
