# chaper2. 동작 파라미터화 코드 전달하기

## 2.1 변화하는 요구사항에 대응
- 매번 변화하는 요구사항에 따라 유연하게 변경 가능한 코드를 만들어야 한다.
  - 엔지닝어링 비용을 최소화하면서 유지보수가 쉬운 코드 작성하기
- 동작 파라미터화, 익명 클래스, 람다 표현식을 사용하지 않고 파라미터로 모든 걸 해결하려 하면, 가독성이 떨어지는 나쁜 코드가 나온다.
```java
public static List<Apple> filterApples(List<Apple> inventory, Color color,
                                       int weight, boolean flag) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if ((flag && apple.getColor().equals(color)) ||
            (!flag && apple.getWeight() > weight)) {
            result.add(apple);
        }
    }
    return result;
}
```

## 2.2 동작 파라미터화
> *동작 파라미터화(Behavior Parameterization)*
>> 메서드가 실행할 구체적인 동작을 아직 정하지 않은 코드 블록을 파라미터로 전달받아, 다양한 요구사항에 유연하게 대응할 수 있도록 하는 프로그래밍 기법

<img width="514" alt="image" src="https://github.com/user-attachments/assets/e071d1e3-e7f7-45db-b9f2-523ee145498d" /><br>

- *프레디케이트와 전략 디자인 패턴*을 함께 사용  
  - 컬렉션을 반복하는 로직과 컬렉션의 각 요소에 적용할 동작을 분리할 수있다는 점에서 소프트웨어 엔지니어링적으로 큰 이득
  - 프레디케이트: 참 또는 거짓을 반환하는 함수
  - 전략 디자인 패턴: 각 알고리즘(전략)을 캡슐화하는 알고리즘 패밀리를 정의해둔 다음에 런타임에 알고리즘을 선택하는 기법

<details>
<summary> 코드 보기</summary>

```java
// 선택 조건 결정 인터페이스 (전략 캡슐화)
public interface ApplePredicate {
  boolean test (Apple apple);
}

public class AppleHeavyWeightPredicate implements ApplePredicate {
  public boolean test(Apple apple) {
    return apple.getWeight() > 150;
  }
}

public class AppleGreenColorPredicate implements ApplePredicate {
  public boolean test(Apple apple) {
    return GREEN.equals(apple.getColor());
  }
}

// 반복하는 부분과 중요한 동작 요소를 분리 -> 유지보수에 유리
public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
  List<Apple> result = new ArrayList<>();
  for(Apple apple : inventory) {
    // 프레디케이트 객체로 사과 검사 조건을 캡슐화
    if(p.test(apple)) {
      result.add(apple);
    } 
  }
}

// 나중에 추가 요구사항 시, 확장 가능
public class AppleRedAndHeavyPredicate implements ApplePredicate {
  public boolean test(Apple apple) {
    return RED.equals(apple.getColor()) && apple.getWeight() > 150;
  }
}

List<Apple> redAndHeavyApples = filterApples(inventory, new AppleRedAndHavyPredicate());
```
</details>

## 2.3 복잡한 과정 간소화
<img width="465" alt="image" src="https://github.com/user-attachments/assets/9a8f3e6c-5eb1-49f1-b525-7cad078a5f05" /><br>

> 🤔 간소화 하는 게 정말 좋을까?
>> 구현이 간단하면 람다를 사용하는 게 가독성은 좋았으나,<br>로직이 복잡한 결제 방법별 서비스 구현 또는 통합주문/신리딩트리/구리딩트리별 동일한 기능의 특화된 서비스 구현은<br>클래스 + 전략 디자인 패턴을 사용했을 때 깔끔했다.
- [ ] cims에서 사용했던 로직 정리해두기

<details>
  <summary>람다(Lambda) 대신 전통적인 전략 패턴(Strategy Pattern)을 사용하는 것이 더 나은 대표적인 예시</summary>

- **여러 메서드가 필요한 복잡한 전략 구현**
  - 람다는 함수형 인터페이스(추상 메서드 1개)만 구현 가능
  - 여러 메서드가 필요하거나 상태(state)가 있는 전략은 람다로 표현 불가
    - 예: 결제 방식 전략(PaymentStrategy)에서 결제, 환불, 포인트 적립 등 여러 메서드 필요
    - 예: 복잡한 초기화나 내부 상태가 필요한 암호화 전략

- **전략 객체가 상태(state)를 가져야 하는 경우**
  - 람다는 상태 없는(Stateless) 함수 객체에 적합
  - 전략 객체가 내부 값을 저장하거나 상태 변화가 필요하면 클래스 구현 권장
    - 예: 캐시를 유지하는 압축 전략
    - 예: 내부 연결 정보를 유지하는 네트워크 전략

- **전략 구현이 매우 복잡하거나 재사용·확장이 필요한 경우**
  - 람다는 간단한 동작에 적합하지만 복잡한 로직이나 재사용 전략은 별도 클래스 관리가 유지보수에 유리
    - 예: 다양한 정렬 알고리즘 전략
    - 예: 여러 단계의 데이터 변환 전략

- **함수형 인터페이스가 아닌 타입을 구현해야 할 때**
  - 람다는 함수형 인터페이스만 구현 가능
  - 추상 메서드가 2개 이상이거나 추상 클래스를 구현해야 하면 전략 패턴 필요

- **자기 자신(this)에 대한 참조가 필요한 경우**
  - 람다에서 `this`는 바깥 객체를 가리킴
  - 익명 클래스나 전략 클래스에서는 자기 자신 참조 가능

---

**요약 표**

| 상황/조건                     | 람다 사용 | 전략 패턴(클래스) 권장 |
|------------------------------|:---------:|:---------------------:|
| 메서드 1개, 간단한 로직       |     O     |           -           |
| 여러 메서드 필요, 상태 보유    |     -     |           O           |
| 복잡한 구현, 재사용/확장 필요 |     △     |           O           |
| 함수형 인터페이스 아님        |     -     |           O           |
| this 참조 필요(자기 자신)     |     -     |           O           |

---

**정리**  
람다는 간결함과 생산성에 강점이 있지만,  
복잡한 전략, 상태 보유, 여러 메서드, this 참조, 함수형 인터페이스가 아닌 경우에는  
전통적인 전략 패턴을 사용하는 것이 더 안전하고 명확함.

</details>



## 2.4 실전 예제: Comparator, Runnabla, GUI
