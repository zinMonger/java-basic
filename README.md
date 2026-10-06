# java-basic — DBA를 위한 Java 코드 읽기와 프로시저 전환

SQL과 DB 운영에 익숙하지만 Java 경험이 적은 학습자가 변수와 메서드부터 Spring Boot의 호출 흐름까지 읽고, 프로시저의 업무 규칙을 Java로 옮길 때 달라지는 의미를 설명하는 교육 프로젝트입니다. 전달자는 환경 구성과 수업을 안내하고, 학습자는 직접 설치·타이핑·디버깅·검증합니다.

## 문서 읽는 순서

1. [교육 설계와 단계별 커리큘럼](docs/01-curriculum.md)
2. [수동 환경 설정과 VS Code 확장](docs/02-environment.md)
3. [Java·Spring 어노테이션 읽기](docs/03-annotations.md)
4. [프로시저 전환 학습 지도](docs/conversion/README.md)
5. [요소별 전환 비교와 예제](docs/conversion/01-elements.md)
6. [실제 프로시저 분석·검증 양식](docs/conversion/02-case-template.md)
7. [멘토링 운영과 사전 질문](docs/04-mentoring.md)
8. [디버거·로그 실습](docs/05-debugging-logging.md)
9. [Oracle·MySQL 비교](docs/conversion/03-oracle-mysql.md)

## 초안의 전제

- Windows + PowerShell + VS Code, JDK 21, Maven Wrapper를 교육 기준으로 제안합니다. 실제 교육 PC 환경은 확인이 필요합니다.
- Spring Boot 버전은 교재와 대상 시스템을 확인한 뒤 반 전체에서 통일합니다. 설치 문서에 선정 기준을 정리했습니다.
- 평일 5일 × 하루 2시간, 총 10시간입니다. 본인 노트북에서 코드 읽기를 중심으로 작은 코드 작성과 전환을 디버거·로그로 확인합니다. 자율 실습은 선택 사항입니다.
- 지정 교재는 없습니다. 일반적인 Java 입문서의 개념 순서를 사용합니다. DB는 Oracle과 MySQL이며 버전·실습 접속 환경은 미정입니다.
- 예제 업무는 가상의 주문 금액 검증입니다. 실제 업무 및 데이터에 맞게 교체합니다.

환경 설치나 애플리케이션 생성은 아직 수행하지 않았습니다. 학습자가 아래 문서대로 직접 만드는 것이 첫 실습입니다.
