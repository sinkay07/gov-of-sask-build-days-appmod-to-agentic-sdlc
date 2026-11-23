---
applyTo: "capstone-src/api-dotnet/tests/**/*.cs"
---

# .NET capstone API: test guidance

- Use xUnit as declared in
  `capstone-src/api-dotnet/tests/WorkItems.Api.Tests/WorkItems.Api.Tests.csproj`
  (`Using Include="Xunit"` is implicit) and follow the style of
  `InMemoryWorkItemRepositoryTests.cs`.
- Test observable behavior of the types in `capstone-src/api-dotnet/src/WorkItems.Api/`
  and the shape in `capstone-src/contracts/openapi.yml`. Inject time through
  `TimeProvider` with the existing `StubTimeProvider` seam; never read the
  system clock or sleep.
- Run `dotnet test WorkItems.slnx --configuration Release --nologo` from
  `capstone-src/api-dotnet`.
- Use only in-memory, synthetic data. Do not call the network, cloud services,
  or production data.
