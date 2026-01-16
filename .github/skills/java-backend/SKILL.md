---
name: java-backend
version: 0.2.0
author: Your Name
contact: your.email@example.com
tags: [java, backend, spring, guidelines]
---

# Skill: Java Backend Development

## Purpose

This skill defines how the AI agent should write, refactor, and review Java code
in this repository. It is both human-readable guidance and machine-consumable metadata
so automation (CI, bots) can validate requests against the rules.

---

## Language & Version

- Java 21 (minimum)
- Prefer modern Java features: records, sealed classes, switch expressions
- Avoid deprecated APIs and internal JDK APIs

## Build tools (examples)

- Gradle: use Gradle 8.x+
- Maven: use Maven 3.8+

---

## Metadata (machine-readable)

- Inputs: `request` (JSON) — action and payload; `config` — runtime configuration
- Outputs: `response` (JSON) — {status, data, errors}
- Automations: lint, fmt, unit-tests, build

Example input schema (JSON):

```json
{
  "action": "createUser",
  "payload": { "name": "Alice", "email": "alice@example.com" }
}
```

Example output schema (JSON):

```json
{
  "status": "ok|error",
  "data": {},
  "errors": []
}
```

---

## Code Style Rules

- Follow clean code principles and meaningful naming
- Prefer immutability and small value objects (records)
- One public class per file
- Max method length: 30 lines (exceptions allowed with justification)
- Keep methods focused and testable

---

## Architecture Rules

- Follow Hexagonal (Ports & Adapters) / Clean Architecture principles. Emphasize clear
  boundaries between core business logic and external frameworks.

  - **Domain (Core):** Entities, value objects, domain services and domain exceptions — pure Java, no framework types.
  - **Application / Use Cases:** Use-case implementations and application services. Orchestrate domain objects.
  - **Ports (Interfaces):** Inbound ports (input) and outbound ports (output) defined by the application layer.
  - **Infrastructure (Adapters):** Implementations of ports (REST controllers, persistence, external API clients). Keep these implementations in `infrastructure` packages.
  - **Framework & Drivers:** Spring, JPA, web servers, DB drivers — keep these at the outermost layer.

- Dependency rule: code dependencies must point inward. Inner layers (domain, use-cases) must not depend on adapters or framework types.

- Ports & adapters conventions:

  - **Inbound ports (input):** interfaces the adapters call to trigger use cases (e.g., `CreateUserUseCase`).
  - **Outbound ports (output):** interfaces the use cases call to access external resources (e.g., `UserRepositoryPort`).
  - Infrastructure adapters live in `infrastructure.*` packages and implement the port interfaces.

- Mapping and boundaries:

  - Keep DTOs, REST models and mappers inside adapter layers; convert to/from domain objects at the boundary.
  - Do not leak framework-specific types (e.g., `Entity`, `@Entity`, Spring `HttpServletRequest`) into domain/use-case packages.

- Example package layout (hexagonal style):

  - `com.example.domain` — entities, value objects, domain exceptions
  - `com.example.application.port.in` — inbound port interfaces (use-case inputs)
  - `com.example.application.port.out` — outbound port interfaces (repositories/gateways)
  - `com.example.application.service` — use-case implementations (application services)
  - `com.example.infrastructure.web` — web controllers, request/response mappers
  - `com.example.infrastructure.persistence` — JPA repositories, DB mappers
  - `com.example.infrastructure.client` — external API clients (HTTP/gRPC), adapters for calling other services

  **Note:** In this repository `infrastructure` packages contain the concrete adapter implementations (i.e., the "adapters"). The term "adapter" is still correct conceptually, but implementations should be grouped under `infrastructure.*` (persistence, client, web, messaging, etc.).

  Example: external client port + adapter (Java)

  ```java
  // application/port/out/ExternalApiPort.java
  package com.example.application.port.out;

  public interface ExternalApiPort {
    ExternalData fetchData(String id);
  }

  // infrastructure/client/ExternalApiHttpClient.java
  package com.example.infrastructure.client;

  @Component
  public class ExternalApiHttpClient implements ExternalApiPort {
    private final WebClient webClient;

    public ExternalApiHttpClient(WebClient webClient) { this.webClient = webClient; }

    @Override
    public ExternalData fetchData(String id) {
      // call remote service, map response to domain-friendly ExternalData
      var resp = webClient.get().uri("/api/data/{id}", id).retrieve().bodyToMono(RemoteDto.class).block();
      return RemoteMapper.toExternalData(resp);
    }
  }
  ```

- Minimal ASCII diagram:

  [Infrastructure In (Web)] -> [Inbound Port Interface] -> [Use Case / Application Service] -> [Outbound Port Interface] -> [Infrastructure Out (DB/External)]

- Rationale: hexagonal boundaries improve testability, make adapters replaceable, and enforce single responsibility and separation of concerns.

---

## Framework Rules (Spring Boot guidance)

- Use constructor injection only — prefer final fields
- Do NOT use field injection
- Prefer `@Service`, `@Component`, `@Repository` stereotypes
- Avoid `@Autowired` in favor of constructor injection
- Use `@ConfigurationProperties` for typed configuration
- Logging: use SLF4J/Logback; do NOT use `System.out.println`

---

## Error Handling

- Do not throw generic `Exception`
- Use custom domain exceptions and explicit mappings to HTTP status codes
- Map exceptions explicitly (controller advice or exception mappers)

---

## Testing Rules

- Use JUnit 5
- Use AssertJ for assertions
- Use Mockito only when necessary (prefer test-friendly design)
- Favor unit tests; require integration tests for critical flows and CI gating

Example unit test (JUnit 5 + AssertJ):

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
  @Mock UserRepository repo;
  @InjectMocks UserService service;

  @Test
  void createUser_savesAndReturnsId() {
    when(repo.save(any())).thenReturn(new User(1L, "Alice"));
    var dto = new CreateUserDto("Alice", "a@example.com");
    var result = service.createUser(dto);
    assertThat(result.getId()).isEqualTo(1L);
  }
}
```

---

## Forbidden Patterns

- No static utility classes for business logic
- No God classes
- No magic numbers (use named constants)
- No `System.out.println` (use logger)

---

## Reasoning Checklist (agent must return before code)

When asked to implement or modify code, the agent must return a short checklist containing:

1. Affected layer(s) (controller/service/domain/repository)
2. Chosen abstraction(s) and DTOs/entities
3. Design validation vs. rules above (list any rule exceptions)
4. Ports affected (inbound/outbound) and mapper responsibilities (DTO -> domain)
5. Dependency validation: confirm no inward-dependencies are violated

Example response the agent must produce before code:

```
Layer: service
Abstraction: UserService (interface) + UserServiceImpl
Validation: Uses constructor injection, domain layer unchanged, method length <= 30
```

If a request violates the rules, explain why and propose an alternative.

---

## Examples (minimal project layout)

```
src/main/java/com/example
  /domain
  /application/port/in
  /application/port/out
  /application/service
  /infrastructure/web
  /infrastructure/persistence
src/test/java/com/example
```

Sample flow (infrastructure -> port -> use case -> port -> infrastructure):

Inbound port interface (application/port/in):

```java
package com.example.application.port.in;

public interface CreateUserUseCase {
  long create(CreateUserCommand command);
}
```

Outbound port interface (application/port/out):

```java
package com.example.application.port.out;

public interface UserRepositoryPort {
  UserEntity save(UserEntity user);
  Optional<UserEntity> findById(long id);
}
```

Use-case implementation (application/service):

```java
package com.example.application.service;

public class CreateUserService implements CreateUserUseCase {
  private final UserRepositoryPort repository;

  public CreateUserService(UserRepositoryPort repository) {
    this.repository = repository;
  }

  @Override
  public long create(CreateUserCommand command) {
    var user = UserEntity.of(command.name(), command.email());
    var saved = repository.save(user);
    return saved.getId();
  }
}
```

Infrastructure web controller (infrastructure/web) — calls inbound port, maps DTO->domain via mapper:

```java
package com.example.infrastructure.web;

@RestController
public class UserController {
  private final CreateUserUseCase createUser;
  private final UserMapper mapper;

  public UserController(CreateUserUseCase createUser, UserMapper mapper) {
    this.createUser = createUser;
    this.mapper = mapper;
  }

  @PostMapping("/users")
  public ResponseEntity<Map<String, Long>> create(@RequestBody CreateUserDto dto) {
    var cmd = mapper.toCommand(dto);
    long id = createUser.create(cmd);
    return ResponseEntity.status(HttpStatus.CREATED).body(Map.of("id", id));
  }
}
```

Mapper example (infrastructure layer):

```java
package com.example.infrastructure.web;

public class UserMapper {
  public CreateUserCommand toCommand(CreateUserDto dto) {
    return new CreateUserCommand(dto.name(), dto.email());
  }
}
```

Persistence adapter (infrastructure/persistence) implements `UserRepositoryPort` and maps between persistence entities and domain entities.

---

---

## CI / Automation (suggested)

Add a GitHub Actions job `.github/workflows/java-ci.yml` to run:

- JDK 21
- Build (Gradle/Maven)
- Run `./gradlew check` or `mvn -DskipTests=false verify`

Example workflow snippet:

```yaml
name: Java CI
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up JDK 21
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: "21"
      - name: Build and test (Gradle)
        run: ./gradlew build --no-daemon
```

---

## Notes & Exceptions

- Method-length rule can be relaxed when a refactor is documented in PR description
- Mockito allowed for legacy code where dependency inversion is impractical

## Architecture verification (optional)

- Use ArchUnit to assert that package dependencies point inward. Minimal test example:

```java
@AnalyzeClasses(packages = "com.example")
class ArchitectureTest {
  @ArchTest
  static final ArchRule domain_should_not_depend_on_infrastructure =
    classes().that().resideInAPackage("..domain..")
      .should().onlyDependOnClassesThat().resideInAnyPackage("..domain..", "java..", "org.slf4j..");
}
```

## CI: optional integration and architecture checks

- Recommended CI extensions:
  - Add an optional `integration` job using Testcontainers for DB-backed flows.
  - Add an `architecture-check` job that runs ArchUnit tests to ensure dependency rules.

## Transactions & AOP

- Prefer to apply `@Transactional` and cross-cutting concerns at the application/use-case layer (`application.service`). Avoid placing transactional boundaries in `infrastructure.web` (controllers).

---

## Change log

- 0.2.0: Added metadata, machine-readable fields, examples, CI skeleton
