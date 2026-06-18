# Architecture Standard

## Purpose

Use this guidance when migrating an existing project toward the common architecture standard being established in Flux. The goal is not architecture ceremony. The goal is code that is easier to maintain, easier to test, and reusable across hosts such as PowerShell, ServiceStack APIs, Background Jobs, Blazor, Terminal TUI, scheduled scripts, and future products.

The target style is:

- Clean Architecture for dependency direction and layer ownership.
- Vertical Slices for feature ownership and folder shape.
- CQRS-lite for clear application boundaries without adding a mediator framework by default.
- Compatibility-first migration for existing public commands, scripts, APIs, and UI flows.

## Layer Ownership

Use layers to decide where code belongs. If a file is hard to place, that is usually a sign that a boundary is doing too much.

## Domain

Domain owns deterministic business concepts and rules.

Put here:

- Value objects, domain records, enums, and exceptions that do not perform IO.
- Rules that can run without databases, HTTP, filesystem, process execution, UI, PowerShell, or ServiceStack.
- Domain concept folders, such as `DataShare/Order`, `DataShare/Download`, `Export`, or `Transact/Column`.

Do not put here:

- EF Core `DbContext`.
- HTTP clients.
- Filesystem access.
- PowerShell, ServiceStack, Blazor, TUI, or scheduler concepts.
- Generic `Entities`, `Enums`, or `Exceptions` folders when the files can belong to a domain concept.

## Application

Application owns user-visible use cases and workflow decisions.

Put here:

- Vertical slice command/query handlers.
- Request records such as `AddOrderCommand` or `GetOrderReadyQuery`.
- Result/outcome records or enums.
- Ports required by a use case, such as `IOrderHistoryClient`, `IExportExecutor`, or `IWebConfigWriter`.
- Application models that are shared by multiple hosts and are not infrastructure objects.

Do not put here:

- EF Core `DbContext` usage once the migration can avoid it.
- Concrete infrastructure adapters.
- PowerShell cmdlet types.
- ServiceStack endpoints, Background Jobs, or API DTO transport concerns.
- Terminal.Gui, ReactiveUI, Blazor rendering, or UI prompts.
- Generic `Services`, `Helpers`, `UseCases`, `Commands`, `Queries`, or `Handlers` folders.

## Infrastructure

Infrastructure owns external implementations behind application ports.

Put here:

- EF Core persistence adapters, repositories, stores, readers, and writers.
- HTTP clients and external API adapters.
- Filesystem, XML, JSON, web.config, IIS, OGR, FME, SMTP, SendGrid, Azure, process execution, and time/delay adapters.
- Composition extensions that register infrastructure implementations, such as `AddHiveInfrastructure()`.

Name infrastructure classes by responsibility, not by a generic `Service` suffix. Prefer names like `DatabaseColumnStore`, `Ogr2OgrRunner`, `WebConfigWriter`, `OrderHistoryClient`, or `ApplicationPoolReader`.

## Presentation And Host Adapters

Presentation and host projects own user interaction and public compatibility surfaces.

Put here:

- PowerShell cmdlets and scripts.
- ServiceStack endpoints, Background Job classes, and request/response DTO mapping.
- Blazor pages/components and ReactiveUI view models.
- Terminal TUI screens, dialogs, keyboard bindings, prompts, and display state.
- Host composition and product-specific wiring.

Host adapters should translate user input into application commands/queries, call handlers, and translate results back to the public surface. They should not contain the workflow itself.
