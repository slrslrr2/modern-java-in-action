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
- 목적: 변하지 않는 코드(설정/정리)와 "변하는 코드(실제 작업)"를 분리
- 흐름: 자원 열기(설정) -> 실제 작업(핵심 로직) -> 자원 닫기(정리)
- 정리:
  - **실행 어라운드 패턴**은 "공통적인 준비/정리"와 "다양한 작업"을 분리
  - **람다**를 활용하면 "작업"을 파라미터로 전달하여 코드 재사용성과 유연성이 높아짐
  - **함수형 인터페이스**를 통해 람다와 기존 자원 처리 코드를 연결
 
### 💡 '실행 어라운드 패턴 + try-with-resources'를 사용할 때의 장점
- 자동 자원 해제
  - try-with-resources를 사용하면, try 블록이 끝난 뒤에 자원이 자동으로 close되어 자원 누수를 방지할 수 있다.
  - 개발자가 직접 finally 블록에서 자원을 닫고 코드를 작성하지 않아도 되므로, 실수로 자원을 닫지 않는 문제가 사라진다.
- 코드 간결화와 가독성 향상
  - 자원을 열고 닫는 반복적인 코드가 훨씬 짧고 명확해진다.
  - finally 블록 없이 try 구문만으로 자원 관리가 끝나므로, 가독성과 유지보수성이 좋아진다.
- 예외 처리의 단순화
  - 여러 자원을 동시에 사용할 때도 try-with-resources 괄호 안에 나열만 하면 되고, 각각의 자원이 자동으로 닫힌다.
  - 예외가 발생해도 모든 자원이 안전하게 정리되므로, 예외 처리 코드가 단순해진다.
- 실행 어라운드 패턴과의 시너지
  - try-with-resources를 사용하면 정리 부분을 완전히 자동화할 수 있으므로, 개발자는 실제 작업(람다로 전달하는 부분)에만 집중할 수 있다.
  - 즉, 공통 자원 관리 코드와 다앙한 작업 코드를 완전히 분리하면서, 자원 누수 위험도 줄이고, 코드도 더 깔끔해진다.
 
### 🤔 가장 깔끔할 것 같은 '실행 어라운드 패턴 + try-with-resources'를 사용하면 안되는 곳
=> 트랜잭션 관리
- try-with-resources 구문에서는 catch 블록 진입 시점에 이미 자원이 close되어 있으므로,<br>conn.rollback()을 호출하면 예외가 발생할 수 있음
- 트랜잭션의 rollback/commit은 try 블록 내에서 처리해야 하며,<br>catch 블록에서 처리하려면 전통적인 try-catch-finally 패턴을 사용해야 함<br>

**=> 그러나 Spring에서는 서비스 계층에서 @Transactional로 처리하므로, 트랜잭션의 try-with-resources는 신경 쓸 필요가 없긴함ㅎ**

<details>
  <summary> 1. 전통적인 방식 </summary>

```java
Connection conn = null;
PreparedStatement pstmt = null;
try {
    conn = dataSource.getConnection();
    conn.setAutoCommit(false);

    pstmt = conn.prepareStatement("UPDATE users SET email = ? WHERE id = ?");
    pstmt.setString(1, "new_email@example.com");
    pstmt.setLong(2, 123L);
    pstmt.executeUpdate();

    conn.commit();
} catch (SQLException e) {
    if (conn != null) {
        try { conn.rollback(); } catch (SQLException ex) { /* 예외 처리 */ }
    }
} finally {
    if (pstmt != null) {
        try { pstmt.close(); } catch (SQLException ex) { /* 예외 처리 */ }
    }
    if (conn != null) {
        try { conn.close(); } catch (SQLException ex) { /* 예외 처리 */ }
    }
}

```
  
</details>

<details>
  <summary> 2. 실행 어라운드 패턴 (트랜잭션 처리 시, 추천) </summary>

```java
@FunctionalInterface
public interface TransactionCallback<T> {
    T doInTransaction(Connection conn) throws SQLException;
}

public <T> T executeInTransaction(TransactionCallback<T> callback) throws SQLException {
    Connection conn = null;
    try {
        conn = dataSource.getConnection();
        conn.setAutoCommit(false);

        T result = callback.doInTransaction(conn);

        conn.commit();
        return result;
    } catch (SQLException e) {
        if (conn != null) conn.rollback();
        throw e;
    } finally {
        if (conn != null) conn.close();
    }
}

// 사용 예시
executeInTransaction(conn -> {
    try (PreparedStatement pstmt = conn.prepareStatement("UPDATE users SET email = ? WHERE id = ?")) {
        pstmt.setString(1, "new_email@example.com");
        pstmt.setLong(2, 123L);
        pstmt.executeUpdate();
    }
    return null;
});

```
  
</details>

<details>
  <summary> 3. 실행 어라운드 패턴 + try-with-resources </summary>

```java
public <T> T executeInTransaction(TransactionCallback<T> callback) throws SQLException {
    try (Connection conn = dataSource.getConnection()) {
        conn.setAutoCommit(false);

        T result = callback.doInTransaction(conn);

        conn.commit();
        return result;
    } catch (SQLException e) {
        // 필요하다면 롤백 처리 => 커넥션이 끊겨있음
        throw e;
    }
}

// 사용 예시
executeInTransaction(conn -> {
    try (PreparedStatement pstmt = conn.prepareStatement("UPDATE users SET email = ? WHERE id = ?")) {
        pstmt.setString(1, "new_email@example.com");
        pstmt.setLong(2, 123L);
        pstmt.executeUpdate();
    }
    return null;
});

```
  
</details>



### 💡 try-with-resources 구문
- try-with-resources가 자원 반납을 자동으로 처리하는 이유는, 해당 자원을 사용하는 객체가 AutoCloseable 인터페이스를 구현하고 있기 때문이다.
- 이 인터페이스에는 close() 메소드가 정의되어 있으며, try-with-resources 구문을 사용하면 try 블록이 끝날 때(정상 종료든 예외 발생이든 상관없이) JVM이 자동으로 close()를 호출해 자원을 해제한다.
- 그러므로 트랜잭션 사용 시, exception에서의 롤백 처리가 필요할 경우 사용하면 안됨.

## 3.4 함수형 인터페이스 사용

### 💡 오토박싱
- 오토박싱은 편리하지만, 아래와 같은 단점이 있어서 성능 저하의 문제가 생길 수 있음<br>특히 반복문처럼 반복되는 작업에 사용하는 걸 지양해야 함<br>

| 타입              | 메모리 사용 | 접근 속도 | 객체 생성 | 가비지 컬렉션 부담 |
|-------------------|-------------|-----------|-----------|-------------------|
| int (기본형)      | 적음        | 빠름      | 없음      | 없음              |
| Integer (박싱)    | 많음        | 느림      | 있음      | 있음              |

<details>
  <summary> 오토박싱의 단점 </summary>
  
1. **메모리 사용 증가**
   - 기본형(예: int)은 4바이트의 메모리만 사용합니다.
   - 래퍼 객체(예: Integer)는 값뿐 아니라 **객체 정보**를 저장하기 때문에 더 많은 메모리를 차지합니다.
   - 래퍼 객체는 **힙(Heap)** 메모리 영역에 저장되므로, 기본형보다 훨씬 많은 공간을 사용하게 됩니다.

2. **객체 생성 비용**
   - 박싱을 할 때마다 새로운 객체가 생성됩니다(일부 값은 캐싱되지만, 대부분은 새로 만들어짐).
   - 객체를 생성하는 것은 기본형 변수에 값을 할당하는 것보다 훨씬 느립니다.

3. **메모리 탐색(접근) 비용**
   - 기본형은 스택(stack)이나 레지스터에 저장되어 바로 접근할 수 있습니다.
   - 래퍼 객체는 힙(Heap)에 저장되어 있기 때문에, 값을 읽거나 쓸 때 포인터를 따라가서 객체를 찾고, 그 안에서 값을 꺼내야 합니다.
   - 이 과정에서 추가적인 메모리 접근이 필요하므로, 성능이 떨어질 수 있습니다.

4. **가비지 컬렉션 부담**
   - 많은 박싱 객체가 생성되면, 불필요한 객체가 많이 쌓이게 되고, 자바의 가비지 컬렉터가 이 객체들을 정리해야 합니다.
   - 이로 인해 프로그램의 성능이 더 저하될 수 있습니다.
     
</details>

<details>
  <summary> 기본형 vs 래퍼 객체의 저장 방식 차이 </summary>

| 구분                 | 저장 위치        | 데이터 구조             | 수명             | 접근 속도           | 메모리 사용량   | 가비지 컬렉션 |
|----------------------|-----------------|-------------------------|------------------|---------------------|----------------|--------------|
| **기본형(int 등)**     | 스택 (또는 레지스터) | 단순 값                 | 메서드 실행 범위  | 매우 빠름           | 매우 적음      | 해당 없음     |
| **래퍼 객체(Integer 등)** | 힙                | 객체(값 + 부가 정보)      | 참조 남을 때까지  | 느림(힙 탐색 필요)  | 많음           | JVM이 처리    |

=> **스택**: 빠른 접근, 지역성, 짧은 수명, 기본형 저장에 최적화.<br>**힙**: 느린 접근, 공유성, 동적 관리, 객체 및 래퍼 타입 저장.

---

- Java 메모리 구조: 스택, 레지스터, 힙 그리고 기본형과 래퍼 객체의 저장 방식:<br>
자바의 메모리 구조는 프로그램의 성능과 변수의 수명, 접근 속도에 큰 영향을 미칩니다. 특히 **기본형(primitive type)**과 **래퍼 객체(wrapper class)**의 저장 방식과 그 차이를 이해하는 것이 중요합니다.

---

### 1. 자바 메모리 영역 개요

자바 프로그램이 실행될 때 JVM은 메모리를 여러 영역으로 나눕니다.

- **스택(Stack) 메모리**
- **힙(Heap) 메모리**
- **메서드(Method) 영역**
- **PC(Program Counter) 레지스터**  
- **네이티브 메서드 스택** 등

여기서는 스택, 힙, 레지스터(PC 레지스터 포함)에 초점을 맞춰 설명합니다.

---

### 2. 스택(Stack) 메모리

- **정의**: 각 스레드가 독립적으로 가지는 메모리 영역.
- **주요 역할**:  
  - 메서드 호출 시마다 “스택 프레임”이라는 블록이 쌓인다.
  - 메서드 내 **지역 변수**(int, boolean 등 기본형)와 **참조 변수**(힙에 저장된 객체의 주소)가 저장된다.
- **수명**:  
  - 메서드 실행 동안만 존재, 메서드가 끝나면 자동으로 제거(pop).
- **특징**:
  - **LIFO(Last-In-First-Out)**. 마지막에 들어온 데이터가 먼저 나간다.
  - 스레드마다 개별적으로 관리되어 **스레드 안전성**이 높음.
  - 접근 및 할당/해제가 **매우 빠름**.

> 기본형 지역변수인 `int a = 10;`은 스택에 10이라는 값이 바로 저장됨.

---

### 3. 레지스터(PC 레지스터)

- **정의**: JVM 각 스레드가 하나씩 가지는 작은 메모리 공간.  
- **용도**:  
  - CPU에서 현재 실행 중인 바이트코드 명령의 주소(Program Counter, PC)를 보관.
- **특성**:
  - 극히 빠른 접근 속도 보장.
  - 자바에서는 변수 저장보다는 코드 실행 위치 추적 용도로만 사용함.

> 실제로 자바 변수들이 레지스터에 저장되는 것은 아니며, 레지스터에서는 변수 값을 임시로 저장하거나 연산에 사용될 수 있으나, JVM 명세상 “PC 레지스터”는 실행 위치 기록용입니다.

---

## 4. 힙(Heap) 메모리

- **정의**: 프로그램 실행 중 동적으로 할당되는 메모리 공간.
- **주요 역할**:  
  - **new 연산자**로 생성되는 모든 **객체**와 **배열**이 저장됨.
  - 모든 스레드가 **공유**함.
- **수명**:  
  - 객체에 대한 참조(Reference)가 남아 있는 한 존재.
  - 참조가 모두 사라지면 **가비지 컬렉터(GC)**에 의해 자동 해제.
- **특징**:
  - 범용성 높음(어디서든 접근 가능).
  - 동적 할당 및 해제가 필요하므로, **접근 속도가 느림**.
  - 메모리 공간이 유동적, 메모리 누수나 단편화에 취약.

> `Integer a = new Integer(10);`과 같이 래퍼 객체 또는 배열 등은 항상 힙에 저장되고, 스택에는 그 객체의 “참조값(주소)”만 저장됨.
  
</details>

### 💡 람다의 예외 처리


