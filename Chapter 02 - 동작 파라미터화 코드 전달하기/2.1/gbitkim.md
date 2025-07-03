
# 동작파라미터화 코드 전달하기

## 2.1 변화하는 요구사항에 대응하기

## 2.2 동작 파라미터화

> 전략 디자인 패턴은 캡슐화하는 알고리즘 패밀리를 정의해둔 다음에 런타임에 알고리즘을 선택하는 기법이다.
>> 해당단어가 생소하였어서 간단하게 내용을 정리해보고자한다.

✅ 전략 디자인 패턴이란?

| 구성 요소                | 역할 설명                                                               |
| -------------------- | ------------------------------------------------------------------- |
| **Context**          | 전략을 사용하는 쪽. 클라이언트 입장에서 실제 동작을 수행하지만 구체적인 전략은 모른 채 **추상화된 인터페이스**에만 의존함. |
| **Strategy 인터페이스**   | 다양한 전략들이 구현해야 하는 공통 인터페이스.                                          |
| **ConcreteStrategy** | 실제 알고리즘을 구현한 클래스들. 다양한 전략이 이 역할을 담당함.                               |

💡 왜 "알고리즘 패밀리를 정의하고 런타임에 선택"이라 했을까?<br>
런타임에 필요에 따라 전략 객체를 바꾸면 다른 동작이 수행되기때문이다.

## 2.3 복잡한 과정 간소화

> new 연산자, 익명클래스는 Heap메모리에 올라갈텐데 그렇다면 람다식도 Heap메모리에 올라가고 GC가 new연산자와 비슷한 시기에 지정될까?

```java
public class PredicateTest {
    public static void main(String[] args) {
        Predicate<String> p1 = new MyPredicate();
        Predicate<String> p2 = new Predicate<String>() {
            @Override
            public boolean test(String s) {
                return s.length() > 3;
            }
        };
        Predicate<String> p3 = s -> s.length() > 3;

        System.out.println(p1.test("hi"));
        System.out.println(p2.test("hello"));
        System.out.println(p3.test("hey"));

        // 이후 아래와 같이 모두 참조를 끊으면 GC 대상이 될 수 있음
        p1 = null;
        p2 = null;
        p3 = null;

        // 명시적으로 GC 요청 (실제로 바로 일어나지는 않음)
        System.gc(); // JVM이 즉시 GC할지는 보장되지 않음
    }
}
```

| 방식               | 객체 생성 방식            | Heap에 올라가는가? | 비고                        |
| ---------------- | ------------------- | ------------ | ------------------------- |
| `new` 연산자        | 명시적 클래스 인스턴스 생성     | ✅ 올라감        | 일반 객체와 동일                 |
| 익명 클래스           | 내부적으로 클래스 + 인스턴스 생성 | ✅ 올라감        | `PredicateTest$1`처럼 컴파일됨  |
| 람다식 (`s -> ...`) | 함수형 인터페이스 구현 객체 생성  | ✅ 올라감 (보통)   | 단, **JVM 최적화**로 캐싱 가능성 존재 |

<br><br>

### 그렇다면 람다는 어떻게 내부적으로 Heap영역에 올라갈까?

```java
Predicate<String> p = s -> s.length() > 3;
```
→ 실제 실행 시엔 JVM이 아래처럼 익명 구현 객체를 동적으로 생성한 것과 같은 효과가 발생한다.

```java
Predicate<String> p = new Predicate<String>() {
    @Override
    public boolean test(String s) {
        return s.length() > 3;
    }
};
```

하지만 이건 **익명 클래스가 아니라 런타임에 생성되는 <b>람다 객체</b>이다.

<br><br>

### new 연산자 없이도 람다가 힙에 객체가 올라가는 이유?

new는 명시적인 객체 생성일 뿐이고,<br>
힙 메모리에 올라가느냐는 <b>"객체가 생성되느냐"</b>의 문제이며<br>
람다는 JVM이 자동으로 객체를 만들어주는 방식이기 때문에 힙에 올라간다.

<br><br>

| 항목      | new 사용 여부 | 객체 생성 여부       | 힙 메모리 할당 | 비고                        |
| ------- | --------- | -------------- | -------- | ------------------------- |
| 명시적 클래스 | ✅ yes     | ✅ yes          | ✅ yes    | 일반적인 방식                   |
| 익명 클래스  | ✅ yes     | ✅ yes          | ✅ yes    | 컴파일 시 내부 클래스 생성           |
| 람다식     | ❌ no      | ✅ yes (JVM 생성) | ✅ yes    | JVM이 `invokedynamic`으로 생성 |

<br><br>

## 2.4 실전예제

> Thread 관련 개념이 나와서 CompletableFuture 나오게 된 흐름 및 관련 주요 클래스 및 인터페이스를 오랜만에 보고싶어졌다...

<br>


| Java 버전 | 명칭                                                             | 도입 이유                         | 장점                          | 단점                          |
| ------- | -------------------------------------------------------------- | ----------------------------- | --------------------------- | --------------------------- |
| Java1.0 | `Thread`, `Runnable`                                           | 기본적인 쓰레드 작업 수행을 위해 도입         | 간단한 구현, 직접 쓰레드 제어 가능        | 자원 관리 어려움, 반환값 없음, 예외 처리 미흡 |
| Java5   | `Callable<V>`                                                  | `Runnable`의 반환값과 예외 처리 한계 극복  | 반환값 처리 가능, 예외 전파 가능         | `Executor`와 함께 사용해야 함       |
| Java5   | `Future<V>`                                                    | `Callable` 결과를 비동기적으로 제어하기 위해 | 결과 조회, 취소, 완료 여부 확인 가능      | `get()` 시 블로킹, 콜백 기능 없음     |
| Java5   | `Executor`, `ExecutorService`                                  | 직접 쓰레드 생성의 비효율성 해결            | 쓰레드 풀 관리, 작업 큐 지원, 확장성 우수   | 설정 및 관리 어려움, 동작 방식 이해 필요    |
| Java5   | `Executors`                                                    | 다양한 유형의 Executor 인스턴스 생성      | 간단한 사용, 다양한 Executor 제공     | 자원 누수 위험, 설정 세부 제어 어려움      |
| Java5   | `ThreadPoolExecutor`                                           | 쓰레드 풀의 상세한 동작 제어 목적           | 코어/최대 쓰레드 수 조절, 큐 제어 가능     | 복잡한 설정, 오용 시 성능 저하 가능       |
| Java5   | `ScheduledExecutorService`                                     | 주기적/지연 실행 지원                  | 정밀한 스케줄링 가능, `Timer` 대체     | 초기 설정 복잡할 수 있음              |
| Java8   | `CompletableFuture`                                            | 비동기 처리 간소화, 콜백 지옥 해소          | 논블로킹, 체이닝, 조합 및 예외 처리 우수    | 디버깅 어려움, 예외 누락 가능성          |
| Java9   | `Flow`, `Publisher`, `Subscriber`, `Subscription`, `Processor` | Reactive Streams 표준 도입        | 비동기 스트림 처리, Backpressure 지원 | 러닝 커브 있음, 구현 복잡성 있음         |


> https://www.notion.so/slrslrr1/CompletableFuture-83d482300b0844d3ac2c9fd86637a787

> 당시 작성할때 외부에서 쓰레드를 상세하게 컨트롤하던 무언가가 존재하였는데.. 하여 스프링컨테이너에 올려서 사용하던것이 존재하였는데.. 기억이 나지않는다..


