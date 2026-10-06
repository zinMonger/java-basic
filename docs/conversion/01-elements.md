# 요소별 전환 비교

## 타입과 반환 계약

| DB/프로시저 요소 | Java 후보 | 검증할 차이 |
|---|---|---|
| 정수 NUMBER/INTEGER | int, long 또는 BigDecimal | 범위·정밀도·NULL 허용; NUMBER가 항상 long은 아님 |
| 금액 DECIMAL | BigDecimal | scale·반올림, 문자열 생성자 사용, compareTo와 equals 차이 |
| VARCHAR | String | 길이 기준, 공백, 인코딩, DB별 빈 문자열 처리 |
| DATE/TIMESTAMP | LocalDate, LocalDateTime, Instant 등 | DB DATE가 시각을 포함하는지, 시간대·정밀도 |
| SQL NULL | 참조형 null 등 | SQL의 UNKNOWN과 Java 조건식은 다름; 언박싱 시 예외 |
| IN/OUT/INOUT | 인자와 반환 DTO | 복수 출력·상태 코드·결과 집합을 계약으로 표현 |

금액에 double을 기본 선택하지 않습니다. SQL의 `= NULL`을 Java `== null`과 대응시켜 읽지 않습니다. SQL에서는 `IS NULL`과 3값 논리를 확인합니다. 특히 Oracle의 빈 문자열 처리와 날짜 의미는 실제 버전·타입을 확인합니다.

## 분기·반복·조회

- IF/CASE는 if/switch로 표현할 수 있지만 NULL 분기와 기본 분기를 명시합니다.
- Cursor 루프를 옮기기 전에 하나의 UPDATE/JOIN/집계 SQL로 유지할 수 있는지 확인합니다. Java에서 한 행마다 조회하면 SQL 횟수와 네트워크 왕복이 늘어납니다.
- 단건 조회는 0건·1건·복수 건 각각의 계약을 정의합니다. DB의 SELECT INTO 예외와 Java 조회 API의 반환 동작을 비교합니다.
- WHERE에 들어가는 입력은 바인딩합니다. 테이블/컬럼 이름 같은 SQL 식별자는 일반 값 파라미터로 바인딩할 수 없으므로 허용된 선택지로 제한합니다.
- List 순서는 명시적 ORDER BY가 없으면 원본과 같다고 가정하지 않습니다.

## 작은 업무 규칙 예제: 음수 금액 거부

아래 프로시저는 특정 DB에서 실행하는 SQL이 아닌 의사코드입니다.

```text
PROCEDURE validate_amount(IN amount, OUT normalized_amount)
  IF amount IS NULL THEN 업무 오류
  IF amount < 0 THEN 업무 오류
  normalized_amount := amount
END
```

대응하는 순수 Java 코드는 `AmountPolicy.java`로 작성할 수 있습니다.

```java
import java.math.BigDecimal;

public class AmountPolicy {
    public BigDecimal validate(BigDecimal amount) {
        if (amount == null) {
            throw new IllegalArgumentException("amount is required");
        }
        if (amount.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("amount must be non-negative");
        }
        return amount;
    }
}
```

읽기 과제: 입력 타입, 반환 타입, 두 오류 경로, `compareTo`의 결과를 설명합니다. 비교 입력은 null, -0.01, 0, 0.00, 100.25, DB 허용 최대값과 범위 초과 값입니다. 이 예제에는 DB 갱신과 반올림이 없습니다. 저장 시 허용 scale과 최대값 검증은 실제 계약에 추가합니다. DB 오류 코드를 HTTP 오류로 바꾸는 규칙도 별도 계약입니다.

## 트랜잭션과 오류

DB 프로시저 내부 COMMIT 위치와 호출자의 트랜잭션을 먼저 조사합니다. 여러 COMMIT이 있는 프로시저를 하나의 `@Transactional`로 감싸면 실패 시 남는 변경이 달라질 수 있습니다. 자동 커밋, 자율 트랜잭션, savepoint, 중첩/전파 동작은 DB와 Spring에서 개별 검증합니다.

일반적인 구조는 Service에 업무 경계, Repository에 SQL입니다. 성공 시 영향 행 수를 확인하고 두 번째 갱신을 실패시켜 첫 번째 갱신까지 롤백되는지 확인합니다. 예외를 삼키거나 각 단계에서 임의 COMMIT을 추가하지 않습니다. 오류는 업무 오류와 연결/제약/타임아웃 같은 기술 오류를 구별하고 원인을 보존합니다.

## 동시성과 성능

DB 패키지/세션 변수는 Spring singleton 필드와 같지 않습니다. singleton에 요청별 금액이나 사용자 ID를 저장하면 요청 간 데이터가 섞일 수 있습니다. 요청별 값은 지역변수와 입력 객체에 둡니다.

동일 주문의 동시 갱신, 재실행, 락 대기, 타임아웃, 데드락, 부분 실패를 시험합니다. 격리 수준·락·유일 제약·멱등성 키를 실제 업무에 맞게 정합니다. 원본과 전환본의 SQL 횟수, 처리 시간, 조회 행 수, 메모리 사용을 비교하고 집합 처리 유지 여부를 결정합니다. 외부 API 호출과 DB 갱신은 일반적인 로컬 DB 트랜잭션만으로 함께 롤백되지 않습니다.
