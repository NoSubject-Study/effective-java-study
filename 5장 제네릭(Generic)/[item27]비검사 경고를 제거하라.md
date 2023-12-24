## 비검사 경고를 제거하라 

### 제네릭 사용시 받는 경고 
1. 비검사 형변환 경고 
2. 비검사 메서드 호출 경고 
3. 비검사 매개변수화 가변인수 타입 경고 
4. 비검사 변환 경고

### 비검사 경고
```` java
Set<Lark> exaltation = new HashSet();

Venery.java:4: warning: [unchecked] unchecked conversion
    Set<Lark> exaltation = new HashSet();
                           ^
  required: Set<Lark>
  found:    HashSet
````

### 자바 7 다이아몬드 연산자
```` java
Set<Lark> exaltation = new HashSet<>();
````

### 만약 경고 제거가 불가능하지만 타입 안전이 보장된 경우
@SuppressWarning("unchecked") 어노테이션 사용  
가능한 한 좁은 범위로 설정 필수  
또한 경고를 무시해도 안전한 이유를 주석으로 설명 필수  

### ArrayList의 toArray 메서드
```` java
public <T> T[] toArray(T[] a) {
  if (a.length < size)
    return (T[]) Arrays.copyOf(elements, size, a.getClass());
    System.arraycopy(elements, 0, a, 0, size);
    if (a.length > size)
      a[size] = null;
    return a;
}

ArrayList.java:305: warning: [unchecked] unchecked cast
    return (T[]) Arrays.copyOf(elements, size, a.getClass());
                               ^
  required: T[]
  found:    Object[]
````

### 지역변수를 추가해 범위를 좁힘
```` java
@SuppressWarnings("unchecked") T[] result = (T[]) Arrays.copyOf(...);
````


