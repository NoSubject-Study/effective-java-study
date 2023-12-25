# 필요없는 검사 예외 사용은 피하라
Avoid Unnecessary Use of Checked Exceptions 

### 검사 예외 사용의 장단점
- ➕ 프로그램의 안전성, 개발자의 실수 감소
- ➖ 예외를 던지는 메소드 혹은 API의 사용자에게 부담
  - 특히 검사 예외를 던지는 메서드는 스트림 안에서는 직접 사용하기 불가능
  - 받는 쪽에서 프로그램의 중단이 아니고, 처리해서 복구 가능
👉 예외를 받는 사용자(프로그래머)가 의미 있는 조치를 취하는 등의 경우가 아니라면 <b>비검사 예외를 사용하자!</b> 

### 검사 예외를 회피하는 방법
API내부에 이미 여러 검사 예외를 던진다면 예외 하나쯤 더 추가하는 것은 무리가 아니지만,   
API내부에 첫 검사 예외를 추가한다면 정말 회피할 수 있는 방법은 없을지??? 고려해보자. 
    
1. 적절한 결과 타입을 담은 옵셔널을 반환하자.
- default, 빈 옵셔널을 반환하자.
- 그러나 예외가 던져주는 구체적인 예외 타입, 예외가 발생한 이유를 알려주는 부가정보, 메시지를 사용할수 없다.

2. 메서드를 2개로 쪼개 비검사 예외로 반환한다.
- 상태검사 메소드를 사용하거나 분기처리를 통해 검사 예외가 아닌 비검사 예외를 사용한다.
- 무조건 성공하리란 걸 안다거나 실패시에 프로그램을 중단 시키고 싶다면 처리하지 않아도 된다. 
``` java
    public static void validateAgeWithCheckedException(int age) throws IllegalArgumentException {
        if (age < 0 || age > 120) {
            throw new IllegalArgumentException("유효한 나이가 아닙니다.");
        }
    }
```
``` java
    public static boolean isValidAge(int age) {
        return age >= 0 && age <= 120;
    }

    public static void main(String[] args) {
        int age = -5;
        
        if (isValidAge(age)) {
            System.out.println("입력한 나이는 유효합니다.");
        } else {
            System.err.println("입력한 나이가 유효하지 않습니다.");
        }
    }
```

### 핵심 정리 (검사 예외, 비검사 예외를 선택하는 지침
> API 호출자가 예외상황에서 복구할 방법이 없고, 프로그램이 실행중단 되어야 하는 경우 : 비검사 예외  
> 복구 가능하고, 호출자가 처리해주길 원한다면 우선 옵셔널/기본 값 반환을 고려   
> 옵셔널로는 상황을 처리하기에 충분한 정보를 제공하기 어려울 때 : 검사 예외   


### Ref
https://dev.to/kylec32/effective-java-avoid-unnecessary-use-of-checked-exceptions-59je
