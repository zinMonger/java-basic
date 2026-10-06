# 디버거·로그로 코드 읽기

## 디버거 순서

1. Controller와 Service에 중단점을 둡니다.
2. Application을 Debug로 실행하고 요청합니다.
3. Variables에서 타입·값·null을 확인합니다.
4. Step Over로 줄을 실행하고 Step Into로 호출 메서드에 들어갑니다. Step Out으로 돌아옵니다.
5. Call Stack에서 호출 경로를 확인합니다.
6. 정상·음수·null 입력을 비교하고 예상과 실제 차이를 기록합니다.

Watch에는 값 조회를 사용합니다. DB 갱신처럼 부작용이 있는 메서드를 디버거 표현식으로 실행하지 않습니다.

## Boot에서 로그 추가하기

Service 클래스에 다음 필드를 추가합니다. 필요한 import는 `org.slf4j.Logger`, `org.slf4j.LoggerFactory`입니다. 아래는 코드 발췌이며 클래스 이름은 실제 이름에 맞춥니다.

```java
private static final Logger log = LoggerFactory.getLogger(AmountService.class);
```

업무 메서드에서 입력 확인 전, 검증 실패, 정상 종료에 로그를 추가합니다.

```java
log.debug("validation started, amount={}", amount);
log.warn("validation rejected");
log.info("validation completed");
```

세 줄은 각 경로에 배치하며 무조건 연속 실행하지 않습니다. `application.properties`에 `logging.level.com.example.javabasic=DEBUG`를 넣습니다. DEBUG는 개발 상세 값, INFO는 주요 진행 상태, WARN은 주의할 사건, ERROR는 처리 실패입니다. 예제는 가상 금액만 출력합니다. 실제 로그에는 비밀번호·개인정보·전체 요청 본문을 넣지 않습니다.

예외는 책임지는 경계에서 원인과 함께 기록하고, 로그만 찍은 뒤 성공으로 반환하지 않습니다. 최상단 메시지와 `Caused by`를 함께 읽습니다. 로그의 완료 문구는 DB 커밋 증거가 아니므로 트랜잭션 종료 뒤 별도 조회로 상태를 확인합니다.

## 매 실습 기록

입력 → 예상 경로 → 중단점 값 → 로그 순서 → 원인 예외 → 최종 DB 상태를 적습니다. SQL 실습은 바인딩 항목, 반환 행 수, 갱신 행 수, 처리 시간을 확인합니다. 로그 출력과 디버거는 서로 보완하는 증거입니다.
