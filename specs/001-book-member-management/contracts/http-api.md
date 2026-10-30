# HTTP API Contract

All endpoints use JSON. The base resources are `/v1/books` and `/v1/members`.
Identifiers are generated positive numeric values. Collection reads return every record and
are not paginated.

## Books

### `POST /v1/books`

Create a book.

Request:

```json
{
  "title": "The Left Hand of Darkness",
  "author": "Ursula K. Le Guin",
  "isbn": "978-0-441-47812-5",
  "copies": 3
}
```

`title`, `author`, and `isbn` are required and non-blank. `copies` is required and must be
zero or greater. A successful response is `201 Created` with the created record, including
`id` and the submitted ISBN representation. A duplicate ISBN after ignoring spaces and
hyphens returns `409 Conflict` and identifies `isbn`.

### `GET /v1/books/{id}`

Return the matching book with `200 OK`. An unknown identifier returns `404 Not Found`.

### `GET /v1/books`

Return all books with `200 OK`; return an empty JSON array when the catalog is empty.

## Members

### `POST /v1/members`

Create a member.

Request:

```json
{
  "name": "Ada Lovelace",
  "email": "ada@example.com"
}
```

`name` and `email` are required; name must be non-blank and email must have a valid format.
The caller cannot provide `id` or `registrationDate`. A successful response is `201 Created`
with the created record, including `id` and system-assigned `registrationDate`. A duplicate
email regardless of letter casing returns `409 Conflict` and identifies `email`.

### `GET /v1/members/{id}`

Return the matching member with `200 OK`. An unknown identifier returns `404 Not Found`.

### `GET /v1/members`

Return all members with `200 OK`; return an empty JSON array when the register is empty.

## Error responses

Errors use Spring `ProblemDetail` with JSON fields `type`, `title`, `status`, `detail`, and
`instance`. Validation and duplicate-conflict responses include an `errors` extension array
containing one `{ "field": "...", "message": "..." }` entry for each invalid or conflicting
field. Validation returns `400 Bad Request`, missing resources return `404 Not Found`, and
duplicate normalized keys return `409 Conflict`. Messages identify the field and the
corrective condition without exposing persistence details.

The single application-wide `@RestControllerAdvice` is responsible for translating domain,
validation, and uniqueness-conflict failures into this response format.
