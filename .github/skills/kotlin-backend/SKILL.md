---
name: kotlin-backend
version: 0.1.0
author: Your Name
contact: your.email@example.com
tags: [kotlin, backend, spring, ktor, guidelines]
---

# Skill: Kotlin Backend Development

## Purpose

This skill describes how the AI agent should author, refactor and review Kotlin backend code
for this repository. It follows Hexagonal / Clean Architecture conventions and Kotlin best practices.

---

## Language & Version

- Kotlin 1.9+ (or latest stable)
- JDK 21
- Prefer Kotlin idioms: `data class`, `sealed`/`sealed interface`, extension functions, null-safety
- Prefer coroutines (`suspend` / `Flow`) for async code

## Build tools (examples)

- Gradle Kotlin DSL (Gradle 8.x+)
- Maven support ok but prefer Gradle for Kotlin projects

---

## Metadata (machine-readable)

- Inputs: `request` (JSON) — action and payload; `config` — runtime configuration
- Outputs: `response` (JSON) — {status, data, errors}
- Automations: lint (detekt), fmt (ktlint), unit-tests, build

---

## Code Style & Kotlin Rules

- Follow Kotlin coding conventions and clean code principles
- Prefer immutability and `data class` for simple DTOs/value objects
- Use `suspend` functions for I/O-bound operations and `Flow` for streams
- Avoid `!!` (use safe calls / explicit checks)
- Keep functions short and single-responsibility (max ~30 lines, justify exceptions)
- Use meaningful names; prefer expression-bodied functions where clear

---

## Architecture Rules (Hexagonal / Ports & Infrastructure)

- Follow Hexagonal (Ports & Adapters) / Clean Architecture with clear boundaries.
- Domain layer: pure Kotlin (no Spring/Ktor types), domain entities, domain exceptions.
- Application/use-case layer: defines inbound/outbound ports (interfaces) and implements use cases.
- Infrastructure: implementations of ports (web controllers, persistence, external API clients); keep framework types here.
- Dependency rule: dependencies point inward; domain/use-cases must not depend on infrastructure/frameworks.

Example package mapping:

- `com.example.domain`
- `com.example.application.port.in`
- `com.example.application.port.out`
- `com.example.application.service`
- `com.example.infrastructure.web`
- `com.example.infrastructure.persistence`

---

## Framework Guidance

- If using Spring Boot: prefer Spring WebFlux + coroutines support or use Spring MVC with suspend where supported.
- Ktor is an acceptable alternative for Kotlin-first microservices.
- Use constructor injection (prefer `@Component` / `@Service` where needed).
- Keep controllers thin: map DTO -> domain command and call inbound port.
- Logging: use SLF4J (Logback) or Kotlin-friendly logger; do NOT use `println`.

---

## Error Handling

- Use domain-specific exceptions; avoid generic `Exception`
- Map exceptions explicitly to HTTP responses using controller advice (Spring) or Ktor features

---

## Testing Rules

- Use JUnit 5
- Use Kotest or plain JUnit + AssertJ for assertions
- Use MockK for mocking Kotlin-friendly tests
- Prefer unit tests; add integration tests (Testcontainers) for DB or external systems

Example unit test (Kotest + MockK):

```kotlin
class CreateUserServiceTest : StringSpec({
  val repo = mockk<UserRepositoryPort>()
  val service = CreateUserService(repo)

  "create user saves and returns id" {
    every { repo.save(any()) } returns UserEntity(1L, "Alice", "a@example.com")
    val cmd = CreateUserCommand("Alice", "a@example.com")
    service.create(cmd) shouldBe 1L
  }
})
```

---

## Examples (Kotlin minimal layout & code)

```
src/main/kotlin/com/example
  /domain
  /application/port/in
  /application/port/out
  /application/service
  /infrastructure/web
  /infrastructure/persistence
src/test/kotlin/com/example
```

Inbound port (application/port/in):

```kotlin
package com.example.application.port.`in`

fun interface CreateUserUseCase {
  suspend fun create(command: CreateUserCommand): Long
}
```

Outbound port (application/port/out):

```kotlin
package com.example.application.port.out

interface UserRepositoryPort {
  suspend fun save(user: UserEntity): UserEntity
  suspend fun findById(id: Long): UserEntity?
}
```

Use-case implementation (application/service):

```kotlin
package com.example.application.service

class CreateUserService(
  private val repository: UserRepositoryPort
) : CreateUserUseCase {
  override suspend fun create(command: CreateUserCommand): Long {
    val user = UserEntity.create(command.name, command.email)
    val saved = repository.save(user)
    return saved.id
  }
}
```

Infrastructure web controller (Spring + coroutines example):

```kotlin
package com.example.infrastructure.web

@RestController
class UserController(
  private val createUser: CreateUserUseCase,
  private val mapper: UserMapper
) {
  @PostMapping("/users")
  suspend fun create(@RequestBody dto: CreateUserDto): ResponseEntity<Map<String, Long>> {
    val cmd = mapper.toCommand(dto)
    val id = createUser.create(cmd)
    return ResponseEntity.status(HttpStatus.CREATED).body(mapOf("id" to id))
  }
}
```

Mapper (infrastructure):

```kotlin
package com.example.infrastructure.web

class UserMapper {
  fun toCommand(dto: CreateUserDto) = CreateUserCommand(dto.name, dto.email)
}
```

Persistence adapter implements `UserRepositoryPort` and maps between DB entities and domain entities.

---

## Reasoning Checklist (agent must return before code)

Before producing code the agent must provide a short checklist containing:

1. Affected layer(s) (infrastructure/application/domain)
2. Chosen abstractions and DTOs/entities
3. Design validation vs. rules above (list exceptions)
4. Ports affected (inbound/outbound) and mapper responsibilities
5. Dependency validation (confirm no inward-dependency violations)

If a request violates these rules, explain why and propose alternatives.

---

## CI / Automation (suggested)

- Use JDK 21 and Gradle Kotlin DSL in CI
- Run `./gradlew build` and `./gradlew detekt ktlintCheck test`
- Optional jobs: integration tests with Testcontainers and architecture checks (ArchUnit)

Example workflow snippet:

```yaml
name: Kotlin CI
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

## Architecture verification (optional)

- Recommend ArchUnit or equivalent checks to assert dependency rules. ArchUnit (Java) works for Kotlin bytecode.

Example (Java/Kotlin test file):

```java
@AnalyzeClasses(packages = "com.example")
class ArchitectureTest {
  @ArchTest
  static final ArchRule domain_should_not_depend_on_infrastructure =
    classes().that().resideInAPackage("..domain..")
      .should().onlyDependOnClassesThat().resideInAnyPackage("..domain..", "java..", "kotlin..", "org.slf4j..");
}
```

---

## Notes & Exceptions

- Prefer coroutines in application and infrastructure layers for asynchronous I/O
- Keep transactional boundaries in application/service layer, not in controllers
- Document any exception to method-length or immutability rules in PR description

---

## Change log

- 0.1.0: Initial Kotlin backend skill (ports, examples, CI suggestions)
