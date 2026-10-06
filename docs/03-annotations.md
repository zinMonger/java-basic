# 어노테이션을 읽는 순서

어노테이션은 코드에 붙이는 메타데이터입니다. 그 자체가 메서드를 실행하는 명령은 아닙니다. 컴파일러·테스트 도구·Spring 등이 읽고 검증하거나 동작을 연결합니다. 어디에 붙었는지, 누가 처리하는지, 어떤 조건에서 효력이 있는지를 함께 읽습니다.

| 단계 | 어노테이션 | 읽을 의미와 주의점 |
|---|---|---|
| 초급 | `@Override` | 상위 타입의 메서드를 구현/재정의하는지 컴파일러가 검사 |
| 초급 | `@Deprecated` | 사용을 권장하지 않는 API 표시, 즉시 실행 불가라는 뜻은 아님 |
| 중급 | `@SpringBootApplication` | Boot 설정·자동 설정·컴포넌트 스캔의 시작점 |
| 중급 | `@Component`, `@Service`, `@Repository` | Spring이 관리할 객체 후보, 업무/데이터 접근의 역할 표시 |
| 중급 | `@Configuration`, `@Bean` | 설정 클래스와 Spring이 관리할 객체 생성 정의 |
| 중급 | `@RestController`, `@GetMapping`, `@PostMapping` | HTTP 경로와 응답 처리 연결 |
| 중급 | `@RequestParam`, `@PathVariable`, `@RequestBody` | 쿼리·경로·본문에서 입력을 받는 위치 |
| 중급 | `@Autowired` | 의존성 주입 표시; 생성자가 하나라면 생략 가능 |
| 중급 | `@Valid` | 검증 요청; 검증 의존성과 제약 조건이 필요 |
| 고급 | `@Transactional` | 트랜잭션 인터셉터를 통한 경계 선언 |
| 검증 | JUnit `@Test`, `@SpringBootTest` | 테스트 실행 표시, 후자는 Spring 컨텍스트 기반 검증 |

## 생성자 주입 읽기

```java
@Service
public class OrderService {
    private final OrderRepository repository;

    public OrderService(OrderRepository repository) {
        this.repository = repository;
    }
}
```

위는 읽기용 일부 코드입니다. `Service` import, `OrderRepository` 선언과 등록이 필요합니다. Spring은 스캔된 Service를 만들 때 Repository를 생성자에 전달합니다. `final`은 필드 참조의 재할당을 막지만 객체 내부가 무조건 불변이라는 뜻은 아닙니다. 구현 후보가 여러 개면 `@Qualifier` 등의 선택 방법이 필요합니다.

## 트랜잭션에서 꼭 읽을 조건

기본 프록시 방식에서 같은 객체 내부의 메서드 호출은 트랜잭션 프록시를 거치지 않습니다. 어노테이션만 보고 적용된 것으로 판단하지 않습니다. 기본 롤백은 RuntimeException과 Error이며 checked exception은 필요에 따라 `rollbackFor` 등을 명시합니다. 예외를 잡아 정상 반환하면 자동 롤백되지 않을 수 있습니다. 전파 옵션, 선택된 트랜잭션 매니저, DB 자원, 호출 경로를 확인합니다. [Spring 공식 @Transactional 설명](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/annotations.html)

과제: 각 어노테이션에 대해 대상 위치, 처리 주체, 제거 시 변화, DB 영향 네 가지를 설명합니다. 요청 매핑과 DB 트랜잭션은 서로 다른 역할임을 호출 흐름으로 보여줍니다.
