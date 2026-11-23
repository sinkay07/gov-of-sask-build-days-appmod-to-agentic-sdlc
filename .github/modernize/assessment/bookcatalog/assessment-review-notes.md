# Companion note: Modernize assessment review (BookCatalog)

This note annotates the generated report in `reports-20261123190524/`. It does not
replace or edit it. Evidence paths are under
`legacy-src/dotnet-bookcatalog/shared-legacy-app/src/BookCatalog.Web/`.

## Corrections to reported findings

- **"Windows authentication detected" (Mandatory)** is a false positive for user
  authentication. `Web.config` only sets `Integrated Security=True` in the LocalDB
  connection string. The app has no user authentication. Treat as a data-access
  setting for the EF Core connection string, not a mandatory auth migration.
- **"Local or network IO" (`Global.asax.cs` line 15)** is `Directory.CreateDirectory`
  on the EF `DataDirectory` used for the LocalDB MDF attach, not general file handling.

## Findings the report did not surface

- ASP.NET MVC 5 / System.Web, `Global.asax`, `App_Start/*` routing and filters, and
  Razor view compatibility work (`packages.config`: Microsoft.AspNet.Mvc 5.2.9).
- `packages.config` and the classic non-SDK `BookCatalog.Web.csproj` (net48).
- EF6 (`EntityFramework 6.4.4`): `ApplicationDbContext`, the
  `DropCreateDatabaseIfModelChanges` initializer and seed data.
- No automated test project exists; behavior checks are manual or by smoke script.
- Azure deployment is a separate, later gate. The report's cost figure and
  managed-instance wording are not authoritative for this change.

The report's "Codebase Insights" section was not generated (requires full analysis
coverage), so no insight claims are relied on.
