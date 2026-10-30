<!--
Sync Impact Report
Version change: not established → 1.0.0 (initial constitution)
Modified principles: None; this is the initial adoption.
Added sections: Core Principles (7), Technology Baseline, Development Workflow.
Removed sections: None.
Follow-up TODOs: None.
-->
# library-lending Constitution

## Core Principles

### I. Supported Stack
The application MUST use Java 17 and Spring Boot, build through the Gradle wrapper, and use
in-memory H2. Stack changes require an approved constitution amendment.

### II. Layered Architecture
Application requests MUST follow the controller → service → repository layers. Dependencies
MUST respect this direction; controllers and services MUST NOT bypass the repository layer to
access persistence directly.

### III. Business Logic Boundaries
Business rules MUST reside in services or the domain model. Controllers MUST be limited to
HTTP handling, input binding and validation, delegation, and response mapping; they MUST NOT
implement business logic.

### IV. Automated Test Strategy
Tests MUST use JUnit 5. Services MUST have unit tests, controllers MUST be tested with
`@WebMvcTest`, and repositories MUST be tested with `@DataJpaTest`. A task is complete only
when `./gradlew test` passes.

### V. Deterministic Money and Time
Monetary values MUST use `BigDecimal`. Date-dependent behavior MUST use an injected
`java.time.Clock`; code MUST NOT call `LocalDate.now()` without that clock.

### VI. HTTP API Contract
The API MUST use JSON, return appropriate HTTP status codes, and represent errors consistently
with Spring `ProblemDetail`. Request input MUST be validated with Bean Validation.

### VII. Focused Changes and Naming
Changes MUST be small and limited to the task's scope. Class and method names MUST be in
English. New dependencies require explicit approval.

## Technology Baseline

The Java 17, Spring Boot, Gradle wrapper, and in-memory H2 stack is the project baseline.
Build and test commands MUST use the checked-in Gradle wrapper so they run with the project's
declared Gradle version.

## Development Workflow

One task MUST produce one commit. A task MUST NOT modify code outside its scope. Before a task
is considered complete, its applicable test suite MUST pass, including the full
`./gradlew test` quality gate.

## Governance

This constitution takes precedence over conflicting project practices. Amendments MUST be made
as documented changes to this file and reviewed and approved before they take effect. Every
implementation change MUST be checked against these principles during planning and review;
non-compliant changes MUST be revised or explicitly approved through a constitution amendment.

Constitution versions follow semantic versioning: MAJOR for backward-incompatible governance
changes or principle removals/redefinitions, MINOR for new principles or materially expanded
requirements, and PATCH for clarifications and non-semantic wording changes. The version and
amendment date MUST be updated with each approved amendment.

**Version**: 1.0.0 | **Ratified**: 2026-10-30 | **Last Amended**: 2026-10-30
