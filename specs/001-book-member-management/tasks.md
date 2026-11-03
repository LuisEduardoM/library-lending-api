---
description: "Commit-sized implementation tasks for Book and Member Management"
---

# Tasks: Book and Member Management

**Input**: Design documents in `specs/001-book-member-management/`

**Quality gate**: Every task includes focused tests, is intended to produce one commit, and is complete only when `./gradlew test` passes. This follows the repository's Kotlin/Gradle build and constitution.

## Phase 1: Setup

**Purpose**: Configure the existing Spring Boot project for Flyway-owned schema and JPA validation.

- [ ] T001 Configure `spring.jpa.hibernate.ddl-auto=validate` in `src/main/resources/application.properties`; extend `src/test/kotlin/mvp/library_lending_api/LibraryLendingApiApplicationTests.kt` to verify the application context starts, and finish only when `./gradlew test` passes.

## Phase 2: Foundational

**Purpose**: Establish the shared schema, API error contract, and injectable system clock before feature work.

- [ ] T002 [P] Add `src/main/resources/db/migration/V1__create_books_and_members.sql` with the Book and Member tables, required columns, generated numeric identifiers, non-negative copies check, and named unique constraints for normalized keys; add migration assertions in `src/test/kotlin/mvp/library_lending_api/common/DatabaseMigrationTests.kt`, and finish only when `./gradlew test` passes.
- [ ] T003 [P] Define field-specific duplicate and resource-not-found exceptions and map them, validation failures, and named database uniqueness violations to the specified `ProblemDetail` responses in `src/main/kotlin/mvp/library_lending_api/common/exception/ApiExceptions.kt` and `ApiExceptionHandler.kt`; test status codes and error fields in `src/test/kotlin/mvp/library_lending_api/common/exception/ApiExceptionHandlerTests.kt`, and finish only when `./gradlew test` passes.
- [ ] T004 [P] Provide a `Clock` bean through `src/main/kotlin/mvp/library_lending_api/common/config/ClockConfiguration.kt`; verify it can be injected in `src/test/kotlin/mvp/library_lending_api/common/config/ClockConfigurationTests.kt`, and finish only when `./gradlew test` passes.

**Checkpoint**: The schema, shared error handling, and clock are ready for both independent user stories.

## Phase 3: User Story 1 - Manage the book catalog (Priority: P1)

**Goal**: Register books, retrieve them by identifier, and list the complete catalog.

**Independent Test**: Register a valid book and verify its returned identifier and submitted details; retrieve it by that identifier and list all books. Verify duplicate normalized ISBN conflicts, unknown identifiers, and an empty catalog.

- [ ] T005 [P] [US1] Create the Book JPA entity and repository in `src/main/kotlin/mvp/library_lending_api/book/entity/Book.kt` and `src/main/kotlin/mvp/library_lending_api/book/repository/BookRepository.kt`. Preserve these data-model constraints: `id` is a “Generated numeric identifier” and “Primary key; assigned on successful creation”; `title` and `author` are “Required; not blank, including whitespace-only values”; `isbn` is “Required; not blank” and “preserved for responses”; `isbnKey` is “Required and unique; derived by removing spaces and hyphens from isbn”; `copies` is “Required; integer greater than or equal to zero.” Test persistence and normalized-key uniqueness in `src/test/kotlin/mvp/library_lending_api/book/repository/BookRepositoryTests.kt`, and finish only when `./gradlew test` passes.
- [ ] T006 [US1] Implement book request/response DTOs, a dedicated mapper, and the `BookService` interface with its implementation for registration, identifier lookup, and complete listing in `src/main/kotlin/mvp/library_lending_api/book/dto/`, `mapper/BookMapper.kt`, and `service/`. Test creation, submitted ISBN preservation, zero copies, duplicate ISBN normalization, missing-book behavior, and listing in `src/test/kotlin/mvp/library_lending_api/book/service/BookServiceTests.kt`, and finish only when `./gradlew test` passes.
- [ ] T007 [US1] Implement `POST /v1/books`, `GET /v1/books/{id}`, and `GET /v1/books` in `src/main/kotlin/mvp/library_lending_api/book/controller/BookController.kt`; test JSON responses, `201`/`200` statuses, duplicate and not-found errors, and empty-list behavior with `@WebMvcTest` in `src/test/kotlin/mvp/library_lending_api/book/controller/BookControllerTests.kt`, and finish only when `./gradlew test` passes.

## Phase 4: User Story 2 - Register and find members (Priority: P1)

**Goal**: Register members with a system-assigned registration date, retrieve them by identifier, and list the complete register.

**Independent Test**: Register a valid member and verify its returned identifier, submitted details, and system-assigned date; retrieve it by that identifier and list all members. Verify case-insensitive duplicate email conflicts, unknown identifiers, and an empty register.

- [ ] T008 [P] [US2] Create the Member JPA entity and repository in `src/main/kotlin/mvp/library_lending_api/member/entity/Member.kt` and `src/main/kotlin/mvp/library_lending_api/member/repository/MemberRepository.kt`. Preserve these data-model constraints: `id` is a “Generated numeric identifier” and “Primary key; assigned on successful creation”; `name` is “Required; not blank, including whitespace-only values”; `email` is “Required; valid email format” and “preserved for responses”; `emailKey` is “Required and unique; derived using locale-independent lowercase”; `registrationDate` is “Required; assigned on creation from an injected Clock; never caller-supplied.” Test persistence and normalized-key uniqueness in `src/test/kotlin/mvp/library_lending_api/member/repository/MemberRepositoryTests.kt`, and finish only when `./gradlew test` passes.
- [ ] T009 [US2] Implement member request/response DTOs, a dedicated mapper, and the `MemberService` interface with its implementation for registration, identifier lookup, and complete listing in `src/main/kotlin/mvp/library_lending_api/member/dto/`, `mapper/MemberMapper.kt`, and `service/`; derive the registration date from the injected `Clock`. Test creation, date assignment, submitted email preservation, case-insensitive duplicates, missing-member behavior, and listing in `src/test/kotlin/mvp/library_lending_api/member/service/MemberServiceTests.kt`, and finish only when `./gradlew test` passes.
- [ ] T010 [US2] Implement `POST /v1/members`, `GET /v1/members/{id}`, and `GET /v1/members` in `src/main/kotlin/mvp/library_lending_api/member/controller/MemberController.kt`; test JSON responses, `201`/`200` statuses, duplicate and not-found errors, and empty-list behavior with `@WebMvcTest` in `src/test/kotlin/mvp/library_lending_api/member/controller/MemberControllerTests.kt`, and finish only when `./gradlew test` passes.

## Phase 5: User Story 3 - Receive clear validation errors (Priority: P2)

**Goal**: Reject missing or invalid book/member fields and identify every invalid field in the response.

**Independent Test**: Submit invalid book and member registrations with multiple invalid fields and confirm each is reported; confirm zero copies remain valid and negative copies are rejected.

- [ ] T011 [P] [US3] Add Bean Validation constraints to the book request DTO in `src/main/kotlin/mvp/library_lending_api/book/dto/`; preserve the data-model rule that title, author, and ISBN are “Required; not blank, including whitespace-only values” and copies are “Required; integer greater than or equal to zero.” Represent missing values so all invalid fields are reported together; add `@WebMvcTest` cases for missing/blank title, author, and ISBN, negative and zero copies, and multiple errors in `src/test/kotlin/mvp/library_lending_api/book/controller/BookControllerTests.kt`, and finish only when `./gradlew test` passes.
- [ ] T012 [P] [US3] Add Bean Validation constraints to the member request DTO in `src/main/kotlin/mvp/library_lending_api/member/dto/`; preserve the data-model rules that name is “Required; not blank, including whitespace-only values” and email is “Required; valid email format.” Represent missing values so all invalid fields are reported together; add `@WebMvcTest` cases for missing/blank name, missing/invalid email, and multiple errors in `src/test/kotlin/mvp/library_lending_api/member/controller/MemberControllerTests.kt`, and finish only when `./gradlew test` passes.

## Phase 6: Polish and Cross-Cutting Verification

**Purpose**: Verify the assembled application and migrations as an integrated service.

- [ ] T013 Add `@SpringBootTest` integration coverage for the Flyway schema and both book/member create-retrieve-list flows in `src/test/kotlin/mvp/library_lending_api/BookAndMemberIntegrationTests.kt`; follow the scenarios in `specs/001-book-member-management/quickstart.md` and finish only when `./gradlew test` passes.

## Dependencies and Execution Order

### Phase dependencies

- **Setup**: T001 establishes JPA schema validation.
- **Foundational**: T002-T004 depend on T001 and block feature stories.
- **User Stories 1 and 2**: Both depend on the foundational phase and are independent of each other.
- **User Story 3**: Depends on the corresponding controllers and DTOs from User Stories 1 and 2.
- **Polish**: T013 depends on all three user stories.

### User story dependencies

- **US1 (P1)**: After Phase 2; no dependency on another story.
- **US2 (P1)**: After Phase 2; no dependency on another story.
- **US3 (P2)**: After US1 and US2 because it validates both registration APIs.

### Parallel opportunities

- T002, T003, and T004 touch separate migration, exception, and configuration files after T001.
- US1 and US2 can be implemented in parallel; T005 and T008, T006 and T009, and T007 and T010 operate on separate feature files.
- T011 and T012 can be implemented in parallel after both registration endpoints exist.

## Parallel Example: User Stories 1 and 2

After Phase 2, separate contributors can implement each story independently:

```text
Contributor A: T005 -> T006 -> T007 (Book catalog)
Contributor B: T008 -> T009 -> T010 (Member register)
```

## Implementation Strategy

### MVP

Deliver User Story 1, the book catalog, as the first P1 increment after the shared foundation. Its independent test covers registration, retrieval, listing, duplicate ISBN handling, and not-found behavior.

### Incremental delivery

1. Complete setup and foundational tasks.
2. Deliver US1 and verify it independently.
3. Deliver US2 and verify it independently; it can also proceed in parallel with US1.
4. Deliver US3 validation coverage for both registration flows.
5. Complete T013 and run the complete `./gradlew test` gate.

## Notes

- Every checklist item includes focused tests and is scoped to one commit.
- Every task's completion gate is the full `./gradlew test` suite.
- `[P]` marks tasks that can proceed concurrently without editing the same files or depending on incomplete tasks.
- No Maven wrapper or Maven project is present; the required gate is the repository's Gradle wrapper.
