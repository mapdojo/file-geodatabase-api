# Test Project Structure Template

Use this template when adding, splitting, or reviewing test projects. The goal is to make the project name tell future agents whether the project is safe to run automatically, needs external services, or only provides shared test helpers.

## Naming Convention

Use explicit suffixes:

- `*.UnitTests` for deterministic automated tests that can run without external services or machine-specific setup.
- `*.IntegrationTests` for automated tests that exercise multiple layers or host/runtime behavior, but still set up their own dependencies or use controlled test infrastructure.
- `*.ManualTests` for development probes that require live client configuration, hard-coded local paths, IIS, SQL Server/DataShare state, SMTP/SendGrid, external APIs, credentials, or other environment-specific dependencies.
- `*.Testing` or `*.TestSupport` for shared helper libraries used by multiple test projects. These projects should not contain runnable tests.

Avoid ambiguous `*.Tests` or `*.Test` project names.

## Project Decision Guide

Before creating or moving a test, classify it by dependency shape:

| Question | Project Type |
| --- | --- |
| Can this run on a clean developer machine or CI without external setup? | `UnitTests` |
| Does it start a test host, exercise middleware, or cross project boundaries with controlled in-memory/test infrastructure? | `IntegrationTests` |
| Does it require real infrastructure, local machine state, credentials, hard-coded paths, client data, or manual inspection? | `ManualTests` |
| Is it only fake clocks, builders, fixtures, test data factories, or assertion helpers? | `Testing` or `TestSupport` |

## Folder And Project Names

When writing paths, do not duplicate product context that is already present in the parent folder.

Use the local area/test project folder as the final path segment:

```text
path\to\<Area>.UnitTests\<Product>.<Area>.UnitTests.csproj
```

For example:

```text
Flux\Hive\Hive.Application.UnitTests\Flux.Hive.Application.UnitTests.csproj
```

The folder is `Hive.Application.UnitTests` because `Flux\Hive` already provides the broader product location. The project file can still use the fully qualified project name.

## Unit Test Project Template

Project file name:

```text
<Product>.<Area>.UnitTests
```

Rules:

- Tests must be deterministic.
- No live network, database, IIS, SMTP, PowerShell host mutation, or machine-specific path assumptions.
- Prefer fake ports, in-memory stores, test doubles, and controlled clocks.
- The project can be included in normal `dotnet test` gates.
- If a unit test starts needing external setup, move it rather than hiding the dependency.

Recommended verification:

```powershell
dotnet test path\to\<Area>.UnitTests\<Product>.<Area>.UnitTests.csproj
```

## Integration Test Project Template

Project file name:

```text
<Product>.<Area>.IntegrationTests
```

Rules:

- Tests may exercise multiple layers, middleware, dependency injection, hosted applications, or persistence adapters.
- Dependencies must be controlled by the test project, documented, and repeatable.
- Prefer in-memory, temporary, containerized, or local test infrastructure.
- Do not use client-specific live data or credentials.
- These tests may be slower than unit tests, but should still be suitable for intentional automated runs.

Recommended README note:

```markdown
# <Product>.<Area>.IntegrationTests

This project contains repeatable integration tests for <area>. It may start test hosts or controlled local dependencies, but it must not rely on client-specific data, credentials, or hard-coded machine paths.
```

## Manual Test Project Template

Project file name:

```text
<Product>.<Area>.ManualTests
```

Rules:

- Use for manual development probes and client-like verification.
- Allowed dependencies include live SQL Server/DataShare state, IIS/application pools, SMTP/SendGrid, external APIs, credentials, local SDK paths, and machine-specific configuration.
- Keep these projects out of unit test gates.
- Add them to the solution for IDE discovery, but run them explicitly.
- Include a README that names the dependencies and warns that the project is not a unit test gate.

Recommended README:

````markdown
# <Product>.<Area>.ManualTests

This project contains manual development probes that exercise <dependencies>.

Run it explicitly when those dependencies are available:

```powershell
dotnet test path\to\<Area>.ManualTests\<Product>.<Area>.ManualTests.csproj
```

Do not use this project as the unit test gate. Use `<Product>.<Area>.UnitTests` for deterministic coverage.
````

## Shared Testing Helper Project Template

Project name:

```text
<Product>.<Area>.Testing
```

or:

```text
<Product>.<Area>.TestSupport
```

Rules:

- This is a helper library, not a runnable test suite.
- Do not add xUnit/NUnit/MSTest test classes here.
- Keep reusable helpers here when both unit and integration tests need them.
- Good candidates include fake clocks, fixture builders, assertion helpers, test data factories, host builders, and fake adapters.
- Prefer `TestSupport` if introducing a new convention; keep `Testing` where it already exists and is understood.

Example:

```text
Apis.Auth.Infrastructure.Testing
  FakeTimeProvider.cs
```

## Solution Inclusion

Add all explicit test projects and shared testing helper projects to the solution when developers need IDE discovery:

```powershell
dotnet sln Flux.slnx add path\to\<Project>\<Project>.csproj
dotnet sln Flux.slnx list
```

After using `dotnet sln add`, check the diff. If the CLI creates an unexpected solution folder, move the project entry into the existing logical folder manually.

## Migration Checklist

- [ ] List all existing `*.Tests` and `*.Test` projects.
- [ ] Classify each project as `UnitTests`, `IntegrationTests`, `ManualTests`, or `Testing`/`TestSupport`.
- [ ] Move hard-coded or live-dependency tests into `ManualTests`.
- [ ] Keep deterministic tests in `UnitTests`.
- [ ] Keep repeatable multi-layer tests in `IntegrationTests`.
- [ ] Keep shared helpers in `Testing`/`TestSupport`.
- [ ] Rename namespaces to match the new project names.
- [ ] Update project references and `InternalsVisibleTo` attributes.
- [ ] Add renamed projects to the solution for IDE discovery.
- [ ] Run focused verification for each affected test project.
- [ ] Document manual project dependencies in a README.
- [ ] Commit the migration with the checklist updated.

## Agent Handoff Prompt

Use this prompt when assigning the work to another project agent:

```markdown
Please review the test projects in this area and apply the repository test naming convention:

- `UnitTests` means deterministic and safe to run without external dependencies.
- `IntegrationTests` means repeatable multi-layer or host/runtime tests with controlled dependencies.
- `ManualTests` means live-dependency or machine-specific development probes that must be run explicitly.
- `Testing` or `TestSupport` means shared helper code only, not a runnable test suite.

Do not leave ambiguous `Tests` or `Test` project names. Preserve backwards-compatible runtime behavior. Add projects to the solution for IDE discovery, update namespaces/references/`InternalsVisibleTo`, run focused verification, and commit the checklist as you go.
```
