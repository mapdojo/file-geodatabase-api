# Naming And File Shape

## Naming Rules

Prefer singular namespace and folder segments when the segment names a workflow, concept, adapter, or boundary.

Examples:

- `DataShare`, not `DataShares`.
- `Order`, not `Orders`.
- `ApplicationPool`, not `ApplicationPools`.
- `Configuration`, not `Configurations`.
- `Entity` only if it is truly a domain concept folder, but usually prefer the actual concept name instead.

Plural or framework names are fine when they are established conventions:

- `UnitTests`, `IntegrationTests`, `ManualTests`.
- EF Core `Migrations`.
- .NET `Properties`.
- Blazor `Pages` and `Components`.
- Collection-shaped values such as `ConnectionStringParameters` or `EnvironmentVariables`.

Generic plural folders are a design smell. Before renaming `Models` to `Model`, ask which vertical slice or domain concept owns the files.

## Extension Method Rule

Extension methods must live in their own file, with a static class name ending in `Extension` and named after the method.

Good:

```text
Security/SecureString/ToSecureStringExtension.cs
Configuration/ServiceCollection/AddHiveApplicationExtension.cs
```

Avoid:

```text
Extensions/Extensions.cs
Extensions/StringExtensions.cs
```

Co-locate the extension with the vertical slice, domain concept, adapter, or shared capability it belongs to.

## File Shape

Prefer small files that make a folder scannable:

- One CQRS-lite artifact per file.
- One extension method class per file.
- One domain concept, value object, port, adapter, or host adapter per file.
- Avoid grouped `UseCases.cs`, `Models.cs`, `Helpers.cs`, or `Extensions.cs` files.
