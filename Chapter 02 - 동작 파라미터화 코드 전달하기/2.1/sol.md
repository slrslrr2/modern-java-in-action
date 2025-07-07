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
> 동작 파라미터화 패턴
>> 동작을 (한 조각의 코드로) 캡슐화한 다음에 메서드로 전달해서 메서드의 동작을 파라미터화한다 (예를 들면 사과의 다양한 프레디케이트)

<br>🤔 현실적으로는 함수형 인터페이스보다는 아닌 경우가 더 많을 것 같아서, 람다의 장점을 살릴 수 있는 예시가 궁금했는데<br>저 3개의 예시가 람다를 사용했을 때의 장점을 잘 말해주는 것 같아 좋았다.<br><br>

<details>
  <summary> 1. '동작 파라미터화 패턴'의 가장 큰 장점은 *지연실행*이 가능하다는 점 같다. </summary>
  
- **불필요한 연산/자원 사용 최소화**  
  필요할 때까지 계산이나 객체 생성을 미루기 때문에, 실제로 필요하지 않은 작업은 아예 수행하지 않음 
  예를 들어, 자바 Stream의 지연 연산은 최종 연산(collect, forEach 등)이 호출될 때까지 중간 연산(map, filter 등)을 실행하지 않음.
  → 이로 인해 불필요한 데이터 처리와 메모리 사용이 줄어듦.

- **메모리 및 CPU 사용의 효율화**  
  큰 데이터 세트나 무거운 객체를 한꺼번에 메모리에 올리지 않고, 실제로 접근할 때만 생성하거나 처리함. 
  예를 들어, 제너레이터/이터레이터 패턴이나 Stream의 lazy evaluation은 한 번에 모든 데이터를 처리하지 않고 필요한 만큼만 메모리에 올려 처리함.  
  결과적으로 메모리 사용량과 CPU 부하가 크게 줄어듦.

- **성능 최적화 및 빠른 응답**  
  연산을 미루는 덕분에, 실제로 필요한 데이터만 빠르게 처리할 수 있음.
  Stream 파이프라인에서 필터링, 매핑 등 여러 연산이 결합되어 있을 때 최종적으로 필요한 데이터만 추려내므로, 불필요한 연산이 줄고 전체 처리 속도가 빨라짐.

- **초기화 순환성 문제 해결**  
  객체나 데이터의 초기화 시점을 늦춤으로써, 복잡한 의존성이나 초기화 순환 문제를 피할 수 있음.
  예를 들어, 어떤 객체가 실제로 사용되기 전까지 초기화하지 않으면 의존성 문제나 불필요한 비용을 방지할 수 있음.

- **유연한 구조와 관심사의 분리**  
  코드의 실행 시점과 정의 시점을 분리할 수 있어 유연한 구조와 관심사의 분리(Separation of Concerns)가 용이합니다.  
  예를 들어, 콜백, 이벤트 핸들러, 스레드 작업 등에서 "나중에 실행할 코드"를 등록해두고 필요할 때 실행하는 구조가 가능합니다.

---

**정리**  
지연 실행: 실제로 필요할 때까지 계산이나 객체 생성을 미루는 기법
이로 인해 메모리와 CPU 등 리소스 사용이 최적화되고, 불필요한 연산을 줄여 성능이 향상되며, 초기화 순환성 문제도 예방할 수 있음
특히 대용량 데이터 처리, 복잡한 의존성 관리, 이벤트/비동기/스레드 작업 등에서 효율적이고 유연한 시스템 설계가 가능함
  
</details>


<details>
  <summary> 2. 람다를 활용했을 때, 지연실행의 효과는 더 크다. 그럼, 람다는 언제 사용하면 좋을까? </summary><br>

**비교 표**

| 방식        | 특징 및 장점                                                                                           | 리소스적 효과/단점                           |
|-------------|--------------------------------------------------------------------------------------------------------|----------------------------------------------|
| 클래스 이용 | - 전략(알고리즘)을 별도의 클래스로 구현<br>- 명확한 타입, 복잡한 상태·여러 메서드 필요시 유리           | - 클래스/객체 수 증가, 메모리 사용 증가      |
| 람다 이용   | - 코드 블록(동작)을 간결하게 전달<br>- 불필요한 클래스 없이 동작만 파라미터로 넘김<br>- 코드가 짧고 명확<br>- 익명 클래스보다 객체 생성량 적음, GC 부담 감소 | - 복잡한 상태·여러 메서드 필요시 부적합      |

---

**구체적 설명**

- 람다 표현식은 코드 블록을 객체처럼 전달할 수 있어, 실제로 필요할 때까지 실행을 미룬다.
  이로 인해 불필요한 객체 생성을 줄이고, 메모리와 GC(가비지 컬렉션) 부담도 줄어든다.
- 람다는 익명 클래스보다 훨씬 적은 코드와 객체로 동작을 표현할 수 있으므로,  
  대량의 동작 파라미터화가 필요한 상황(예: 스트림, 비동기, 이벤트 처리 등)에서 리소스 효율성이 높아진다.
- 클래스 기반 전략 패턴은 명확한 타입, 복잡한 상태, 여러 메서드가 필요한 경우에만 사용하는 것이 좋고,  
  단순히 "나중에 실행할 코드"를 전달하는 목적이라면 람다가 훨씬 간결하고 효율이다.

---

**결론**

- 지연 실행의 장점을 최대한 활용하려면, 동작 파라미터화에서 람다를 사용하는 것이 클래스보다 리소스적으로도 더 효과적이다.
- 람다는 코드의 간결성, 메모리 효율, GC 부담 감소, 실행 시점의 유연성 등 다양한 측면에서 이점을 제공한다.
- 단, 복잡한 상태 관리나 여러 메서드가 필요한 경우에는 전통적인 클래스 기반 전략 패턴이 더 적합할 수 있다.
- **즉, 단순 동작 파라미터화와 지연 실행이 목적이라면 람다 사용이 더 나은 선택이다.**
  
</details>

## 2.5 마치며

| 키워드                         | 설명                                                                 |
|-----------------------------|----------------------------------------------------------------------|
| 1. 동작 파라미터화, 익명 클래스, 람다 표현식 | 무한 조건문 생성 방지 → 엔지니어링 비용 최소화 & 유지보수 쉬운 코드 |
| 2. 프레디케이트 & 전략 디자인 패턴        | 주요 로직 캡슐화                                                     |
| 3. 지연 실행                     | - 연산/자원 사용 최소화  <br> - 메모리 및 CPU 사용 효율화 <br> - 성능 최적화 및 빠른 응답 <br> - 초기화 순환성 문제 해결 <br> - 유연한 구조와 관심사의 분리 |

## 번외. ThreadPoolTaskExecutor
![image](https://github.com/user-attachments/assets/51fdc30a-0c75-4fca-a336-a57b84b45883)
🤔 금빛님의 물음표에서 시작된 ThreadPoolTaskExecutor 정리

> ThreadPoolTaskExecutor
>> ThreadPoolTaskExecutor는 Spring에서 많이 사용하는 스레드 풀 관리 도구로, 세밀한 설정이 가능하고,<br>@Async, TaskScheduler, CompletableFuture, WebClient 등과 함께 사용하기 위해 Bean으로 등록해두고 활용함<br>
>> 개발자 → ThreadPoolTaskExecutor ← 내부적으로 ThreadPoolExecutor 사용 (JDK)<br>

### 1) 다른 대안들과의 비교
| 이름                       | 설명                            | 언제 사용?                         |
|----------------------------|---------------------------------|------------------------------------|
| ThreadPoolExecutor         | Java 기본 제공. 가장 세밀한 설정 가능. | 스프링 안 쓰거나, 아주 커스터마이징할 때 |
| Executors.newFixedThreadPool() 등 | 간편하게 쓸 수 있는 팩토리 메서드       | 테스트용, 간단한 병렬 처리                |
| ForkJoinPool               | 병렬 작업을 자동으로 분할해서 실행       | CPU 중심의 계산작업 (예: 그래픽 처리)     |
| ScheduledThreadPoolExecutor| 예약된 작업 (스케줄링)                 | 일정 시간마다 작업 실행할 때              |

### 2) 장점
| 기능       | 설명                                              |
| -------- | ----------------------------------------------- |
| 성능 제어    | 동시에 실행할 수 있는 작업 수를 설정하여 서버 과부하 방지               |
| 예외 처리    | CompletableFuture로 try-catch 없이 체계적인 예외 처리 가능 |
| 응답 속도 개선 | 병렬 처리를 통해 여러 요청을 동시에 빠르게 처리 가능                  |
| 재사용성     | 스레드 풀을 Bean으로 등록해 여러 서비스에서 재사용 가능               |

### 3) 간단한 코드 예시
<details>
  <summary> ThreadPoolTaskExecutor의 간단한 코드 </summary><br>
  
  ```java
  // 1. ThreadPoolTaskExecutor 설정 (스프링 빈 등록)
  // AsyncConfig.java
  @Configuration
  public class AsyncConfig {
  
      @Bean(name = "customExecutor")
      public ThreadPoolTaskExecutor threadPoolTaskExecutor() {
          ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
          executor.setCorePoolSize(4);         // 최소 스레드 수
          executor.setMaxPoolSize(8);          // 최대 스레드 수
          executor.setQueueCapacity(50);       // 큐에 대기 가능한 작업 수
          executor.setThreadNamePrefix("CustomExecutor-");
          executor.initialize();               // 초기화 필수
          return executor;
      }
  }

// 2. 서비스 로직에서 CompletableFuture와 함께 사용
// MyAsyncService.java
@Service
public class MyAsyncService {

    private final ThreadPoolTaskExecutor executor;

    public MyAsyncService(@Qualifier("customExecutor") ThreadPoolTaskExecutor executor) {
        this.executor = executor;
    }

    public CompletableFuture<String> asyncTask(String input) {
        return CompletableFuture.supplyAsync(() -> {
            log.info("비동기 작업 시작: {}", input);
            try {
                Thread.sleep(1000); // 실제 작업 시뮬레이션
            } catch (InterruptedException e) {
                throw new IllegalStateException(e);
            }
            return "처리 완료: " + input;
        }, executor);
    }
}

// 3. 컨트롤러에서 호출
// MyController.java
@RestController
@RequestMapping("/api")
public class MyController {

    private final MyAsyncService asyncService;

    public MyController(MyAsyncService asyncService) {
        this.asyncService = asyncService;
    }

    @GetMapping("/process")
    public CompletableFuture<ResponseEntity<String>> process(@RequestParam String input) {
        return asyncService.asyncTask(input)
                .thenApply(ResponseEntity::ok)
                .exceptionally(ex -> ResponseEntity.status(500).body("에러 발생: " + ex.getMessage()));
    }
}

  
  ```

</details>

DOTO 졸려서 내일 이어서 정리해보겠음
### 4) 왜 @Async보다 더 유연하고 제어 가능할까?
### 5) 왜 실무에서 API 호출, 대용량 데이터 처리, 외부 연동 등에 자주 쓰일까?


