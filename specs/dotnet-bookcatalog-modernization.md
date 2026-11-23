# Change specification: Modernize BookCatalog to .NET 10 ASP.NET Core MVC and EF Core

## Metadata

- **Owner:** sinkay07
- **Status:** Draft
- **Source root:** `legacy-src/dotnet-bookcatalog/shared-legacy-app`
- **Related issue:** not created

## Problem and outcomes

### Problem

BookCatalog is a disposable workshop sample on .NET Framework 4.8, ASP.NET MVC
5/System.Web, EF6 and LocalDB (`src/BookCatalog.Web/packages.config`,
`src/BookCatalog.Web/Web.config`). It cannot run on a supported cross-platform
runtime. The Modernize assessment
(`.github/modernize/assessment/bookcatalog/reports-20261123190524/repos/shared-legacy-app/report.md`)
reports 6 issues, 1 of them mandatory. The classic Release build passes with
MSBuild.

### Desired outcomes

1. A user sees the same catalog behavior from an ASP.NET Core MVC app on .NET 10.
2. The source root builds and runs on .NET 10 with EF Core SQL Server and has no
   System.Web, EF6 or `packages.config` references.
3. Preserved behavior is checked by a repeatable local script, not only by
   manual clicking.

### Success measures

- The primary validation command exits 0.
- Every acceptance criterion passes and the diff touches only allowed paths.

## Functional requirements

### Preserved behavior

| ID | Requirement | Evidence or rationale |
| --- | --- | --- |
| FR-1 | `/` and `/Books` show the catalog (default route Books/Index/{id?}). | `src/BookCatalog.Web/App_Start/RouteConfig.cs` |
| FR-2 | Index lists only active books, ordered by title. | `Controllers/BooksController.cs:Index` |
| FR-3 | Details, Create, Edit and Delete (GET and confirmed POST) work. An unknown id on Details, Edit or Delete returns 404. A confirmed delete of a missing id redirects to Index. | `Controllers/BooksController.cs` |
| FR-4 | Title is required (max 200). Author is required (max 100). ISBN max 13. PublishedYear is 1800-2100 or empty. Invalid input re-renders the form with errors and persists nothing. | `Models/Book.cs` |
| FR-5 | Create, Edit and Delete POSTs require an antiforgery token. A tokenless POST is rejected and changes nothing. | `Controllers/BooksController.cs` (`ValidateAntiForgeryToken`) |
| FR-6 | New books default to Active and CreatedDate is now. Edit updates only Title, Author, ISBN, PublishedYear and IsActive. | `Models/Book.cs`, `BooksController.Edit` |
| FR-7 | A fresh database seeds seven books (six active, one inactive "The Matrix: The Shooting Script"). Seeding is deterministic and runs only when the table is empty. | `Models/ApplicationDbContext.cs` |
| FR-8 | Saved changes survive an app restart. The database is not dropped or recreated at startup. | `README.md` |
| FR-9 | Index shows row number, Title, Author, ISBN, Year, status, View/Edit/Delete actions, a record count and an empty-state message. The classic look from `Content/Site.css` is kept. | `Views/Books/Index.cshtml` |

### Intentionally changed behavior

| ID | Change | Reason |
| --- | --- | --- |
| CH-1 | Host moves from IIS/System.Web to a single ASP.NET Core host. No side-by-side host. | Target direction |
| CH-2 | EF6 with `DropCreateDatabaseIfModelChanges` and MDF attach becomes EF Core SQL Server with migrations. | Assessment; data cutover |
| CH-3 | The database is new, named `BookCatalogModernizedLab`. Records in the old `BookCatalog.mdf` are not migrated. | Data is disposable |
| CH-4 | The request User-Agent diagnostic ("YOUR BROWSER:" box) is removed. It is not catalog behavior. | `Controllers/BooksController.cs:Index`, `Views/Books/Index.cshtml` |
| CH-5 | `customErrors Off` and `compilation debug=true` are not carried forward. Detailed errors appear only in Development. | `Web.config` |
| CH-6 | Inactive-book URL access follows the separate spec `specs/dotnet-inactive-book-404.md` (issue #1). This modernization preserves whatever the legacy app does at the time it starts, and the smoke script is the shared contract. | Decision B1 |

## Constraints and out of scope

### Constraints

- **Allowed paths:**
  - `legacy-src/dotnet-bookcatalog/shared-legacy-app/src/BookCatalog.Web/**`
    (excluding `bin/` and `obj/`)
  - `legacy-src/dotnet-bookcatalog/shared-legacy-app/BookCatalog.sln`
  - `legacy-src/dotnet-bookcatalog/shared-legacy-app/NuGet.Config` and
    `global.json`, only if the SDK-style migration requires it
  - `legacy-src/dotnet-bookcatalog/shared-legacy-app/README.md`
  - `legacy-src/dotnet-bookcatalog/AGENTS.md`, only to update build and format
    guidance after migration
  - `.github/modernize/bookcatalog-dotnet10/**` (plan artifacts only)
  - this specification
- **Primary validation command** (run from the monorepo root; the repository
  gate lives there):
  `pwsh scripts/validation/validate-dotnet-target.ps1 -EvidenceDirectory <temp dir>`.
  It restores, builds Release on net10.0, runs any test projects, and runs
  `scripts/validation/Test-BookCatalogSmoke.ps1` on a task-owned disposable
  LocalDB instance. The validation profile switches from baseline to target
  automatically once the project is SDK-style .NET 10 (`Get-ValidationProfile`
  in `scripts/modernization/Modernization.psm1`).
  Until then the baseline gate is `.\scripts\validation\validate-dotnet.ps1 -Configuration Release`.
- **Compatibility:** routes, model fields, validation limits and status codes in
  the requirements above. The smoke script expects
  `src/BookCatalog.Web/bin/Release/net10.0/BookCatalog.Web.dll`, so the project
  path, assembly name and target framework are fixed.
- **Dependencies:** target `net10.0`. ASP.NET Core MVC and EF Core SqlServer with
  their tooling only. Exact packages and versions are decided in the technical
  plan. No System.Web adapters, YARP or shared legacy schema. No new test
  project.
- **Data and security:** SQL Server LocalDB with integrated authentication and
  disposable synthetic data only. No passwords or secrets in source or config.
  No business database. No empty-MDF workaround. Antiforgery is kept. No new
  telemetry or request-metadata logging. Timestamps stay in local time as in
  the legacy app.
- **Cost and deployment:** local verification is the completion bar. Azure
  deployment (assets under `legacy-src/dotnet-bookcatalog/examples/azure`) is out
  of this change unless a separate approval names subscription, region, roles and
  cost owner. The sample is public and unauthenticated; no production-hardening
  claim is made. No-deployment completion is valid.
- **Cleanup obligations:** remove temp evidence directories and task-owned
  LocalDB smoke instances and state. Drop `BookCatalogModernizedLab` only if this
  task created it and only with user consent. Do not commit `bin/`, `obj/` or
  `packages/`.

### Exclusions (out of scope)

- The Java app, `capstone-src/`, and other `legacy-src` roots.
- Azure deployment, Bicep changes and CI workflows.
- Data migration from the old MDF.
- Windows/Entra user authentication.
- New features, UI redesign, JSON endpoints and telemetry.
- Changes to `scripts/validation/**` or `legacy-src/dotnet-bookcatalog/examples/**`.
- Hand-editing `packages/`.

## Edge cases

| Type | Condition | Expected behavior |
| --- | --- | --- |
| Normal | Fresh database | Six active books in title order, no Matrix row |
| Failure | LocalDB unavailable or migration fails | Startup fails with the first error; no silent recreate or empty-database fallback |
| Failure | POST without antiforgery token | 400 or 403 and no change |
| Boundary | Title 201 chars, Year 1799 or 2101, ISBN 14 chars | Rejected with a visible error |
| Boundary | Empty year, or zero active books | Empty year accepted; empty-state message shown |
| Boundary | Unknown or non-numeric id | 404, and never a 500 |
| Compatibility | Restart after an edit | Edit still present, no reseed |

## Acceptance criteria

### AC-1: List

- **Given** a fresh seeded database
- **When** GET `/` and `/Books`
- **Then** 200, six active books in title order, Matrix absent, no User-Agent block
- **Verification:** `Test-BookCatalogSmoke.ps1` via the primary validation command

### AC-2: Create and validation

- **Given** the create form with a valid token
- **When** Title and Author are valid, or Title is missing or Year is 1799
- **Then** valid input redirects to Index and the book appears once; invalid input re-renders with errors and adds no row
- **Verification:** smoke script

### AC-3: Antiforgery

- **Given** no antiforgery token
- **When** POST Create, Edit or Delete
- **Then** 400 or 403 and no data change
- **Verification:** smoke script

### AC-4: Edit, delete and not found

- **Given** an existing book
- **When** it is edited and then deleted, and when an unknown or non-numeric id is requested
- **Then** the edit appears, then the row disappears and Details returns 404; unknown and non-numeric ids return 404
- **Verification:** smoke script

### AC-5: Persistence

- **Given** a saved edit
- **When** the app is stopped and restarted on the same database
- **Then** the edit remains and seed data is not added again
- **Verification:** smoke script

### AC-6: Stack

- **Given** the source root
- **When** the primary validation runs
- **Then** it exits 0 on net10.0 and no `System.Web`, EF6 `EntityFramework` or `packages.config` reference remains under the source root
- **Verification:** validation output and `git grep`

### AC-7: Data and secrets

- **Given** the diff
- **When** it is reviewed
- **Then** it contains no password, secret or non-LocalDB connection string, and the manual-run database is `BookCatalogModernizedLab`
- **Verification:** diff review and `git grep`

### AC-8: Scope

- **Given** the diff
- **When** `git diff --name-only` runs
- **Then** every path is inside the allowed paths
- **Verification:** command

## Non-functional requirements

| ID | Quality | Testable requirement | Verification |
| --- | --- | --- | --- |
| NFR-1 | Reliability | Startup never drops the database and migrations apply idempotently | Restart smoke check |
| NFR-2 | Security/privacy | No secrets; detailed errors only in Development; no User-Agent in UI or logs | Diff review |
| NFR-3 | Maintainability | Single net10.0 web project and no deprecated compatibility shims | AC-6 |
| NFR-4 | Cost | Zero Azure spend | No `az` or deploy commands run |

## Assumptions, dependencies, and risks

| Item | Type | Validation or mitigation |
| --- | --- | --- |
| Windows, LocalDB and a .NET 10 SDK are available (`global.json` 10.0.100) | Dependency | Stop and report if missing |
| The assessment's mandatory "Windows authentication" finding refers to `Integrated Security=True` to LocalDB, not user authentication; the app has none | Assumption | Annotate the report with a companion note; do not edit the generated report |
| The assessment mentions App Service Managed Instance and a ~$63/month estimate, which differ from the lab's App Service + Azure SQL design | Risk | Deployment is out of scope; resolve in any later Azure spec |
| A single-host cutover is a large diff | Risk | Incremental commits and validation at each step |
| No pre-change observed behavior baseline (IIS Express CRUD was not run) | Risk | Accepted: preserved behavior comes from source evidence and the passing build |
| The inactive-404 change (issue #1) is separate and may land first | Dependency | Re-check CH-6 and the smoke script before execution |
| Views use no jQuery or unobtrusive client-side validation | Assumption | Verified by grep of `Views/Books`; server-side validation only |

## Plan-mode review checkpoint

### Evidence

- **Current behavior:** cited per requirement above.
- **Existing tests:** none found (`legacy-src/dotnet-bookcatalog/AGENTS.md`).
- **Baseline validation evidence:** `MSBuild.exe BookCatalog.sln -restore -p:Configuration=Release` from `legacy-src/dotnet-bookcatalog/shared-legacy-app` - exit 0, `BookCatalog.Web.dll` built (2026-11-23).

### Unresolved questions

None.

### Boundary and validation review

- [x] Evidence citations were checked against the repository.
- [x] Allowed paths include all anticipated implementation and test changes and
      nothing unrelated.
- [x] The primary validation command is safe, non-interactive, and runnable
      from the stated directory.
- [x] Acceptance criteria cover the required normal, failure, boundary, and
      compatibility behavior.
- [x] Exclusions prevent unrelated modernization or cleanup.

### Stop conditions

Stop and request human guidance if:

- repository evidence contradicts this specification;
- an unresolved question could materially change behavior or scope;
- satisfying a criterion requires a path outside the allowed paths;
- the baseline or primary validation fails for a reason caused by the proposed
  change, or the command is missing, destructive, or irreproducible;
- LocalDB or the .NET 10 SDK is missing;
- a secret, credential, production resource, Azure resource, destructive action,
  dependency change not listed above, schema change, or public contract change is
  required but not explicitly approved;
- the smoke script contract conflicts with this specification (for example the
  inactive-book 404 decision);
- the target database already exists with non-synthetic data; or
- assessment evidence would change intent, acceptance criteria, allowed paths or
  validation (return to Plan mode and obtain reapproval).

### Human approval

- [ ] I reviewed the evidence, unresolved questions, allowed paths, acceptance
      criteria, validation command, exclusions, risks, and stop conditions.
- **Reviewer:**
- **Approval date:**
- **Approved specification commit:**

Change **Status** to `Approved` only after the checkbox is selected and the
reviewer, date, and commit fields are complete. Planning approval authorizes
issue preparation, not implementation.
