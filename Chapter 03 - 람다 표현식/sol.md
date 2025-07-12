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

### 💡 람다의 예외 처리


