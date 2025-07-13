# 3. 람다표현식

## 3.1 람다란 무엇인가?
## 3.2 어디에, 어떻게 람다를 사용할까?
### 3.2.1 함수형 인터페이스
### 3.2.2 함수 디스크립터
## 3.3 람다 활용 : 실행 어라운드 패턴
### 3.3.1 1딘계 : 동작 파라미터화를 기억하라
### 3.3.2 2단계 함수형 인터페이스를 이용해서 동작 전달
### 3.3.3 3단계 : 동작 실행
### 3.3.4 4단계 : 람다 전달

1단계: 고정된 파일 처리
```
public String processFile() throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
        return br.readLine(); // 고정된 동작 (1줄 읽기)
    }
}
```
> data.txt 파일에서 첫 줄만 읽는 고정된 동작<br>동작을 바꾸려면 메서드 자체를 수정해야 함 → 유연성 낮음

2단계: 동작을 함수형 인터페이스로 분리
```
@FunctionalInterface
public interface BufferedReaderProcessor {
    String process(BufferedReader br) throws IOException;
}
```
> BufferedReader를 받아서 String을 반환하는 동작의 틀을 정의<br>이걸 통해 다양한 파일 처리 방식을 외부에서 전달할 수 있음

3단계: 동작을 파라미터로 받아 실행
```
public String processFile(BufferedReaderProcessor processor) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
        return processor.process(br); // 전달된 동작 실행
    }
}
```
이제 파일을 어떻게 읽을지 결정하는 "동작"을 외부에서 받아서 실행함
코드 재사용성과 유연성이 높아짐

4단계: 람다로 동작 전달
```
String oneLine = processFile((BufferedReader br) -> br.readLine());
String twoLines = processFile((BufferedReader br) ->
    br.readLine() + br.readLine()
);
```
람다식을 통해 구현 객체를 만들지 않고 바로 동작만 전달
oneLine: 한 줄만 읽음
twoLines: 두 줄 읽어서 이어붙임

최종 구조 요약
```
// 1. 함수형 인터페이스
@FunctionalInterface
public interface BufferedReaderProcessor {
    String process(BufferedReader br) throws IOException;
}

// 2. 템플릿 메서드
public String processFile(BufferedReaderProcessor processor) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
        return processor.process(br);
    }
}

// 3. 실행
String oneLine = processFile(br -> br.readLine());
String twoLines = processFile(br -> br.readLine() + br.readLine());
```
> 파일 읽는 동작을 "파라미터(함수)"로 전달해서 유연하게 처리할 수 있도록 람다 + 함수형 인터페이스를 활용한 구조


## 3.4 함수형 인터페이스 사용
### 3.4.1 Predicate
### 3.4.2 Consumer
### 3.4.3 Functional

#### 기본형 특화
자바의 모든 형식은 참조형, 기본현에 해당한다 하지만 제네릭 파라미터에는 참조형만 사용할 수 있다.
제네릭의 내부구현때문에 어쩔 수 없는 일이다.

✅ 이유: 제네릭의 내부 구현 때문 ? 이게 어떤의미일까
자바의 제네릭은 컴파일 타임에 타입 체크만 하고,
**런타임에는 타입 정보를 지우는 방식(type erasure)**으로 동작한다

예:
```
List<String> list = new ArrayList<>();
이건 컴파일 후에 아래처럼 동작하게 바뀜:

List list = new ArrayList(); // 타입 정보 사라짐!
자바는 이걸 통해 모든 제네릭 타입을 Object 기반으로 처리한다.
```

> 내부적으로는 **Object[]** 같은 배열로 처리되는데,<br>int, double 같은 기본형은 Object로 처리 불가능하다 <br>→ 기본형은 Object의 서브타입이 아니기 때문!<br>✅ 그래서 자바는 기본형 대신 **랩퍼 클래스(wrapper class)**를 사용!

> 요약<br>자바의 모든 타입은 기본형 또는 참조형인데,<br>제네릭은 내부적으로 Object 기반으로 동작하기 때문에,<br>기본형은 사용할 수 없고 참조형만 가능하다는 의미!


#### Predicate<Integer> vs IntPredicate
✅ Predicate<Integer>의 문제점
```
Predicate<Integer> isPositive = i -> i > 0;
여기서 i는 사실 Integer 타입이에요.
```
즉, 이런 식으로 사용하면:<br>
```
isPositive.test(5);
```
> ⚠️ 5는 int니까 → 자바가 박싱해서 Integer로 변환<br> → 내부적으로 Integer.valueOf(5) 같은 동작 수행<br>그리고 결과는 boolean, 즉 기본형으로 처리되긴 하지만,<br>이 과정에서 불필요한 객체 생성 비용과 힙 메모리 사용이 발생합니다.

✅ IntPredicate가 등장한 이유
자바 8에서는 이런 박싱/언박싱으로 인한 성능 저하를 막기 위해<br>IntPredicate, IntFunction, IntConsumer 같은<br>**"기본형 특화형 함수형 인터페이스"**를 따로 만들었어요.

```
IntPredicate isPositive = i -> i > 0;
```
여기서 i는 진짜 int예요.<br>→ 그래서 박싱이 전혀 발생하지 않음


✅ 핵심 요약

> IntPredicate가 박싱이 없다?	파라미터가 int라서 Integer로 바꾸는 박싱이 발생하지 않음<br>결과는 boolean인데 왜 걱정 안 해도 됨?	결과는 원래 기본형이라 박싱 없음<br>Predicate<Integer>보다 뭐가 좋음?	성능: 메모리 덜 쓰고, GC 부담 덜고, 속도 빠름


## 3.5 형식검사, 형식 추론, 제약
### 3.5.1 형식검사
### 3.5.2 같은람다, 다른 함수형 인터페이스
### 3.5.3 형식추론
### 3.5.4 지역 변수 사용

## 3.6 메서드 참조
## 3.7 람다, 메서드 참조 활용하기
