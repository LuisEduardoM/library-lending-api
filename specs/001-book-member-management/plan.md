# Implementation Plan: Book and Member Management

**Branch**: `001-book-member-management` | **Date**: 2026-10-30 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from `specs/001-book-member-management/spec.md`

## Summary

Provide JSON REST resources to register, retrieve, and list books and members. Use the existing
Kotlin/JVM 17, Spring Boot, Gradle, H2, and Flyway baseline; organize code by feature and keep
HTTP DTOs separate from JPA entities. Enforce ISBN and email uniqueness using normalized
database keys, with the original submitted values preserved for responses. See
[research.md](research.md) for decisions and tradeoffs.

## Technical Context

**Language/Version**: Kotlin on JVM 17.

**Primary Dependencies**: Spring Boot 4, Spring MVC, Spring Data JPA, Bean Validation,
Jackson Kotlin module, Flyway; already declared in `build.gradle.kts`. No new dependencies.

**Storage**: In-memory H2; Flyway versioned SQL migrations own the schema.

**Testing**: JUnit 5 through the Gradle wrapper; service unit tests, `@WebMvcTest` controller
tests, and `@DataJpaTest` repository tests. The required quality gate is
`.\gradlew.bat test` on Windows (`./gradlew test` on POSIX).

**Target Platform**: JVM 17 server application.

**Project Type**: JSON REST web service.

**Performance Goals**: No numeric performance target is defined for this feature.

**Constraints**: Feature packages under `mvp.library_lending_api`; one controller, service,
repository, DTO set, and dedicated mapper per feature. Constructor injection only. One
`@RestControllerAdvice` returns `ProblemDetail` for standardized errors. No updates, deletes,
search, filters, pagination, authentication, or loan-related behavior.

**Scale/Scope**: Complete in-memory book catalog and member register for the current process;
no volume target or persistent production database is specified.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Gate | Result |
|-----------|------|--------|
| Supported Stack | Kotlin/JVM 17, Spring Boot, Gradle wrapper, in-memory H2 | PASS |
| Layered Architecture | Feature packages; controller → service → repository | PASS |
| Business Logic Boundaries | Services/domain own rules; constructor injection; thin controllers | PASS |
| Automated Test Strategy | JUnit 5 service, MVC slice, and JPA slice tests; full wrapper test gate | PASS |
| Deterministic Money and Time | No money rules; registration date supplied by injected `Clock` | PASS |
| HTTP API Contract | JSON, DTOs separate from entities, dedicated mappers, Bean Validation, one advice with `ProblemDetail` | PASS |
| Focused Changes and Naming | Scope limited to create/read/list; simple patterns and no extra dependencies | PASS |
| Development Workflow | Feature-scoped work, one task per commit, tests required | PASS |

The user requested Kotlin "DTO records"; Kotlin-native `data class` DTOs are selected because
the constitution explicitly requires them and Java record syntax is not Kotlin's DTO form.
No governance exception is required.

## Phase 0: Research

Research decisions and alternatives are recorded in [research.md](research.md). The design
uses database unique constraints as the authority for conflict prevention, in addition to
service-level checks for readable field-specific duplicate errors.

## Phase 1: Design & Contracts

- Entity and validation design: [data-model.md](data-model.md)
- HTTP interface: [contracts/http-api.md](contracts/http-api.md)
- Runnable validation scenarios: [quickstart.md](quickstart.md)

## Project Structure

### Documentation (this feature)

```text
specs/001-book-member-management/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── http-api.md
├── checklists/
│   └── requirements.md
└── tasks.md
```

### Source Code (repository root)

```text
src/
├── main/
│   ├── kotlin/mvp/library_lending_api/
│   │   ├── common/exception/
│   │   │   ├── ApiExceptionHandler.kt
│   │   │   └── domain exceptions
│   │   ├── book/
│   │   │   ├── controller/
│   │   │   ├── service/
│   │   │   ├── repository/
│   │   │   ├── dto/
│   │   │   ├── mapper/
│   │   │   └── entity/
│   │   ├── member/
│   │   │   ├── controller/
│   │   │   ├── service/
│   │   │   ├── repository/
│   │   │   ├── dto/
│   │   │   ├── mapper/
│   │   │   └── entity/
│   │   └── LibraryLendingApiApplication.kt
│   └── resources/
│       ├── application.properties
│       └── db/migration/V1__create_books_and_members.sql
└── test/
    └── kotlin/mvp/library_lending_api/
        ├── book/
        │   ├── controller/
        │   ├── service/
        │   └── repository/
        ├── member/
        │   ├── controller/
        │   ├── service/
        │   └── repository/
        └── common/exception/
```

**Structure Decision**: One Gradle Spring Boot application matching the existing package root.
Book and member features are independent slices; shared exception handling lives in a small
`common.exception` package. Keep the existing application entry point and dependency set.

## Design Patterns

- **Repository (Spring Data JPA)**: use one Spring Data repository per aggregate to isolate
  persistence and provide the database-backed lookup/list operations this feature needs.
  Do not add a second repository abstraction or custom implementation.
- **No Strategy, State, or Factory pattern**: neither varying policies nor lifecycle
  transitions nor complex validated object construction exist in this scope. Dedicated DTO
  mappers are required by the constitution, but no additional pattern is introduced for them.

## Constitution Check (Post-Design)

| Gate | Result |
|------|--------|
| Required stack and existing dependencies | PASS |
| Feature boundaries, layering, constructor injection | PASS |
| Business rules kept out of controllers | PASS |
| JUnit 5 test slices and Gradle wrapper gate | PASS |
| Injected `Clock` for registration date | PASS |
| Kotlin `data class` DTOs, dedicated mappers, Bean Validation, one `ProblemDetail` advice | PASS |
| Justified patterns; no unnecessary abstraction or dependency | PASS |
| Feature-scoped scope and workflow | PASS |

No constitution violations remain.

## Complexity Tracking

No constitution violations requiring justification.
