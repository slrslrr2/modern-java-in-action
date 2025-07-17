## 전략 패턴과 팩토리 메서드를 사용한 결제 취소 관련 로직의 예를 들고와서 분석해보았다.

## 1. Strategy Pattern

결제 방식에 따른 서로 다른 구현을 각각의 service 클래스에서 처리  
결제 방식에 따라 행동이 달라지는 부분을 서비스 클래스로 캡슐화함  
결제 방식별 전략을 런타임에 주입 받음

### 장점

- OCP(개방/폐쇄 원칙): 새로운 결제 수단이 추가돼도 기존 코드를 수정하지 않고, 새 클래스만 추가하면 됨  
- SRP(단일 책임 원칙): 각 결제 방식에 따른 승인/취소/검증 로직이 한 클래스에 모여 있어 유지보수가 쉬움  
- DI 컨테이너의 힘: 전략 객체를 Spring이 자동 주입 → 관리 편리 & 테스트 용이  
- 런타임 전략 교체 가능: 코드가 아니라 설정/입력 값에 따라 전략 선택 가능 (`PaymentCancelCode`)

### 실제 효과

- `KakaoPayCancelService`, `NaverPayCancelService` 각각의 내부 로직은 독립적으로 테스트, 수정, 교체 가능  
  예: PG사 정책 변경 시, 그 서비스만 바꾸면 됨 (다른 전략엔 영향 없음)

---

## 2. Factory Method

객체 생성을 서브클래스에 위임하는 패턴  
여기선 `PaymentCancelCode.of(String)` + `Map<PaymentCancelCode, PaymentCancelInPort>`가 팩토리 역할 수행  
(enum과 구현클래스(service)를 통해 전략 객체 선택)  
즉, 문자열 입력만으로 적절한 전략 객체를 자동 선택

### 장점

- 복잡한 분기 제거: `if-else`, `switch`문 없이 enum으로 전략 선택 가능  
- 확장에 유리: 새로운 코드가 추가되면 enum과 서비스만 확장하면 됨  
- 실행 흐름이 명확: 어떤 코드에 어떤 객체가 매핑되는지 Map을 보면 한눈에 파악 가능  
- 타입 안정성: enum 기반이므로 컴파일 시점에 타입 보장됨 (문자열 비교 실수 방지)

### 실제 효과

- "결제방식 코드 → 전략 객체" 매핑을 명확하고 안정적으로 처리  
- 복잡한 조건문 없이도 유지보수 쉬운 확장 구조 확보

---

## 3. Interface Segregation Principle

인터페이스 분리 원칙은 큰 인터페이스를 목적별로 나누고,  
클라이언트는 필요한 것만 의존하게 하자는 SOLID 원칙 중 하나임  

`PaymentCancelInPort`는 모든 결제 수단이 반드시 구현해야 할 공통 기능만 정의  
예: `cancelPayment()`, `cancelPaymentLinkages()` 등

### 장점

- 명세 기반 개발: 전략 클래스는 인터페이스를 구현함으로써 일관된 계약(Contract)을 따름  
- 유지보수 용이: 공통 로직 변경 시 한 곳만 수정하면 됨  
- 테스트 편의성: 인터페이스로 mocking → 단위 테스트 유리  
- 낮은 결합도: Controller나 Finder는 인터페이스만 알면 되므로 구체 구현체에 의존하지 않음

### 실제 효과

- 모든 결제 수단은 동일한 인터페이스를 통해 호출되므로, 서비스별 차이는 숨기고 공통 방식으로 조작 가능  
- 예: `cancelPayment()` 호출 시, 결제 방식마다 다르게 동작하더라도 외부에서는 동일하게 취급 가능

---

## 4. 예제 코드 흐름 설명
1. enum이자 팩토리인 PaymentCancelCode을 통해 결제수단에 따른 객체 생성
2. Strategy Selector(혹은 Registry)인 PaymentCancelFinder를 통해 결제 수단 코드에 맞는 구현체를 반환 받음
3. 특정 결제수단의 구현체(service)에서 cancelPayment 실행

<details>
  <summary> Strategy Pattern + Factory Method + Interface Srgregation Principle 예시 코드 </summary><br>

> PaymentController.java 의 일부
```java
PaymentCancelInPort paymentCancelInPort = paymentCancelFinder.findByPaymentCancel(PaymentCancelCode.of(cancelPayment.getStlmMthdCode()));
// 승인취소 처리
ResponseMessage<CancelPaymentResDto> responseMessage = paymentCancelInPort.cancelPayment(cancelPayment);
```
<br>

> PaymentCancelFinder.java
>> 결제 취소 전략 객체를 런타임에 주입받고, 특정 결제 수단 코드에 따라 적절한 구현체를 반환하는 '전형적인 전략 패턴 + DI' 조합 구조 
```java
@Component
public class PaymentCancelFinder {
    private final Map<PaymentCancelCode, PaymentCancelInPort> paymentCancelOutAdapter;

    public PaymentCancelFinder(final List<PaymentCancelInPort> apiOutPorts) {
        this.paymentCancelOutAdapter = apiOutPorts.stream()
                .flatMap(
                        apiOutPort -> apiOutPort.cancelPaymentLinkages()
                                .stream()
                                .map(linkage -> new AbstractMap.SimpleEntry<>(linkage, apiOutPort))
                )
                .collect(
                        Collectors.toMap(
                                Map.Entry::getKey,
                                Map.Entry::getValue,
                                (existing, replacement) -> {
                                    throw new IllegalStateException(String.format("PaymentCancelFinder Duplicate key detected for %s", existing));
                                },
                                () -> new EnumMap<>(PaymentCancelCode.class)
                        )
                );
    }

    public PaymentCancelInPort findByPaymentCancel(final PaymentCancelCode paymentCancelCode) {
        PaymentCancelInPort apiOutPort = paymentCancelOutAdapter.get(paymentCancelCode);
        Objects.requireNonNull(apiOutPort, "PaymentCancelFinder.findByPaymentCancel PaymentCancelOutPort is non null");
        return apiOutPort;
    }
}
```
<br>

<details>
  <summary> PaymentCancelFinder의 상세 동작 설명 </summary>
  - final List<PaymentCancelInPort> apiOutPorts : 모든 전략 구현체를 Spring이 자동 주입<br>
  - flatMap : 각 전략 객체가 지원하는 코드를 펼처서 (code, 전략) 쌍 생성(평탄화 과정)<br>
  - SimpleEntry: Map.Entry 객체로 쌍을 표현하기 위해 사용<br>
  - Collectors.toMap(...): 결과를 맵으로 수집<br>
  - (existing, replacement) -> {throw ...}: 중복 키 방지용 예외 처리(매핑 오류나 중복 시 빠르게 예외로 알려줌(Fail-fast))<br>
  - EnumMap: enum key에 최적화된 Map 구조 사용(성능 최적화)
</details>

<br>

> PaymentCancelInPort.java
>> Interface Segregation Principle
 
```java
public interface PaymentCancelInPort {
    /**
     * @description : 해당 금권에 대한 승인취소 유효성 처리 진행
     */
    void isValidStlmMthdCodeCancelPayment(CancelPayment cancelPayment);

    /**
     * @description : 전체 금권 환불 취소
     */
    ResponseMessage<CancelPaymentResDto> cancelPayment(CancelPayment cancelPayment);

    /**
     * @description : 전체 금권과 서비스 클래스 결제취소 Enum과 매핑
     */
    List<PaymentCancelCode> cancelPaymentLinkages();
}
```

<br>

> 각 결제수단별 service.java
>> 구현 service의 예시(PaymentCancelInPort 구체 구현체)

```java
@Component
public class KakaoPayCancelService implements PaymentCancelInPort {
    @Override
    public void isValidStlmMthdCodeCancelPayment(CancelPayment cancelPayment) {
        ...
    }

    /**
     * @description : 해당 금권에 대한 승인취소를 하는 메소드
     */
    @Override
    public ResponseMessage<CancelPaymentResDto> cancelPayment(CancelPayment cancelPayment) {
        ...
        return ResponseMessage.ok(CancelPaymentResDto.of(successOrdrStlmNum));
    }

    @Override
    public List<PaymentCancelCode> cancelPaymentLinkages() {
        return List.of(PaymentCancelCode.KAKAO_PAY, PaymentCancelCode.KAKAO_PAY_DDCT);
    }    
}

@Component
public class NaverPayCancelService implements PaymentCancelInPort { ... }

@Component
public class BookGiftCertificateCancelService implements PaymentCancelInPort { ... }
```
  
</details>

---
## ✚ 금빛님의 🤔에서 시작된 정리
<img width="918" height="263" alt="image" src="https://github.com/user-attachments/assets/cae5a7a6-989a-4ab0-9016-2027d634dacd" />

### intro. 전통적인 factory 및 factory method pattern의 의미
- 팩토리란: 객체를 생성하는 역할만 전담하는 클래스 또는 함수
- 팩토리 메서드 패턴: (새로운 객체가 추가되어도 공통 코드를 수정하지 않도록) 객체 생성 방법을 서브클래스(자식 클래스)에 위임하는 디자인 패턴
<details>
  <summary> 간단한 예시: 새로운 핸드폰 타입이 생겨도, 새로운 팩토리만 만들면 나머지 코드를 고칠 필요 없이 확장 가능 </summary>

  ```java
  
  // 제품 인터페이스
  public interface Phone {
      void call();
  }
  
  // 제품 구현체
  public class IPhone implements Phone {
      public void call() {
          System.out.println("아이폰으로 호출합니다.");
      }
  }
  
  public class AndroidPhone implements Phone {
      public void call() {
          System.out.println("안드로이드폰으로 호출합니다.");
      }
  }
  
  // 팩토리 인터페이스
  public interface PhoneFactory {
      // 팩토리 메서드
      Phone createPhone();
      // 공통 사전처리, 후처리 메서드도 추가 가능
      default Phone orderPhone() {
          Phone phone = createPhone();
          System.out.println("생산 완료!");
          return phone;
      }
  }
  
  // 각 서브 클래스가 객체 생성 방식 결정
  public class IPhoneFactory implements PhoneFactory {
      public Phone createPhone() {
          return new IPhone();
      }
  }
  
  public class AndroidPhoneFactory implements PhoneFactory {
      public Phone createPhone() {
          return new AndroidPhone();
      }
  }
  
  // 사용 예시
  public class Practice {
      public static void main(String[] args) {
          PhoneFactory factory = new IPhoneFactory();
          Phone phone = factory.orderPhone();
          phone.call(); // 결과: 생산 완료! 아이폰으로 호출합니다.
      }
  }


  ```  

</details>
<br>

### 1. 팩토리 메서드 패턴이라고 생각한 이유
- 팩토리의 **역할**인 '객체 생성' 자체는 Spring DI를 활용합니다.
- 하지만, '객체 생성 로직을 캡슐화하여 결합도를 감소 시키고, 새로운 객체 추가 시 기존 코드 변경을 최소화 시키는 유연성 등' 팩토리 메서드 패턴의 **의도**부분에서 결이 같다고 생각했습니다.
- 그래서 DI를 활용한 '변형된 혹은 발전된 팩토리 메서드 패턴'이라고 생각했습니다.
- 이와 비슷한 예제로 정리해 놓은 url을 🎁 선물로 드립니다! (https://zorba91.tistory.com/306)
<br>

### 2. 기존 팩토리 메서드 패턴(GoF)과의 차이점
| 항목       | 전통적인 Factory Method  | 지금 코드                       |
| -------- | -------------------- | --------------------------- |
| 객체 생성 책임 | Creator 서브클래스에 위임    | Spring이 생성하고, Finder가 선택    |
| 핵심 역할    | 객체를 생성해 반환           | 등록된 전략 객체 중 선택하여 반환         |
| 사용 방식    | `creator.create()`   | `finder.find(code)`         |
| 패턴 구조    | 상속 기반                | 전략 매핑 기반                    |
| 패턴 명칭    | Factory Method (GoF) | 전략 + 매핑 + Simple Factory 성격 |

<br>

### 3. 차이에 따른 코드 예제
<details>
  <summary> GoF의 팩토리 메서드 패턴 </summary>

```java
// Product interface
interface PaymentProcessor {
    void process();
}

// Concrete Products
class KakaoPayProcessor implements PaymentProcessor {
    public void process() {
        System.out.println("KakaoPay 처리");
    }
}
class TossPayProcessor implements PaymentProcessor {
    public void process() {
        System.out.println("TossPay 처리");
    }
}

// Creator abstract class
abstract class PaymentProcessorCreator {
    public abstract PaymentProcessor createProcessor();
}

// Concrete Creator
class KakaoPayProcessorCreator extends PaymentProcessorCreator {
    public PaymentProcessor createProcessor() {
        return new KakaoPayProcessor();
    }
}

class TossPayProcessorCreator extends PaymentProcessorCreator {
    public PaymentProcessor createProcessor() {
        return new TossPayProcessor();
    }
}

```

```java
PaymentProcessorCreator creator = new KakaoPayProcessorCreator();
PaymentProcessor processor = creator.createProcessor();
processor.process();  // KakaoPay 처리
```
  
</details>

<details>
  <summary> DI + 팩토리 메서드 패턴 </summary>

  ```java
public class PaymentCancelFinder {

    private final Map<PaymentCancelCode, PaymentCancelInPort> paymentCancelOutAdapter;

    public PaymentCancelFinder(final List<PaymentCancelInPort> apiOutPorts) {
        this.paymentCancelOutAdapter = apiOutPorts.stream()
                .flatMap(apiOutPort -> apiOutPort.cancelPaymentLinkages().stream()
                        .map(linkage -> new AbstractMap.SimpleEntry<>(linkage, apiOutPort)))
                .collect(Collectors.toMap(
                        Map.Entry::getKey,
                        Map.Entry::getValue,
                        (existing, replacement) -> {
                            throw new IllegalStateException(String.format("중복 키: %s", existing));
                        },
                        () -> new EnumMap<>(PaymentCancelCode.class)
                ));
    }

    public PaymentCancelInPort find(PaymentCancelCode code) {
        return paymentCancelOutAdapter.get(code);
    }
}
  ```

```java
PaymentCancelInPort cancelHandler = paymentCancelFinder.find(PaymentCancelCode.KAKAO);
cancelHandler.cancel();
```

</details>

<br>

### 4. GoF 대비 변형된 해당 패턴(Spring DI 기반 전략 선택기 + Simple Factory 성격)의 장점

- 객체 생성을 개발자가 직접 관리하지 않아도 됨
  - 생성 책임을 추상화 수준이 아닌, 프레임워크 수준에서 해결
  - 전통 패턴에서는 개발자가 직접 생성 로직을 써야하고, 새로운 구현이 생기면 Creator 클래스 추가 필요
- 유연하고 확장에 강한 구조
  - PaymentCancelInPort 구현체만 추가하면 자동으로 매핑
  - PaymentCancelFinder는 변경 없이 그대로 사용 가능(OCP원칙)
  - GoF 패턴은 구현체 추가 시, ConcreteCreator도 만들어야 하고, Factory도 확장 필요
- EnumMap을 활용하여 성능과 가독성 둘 다 향상 가능



