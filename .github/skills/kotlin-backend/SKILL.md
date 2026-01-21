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
  - **Domain (Core):** Entities, value objects, domain services and domain exceptions — pure Kotlin, no framework types.
  - **Domain Use Cases:** Use-case interfaces and implementations. Orchestrate domain objects and call gateways.
  - **Domain Gateways:** Interfaces (outbound ports) for external resources like repositories, email services, etc.
  - **Infrastructure (Adapters):** Implementations of gateways (driven-adapters) and REST controllers (entry-points). Keep these implementations in `infrastructure` packages.
  - **Framework & Drivers:** Spring, Ktor, R2DBC, web servers — keep these at the outermost layer.

- Dependency rule: code dependencies must point inward. Inner layers (domain, use-cases) must not depend on adapters or framework types.

- Use cases & gateways conventions:
  - **Use cases (domain layer):** interfaces and implementations that orchestrate business logic (e.g., `CreateUserUseCase`, `CreateUserUseCaseImpl`).
  - **Gateways (domain layer):** interfaces for external resources (e.g., `UserRepository`, `EmailService`, `PasswordEncoder`).
  - Infrastructure adapters live in `infrastructure.drivenAdapters.*` packages and implement the gateway interfaces.
  - Entry points live in `infrastructure.entryPoints.*` packages and call use case interfaces.

Example package mapping:

- `com.example.domain.model` — entities, value objects, domain exceptions
- `com.example.domain.usecase` — use-case interfaces and implementations
- `com.example.domain.gateway` — outbound port interfaces (repositories, external services)
- `com.example.infrastructure.drivenAdapters` — gateway implementations (R2DBC, email, security)
- `com.example.infrastructure.providers` — technical providers (R2DBC entities, SMTP clients, BCrypt)
- `com.example.infrastructure.entryPoints` — web controllers, request/response DTOs and mappers
- `com.example.infrastructure.config` — bean configuration and dependency injection setup

**Note:** In this repository `infrastructure` packages contain the concrete adapter implementations (i.e., the "adapters"). The term "adapter" is still correct conceptually, but implementations should be grouped under `infrastructure.*` (drivenAdapters, providers, entryPoints, config, etc.).

Kotlin example: external service gateway + adapter

```kotlin
// domain/gateway/ExternalApiService.kt
package com.example.domain.gateway

interface ExternalApiService {
  suspend fun fetchData(id: String): ExternalData
}

// infrastructure/drivenAdapters/externalApi/ExternalApiServiceAdapter.kt
package com.example.infrastructure.drivenAdapters.externalApi

import com.example.domain.gateway.ExternalApiService
import org.springframework.stereotype.Component

@Component
class ExternalApiServiceAdapter(private val webClient: WebClient) : ExternalApiService {
  override suspend fun fetchData(id: String): ExternalData {
    val dto = webClient.get().uri("/api/data/{id}", id).retrieve().bodyToMono(RemoteDto::class.java).awaitFirst()
    return RemoteMapper.toExternalData(dto)
  }
}
```

- Minimal ASCII diagram:

  [Entry Points (REST)] -> [Use Case Interface] -> [Use Case Implementation] -> [Gateway Interface] -> [Driven Adapters (DB/External)]

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

## Examples (project structure)

### Hexagonal Architecture - Detailed Structure

```
user-management/
├── applications/
│   └── app-service/
│       └── src/main/kotlin/com/example/
│           └── MainApplication.kt
│
├── domain/
│   ├── model/
│   │   ├── User.kt
│   │   ├── Email.kt
│   │   └── exception/
│   │       ├── UserNotFoundException.kt
│   │       └── InvalidEmailException.kt
│   │
│   ├── usecase/
│   │   ├── CreateUserUseCase.kt
│   │   ├── GetUserUseCase.kt
│   │   ├── UpdateUserUseCase.kt
│   │   └── DeleteUserUseCase.kt
│   │
│   └── gateway/
│       ├── UserRepository.kt
│       ├── EmailService.kt
│       └── PasswordEncoder.kt
│
└── infrastructure/
    ├── driven-adapters/
    │   ├── r2dbc-repository/
    │   │   ├── UserRepositoryAdapter.kt
    │   │   └── mapper/
    │   │       └── UserMapper.kt
    │   │
    │   ├── email-sender/
    │   │   └── EmailServiceAdapter.kt
    │   │
    │   └── security/
    │       └── PasswordEncoderAdapter.kt
    │
    ├── providers/
    │   ├── r2dbc/
    │   │   ├── UserR2dbcRepository.kt
    │   │   ├── entities/
    │   │   │   └── UserEntity.kt
    │   │   └── config/
    │   │       └── R2dbcConfig.kt
    │   │
    │   ├── smtp/
    │   │   ├── SmtpClient.kt
    │   │   └── config/
    │   │       └── EmailConfig.kt
    │   │
    │   └── bcrypt/
    │       └── BCryptProvider.kt
    │
    ├── entry-points/
    │   └── api-rest/
    │       ├── UserController.kt
    │       ├── dto/
    │       │   ├── CreateUserRequest.kt
    │       │   ├── UpdateUserRequest.kt
    │       │   └── UserResponse.kt
    │       └── mapper/
    │           └── UserDtoMapper.kt
    │
    └── config/
        └── BeanConfiguration.kt
```

### Minimal project layout

```
src/main/kotlin/com/example
  /domain
    /model
    /usecase
    /gateway
  /infrastructure
    /drivenAdapters
    /providers
    /entryPoints
    /config
src/test/kotlin/com/example
```

Sample flow (entry-points -> use case -> gateway -> driven-adapters):

Use case interface (domain/usecase):

```kotlin
package com.example.domain.usecase

fun interface CreateUserUseCase {
  suspend fun create(command: CreateUserCommand): Long
}
```

Gateway interface (domain/gateway):

```kotlin
package com.example.domain.gateway

import com.example.domain.model.User

interface UserRepository {
  suspend fun save(user: User): User
  suspend fun findById(id: Long): User?
}
```

Use-case implementation (domain/usecase):

```kotlin
package com.example.domain.usecase

import com.example.domain.gateway.UserRepository
import com.example.domain.model.User

class CreateUserUseCaseImpl(
  private val repository: UserRepository
) : CreateUserUseCase {
  override suspend fun create(command: CreateUserCommand): Long {
    val user = User.create(command.name, command.email)
    val saved = repository.save(user)
    return saved.id
  }
}
```

REST Controller (infrastructure/entryPoints/apiRest) — calls use case, maps DTO->domain via mapper:

```kotlin
package com.example.infrastructure.entryPoints.apiRest

import com.example.domain.usecase.CreateUserUseCase
import org.springframework.http.HttpStatus
import org.springframework.http.ResponseEntity
import org.springframework.web.bind.annotation.*

@RestController
@RequestMapping("/api/users")
class UserController(
  private val createUser: CreateUserUseCase,
  private val mapper: UserDtoMapper
) {
  @PostMapping
  suspend fun create(@RequestBody dto: CreateUserRequest): ResponseEntity<Map<String, Long>> {
    val cmd = mapper.toCommand(dto)
    val id = createUser.create(cmd)
    return ResponseEntity.status(HttpStatus.CREATED).body(mapOf("id" to id))
  }
}
```

DTO Mapper (infrastructure/entryPoints/apiRest/mapper):

```kotlin
package com.example.infrastructure.entryPoints.apiRest.mapper

import com.example.domain.usecase.CreateUserCommand
import com.example.infrastructure.entryPoints.apiRest.dto.CreateUserRequest

class UserDtoMapper {
  fun toCommand(dto: CreateUserRequest) = CreateUserCommand(dto.name, dto.email)
}
```

Repository Adapter (infrastructure/drivenAdapters/r2dbcRepository) implements `UserRepository` gateway:

```kotlin
package com.example.infrastructure.drivenAdapters.r2dbcRepository

import com.example.domain.gateway.UserRepository
import com.example.domain.model.User
import com.example.infrastructure.providers.r2dbc.UserR2dbcRepository
import org.springframework.stereotype.Component

@Component
class UserRepositoryAdapter(
  private val r2dbcRepository: UserR2dbcRepository,
  private val mapper: UserMapper
) : UserRepository {
  override suspend fun save(user: User): User {
    val entity = mapper.toEntity(user)
    val saved = r2dbcRepository.save(entity)
    return mapper.toDomain(saved)
  }

  override suspend fun findById(id: Long): User? {
    return r2dbcRepository.findById(id)?.let { mapper.toDomain(it) }
  }
}
```

---

## Reasoning Checklist (agent must return before code)

Before producing code the agent must provide a short checklist containing:

1. Affected layer(s) (entry-point/use-case/domain-model/gateway/driven-adapter)
2. Chosen abstractions and DTOs/entities
3. Design validation vs. rules above (list exceptions)
4. Use cases and gateways affected, mapper responsibilities (DTO -> domain)
5. Dependency validation (confirm no inward-dependency violations)

Example response the agent must produce before code:

```
Layer: domain/usecase + infrastructure/drivenAdapters
Abstraction: CreateUserUseCase (interface + implementation) + UserRepositoryAdapter
Validation: Uses constructor injection, domain layer unchanged, function length <= 30
```

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

- Use ArchUnit to assert that package dependencies point inward. ArchUnit (Java) works for Kotlin bytecode. Example tests:

```java
@AnalyzeClasses(packages = "com.example")
class ArchitectureTest {

  @ArchTest
  static final ArchRule domain_should_not_depend_on_infrastructure =
    classes().that().resideInAPackage("..domain..")
      .should().onlyDependOnClassesThat()
      .resideInAnyPackage("..domain..", "java..", "kotlin..", "kotlinx..", "org.slf4j..");

  @ArchTest
  static final ArchRule use_cases_should_only_depend_on_domain_and_gateways =
    classes().that().resideInAPackage("..domain.usecase..")
      .should().onlyDependOnClassesThat()
      .resideInAnyPackage("..domain..", "java..", "kotlin..", "kotlinx..", "org.slf4j..");

  @ArchTest
  static final ArchRule driven_adapters_should_implement_gateways =
    classes().that().resideInAPackage("..infrastructure.drivenAdapters..")
      .should().implement(
        DescribedPredicate.describe("gateway interfaces",
          cls -> cls.getPackageName().contains("domain.gateway"))
      );
}
```

---

## Notes & Exceptions

- Prefer coroutines in use-case and infrastructure layers for asynchronous I/O
- Keep transactional boundaries in use-case layer (`domain.usecase`), not in controllers (`infrastructure.entryPoints`)
- Document any exception to function-length or immutability rules in PR description
- Use `@Transactional` sparingly and only at the use-case implementation level

---

## Change log

- 0.1.0: Initial Kotlin backend skill (ports, examples, CI suggestions)
