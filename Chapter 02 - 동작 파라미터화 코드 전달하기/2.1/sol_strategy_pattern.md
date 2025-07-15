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



