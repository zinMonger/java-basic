# Oracle·MySQL·Java 비교

대상은 Oracle과 MySQL이며 버전은 추가 확인합니다. 입문 과정은 한 번에 한 DB를 연결하고 같은 업무 규칙을 비교합니다. 두 DataSource와 복수 트랜잭션 매니저 구성은 후속 과정입니다.

| 요소 | Oracle PL/SQL | MySQL 프로시저 | Java |
|---|---|---|---|
| 대입 | `:=` | SET | `=` |
| 입력/출력 | IN/OUT/IN OUT | IN/OUT/INOUT | 인자/반환 DTO |
| 오류 | RAISE_APPLICATION_ERROR | SIGNAL | throw |
| 처리 | EXCEPTION | DECLARE HANDLER | try/catch와 전파 |
| 빈 문자열 | 길이 0 문자열을 NULL로 취급 | 일반적으로 빈 문자열과 NULL 구별 | 빈 String과 null 구별 |
| 블록 | BEGIN | BEGIN | 메서드 블록 |
| 금액 | NUMBER(p,s) | DECIMAL(p,s) | BigDecimal |

MySQL 저장 프로그램의 BEGIN은 블록 시작이며 트랜잭션 시작에는 START TRANSACTION을 사용합니다. 오류 처리 구문이 있다고 모든 변경이 자동 롤백된다고 판단하지 않습니다. 근거: [Oracle PL/SQL 기본 규칙](https://docs.oracle.com/en/database/oracle/oracle-database/18/lnpls/plsql-language-fundamentals.html), [Oracle 트랜잭션](https://docs.oracle.com/en/database/oracle/oracle-database/26/lnpls/transaction-processing-and-control.html), [MySQL 저장 프로그램 제약](https://dev.mysql.com/doc/mysql-reslimits-excerpt/8.0/en/stored-program-restrictions.html). 교육 DB 버전이 정해지면 해당 버전 자료로 다시 확인합니다.

## 금액 검증 비교

아래는 읽기용 본문 발췌입니다. CREATE PROCEDURE·타입 선언·클라이언트 구분자가 없는 완성 전 코드입니다.

Oracle:

```sql
IF p_amount IS NULL OR p_amount < 0 THEN
    RAISE_APPLICATION_ERROR(-20001, 'invalid amount');
END IF;
p_result := p_amount;
```

MySQL:

```sql
IF p_amount IS NULL OR p_amount < 0 THEN
    SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'invalid amount';
END IF;
SET p_result = p_amount;
```

Java는 [AmountPolicy 예제](01-elements.md)의 validate와 비교합니다. null/-1/0/100.25에서 조건·오류·정상 반환을 표시합니다. DB 오류 코드와 Java 예외는 업무 계약을 기준으로 대응시킵니다.

전달자는 DB 버전·교육 계정·JDBC 드라이버·접속 URL·초기 데이터를 준비합니다. 후속 실습은 0건/복수 건 조회, cursor 종료, OUT 결과, 중간 갱신 실패의 커밋 여부를 비교합니다.
