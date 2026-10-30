# Research: Book and Member Management

**Date**: 2026-10-30

## Kotlin DTOs and JPA entities

**Decision**: Use Kotlin `data class` types for request/response DTOs, separate from ordinary
JPA entity classes. Continue using the repository's existing Kotlin JPA and Spring compiler
plugins.

**Rationale**: Kotlin data classes provide concise value types suitable for API data. JPA
entities have identity, persistence lifecycle, and proxy behavior; ordinary entity classes
avoid data-class-generated equality and copying semantics over mutable or proxy-backed state.
The existing JPA plugin configuration already supplies the required no-arg/open support.
This follows the constitution's Kotlin `data class` rule while interpreting the user's
"records" as the DTO role rather than Java record syntax.

**Alternatives considered**:
- Java records: not Kotlin-native and conflict with the constitution's explicit DTO form.
- Data classes as JPA entities: concise, but generated equality/copy behavior is unsuitable
  for mutable persistence entities.

## Normalized uniqueness

**Decision**: Preserve submitted ISBN and email text for display and store separate canonical
keys with named, non-null unique constraints. ISBN key creation removes spaces and hyphens;
email key creation uses locale-independent lowercase conversion.

**Rationale**: The key columns enforce clarified uniqueness semantics at the database boundary
and prevent concurrent inserts from bypassing application-level checks. A service-level
pre-check can provide a direct domain error for common duplicate attempts; the unique
constraint remains authoritative. If a race still causes a constraint violation, translate
the persistence error at the request boundary to the correct field-specific conflict.

**Alternatives considered**:
- Exact-value uniqueness: violates the accepted ISBN formatting and email casing rules.
- Expression indexes or generated columns: compact, but more coupled to database-specific
  SQL behavior than explicit stored key columns.
- Service pre-check only: cannot guarantee uniqueness under concurrent requests.

## Schema ownership and migrations

**Decision**: Use ordered Flyway SQL migrations under `src/main/resources/db/migration`,
starting with a versioned migration that creates the book and member tables and their
normalized unique constraints. Configure JPA schema handling to validate, not create, the
schema.

**Rationale**: Flyway and H2 are already declared dependencies. Explicit migrations provide a
reviewable schema history and make repository tests exercise the same schema authority as the
application.

**Alternatives considered**:
- JPA-generated DDL: convenient for a prototype but duplicates schema ownership and does not
  provide an explicit migration history.
- Add another migration tool: unnecessary and would require dependency approval.

## API errors and concurrency

**Decision**: Use domain exceptions for duplicate and missing records, one
`@RestControllerAdvice` to return `ProblemDetail`, and translate database uniqueness failures
to field-specific conflict responses. Use `400 Bad Request` for validation errors, `404 Not
Found` for missing identifiers, and `409 Conflict` for duplicate keys.

**Rationale**: This preserves one API error contract and handles both predictable duplicates
and concurrent uniqueness races without broad catches or silent fallbacks.

**Alternatives considered**:
- Expose persistence exception details: leaks implementation-specific details and gives an
  unstable API response.
- Treat all failures as `400`: loses the distinction between invalid input, missing resource,
  and conflicting resource.

## Sources

- Repository baseline: `build.gradle.kts` (Kotlin JPA/Spring plugins, Spring Boot, Flyway,
  H2, validation, JPA, MVC, and test dependencies).
- Kotlin documentation: [No-arg compiler plugin](https://kotlinlang.org/docs/no-arg-plugin.html),
  [All-open compiler plugin](https://kotlinlang.org/docs/all-open-plugin.html), and
  [Data classes](https://kotlinlang.org/docs/data-classes.html).
- Spring Boot documentation:
  [Database initialization and Flyway](https://docs.spring.io/spring-boot/how-to/data-initialization.html#howto.data-initialization.migration-tool.flyway).
- Spring Framework documentation:
  [DAO exception translation](https://docs.spring.io/spring-framework/reference/data-access/dao.html#dao-exceptions)
  and [declarative transactions](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/annotations.html).
- Java 17 API:
  [`String.toLowerCase(Locale)`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/String.html#toLowerCase(java.util.Locale)).
- Database rationale:
  [PostgreSQL constraints reference](https://www.postgresql.org/docs/current/ddl-constraints.html)
  (used for general unique-constraint semantics; deployment DB remains H2 for this feature).
