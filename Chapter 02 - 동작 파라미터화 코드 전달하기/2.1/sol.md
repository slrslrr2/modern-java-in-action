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
## 2.3 익명 클래스
## 2.4 람다 표현식 미리보기
## 2.5 실전 예제: Comparator, Runnabla, GUI
