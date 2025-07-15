#Chapter 02 - 동작 파라미터화 코드 전달하기 
--- 

## 전략 디자인 패턴 

### 알고리즘을 캡슐화하고 실행 중에 교체할 수 있도록 해주는 패턴  

동일한 작업을 수행하는 여러 알고리즘(전략)이 있을 때, 이를 각각 클래스로 분리  
실행 중에 원하는 전략을 선택하거나 교체할 수 있도록 설계  
상속 대신 위임(Delegation)을 사용하여 유연성을 확보  

  


## 동작 파라미터화   
### 동작을 파라미터로 전달해 하나의 메서드로 다양한 동작 처리    
함수형 인터페이스 + 람다식으로 표현  


```java
public List<Apple> filterApples(List<Apple> inventory, ApplePredicate predicate) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if (predicate.test(apple)) {
            result.add(apple);
        }
    }
    return result;
}
```

```java
filterApples(inventory, apple -> "green".equals(apple.getColor()));
filterApples(inventory, apple -> apple.getWeight() > 150);
```
 

# Runnable vs Callable



| 항목          | Runnable                                      | Callable                                      |
|---------------|-----------------------------------------------|-----------------------------------------------|
| 도입 시기      | Java 1.0                                      | Java 5                                         |
| 추상 메서드     | `run()`                                       | `call()`                                       |
| 반환값         | 없음 (`void`)                                  | 있음 (`T`)                                     |
| 예외 처리       | `throws` 불가 → 내부에서 try-catch 필요         | `throws Exception` 가능                        |
| 실행 방식       | `Thread` 또는 `ExecutorService.execute()`       | `ExecutorService.submit()` → `Future`로 결과 받음 |




# 병렬 처리 API
자바에는 스레드를 직접 만들지 않아도 병렬 처리를 도와주는 API

API	설명
| API                 | 설명                                      |
|---------------------|-------------------------------------------|
| `parallelStream()`  | 컬렉션을 병렬로 처리                      |
| `CompletableFuture` | 비동기 작업을 체이닝으로 처리             |
| `ForkJoinPool`      | 큰 작업을 나눠서 병렬로 처리              |


```java
public List<Apple> filterApplesParallel(List<Apple> inventory, ApplePredicate predicate) {
    List<Apple> result = new ArrayList<>();
    List<Thread> threads = new ArrayList<>();

    for (Apple apple : inventory) {
        Thread t = new AppleFilterThread(apple, predicate, result);
        threads.add(t);
        t.start();
    }

    for (Thread t : threads) {
        try {
            t.join(); 
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    return result;
}
```
```java
public List<Apple> filterApplesParallel(List<Apple> inventory, ApplePredicate predicate) {
    return inventory.parallelStream()
                    .filter(predicate::test)
                    .collect(Collectors.toList());
}

```
