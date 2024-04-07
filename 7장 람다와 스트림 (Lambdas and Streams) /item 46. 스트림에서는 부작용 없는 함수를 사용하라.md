# 스트림에서는 부작용 없는 함수를 사용하라
Prefer side-effect-free functions in streams

## 스트림 패러다임의 핵심
- 스트림의 패러다임의 핵심은 계산을 일련의 변환(transformation)으로 재구성하는 것이다.
- 각 변환 단계는 가능한 이전 단계의 결과를 받아 처리하는 `순수 함수(pure function)`여야 한다.

## 순수 함수란 ?
- 오직 입력만이 결과에 영향을 주는 함수
- 다른 가변 상태를 참조하지 않고, 함수 스스로도 다른 상태를 변경하지 않는 함수
   
``` java
// 순수 함수
IntFunction pureMethod= (x -> x+1);

// 부작용이 있는 함수
Random<Integer> randomizer;
IntFunction<Integer> nonPureMethod = (x -> x + randomizer.nextInt());
```

### 예시
- ⛔️ 스트림 패러다임을 이해하지 못한 채 API 만 사용한 예시  
- 종단 연산 foreach문에서 외부 상태(freq)를 수정하고 있음 
  - *foreach 연산은 스트림 계산 결과를 보고할 때만 사용하고, 계산하는데는 쓰지 말자*
```java
   // Uses the streams API but not the paradigm--Don't do this!
   Map<String, Long> freq = new HashMap<>();
   try (Stream<String> words = new Scanner(file).tokens()) {
       words.forEach(word -> {
           freq.merge(word.toLowerCase(), 1L, Long::sum);
      });
   }
```
- ✅ 스트림 연산을 제대로 활용해서 빈도표(freq)를 초기화하는 예시
```java
   // Proper use of streams to initialize a frequency table
   Map<String, Long> freq;
   try (Stream<String> words = new Scanner(file).tokens()) {
       freq = words
           .collect(groupingBy(String::toLowerCase, counting()));
    }
```

## java.util.stream.Collectors
- 스트림을 사용하려면 꼭 배워야 하는 클래스
- 축소 전략을 캡슐화한 블랙박스 객체
- 스트림의 원소를 손쉽게 컬렉션으로 모을 수 있음
- toList(), toSet(), toCollection(collectionFactory)

#### 📍 빈됴표에서 가장 흔한 단어 10개를 뽑아내는 파이프라인
✅ Collectors의 정적메소드 toList를 활용, Collectors의 멤버를 정적 임포트해서 사용하면 가독성이 좋아진다. 
``` java
List<String> topTen = freq.keySet().stream()
                .sorted(Comparator.comparing(freq::get).reversed()) 
                .limit(10)
                .collect(Collectors.toList());
```

### Collectors의 toMap 활용하기
#### 📍 매핑 : 2개의 인수를 받는 간단한 toMap 활용 (keyMapper, valueMapper)
- toMap을 활용해서 문자열을 열거 타입 상수에 매핑
``` java
private static final Map<String, Operation> stringToEnum =
        Stream.of(values()).collect(toMap(Object::toString, e -> e));
```
#### 📍 새로운 맵 생성 : 각 키와 해당 키의 특정 원소를 연관 짓는 맵을 생성
``` java
 Map<Artist, Album> topHits = albums.collect(
        toMap(Album::artist, a->a, maxBy(comparing(Album::sales))));
```
#### 📍 마지막에 쓴 값을 취하는 예시
``` java
toMap(keyMapper, valueMapper, (oldVal, newVal) -> newVal)
```

### Collectors의 groupingBy 활용하기
- groupingBy는 입력으로 분류 함수 (classifier)를 받고 출력으로는 원소들을 카테고리별로 모아 놓은 맵을 담은 수집기를 반환
- 분류함수(classifier) : 입력받은 원소가 속하는 카테고리 반환
  - 반환된 카테고리는 해당 원소의 맵 키로 사용
  - 반환된 맵에 담긴 각각의 값은 해당 카테고리(키)에 속하는 원소들을 모두 담은 리스트

#### 📍 분류 함수(classifier)만 받는 groupingBy 예시
``` java
words.collect(groupingBy(word -> alphaetize(word)));
```
#### 📍 다운스트림, 분류함수를 받는 groupingBy 예시
``` java
Map<String, Long> freq = words.collect(groupingBy(String::toLowerCase, counting());
```
#### 📍 맵 팩터리, 다운스트림, 분류함수를 받는 groupingBy 예시
``` java
// 학생 객체를 학급 별로 그룹화하고, 각 그룹에 대해 학생 수 계산
// TreeMap을 사용하여 결과를 정렬된 맵으로 저장
Map<String, Long> studentCountByClass = students.stream()
               .collect(Collectors.groupingBy(
                Student::getClassroom, // classifier: 학급을 기준으로 그룹화
                TreeMap::new, // mapFactory: TreeMap을 사용하여 그룹화된 결과 저장
                Collectors.counting() // downstream: 학생 수를 계산
));
```

### Collectors의 joining
- CharSequence 인스턴스의 스트림에만 적용 가능, 단순히 원소들은 concat 한다.
- charSequence타입의 구분문자(delimeter)를 매개변수로 받아 연결 부위에 이 구분문자를 삽입해준다.
``` java
words.stream()
     .filter(w -> w.length() > 5)
     .collect(Collectors.joining(","));
```

## 핵심 정리 
- 스트림 파이프라인 프로그래밍의 핵심은 부작용 없는 함수 객체에 있다.
- 스트림 관련 객체에 건네지는 모든 함수 객체가 부작용이 없어야 한다.
- forEach는 계산 자체에 이용하지 말자
- 스트림을 올바르게 사용하기 위해서는 Collector를 잘 이해하자
    - toList, toSet, toMap, groupingBy, joining 



