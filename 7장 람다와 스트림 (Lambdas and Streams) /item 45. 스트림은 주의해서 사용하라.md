# 스트림은 주의해서 사용하라
Use streams judiciously

## 스트림 API란?
- 다량의 데이터 처리 작업을 위해 자바 8부터 추가됨
- 스트림API의 핵심 개념은 두 가지, **스트림의 시퀀스(sequence)**, **스트림 파이프라인(pipeline)**

### 스트림 (stream sequence)
- 데이터 원소의 유한 혹은 무한 시퀀스를 뜻하는 스트림
  - A sequence of elements supporting sequential and parallel aggregate operations
  - 데이터 처리 연산을 지원하도록 소스에서 추출된 요소
  - 컬렉션, 배열, 파일, 정규표현식 패턴 매처, 난수 생성기 혹은 다른 스트림 등이 스트림의 원소가 될 수 있음
  - 스트림 안의 데이터 원소는 객체 참조, 기본 타입 값
    - 기본 타입으로는 int, long, double을 지원 (char은 지원하지 않음)
- 이 원소들로 수행하는 연산 단계를 표현하는 스트림 파이프 라임

### 스트림 파이프라인(pipeline)
``` java
        List<String> words = Arrays.asList("hello", "world", "java", "stream");
        words.stream()
             // 중간 연산: 문자열 길이가 5 이상인 것만 필터링
             .filter(word -> word.length() > 5)
             // 종단 연산: 각 문자열 출력
             .collect(Collectors.toList());
```
- 소스 스트림에서 시작해 중간연산(intermediate operation)을 거쳐 종단연산(terminal operation)으로 끝남
- 각 **중간연산**은 스트림을 어떠한 형식으로 변환할 수 있다.
  - ex) 각 원소에 함수를 적용하거나, 특정 조건에 따라 필터링하거나, ....
  - 중간연산은 모두 스트림을 다른 스트림으로 **변환**하는데 변환된 스트림의 원소 타입은 전과 다를 수도, 같을 수도 있다.
  - filter, map, flatMap, distinct, sorted 
- **종단 연산**은 마지막 중간 연산의 결과에 최후의 연산을 한다.
  - ex) 원소를 정렬해 컬렉션에 담거나, 특정 원소를 선택하거나..
  - forEach, collect, reduce, count 
- 스트림 파이프라인은 `지연 평가(lazy evaluation)`된다.
  - 즉, 평가는 종단 연산이 호출될 때 이루어지며, 종단 연산에 쓰이지 않는 데이터 원소는 계산에 쓰이지 않는다.
- 스트림 파이프라인은 **순차적**으로 수행된다.
  - 파이프라인을 병렬로 실행하려면, 파이프라인을 구성하는 스트림 중에서 pararell 메소드를 호출하면 되나, 효과를 볼 수 있는 상황은 많지 않다.
- **지연 평가(lazy evaluation)**된다
  - 종단 연산이 수행될 때 평가된다 = 종단 연산이 수행되지 않으면 아무일도 일어나지 않는다.
  - 필요할 때만 평가가 되므로 메모리를 효율적으로 사용, 대량의 데이터일때 더 효율적
- **병렬 처리**?
  - 병렬로 실행하기 위해서는 parallel() 을 호출하면 되지만, 효과를 볼 수 있는 상황은 많지 않다고 한다 (item 48)

## 스트림을 지나치게 사용해서 가독성이 저하된 경우
- 사용자 입력수(minGroupSize) 보다 많은 아나그램 값을 가지고 있는 아나그램 출력
- <aelpst> : [aelpst, staple, petals]
### 스트림을 사용하기 전
``` java
public class OriginalAnagrams {
    public static void main(String[] args) throws IOException {
        File dictionary = new File(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        Map<String, Set<String>> groups = new HashMap<>();
        try (Scanner s = new Scanner(dictionary)) {
            while (s.hasNext()) {
                String word = s.next();
                // 맵 안에 키가 있는지 찾은 후 있으면 키에 매핑된 값을 반환해서 맵에 추가
                groups.computeIfAbsent(alphabetize(word), 
                        (unused) -> new TreeSet<>()).add(word);
            }
        }

        // groups : {aelpst=[petals, staple]}
        for (Set<String> group : groups.values())
            if (group.size() >= minGroupSize)
                System.out.println(group.size() + ": " + group);
    }

    // 알파벳 순서대로 정렬 (staple -> aelpst) == anagram의 key
    private static String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
```

### Bad Practice ! 스트림 과용
``` java
// Overuse of streams - don't do this!
// 스트림을 과용하면 가독성이 너무 떨어짐
public class StreamAnagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(
                    groupingBy(word -> word.chars().sorted()
                            .collect(StringBuilder::new,
                                    (sb, c) -> sb.append((char) c),
                                    StringBuilder::append).toString()))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .map(group -> group.size() + ": " + group)
                    .forEach(System.out::println);
        }
    }
}
```

### Best Practice ! 스트림의 적절한 활용예시
``` java
public class HybridAnagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);try (Stream<String> words = Files.lines(dictionary)) {
    words.collect(Collectors.groupingBy(word -> alphabetize(word)))
             .values().stream()
             .filter(group -> group.size() >= minGroupSize)
             .forEach(g -> System.out.println(g.size() + ": " + g));
        }
    }private static String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
```
- 📌 스트림으로 바꾸는게 가능할지라도 코드 가독성과 유지보수 측면에서 손해를 볼 수도 있다 
- 📌 기존 코드는 스트림을 사용해서 리팩토링하되, 새 코드가 나아 보일때만 반영하자

## 스트림 사용이 적합한 경우 VS 적합하지 않은 경우
### 적합한 경우
- *반복문의 특징을 사용하지 않아도 되는 경우* 
  - 원소들의 시퀀스를 일관되게 변환하는 경우
  - 원소들의 시퀀스를 필터링하는 경우
  - 원소들의 시퀀스를 하나의 연산을 사용해 결합하는 경우
  - 원소들의 시퀀스를 컬렉션에 모으는 경우(공통된 속성을 기준으로)
  - 원소들의 시퀀스에서 특정 조건을 만족하는 원소를 찾는 경우
 
### 적합하지 않는 경우
- *반복문의 특징을 사용해야 하는 경우*
  - 범위 안의 지역 변수를 읽고 수정이 가능
  - break, continue를 활용하여 반복문을 종료, 반복을 뛰어 넘을 수 있음
  - 메서드 선언에 예외 처리가 가능함

## 스트림으로 처리하기 어려운 일
- 한 데이터가 파이프라인의 여러 단계를 통과할 때 이 데이터의 각 단계에서의 값들에 동시에 접근하기는 어려운 경우
- 스트림 파이프라인은 한 값을 다른 값에 매핑하고 나면 원래의 값은 잃는 구조이기 때문이다.
- 원래 값과 새로운 값의 쌍을 저장하는 객체를 사용해 매핑하는 우회 방법도 있지만, 복잡하기 떄문에 사용하지 않는 경우가 좋다

## 스트림 VS 반복문 어느 쪽을 써야할지 어려울 때
👉 본인과 같이 일하는 동료들이 이해하기 쉬운 쪽으로
```java
// Iterative Cartesian product computation
// CartesianProduct(['1','2','3'],['a','b','c'])
// == ['1a','1b','1c','2a','2b','2c','3a','3b','3c']
private static List<Card> newDeck(){
	List<Card> result=new ArrayList<>();
	for(Suit suit:Suit.values())
	  for(Rank rank:Rank.values())
	      result.add(new Card(suit,rank));
	          return result;
	}

```  

```java
// Stream-based Cartesian product computation
   private static List<Card> newDeck() {
       return Stream.of(Suit.values())
       .flatMap(suit -> Stream.of(Rank.values())
       .map(rank -> new Card(suit, rank))) .collect(toList());
```
  
