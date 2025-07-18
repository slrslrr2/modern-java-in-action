# 자바 8, 9, 10에서는 무슨 일이?

## 1. 주요 변화

- **1.1 멀티코어 병렬성**
- **1.2 코드를 메서드로 전달**
- **1.3 스트림 API**
- **1.4 디폴트 메서드**
- **1.5 함수형 프로그래밍**

---  


## ✅ 함수형 인터페이스

```java
public interface Predicate<T> {
    boolean test(T t);
}

```
어떤 조건을 검사하여 true or false 로 반환함  


  
### 함수형 인터페이스란?

- ⭐ 단 하나의 추상 메서드만 가지는 인터페이스
- 람다식으로 해당 메서드를 구현 가능
- `@FunctionalInterface` 어노테이션을 붙이면 컴파일러가 규칙을 체크해줌

| 인터페이스      | 설명                                 |
|-----------------|--------------------------------------|
| `Predicate<T>`  | T를 받아 `boolean` 반환 (`test`)     |
| `Function<T, R>`| T를 받아 R 반환 (`apply`)            |
| `Consumer<T>`   | T를 받아 소비하고 반환 없음 (`accept`)|
| `Supplier<T>`   | 아무것도 받지 않고 T 반환 (`get`)     |
| `Runnable`      | 매개변수와 반환값 없이 실행 (`run`)  |


  
## ✅ 컬렉션API VS 스트림API


| 항목             | 컬렉션 API                  | 스트림 API                     |
|------------------|-----------------------------|-------------------------------|
| 데이터 저장 방식 | 메모리에 모든 데이터를 저장 | 요청 시 계산 (게으른 계산)     |
| 반복 방식        | 외부 반복 (for-each 등)     | 내부 반복 (자동 처리)          |
| 데이터 처리 목적 | 데이터 저장 및 관리         | 데이터 계산 및 변환            |
| 병렬 처리        | 직접 구현 필요              | 병렬 처리 지원 (parallelStream) |
| 재사용성         | 여러 번 탐색 가능           | 한 번만 탐색 가능              |
| 연산 방식        | 명령형 프로그래밍           | 선언형 프로그래밍              |


  
컬렉션 방식
```java
List<String> names = new ArrayList<>();
for (Dish dish : menu) {
    if (dish.getCalories() > 300) {
        names.add(dish.getName());
    }
}
```  
스트림 방식
```java
List<String> names = menu.stream()
    .filter(dish -> dish.getCalories() > 300)
    .map(Dish::getName)
    .collect(Collectors.toList());

```

💎 다이아몬드 상속 구조란?


```java
   A
  / \
 B   C
  \ /
   D
```

클래스 B와 C는 A를 상속받고

클래스 D는 B와 C를 상속받는 구조

이렇게 되면 D는 A를 두 번 상속받게 되는 셈이라, A의 멤버에 접근할 때 모호성이 발생  
예를 들어 D에서 A의 age라는 멤버를 사용하려고 하면, B를 통해 상속받은 age인지, C를 통해 상속받은 age인지 알 수 없게 된다 

⚠️ 발생하는 문제들
중복 멤버: A의 멤버가 D에 두 번 존재 

Java 클래스의 다중 상속은 금지하지만, 인터페이스의 다중 구현은 허용 

```java
interface InterfaceA {
    default void greet() {
        System.out.println("Hello from A");
    }
}

interface InterfaceB {
    default void greet() {
        System.out.println("Hello from B");
    }
}

class MyClass implements InterfaceA, InterfaceB {
    @Override
    public void greet() {
        // 충돌 해결: 명시적으로 호출
        InterfaceA.super.greet();
        InterfaceB.super.greet();
    }
}
```


