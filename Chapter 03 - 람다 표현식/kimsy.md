# Chanpter 03 - 람다 표현식 


###  람다 표현식이란?
- 이름이 없는 함수 (익명 함수)
- 메서드를 인수로 전달 가능
- 익명 클래스와 유사하지만 더 간결함
- 자바 8부터 도입

---

###  문법 구성
```java
(파라미터 리스트) -> { 람다 바디 }
```

- `return`은 **암묵적으로 포함**되어 있어 생략 가능
- 여러 문장을 포함할 수 있음

---

### 람다 표현식 구성 요소
- **파라미터 리스트**  
- **람다 바디**  
- **변환 타입 (타입 추론 가능)**  
- **예외 리스트 (필요 시 명시 가능)**

---

### 함수형 인터페이스
람다 표현식을 사용하려면 반드시 함수형 인터페이스 필요:

| 인터페이스      | 설명                                |
|----------------|-------------------------------------|
| `Predicate<T>` | 조건 판별 (`true` / `false`)         |
| `Function<T,R>`| 입력 `T` → 결과 `R` 변환              |
| `Consumer<T>`  | 입력 `T` 소비 (반환 없음)             |
| `Supplier<T>`  | 공급자, 값을 생성                    |
| `Comparator<T>`| 두 객체 비교                         |
| `Runnable`     | 매개변수 없이 실행만 하는 인터페이스 |

---

### 사용 팁
- 한 개의 `void` 메서드 호출일 경우 **중괄호 생략** 가능
- **컬렉션 + 스트림 API**에서 자주 활용됨
- 메서드 참조(`::`)를 쓰면 더 간결한 코드 가능

예시:
```java
names.stream()
     .filter(name -> name.startsWith("A"))
     .forEach(System.out::println);
```



# 람다, 메서드 참조 활용하기

---

## 코드 전달 방식

✅ 3.7.1 코드 전달: 익명 클래스 → 람다 → 메서드 참조
```java
public class AppleComparator implements Comparator<Apple> {
    public int compare(Apple a1, Apple a2) {
        return a1.getWeight().compareTo(a2.getWeight());
    }
}
inventory.sort(new AppleComparator());

```
-명시적으로 클래스를 생성해서 코드를 전달   
-가장 전통적인 방식  


✅ 3.7.2 익명 클래스 사용

```java
inventory.sort(new Comparator<Apple>() {
    public int compare(Apple a1, Apple a2) {
        return a1.getWeight().compareTo(a2.getWeight());
    }
});
```

- 클래스를 따로 선언하지 않고 익명 클래스로 구현  
- 구현은 조금 더 간결하지만 여전히 반복되는 코드 존재  



✅ 3.7.3 람다 표현식 사용 

```java
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
```

- 훨씬 간결하고 가독성 높은 코드  
- 함수형 프로그래밍 스타일을 적극 활용  


✅ 3.7.4 메서드 참조 활용

```java
import static java.util.Comparator.comparing;

inventory.sort(comparing(Apple::getWeight));
```

- 코드 간결성 극대화  
- 재사용성과 가독성이 뛰어남  


