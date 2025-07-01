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
<img width="514" alt="image" src="https://github.com/user-attachments/assets/e071d1e3-e7f7-45db-b9f2-523ee145498d" />

- *동작 파라미터화(Behavior Parameterization)*: 메서드가 실행할 구체적인 동작을 아직 정하지 않은 코드 블록을 파라미터로 전달받아, 다양한 요구사항에 유연하게 대응할 수 있도록 하는 프로그래밍 기법
- *프레디케이트와 전략 디자인 패턴*을 함께 사용  
  - 컬렉션을 반복하는 로직과 컬렉션의 각 요소에 적용할 동작을 분리할 수있다는 점에서 소프트웨어 엔지니어링적으로 큰 이득
  - 프레디케이트: 참 또는 거짓을 반환하는 함수
  - 전략 디자인 패턴: 각 알고리즘(전략)을 캡슐화하는 알고리즘 패밀리를 정의해둔 다음에 런타임에 알고리즘을 선택하는 기법

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

- 구현이 간단하면 구현부를 줄이고 람다를 사용해도 되지만,<br>로직이 복잡한 결제 방법별 서비스 구현 또는 통합주문/신리딩트리/구리딩트리별 동일한 기능의 특화된 서비스 구현은<br>위의 예처럼 전략 디자인 패턴을 사용했을 때 깔끔했다.
- [ ] cims에서 사용했던 로직 정리해두기

## 2.3 익명 클래스
## 2.4 람다 표현식 미리보기
## 2.5 실전 예제: Comparator, Runnabla, GUI
