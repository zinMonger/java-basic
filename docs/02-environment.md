# 수동 환경 설정 — Windows / PowerShell / VS Code

학습자가 단계마다 설치 목적과 확인 결과를 기록합니다. 아래는 Windows 기준이며 OS·설치 권한·망 정책을 확인한 뒤 수정합니다. 최초 다운로드와 Maven 의존성 다운로드에는 인터넷 또는 사내 저장소가 필요합니다.

## 1. 사용할 버전 정하기

교육용 제안은 JDK 21 + Maven + Spring Boot입니다. 2026-10-06 확인 시 공식 문서는 Spring Boot 4.1.1의 Java 범위를 17~26, Maven 최소 버전을 3.6.3으로 안내합니다. JDK 21은 이 범위 안에 있습니다. [Spring Boot 시스템 요구사항](https://docs.spring.io/spring-boot/system-requirements.html)

교재나 실무가 Boot 3.x 기준이면 해당 버전의 공식 요구사항과 지원 상태를 확인하여 동일 계열로 맞춥니다. Boot 4 예제와 Boot 3 교재의 의존성을 섞지 않습니다. Initializr에서 안정 버전을 선택하고 SNAPSHOT/Milestone은 교육 기본값에서 제외합니다. 교육 시작 시 정확한 JDK 패치, Boot, Wrapper 버전을 아래에 기록하고 유지합니다.

| 항목 | 교육 반 확정값 |
|---|---|
| OS / 아키텍처 | 확인 필요 |
| JDK 공급자 / 전체 버전 | 21 계열 제안, 패치 미정 |
| Spring Boot | 미정 |
| Maven Wrapper 버전 | 생성 후 기록 |
| DB / JDBC 드라이버 | DB 결정 후 기록 |

## 2. JDK 설치와 환경 변수

1. 조직에서 허용한 JDK 공급자의 공식 배포처에서 OS/CPU에 맞는 **JDK 21**을 내려받아 직접 설치합니다. 예: [Eclipse Temurin](https://adoptium.net/temurin/releases/?version=21). JDK에는 컴파일러가 포함됩니다.
2. 설치 폴더를 확인합니다. Windows의 환경 변수 편집 화면에서 사용자 변수 `JAVA_HOME`을 JDK 루트 폴더로 지정합니다(`bin` 제외).
3. 사용자 `Path`에 `%JAVA_HOME%\bin`을 추가합니다. 기존 항목을 삭제하거나 Path 전체를 덮어쓰지 않습니다.
4. 터미널과 VS Code를 닫았다가 다시 열고 아래를 각각 실행합니다.

```powershell
java -version
javac -version
$env:JAVA_HOME
Get-Command java
Get-Command javac
```

두 버전이 21 계열인지, 명령 경로가 설치한 JDK인지 확인합니다. 여러 JDK가 있다면 `where.exe java`로 경로 순서를 확인합니다. [Java 첫 실행 안내](https://dev.java/learn/first-steps/)

## 3. VS Code 설치와 확장

[VS Code 공식 다운로드](https://code.visualstudio.com/download)에서 설치합니다. `Ctrl+Shift+X`에서 이름과 게시자를 확인하고 다음 팩을 직접 설치합니다.

| 우선순위 | 확장 / ID | 목적 |
|---|---|---|
| 필수 | Extension Pack for Java / `vscjava.vscode-java-pack` | Java 언어 지원, 디버거, 테스트, Maven, 프로젝트 탐색 |
| 필수 | Spring Boot Extension Pack / `vmware.vscode-boot-dev-pack` | Spring 설정 지원, Initializr, Boot Dashboard |

공식 안내: [Java 확장과 시작](https://code.visualstudio.com/docs/java/java-tutorial), [Spring Boot 개발](https://code.visualstudio.com/docs/java/java-spring-boot), [Spring 확장 팩](https://marketplace.visualstudio.com/items?itemName=vmware.vscode-boot-dev-pack).

Markdown 미리보기와 Git 기본 기능은 VS Code에 포함되어 있습니다. DB 실습은 기존에 익숙한 DBA 도구를 사용해도 됩니다. Lombok은 생성자·필드를 직접 읽는 초기 교육 이후에 도입합니다.

명령 팔레트(`Ctrl+Shift+P`) → `Java: Configure Java Runtime`에서 프로젝트 JDK를 확인합니다. 확장의 언어 서버 런타임과 프로젝트 컴파일 JDK는 다를 수 있으므로 확장 안내도 확인합니다.

## 4. Java 첫 파일을 손으로 만들기

프로젝트 안에 `labs/hello` 폴더와 `HelloWorld.java`를 직접 만들고 다음을 타이핑합니다. 파일 이름과 public 클래스 이름을 맞춥니다.

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, DBA!");
    }
}
```

터미널에서 해당 폴더로 이동한 뒤 실행합니다.

```powershell
javac -encoding UTF-8 HelloWorld.java
java HelloWorld
```

예상 출력은 `Hello, DBA!`입니다. 생성된 `.class`와 소스를 비교하고, VS Code에서 출력 줄에 중단점을 넣어 Debug를 실행합니다. `main`, `String[]`, `static`은 1회차에 역할을 소개하고 이후 수업에서 다시 다룹니다.

## 5. Spring Boot 프로젝트 생성

1. [Spring Initializr](https://start.spring.io/)를 브라우저에서 직접 엽니다.
2. Project: Maven, Language: Java, Packaging: Jar, Java: 21을 선택합니다. Boot는 1단계에서 확정한 안정 버전입니다.
3. Group: `com.example`, Artifact: `java-basic-app`, Package: `com.example.javabasic`, Dependencies: `Spring Web`을 선택합니다.
4. Generate로 ZIP을 받고 프로젝트의 `labs/java-basic-app`에 압축을 풉니다. `pom.xml`이 그 폴더 바로 아래에 오도록 확인합니다.
5. VS Code의 Open Folder로 `pom.xml`이 있는 폴더를 엽니다. 신뢰할 수 있는 본인이 생성한 파일인지 확인하고 Java 프로젝트 가져오기가 끝날 때까지 기다립니다.
6. `pom.xml`, `src/main/java`, `src/main/resources`, `src/test/java`, `mvnw.cmd`, `.mvn/wrapper`의 목적을 설명합니다.

Maven Wrapper가 Maven을 준비하므로 Maven 전역 설치는 이 실습의 필수 단계가 아닙니다. 다음은 **생성된 앱 폴더**에서 실행합니다.

```powershell
.\mvnw.cmd -v
.\mvnw.cmd test
.\mvnw.cmd spring-boot:run
```

최초 실행은 Maven과 라이브러리를 다운로드할 수 있습니다. `-v`에서 Java 21과 실제 Maven 버전을 기록합니다. 서버 시작 로그를 확인하고 `Ctrl+C`로 종료합니다.

## 6. 첫 요청 처리 코드를 직접 만들기

`src/main/java/com/example/javabasic/HelloController.java`를 작성합니다. 생성된 Application 클래스가 `com.example.javabasic`에 있는지 확인합니다.

```java
package com.example.javabasic;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
        return "Hello, DBA!";
    }
}
```

앱을 다시 실행하고 새 터미널에서 확인합니다.

```powershell
Invoke-RestMethod http://localhost:8080/hello
```

예상 응답은 `Hello, DBA!`입니다. Controller 메서드에 중단점을 넣고 VS Code의 Application 실행 항목에서 Debug로 시작한 뒤 다시 요청합니다. 별도 터미널에서 이미 실행 중인 서버는 먼저 종료합니다.

## 7. DB 연결은 Java/HTTP 기초 이후

DB가 확정되면 Initializr 또는 `pom.xml`에 JDBC용 starter와 해당 DB 드라이버를 추가합니다. 초기에는 SQL을 읽기 쉬운 JdbcTemplate을 사용하고 MyBatis/JPA는 실무 필요에 따라 확장합니다. 교육 DB에서 SELECT 1 또는 해당 DB의 동등 쿼리 → 파라미터 조회 → 단일 갱신 → 트랜잭션 순으로 확인합니다.

접속 정보는 환경 변수로 전달하고 실제 비밀번호를 학습 기록이나 Git에 쓰지 않습니다. 설정 예시는 다음과 같으며 DB별 URL은 별도로 확정합니다.

```properties
spring.datasource.url=${DB_URL}
spring.datasource.username=${DB_USER}
spring.datasource.password=${DB_PASSWORD}
```

H2를 사용할 경우 원본 DB의 NULL·날짜·잠금 동작까지 검증한 것으로 보지 않습니다.

## 오류 확인 순서

| 증상 | 먼저 확인할 사항 |
|---|---|
| java/javac 인식 실패 | JDK 설치, Path, 터미널 재시작 |
| 컴파일 버전 불일치 | JAVA_HOME, 프로젝트 JDK, Wrapper의 `-v` |
| 의존성 다운로드 실패 | 인터넷/사내 Maven 저장소, 프록시·인증서 설정 |
| 8080 사용 중 | 본인이 실행한 앱 종료 또는 `server.port=8081` 설정 후 URL 변경 |
| /hello 404 | 패키지 위치, 재시작, URL과 실제 포트 |
| 프로젝트 인식 실패 | pom.xml 폴더 열기, Java 가져오기 로그 |

완료 기록: 설치 버전, java/javac 경로, HelloWorld 출력, 테스트 결과, HTTP 응답, 직접 해결한 오류 한 가지. 이 문서의 코드는 실행 안내용이며 현재 프로젝트에서 설치·빌드 검증한 결과는 아닙니다.
