## 타입 안전 이종 컨테이너를 고려하라  
제네릭의 모든 쓰임에서 매개변수화되는 대상은 원소가 아닌 컨테이너 자신이다.  
하나의 컨테이너에서 매개변수화할 수 있는 타입의 수는 제한된다.  
만약 컨테이너 대신 키를 매개변수화한다면 더욱 유연하게 사용가능하다.  

</br>

### 타입 안전 이종 컨테이너 패턴(type safe heterogeneous container pattern)  
키를 매개변수화해서 컨테이너에 값을 삽입, 조회할때 매개변수화한 키를 함께 제공한다.  

```` java
public class Favorites {
  public <T> void putFavorite(Class<T> type, T instance);
  public <T> T getFavorite(Class<T> type);
}

public static void main(String[] args) {
  Favorites f = new Favorites();

  f.putFavorite(String.class, "Java");
  f.putFavorite(Integer.class, 0xcafebabe);
  f.putFavorite(Class.class, Favorites.class);

  String favoriteString = f.getFavorite(String.class);
  int favoriteInteger = f.getFavorite(Integer.class);
  Class<?> favoriteClass = f.getFavorite(Class.class);

  System.out.printf("%s %x %s%n", favoriteString, favoriteInteger, favoriteClass)

}

public class Favorites {
  private Map<Class<?>, Object> favorites = new HashMap<>();

  public <T> void putFavorite(Class<T> type, T instance) {
    favorites.put(Objects.requireNonNull(type), instance);
  }

  public <T> T getFavorite(Class<T> type) {
    return type.cast(favorites.get(type));
  }
}
````

</br>

### Favorites 클래스 제약  
1. 악의적인 클라이언트가 로 타입을 사용할 경우 타입 안전성이 쉽게 깨진다.  
2. 실체화 불가 타입에는 사용할 수 없다.  

</br>

### 슈퍼 타입 토큰(super type token)  
자바 업계 거장인 닐 개프터(Neal Gafter)가 고안한 방식이다.  
스프링 프레임워크에서는 아예 ParameterizedTypeReference 클래스로 미리 구현해놓았다.  
슈퍼 타입 토큰을 이용해서 제네릭 타입 클래스 레터럴을 문제업이 저장가능하다.  

```` java
Favorites f = new Favorites();
List<String> pets = Arrays.asList("dog", "cat", "chicken");
f.putFavorie(new TypeRef<List<String>>(){}, pets);
List<String> listofStrings = f.getFavorite(new TypeRef<List<String>>(){});
```` 

