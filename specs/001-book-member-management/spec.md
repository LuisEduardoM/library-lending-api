# Feature Specification: Book and Member Management

**Feature Branch**: `001-book-member-management`

**Created**: 2026-10-30

**Status**: Draft

**Input**: User description: "Books and members management for the library. The library needs a catalog of books and a register of members before loans can exist in later features. Books have a title, an author, a unique ISBN and a number of copies (zero or more). Members have a name, a unique email and a registration date set by the system when the member is created. Registering a book or member returns the created record with its identifier. Duplicate ISBNs or emails are rejected with a field-specific error. Missing required fields, blank titles or names, invalid email formats and negative copy counts are rejected with a clear error for each invalid field. Books and members can be retrieved by identifier or listed. Loans, fines, reservations, updates, deletion, search, filters, pagination, and authentication are out of scope."

## Clarifications

### Session 2026-10-30

- Q: Should the system treat ISBNs with different spacing or hyphenation as the same ISBN? → A: Ignore spaces and hyphens when comparing ISBNs; preserve the submitted ISBN for display.
- Q: Should a book be rejected when its author or ISBN contains only whitespace? → A: Reject whitespace-only author and ISBN values.
- Q: Should member emails that differ only by letter casing be treated as duplicates? → A: Ignore letter casing when checking email uniqueness.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Manage the book catalog (Priority: P1)

A library staff member registers books so the library has an accurate catalog with identifiers
and copy counts. They can retrieve a book by identifier or see all registered books.

**Why this priority**: A usable catalog is a prerequisite for later lending features and is a
primary library record.

**Independent Test**: Register a book, verify the returned identifier and book details, then
retrieve it by that identifier and list the catalog.

**Acceptance Scenarios**:

1. **Given** valid book details and no book with that ISBN, **When** a staff member registers
   the book, **Then** the created book is returned with its identifier, title, author, ISBN,
   and copy count.
2. **Given** a registered book, **When** a staff member retrieves it by its identifier,
   **Then** the matching book and its details are returned.
3. **Given** multiple registered books, **When** a staff member lists books, **Then** every
   registered book is included.
4. **Given** a book with the same ISBN is already registered, **When** a staff member tries
   to register another book with that ISBN, **Then** registration is rejected with a clear
   error identifying the ISBN field.
5. **Given** an identifier that matches no book, **When** a staff member retrieves that
   identifier, **Then** a clear book-not-found error is returned.

---

### User Story 2 - Register and find members (Priority: P1)

A library staff member registers each person as a member and can later retrieve an individual
member or list the register.

**Why this priority**: The member register is required before people can participate in later
lending features.

**Independent Test**: Register a member, verify the returned identifier and system-assigned
registration date, then retrieve that member and list the register.

**Acceptance Scenarios**:

1. **Given** a name and a valid email address not used by another member, **When** a staff
   member registers the member, **Then** the created member is returned with an identifier,
   name, email, and a registration date assigned by the system.
2. **Given** a registered member, **When** a staff member retrieves them by identifier,
   **Then** the matching member and their details are returned.
3. **Given** multiple registered members, **When** a staff member lists members, **Then**
   every registered member is included.
4. **Given** a member with the same email is already registered, **When** a staff member
   tries to register another member with that email, **Then** registration is rejected with
   a clear error identifying the email field.
5. **Given** an identifier that matches no member, **When** a staff member retrieves that
   identifier, **Then** a clear member-not-found error is returned.

---

### User Story 3 - Receive clear validation errors (Priority: P2)

A library staff member receives field-specific feedback when book or member information is
missing or invalid, so they can correct the entry.

**Why this priority**: Clear validation feedback prevents invalid records and makes both
registration workflows usable.

**Independent Test**: Attempt registrations with each required field missing or invalid and
verify that every invalid field is identified in the rejection.

**Acceptance Scenarios**:

1. **Given** a book registration with one or more missing required fields, **When** submitted,
   **Then** it is rejected and every missing field is identified.
2. **Given** a blank or whitespace-only book title, **When** submitted, **Then** it is
   rejected with an error identifying the title field.
3. **Given** a book registration with a negative copy count, **When** submitted, **Then** it
   is rejected with an error identifying the copies field.
4. **Given** a book registration with zero copies, **When** submitted, **Then** it is
   accepted.
5. **Given** a member registration with one or more missing required fields, **When**
   submitted, **Then** it is rejected and every missing field is identified.
6. **Given** a blank or whitespace-only member name, **When** submitted, **Then** it is
   rejected with an error identifying the name field.
7. **Given** a member registration with an invalid email format, **When** submitted,
   **Then** it is rejected with an error identifying the email field.

### Edge Cases

- A copy count of zero is valid; any negative copy count is invalid.
- A value containing only whitespace is blank for the title and name fields.
- A value containing only whitespace is also invalid for the author and ISBN fields.
- A submitted ISBN or email that conflicts with an existing record is rejected without
  creating a second record.
- ISBN uniqueness ignores spaces and hyphens in the submitted value; the submitted ISBN
  remains unchanged when returned or listed.
- Email uniqueness ignores letter casing; an email differing only by case conflicts with
  the existing member.
- A lookup for an identifier that does not exist returns a not-found error specific to the
  requested record type.
- A failed registration does not create a partial book or member record.
- Listing books or members when none have been registered returns an empty list.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The system MUST allow a staff member to register a book with a title, author,
  ISBN, and copy count.
- **FR-002**: The system MUST assign each registered book an identifier and return the
  identifier and submitted book details after successful registration.
- **FR-003**: The system MUST reject a book registration when its ISBN conflicts with an
  existing book after ignoring spaces and hyphens, and identify ISBN as the conflicting
  field. The system MUST preserve the submitted ISBN value when displaying the book.
- **FR-004**: The system MUST reject book registration when any required field is missing,
  the title, author, or ISBN is blank, or the copy count is negative, identifying every
  invalid field in the rejected registration. A field containing only whitespace is blank.
- **FR-005**: The system MUST allow a copy count of zero.
- **FR-006**: The system MUST allow a staff member to retrieve a book by identifier and
  return a clear not-found error when no book matches.
- **FR-007**: The system MUST allow a staff member to list all registered books, returning
  an empty list when no books exist.
- **FR-008**: The system MUST allow a staff member to register a member with a name and
  email address.
- **FR-009**: The system MUST assign each registered member an identifier and system-set
  registration date, and return the identifier and member details after successful
  registration.
- **FR-010**: The system MUST reject a member registration when its email conflicts with an
  existing member regardless of letter casing, and identify email as the conflicting field.
- **FR-011**: The system MUST reject member registration when any required field is missing,
  the name is blank, or the email format is invalid, identifying every invalid field in the
  rejected registration.
- **FR-012**: The system MUST allow a staff member to retrieve a member by identifier and
  return a clear not-found error when no member matches.
- **FR-013**: The system MUST allow a staff member to list all registered members, returning
  an empty list when no members exist.
- **FR-014**: A rejected registration MUST NOT create a partial or otherwise invalid record.
- **FR-015**: The system MUST NOT provide loan, fine, reservation, update, delete, search,
  filter, pagination, or authentication capabilities as part of this feature.

### Key Entities

- **Book**: A library catalog entry with an identifier, title, author, unique ISBN, and
  non-negative number of copies.
- **Member**: A registered library member with an identifier, name, unique email address,
  and a registration date assigned when the member is created.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: In acceptance testing, 100% of valid book registrations return an identifier
  and the submitted book details.
- **SC-002**: In acceptance testing, 100% of valid member registrations return an identifier,
  the submitted member details, and a system-assigned registration date.
- **SC-003**: Every duplicate ISBN and duplicate email attempt is rejected, and the error
  identifies the conflicting field.
- **SC-004**: Every tested missing or invalid field is rejected with an error identifying
  that field; zero copies are accepted and negative copies are rejected.
- **SC-005**: Every registered book and member can be retrieved by its identifier and appears
  in its respective complete listing.
- **SC-006**: Staff can distinguish a missing book from a missing member through a clear
  not-found error.

## Assumptions

- The staff member is authorized to perform all included catalog and member-register
  operations; authentication and permissions are outside this feature.
- ISBN comparison ignores spaces and hyphens, but ISBN syntax and validity checks are not
  defined by this feature.
- Email uniqueness applies without regard to letter casing; email validation checks that
  the submitted value has a valid email format.
- A member's registration date is the calendar date on which the system creates the member;
  callers do not set that date.
- Listing returns every registered record because pagination and filtering are out of scope.
- Author is required and must not be missing or blank; no additional author-format rules
  are imposed. ISBN is also required and must not be blank.
