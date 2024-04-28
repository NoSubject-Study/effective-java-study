## 지연 초기화는 신중히 사용하라

### 지연 초기화(lazy initialization)
필드의 초기화 시점을 그 값이 실제로 필요한 때까지 늦추는 기법  
값이 사용되지 않는 경우 초기화되지 않음  
정적 필드와 인스턴스 필드 모두에 사용 가능  
대부분의 상황에서 일반적인 초기화가 지연 초기화보다 낫다  
초기화 순환성(initialization circularity)을 깨뜨릴 것 같으면 synchronized 접근자 사용  

<br>

### 인스턴스 필드 지연 초기화
````java
private FieldType field;

private synchronized FieldType getField() {
  if (field == null) {
    filed = computeFieldValue();
  }
  return field;
}
````

<br>

성능 때문에 정적 필드를 지연 초기화하는 경우 `지연 초기화 홀더 클래스`(lazy initialization holder class) 관용구 사용  

<br>

### 정적 필드용 지연 초기화 홀더 클래스 관용구
````java
private static class FieldHolder {
  static final FieldType field = computeFieldValue();
}

private static FieldType getField() {
  return FieldHolder.field;
}
````

<br>

성능 때문에 인스턴스 필드를 지연 초기화하는 경우 `이중검사`(double check) 관용구 사용  

<br>

### 인스턴스 필드 지연 초기화용 이중검사 관용구
````java
private volatile FieldType field;

private FieldType getField() {
  FieldType result = field;
  if (result != null) {
    return result;
  }

  synchronized(this) {
    if (field == null) {
      field = computeFieldValue();
    }
    return field;
  }
}
````

<br>

이중검사에서 두 번째 검사를 생략할 수 있는 경우 `단일검사`(single check) 관용구  

<br>

### 단일검사 관용구
````java
private volatile FieldType field;

private FieldType getField() {
  FieldType result = field;
  if (result == null) {
    field = result = comoputeFieldValue();
  }
  return result;
}
````

<br>



