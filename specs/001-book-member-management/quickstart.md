# Quickstart: Book and Member Management

## Prerequisites

- JDK 17
- The repository's Gradle wrapper

The existing project declares Spring Boot, Spring Data JPA, Bean Validation, Flyway, H2,
Jackson Kotlin support, and JUnit 5 test dependencies.

## Run the test suite

From the repository root on Windows:

```powershell
.\gradlew.bat test
```

Expected result: build succeeds and all service, controller, repository, and application
tests pass.

## Start the application

```powershell
.\gradlew.bat bootRun
```

Flyway creates the in-memory H2 schema on startup. The database is ephemeral and resets when
the application process ends.

## Validate the book flow

Create a book:

```powershell
curl.exe -i -X POST http://localhost:8080/v1/books `
  -H "Content-Type: application/json" `
  -d '{"title":"The Left Hand of Darkness","author":"Ursula K. Le Guin","isbn":"978-0-441-47812-5","copies":3}'
```

Expected: `201 Created`, a generated `id`, and the submitted fields. Repeat with
`isbn` equal to `9780441478125`; expected: `409 Conflict` identifying `isbn`.

Use the returned identifier to request `GET /v1/books/{id}`; expected: `200 OK` with that
book. `GET /v1/books` returns the complete collection, or `[]` when empty.

## Validate the member flow

Create a member:

```powershell
curl.exe -i -X POST http://localhost:8080/v1/members `
  -H "Content-Type: application/json" `
  -d '{"name":"Ada Lovelace","email":"ada@example.com"}'
```

Expected: `201 Created`, a generated `id`, and a system-assigned `registrationDate`.
Repeat with `ADA@example.com`; expected: `409 Conflict` identifying `email`.

Use the returned identifier to request `GET /v1/members/{id}`; expected: `200 OK` with that
member. `GET /v1/members` returns the complete register, or `[]` when empty.

## Validate error cases

- Post a book with a blank title, author, or ISBN, or a negative `copies` value: expect
  `400 Bad Request` and an `errors` entry for each invalid field.
- Post a book with `copies: 0`: expect `201 Created`.
- Post a member with a blank name or invalid email: expect `400 Bad Request` with
  field-specific errors.
- Request a nonexistent numeric identifier from either resource: expect `404 Not Found`
  with a resource-specific `ProblemDetail`.

See [the HTTP contract](contracts/http-api.md) and [the data model](data-model.md) for full
field and response requirements.
