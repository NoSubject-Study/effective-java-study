# 스트림 병렬화는 주의해서 적용해라
Use caution when making streams parallel
----

- **자바 8**부터 pararell 메소드 지원
  - pararell메소드를 스트림처리에 호출하면 파이프라인을 병렬 실행할 수 있음
  - 동시성 프로그래밍을 할 때는 **안전성(Safety)**, **응답 가능(Liveness)**에 주의해야하는 것과 마찬가지로 *pararell*메소드를 사용할 때도 이를 고려해야 함
 

## pararell()메소드를 사용하면 안되는 예시
``` java

// Stream-based program to generate the first 20 Mersenne primes
public static void main(String[] args) {
       primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
           .filter(mersenne -> mersenne.isProbablePrime(50))
           .limit(20)
           .forEach(System.out::println);
}
static Stream<BigInteger> primes() {
       return Stream.iterate(TWO, BigInteger::nextProbablePrime);
}

```
- 처음 20개의 메르센 소수를 구하는 프로그램
  - 메르센 수 : M(n) = 2^n -1 형태의 수
  - 메르센 소수는 메르센 수 중 소수인 것, n이 소수이면 해당 메르센 수도 소수
 
- ❗️ 성능 개선을 기대했으나, 실제로 pararell()메소드를 추가해서 실행하면 1시간 반이 지날때까지 아무 결과도 출력하지 않음.
  - 스트림 라이브러리가 이 파이프라인을 병렬화하는 방법을 찾아내지 못했기 때문
  - 데이터 소스가 *Stream.iterate*거나 중간 연산으로 *limit*을 쓰면 성능 개선이 힘듦
 
## 병렬화 효과가 좋은 케이스
1. 스트림의 소스가 ArrayList, HashMap, HashSet, ConcurrentHashMap의 인스턴스거나 배열, int 범위, long 범위일 때
- 이 자료구조들의 공통점
  - **정확성**
  - 모두 데이터를 원하는 크기로 정확하고 손쉽게 나눌 수 있어 다수의 스레드에 일을 분배하기에 좋음
  - 나누는 작업은 Spliterator 를 통해 이루어지며, Iterable 과 Stream 에서 얻을 수 있다.
  - **참조 지역성 (locality of refrence)**
  - 위 자료구조들은 원소들을 순차적으로 실행할 때 참조지역성이 뛰어남.
    - 참조지역성이 높다? = 이웃한 원소들의 참조들이 메모리에 연속해서 저장되어 있다
    - 참조지역성이 낮다? = 참조들이 가리키는 실제 객체가 메모리에서 서로 떨어져있어 데이터가 캐시 메모리로 오는 시간 소요
  - **참조지역성**은 다량의 데이터를 처리하는 벌크 연산을 병렬화 할때 아주 중요함 요소!
  - 가장 뛰어난 자료구조 : 기본 타입 배열 (데이터 자체가 메모리에 연속해서 저장)
 
2. 종단연산으로 축소(reduction)을 사용하는 경우
- 종단연산의 동작방식도 병렬 수행 효율에 큰 영향
- 종단연산은 파이프라인에서 만들어진 원소를 하나로 합치는 작업으로, 비교적 간단한 연산들이 적합
  - reduce, anyMatch, allMatch, noneMatch
  - *collect 와 같이 컬렉션들을 합치는 부담이 큰 메서드는 병렬화에 부적합*
 
  - 
