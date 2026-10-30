# Data Model: Book and Member Management

## Book

Represents a cataloged title.

| Field | Type / meaning | Constraints |
|-------|----------------|-------------|
| `id` | Generated numeric identifier | Primary key; assigned on successful creation |
| `title` | Submitted title | Required; not blank, including whitespace-only values |
| `author` | Submitted author | Required; not blank, including whitespace-only values |
| `isbn` | Submitted ISBN representation | Required; not blank; preserved for responses |
| `isbnKey` | Canonical uniqueness key | Required and unique; derived by removing spaces and hyphens from `isbn` |
| `copies` | Number of available copies recorded in the catalog | Required; integer greater than or equal to zero |

### Book validation and invariants

- Missing required fields are rejected and each invalid field is identified.
- Title, author, and ISBN cannot be blank or whitespace-only.
- Zero copies is valid; negative copies are invalid.
- ISBN uniqueness is evaluated using `isbnKey`; spaces and hyphens do not distinguish ISBNs.
- The original submitted ISBN is returned and listed without normalization.
- Failed registration does not persist a book.

## Member

Represents a person registered with the library.

| Field | Type / meaning | Constraints |
|-------|----------------|-------------|
| `id` | Generated numeric identifier | Primary key; assigned on successful creation |
| `name` | Submitted member name | Required; not blank, including whitespace-only values |
| `email` | Submitted email address | Required; valid email format; preserved for responses |
| `emailKey` | Canonical uniqueness key | Required and unique; derived using locale-independent lowercase |
| `registrationDate` | System-assigned calendar date | Required; assigned on creation from an injected `Clock`; never caller-supplied |

### Member validation and invariants

- Missing required fields are rejected and each invalid field is identified.
- Name cannot be blank or whitespace-only.
- Email must have a valid format.
- Email uniqueness ignores letter casing; original email text is preserved for responses.
- Failed registration does not persist a member.

## Relationships and lifecycle

- Books and members have no relationship in this feature.
- Both entities are created once and can be retrieved individually or listed.
- Updates, deletions, loans, fines, and reservations are out of scope.
- The schema is created by Flyway; JPA entities map the migrated tables.
- Tables use generated numeric primary keys and named unique constraints for normalized keys.
