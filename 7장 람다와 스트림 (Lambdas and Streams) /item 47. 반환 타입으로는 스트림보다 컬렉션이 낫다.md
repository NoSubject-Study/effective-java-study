# item 47. 반환 타입으로는 스트림보다 컬렉션이 낫다 
Prefer Collection to Stream as a return type

## 스트림 반복문
- 스트림은 반복(iteration)을 지원하지 않는다 
  - 따라서 스트림과 반복을 알맞게 조합해야 좋은 코드가 나온다.
- 그런데 API가 스트림만 반환한다면? → for-each로 반복하길 원하는 API사용자들의 불편함
- Stream 인터페이스는 Iterable 인터페이스가 정의한 추상 메서드를 전부 포함할 뿐만 아니라 ,Iterable 인터페이스가 정의한 방식대로 동작한다.
  - **하지만, ArrayList, LinkedList같은 자료형과 다르게 스트림은 Iterable을 확장하고 있지 않기 때문에 for-each로 스트림을 반복할 수 없다.**
  - 그렇다면, 어떻게 ?
 
### Stream ↔ Iterator 어댑터 활용
#### Stream → Iterator
``` java
public static <E> Iterable<E> iterableOf(Stream<E> stream){
  return stream::iterator;
}
```
``` java
for(ProcessHandle p : iterableOf(ProcessHandle.allProcesses())){
  // 프로세스 처리 로직
}
```
``` java
    public static void main(String[] args) {
        // Assume we have a stream of user information
        Stream<User> userStream = getUserStream();

        // 스트림 -> itreable
        Iterable<User> users = iterableOf(userStream);

        // 반복문 활용
        for (User user : users) {
            System.out.println("Name: " + user.getName());
            System.out.println("Age: " + user.getAge());
            System.out.println("----------------------");
        }
    }
```
#### Iterator → Stream
``` java
public static <E> Stream<E> streamOf(Iterable<E> iterable){
  return StreamSupport.stream(iterable.spliterator(), false);
}
```
``` java
    public static void main(String[] args) {
        // Assume we have a list of users
        List<User> userList = new ArrayList<>();
        userList.add(new User("Alice", 25));
        userList.add(new User("Bob", 30));
        userList.add(new User("Charlie", 35));

        // Iterable -> 스트림
        Stream<User> userStream = streamOf(userList);

        // 스트림 처리
        userStream.filter(user -> user.getAge() > 30)
                  .forEach(user -> System.out.println(user.getName() + " is older than 30."));
    }
```
- 📌 **Stream이나 Iterator 한 가지로만 반환하면 API를 사용하는 사용자가 불편해하게 된다. 이를 중개해주는 어댑터를 활용하는 방안도 있다.**
- 📌 **하지만, 어댑터를 사용하는 방법은 난잡하고, 직관성이 떨어지므로 쓰지 않는 것을 권장한다.**
- 📌 API사용자가 스트림/Iterable으로만 활용할 것이 확실하다면 스트림으로만 반환하거나 Iterable로만 반환해도 되지만, 확신하기는 어렵다.
- **💡둘다 지원하기 위해서는 Collection으로 리턴하자!**
  - Collection 인터페이스는 Itreable의 하위 타입이고 stream 메소드도 제공해서 반복과 스트림을 동시에 지원한다.
  - 원소 시퀀스를 반환하는 public API의 리턴 타입으로는 Collection 또는 Collection의 하위 타입을 쓰는게 일반적이다.
    - List, Set, Map..

```java
    public static List<User> getUsers() {
        List<User> userList = new ArrayList<>();
        userList.add(new User("Alice", 25));
        userList.add(new User("Bob", 30));
        userList.add(new User("Charlie", 35));
        return userList;
    }

    public static void main(String[] args) {
        // Collection으로 전달받은 사용자 리스트
        List<User> userList = getUsers();

        // 스트림으로 활용 가능
        userList.stream()
                .filter(user -> user.getAge() > 30)
                .forEach(user -> System.out.println(user.getName() + " is older than 30."));

        // 반복문으로도 활용 가능
        for (User user : userList) {
            System.out.println("Name: " + user.getName());
            System.out.println("Age: " + user.getAge());
            System.out.println("----------------------");
        }
    }

```
## 전용 컬렉션 구현
- 컬렉션을 반환한다는 이유로 덩치 큰 시퀀스를 메모리에 올려서는 안된다. -> 원소의 개수가 너무 많아지면 컬렉션은 그만큼 많은 메모리를 차지하므로 커스텀 컬렉션을 구현하는 방안을 생각해보아야 한다.
-  **반환할 시퀀스가 크지만 표현을 간결하게 할 수 있다면 전용 컬렉션을 구현하는 방안을 검토하는 것이 좋다**

### 전용 컬렉션 예시 - 멱집합(Power Set) 구하기
- 멱집합(power set)이란? 한 집합의 모든 부분집합을 원소로 하는 집합
- ex) {a, b, c}의 멱집합 : {{}, {a}, {b}, {c}, {a, b}, {a, c}, {b, c}, {a, b, c}}
- 원소 개수가 N개 일때, 멱집합 원소의 개수는 2^n개 (2^3 = 8)
- 원소 개수가 많아지먄 많아 질수록 멱집합 원소가 매우 커질 가능성 → 표준 컬렉션에 저장하기엔 메모리에 부담
- AbstractList를 활용해서 전용 컬렉션 구현!

``` java
public class PowerSet {
  public static final <E> Collection<Set<E>> of(Set<E> s) {
    List<E> src = new ArrayList<>(s);
    if(src.size() > 30) {
      // 과도한 메모리 사용을 방지하기 위해 제한
      throw new IllegalArgumentException("집합에 원소가 너무 많습니다.(원소 최대 30개) : " + s);
    }

    return new AbastractList<Set<E>>() {
      @Override public int size() {
        // 멱집합의 크기는 2를 원래 집합의 원소 수 만큼 거듭제곱된 것과 같다.
        // 멱집합 크기 계산
        return 1 << src.size();
      }

      @Override public boolean contains(Object o){
        //  주어진 객체가 부분 집합인지 확인
        return o instanceof Set && src.containsAll((Set)o);
      }

      @Override public Set<E> get(int index){
        Set<E> result = new HashSet<>();
        for(int i = 0; index !=0; i++, index >>= 1){
          if((index & 1) == 1){
            result.add(src.get(i));
          }
        }
        return result;
      }
    }
  }
}
```
``` java
        Set<Integer> s = new HashSet<>(Arrays.asList(1, 2, 3));
        Collection<Set<Integer>> p = PowerSet.of(s);
        System.out.println(p);
```

## 핵심 정리
- 원소 시퀀스를 반환하는 메소드를 작성할 때는, 이를 스트림으로 처리하기 원하는 사용자와 반복으로 처리하길 원하는 사용자가 모두 있을 수 있다!
  - 양쪽을 모두 만족 시키려고 노력하자!
- 컬렉션을 리턴할 수 았다면 그렇게 하라!
  - 이미 원소들을 컬렉션에 담아 관리하고 있거나, 컬렉션을 하나 더 만들어도 될정도로 원소개수가 적다면 ArrayList같은 표준 컬렉션에 담아 리턴해라.
  - 그렇지 않고 원소의 개수가 너무 크다면 PowerSet예시같이 커스텀 컬렉션 구현을 고민해라.
- 컬렉션 반환이 불가능하다면, 스트림과 Iterable중 더 자연스러운것을 반환해라  
