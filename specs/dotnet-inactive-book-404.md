# Change specification: Return 404 for inactive books

## Metadata

- **Owner:** sinkay07
- **Status:** Approved
- **Source root:** `legacy-src/dotnet-bookcatalog/shared-legacy-app`
- **Related issue:** Not created

## Problem and outcomes

### Problem

`BooksController.Index` hides inactive books with `Where(b => b.IsActive)`, but
`Details`, `Edit` (GET and POST), `Delete`, and `DeleteConfirmed` load the book
with `db.Books.Find(id)` and never check `IsActive`. The seeded inactive book
"The Matrix: The Shooting Script" (`ApplicationDbContext.cs`, `SeedBooks`) is
therefore not listed but can still be viewed, edited, or deleted by URL.

### Desired outcomes

1. Inactive books are unreachable through every `Books` action, consistent with
   the list view.
2. The smoke script asserts this behavior so it becomes part of the contract
   that migrated applications must keep.

### Success measures

- GET `Details`, `Edit`, and `Delete` for the seeded inactive book return 404.
- Active-book behavior and the six-book list are unchanged.

## Functional requirements

| ID | Requirement | Evidence or rationale |
| --- | --- | --- |
| FR-1 | When a book is inactive, `Details`, `Edit` GET, and `Delete` GET shall return 404 (`HttpNotFound()`). | `src/BookCatalog.Web/Controllers/BooksController.cs:Details,Edit,Delete` |
| FR-2 | When the book loaded by `Edit` POST or `DeleteConfirmed` is inactive, the action shall return 404 and change no data. | Same file; `Edit` POST loads `existingBook` |
| FR-3 | When the id does not exist, `DeleteConfirmed` shall keep its current redirect to `Index`. | Preserves existing idempotent delete; changing it is out of scope |
| FR-4 | Active-book behavior for every action shall be unchanged. | Behavior preservation |

## Constraints and out of scope

### Constraints

- **Allowed paths:**
    - `legacy-src/dotnet-bookcatalog/shared-legacy-app/src/BookCatalog.Web/Controllers/BooksController.cs`
    - `scripts/validation/Test-BookCatalogSmoke.ps1`
    - `specs/dotnet-inactive-book-404.md`
- **Primary validation command:** `msbuild BookCatalog.sln -restore -p:Configuration=Release`
- **Compatibility:** Routes, models, views, database schema, seed data, and the
  active-book flows are unchanged.
- **Dependencies:** Add no package or configuration change.
- **Data and security:** Use only the synthetic seed data. Commit no secrets.

### Exclusions (out of scope)

- Schema or data changes, new UI, soft-delete redesign, and .NET migration work.
- Changing the missing-id behavior of `DeleteConfirmed`.
- Editing the facilitator reference solution, lab documents, or CI workflows.

## Edge cases

| Type | Condition | Expected behavior |
| --- | --- | --- |
| Normal | Active book | `Details`, `Edit`, and `Delete` work as today. |
| Failure | Missing id | GET actions return 404; `DeleteConfirmed` still redirects to `Index`. |
| Boundary | Active book saved with `IsActive = false` through `Edit` | It disappears from `Index`, and every action then returns 404. This is the accepted consequence, not a bug. |
| Compatibility | Inactive book previously reactivated through the UI | Not possible after this change. This is the accepted consequence. |

## Acceptance criteria

### AC-1: Inactive book GET actions return 404

- **Given** the seeded inactive "The Matrix: The Shooting Script"
- **When** a client requests `Details`, `Edit`, or `Delete` for it
- **Then** each response is 404
- **Verification:** Smoke script assertions, run only against a net10.0 target.
  For the legacy app, code review plus the primary validation command.

### AC-2: Inactive book POST actions return 404

- **Given** an inactive book
- **When** a client POSTs `Edit` or `DeleteConfirmed` for it
- **Then** the response is 404 and no data changes
- **Verification:** Code review and the primary validation command only. The
  smoke script does not cover these paths.

### AC-3: Missing id is unchanged

- **Given** an id that does not exist
- **When** a client requests `Details`, `Edit`, or `Delete`, or POSTs `DeleteConfirmed`
- **Then** the GET actions return 404 and `DeleteConfirmed` redirects to `Index`
- **Verification:** Existing smoke assertions (404 for a missing id); code review
  for `DeleteConfirmed`.

### AC-4: Active books and the list are unchanged

- **Given** a fresh synthetic database
- **When** a client lists and exercises active books
- **Then** `Index` shows exactly six books, and `Details`, `Edit`, `Delete`,
  and `DeleteConfirmed` work for an active book
- **Verification:** Existing smoke flow, run only against a net10.0 target.
  Otherwise code review.

### AC-5: Build remains valid

- **Given** the documented .NET Framework build prerequisites are installed
- **When** the primary validation command runs from the source root
- **Then** restore and build succeed
- **Verification:** Record the command, exit status, and concise result.

## Non-functional requirements

| ID | Quality | Testable requirement | Verification |
| --- | --- | --- | --- |
| NFR-1 | Maintainability | The diff touches only the allowed paths. | `git diff --name-only` |
| NFR-2 | Maintainability | No package, configuration, schema, or view change. | Diff review |

## Assumptions, dependencies, and risks

| Item | Type | Validation or mitigation |
| --- | --- | --- |
| The smoke script targets `bin\Release\net10.0` and cannot run against the .NET Framework 4.8 legacy build. Its only caller is `scripts/validation/validate-dotnet-target.ps1`. | Risk | The 404 behavior in the legacy app has no automated gate in this repository. Mark every smoke change "not run" unless a net10.0 build and LocalDB are available. |
| Changing the smoke script redefines the contract for migrated apps, so a migration that keeps the old behavior fails the target gate. The facilitator reference solution (`facilitator/reference-solutions/dotnet-modernization`, outside the allowed paths) may need a follow-up. | Risk | Reviewer confirmed this scope. Shared-contract work needs broader review before merge. Report a failing target gate; do not edit outside the allowed paths. |
| The existing smoke flow deactivates a book and then reactivates it through `Edit`. After this change that reactivation returns 404. | Dependency | Restructure the flow: edit an active book (Book A); deactivate a second book (Book B) and assert 404 for its GET actions; probe ids to confirm the seeded Matrix book returns 404. |
| The Matrix id is not exposed in the UI; it is seeded last, so likely id 7 on a fresh database. | Assumption | Probe ids from 1 to one past the highest listed id and assert that every id absent from `Index` returns 404. |
| Antiforgery-token reuse for POST checks is fragile and untestable here. | Risk | Not attempted. AC-2 is verified by code review and build only. |
| Nothing in `scripts/modernization` depends on the smoke script's output text. | Assumption | Checked by glob and grep (`tests/Test-WorkflowModes.ps1`, `tests/Test-Modernization.ps1`). |

## Plan-mode review checkpoint

### Evidence

- **Current behavior:** `legacy-src/dotnet-bookcatalog/shared-legacy-app/src/BookCatalog.Web/Controllers/BooksController.cs`:
  `Index` filters on `IsActive` (line 15); `Details` (25), `Edit` GET (55) and
  POST (69), `Delete` (89), and `DeleteConfirmed` (101) use `db.Books.Find`
  with no `IsActive` check. `src/BookCatalog.Web/Models/ApplicationDbContext.cs`
  seeds the Matrix book with `IsActive = false`.
- **Existing tests:** `scripts/validation/Test-BookCatalogSmoke.ps1` (net10.0
  target only). It checks that the fresh list has six books and no Matrix row,
  and that missing ids and deleted books return 404. It never requests the
  inactive Matrix book directly.
- **Baseline validation evidence:** `msbuild BookCatalog.sln -restore -p:Configuration=Release`
  run from `legacy-src/dotnet-bookcatalog/shared-legacy-app` — exit 0;
  `BookCatalog.Web.dll` built; nothing to restore.

### Unresolved questions

- None.

### Boundary and validation review

- [x] Evidence citations were checked against the repository.
- [x] Allowed paths include all anticipated implementation and test changes and
      nothing unrelated.
- [x] The primary validation command is safe, non-interactive, and runnable
      from the stated source root.
- [x] Acceptance criteria cover the required normal, failure, boundary, and
      compatibility behavior.
- [x] Exclusions prevent unrelated modernization or cleanup.

### Stop conditions

Stop and request human guidance if:

- repository evidence contradicts this specification;
- satisfying a criterion requires a path outside the allowed paths;
- the baseline or primary validation fails for a reason caused by the proposed
  change, or the command is missing, destructive, or irreproducible;
- LocalDB or a net10.0 build is unavailable (record the check as "not run"; do
  not claim a pass);
- the facilitator reference solution or the CI target gate fails because of this
  contract change (report it; do not edit outside the allowed paths); or
- a secret, production resource, destructive action, dependency change, schema
  change, or public API change would be required.

### Human approval

- [x] I reviewed the evidence, unresolved questions, allowed paths, acceptance
      criteria, validation command, exclusions, risks, and stop conditions.
- **Reviewer:** sinkay07
- **Approval date:** 2026-11-23
- **Approved specification commit:** cd4cf52

Change **Status** to `Approved` only after the checkbox is selected and the
reviewer, date, and commit fields are complete. Planning approval authorizes
issue preparation, not implementation.
