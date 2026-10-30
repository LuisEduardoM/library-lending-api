<!--
Sync Impact Report
Version change: 1.0.0 → 1.1.0 (MINOR: materially expanded project requirements)
Modified principles:
- Supported Stack → Supported Stack (Kotlin and explicit runtime/build requirements)
- Layered Architecture → Layered Architecture (feature-oriented packages and layer boundaries)
- Business Logic Boundaries → Business Logic Boundaries (constructor injection requirement)
- Automated Test Strategy → Automated Test Strategy (behavior-based names and Gradle gate)
- Deterministic Money and Time → Deterministic Money and Time (configurable business values)
- HTTP API Contract → HTTP API Contract (DTOs, mappers, validation, and centralized errors)
- Focused Changes and Naming → Focused Changes and Naming (readability and pattern guidance)
Added sections: None.
Removed sections: None.
Follow-up TODOs: None.
-->
# library-lending Constitution

## Core Principles

### I. Supported Stack
The application MUST use Kotlin on JVM 17, Spring Boot, the checked-in Gradle wrapper, and
in-memory H2. Stack changes require an approved constitution amendment.

### II. Layered Architecture
Code MUST be organized by feature (for example, `book`, `member`, and `loan`), with each
feature owning its controller, service, and repository. Dependencies MUST follow the
controller → service → repository direction.

### III. Business Logic Boundaries
Business rules MUST reside in services or the domain model. Controllers MUST be limited to
HTTP handling, input binding and validation, delegation, and response mapping; they MUST NOT
implement business logic. Dependencies MUST be injected through constructors; field
`@Autowired` injection is prohibited.

### IV. Automated Test Strategy
Tests MUST use JUnit 5. Services MUST have unit tests, controllers MUST be tested with
`@WebMvcTest`, and repositories MUST be tested with `@DataJpaTest`. Test names MUST describe
the behavior, such as `shouldRejectLoanWhenMemberHasThreeActiveLoans`. A task is complete
only when `./gradlew test` passes.

### V. Deterministic Money and Time
Monetary values MUST use `BigDecimal`. Date-dependent behavior MUST use an injected
`java.time.Clock`; code MUST NOT call `LocalDate.now()` directly. Business limits and values,
including loan limits, loan periods, and fine amounts, MUST be provided through configuration
and MUST NOT be embedded as unexplained literals in code.

### VI. HTTP API Contract
The API MUST use JSON and return appropriate HTTP status codes. API inputs and outputs MUST
use Kotlin `data class` DTOs separate from JPA entities, with conversion handled by dedicated
mappers. Inputs MUST be validated with Bean Validation. Business errors MUST use specific
domain exceptions and be translated centrally by one `@RestControllerAdvice` into standardized
Spring `ProblemDetail` responses.

### VII. Focused Changes and Naming
Code MUST favor clear, intention-revealing names, short single-responsibility methods, and
early returns over nested conditionals. Comments MUST explain non-obvious rationale, not
restate code. Class and method names MUST be in English. Patterns MUST be used only to solve
real design needs: Spring Data Repository for persistence, Strategy for varying rules, State
for lifecycle behavior, and a factory or static factory for validated creation. The plan MUST
identify the pattern and its reason when one is introduced. Unnecessary interfaces or layers
without a second implementation or clear need are prohibited; when in doubt, choose the
simplest solution.

## Technology Baseline

Kotlin, JVM 17, Spring Boot, the checked-in Gradle wrapper, and in-memory H2 are the project
baseline. Build and test commands MUST use the checked-in Gradle wrapper.

## Development Workflow

Changes MUST be small and limited to the task's scope. One task MUST produce one commit, and
a task MUST NOT modify code outside its scope. New dependencies require explicit approval.
Before a task is considered complete, the full `./gradlew test` quality gate MUST pass.

## Governance

This constitution takes precedence over conflicting project practices. Amendments MUST be
documented in this file and reviewed and approved before they take effect. Every implementation
change MUST be checked against these principles during planning and review; non-compliant
changes MUST be revised or explicitly approved through a constitution amendment.

Constitution versions follow semantic versioning: MAJOR for backward-incompatible governance
changes or principle removals or redefinitions, MINOR for new principles or materially expanded
requirements, and PATCH for clarifications and non-semantic wording changes. The version and
amendment date MUST be updated with each approved amendment.

**Version**: 1.1.0 | **Ratified**: 2026-10-30 | **Last Amended**: 2026-10-30
