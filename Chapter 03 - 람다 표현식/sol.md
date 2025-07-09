# chaper3. 람다 표현식

## 3.1 람다란 무엇인가?
- 깔끔한 익명 클래스: 익명 클래스처럼 이름이 없는 함수면서 메서드를 인수로 전달 가능

## 3.2 어디에, 어떻게 람다를 사용할까?
- 함수형 인터페이스

<details>
  <summary> 함수형 인터페이스 + 람다 </summary>

```java
// 함수형 인터페이스는 추상메서드가 1개여서 바로 구현 가능
Runnable r1 = () -> System.out.println("Hello World 1");

Runnable r2 = new Runnable() {
  public void run() {
    System.out.println("Hello World 2");
  }
}

public static void process(Runnable r) {
  r.run();
}

process(r1);
process(r2);
process(() -> System.out.println("Hello World 3"));
```
  
</details>

## 3.3 람다 활용: 실행 어라운드 패턴

### 💡 실행 어라운드 패턴

### 💡 try-with-resources 구문

## 3.4 함수형 인터페이스 사용

### 💡 오토박싱

### 💡 람다의 예외 처리


