# Modernization Plan: BookCatalog .NET 10 Modernization

**Project**: BookCatalog

---

## Technical Framework

- **Language**: C# (.NET Framework 4.8)
- **Framework**: ASP.NET MVC 5 (classic web application project format)
- **Build Tool**: MSBuild (`nuget restore` + `msbuild`), `packages.config`
- **Database**: SQL Server LocalDB (`MSSQLLocalDB`), integrated authentication, disposable sample data
- **Key Dependencies**: Entity Framework 6, `System.Data.SqlClient`, Web.config connection strings without configuration builders, static content served from the classic web project

---

## Overview

> This migration moves BookCatalog from ASP.NET MVC 5 on .NET Framework 4.8 with EF6 to ASP.NET Core MVC on .NET 10 with EF Core, preserving the approved catalog behavior (active/inactive book list, sorting, add/edit/remove flows). The application currently runs as a classic SDK-less web project with Windows/integrated authentication against a local SQL Server LocalDB instance. The new architecture will:
>
> - Convert the project to the modern SDK-style .NET 10 format, replacing System.Web with ASP.NET Core MVC hosting
> - Replace EF6 with EF Core while keeping the same disposable, re-creatable LocalDB-based data model and seed behavior
> - Remediate known dependency vulnerabilities (for example `System.Data.SqlClient`) as part of the upgrade
>
> The migration follows a single, in-place upgrade of the existing copy (no side-by-side host, shared schema, YARP, or System.Web compatibility shims), verified locally with the repository smoke script; Azure and Bicep are out of scope.

---

## Migration Impact Summary

| Application | Original Service | New Azure Service | Authentication | Comments |
|-------------|------------------|--------------------|-----------------|----------|
| BookCatalog.Web | ASP.NET MVC 5 / .NET Framework 4.8 | ASP.NET Core MVC / .NET 10 | N/A (in-place upgrade) | Preserve catalog behavior; no deployment |
| BookCatalog.Web | EF6 + SQL Server LocalDB | EF Core + SQL Server LocalDB | Integrated/local, unchanged | Data remains disposable, re-creatable |

---

## Reconciliation with the approved specification

This plan is a proposal subordinate to `specs/dotnet-bookcatalog-modernization.md`
(approved commit `d2fcb40`). On any conflict the specification wins.

- **Tasks kept:** 001 upgrade (increments, allowed paths, validation, containment, stop
  conditions), 004 CVE check (within the spec's dependency rule), and 005 local
  verification via `pwsh scripts/validation/validate-dotnet-target.ps1 -EvidenceDirectory <temp dir>`.
- **Tasks removed from the generated plan:** 002 baseline bundle (not in the spec; the
  smoke script is the contract) and 003 Bicep generation (Azure and Bicep are
  excluded).
- **Wording corrected:** "preserve exactly as currently implemented" now follows FR-1
  to FR-9 and the intentional changes CH-1 to CH-5, including removal of the User-Agent
  diagnostic and the new `BookCatalogModernizedLab` database.
- **Not executed:** nothing in this plan has been run. No Azure deployment, no
  new test project.
- **Generator defaults replaced:** the tool's auto-chosen answers (Mock mode,
  no infrastructure, standard CVE scan) were replaced by the spec's decisions.

## Plan approval

- **Reviewer:** sinkay07
- **Approval date:** 2026-11-23
- **Reviewed against:** specs/dotnet-bookcatalog-modernization.md (approved commit d2fcb40)
- **Accepted risks:** no observed pre-change IIS Express baseline; the inactive-book 404 change (issue #1) is separate; the plan was generated with app feature flags cleared for the CLI process

