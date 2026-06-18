# Agent Architecture Standards Template

Use this template in `AGENTS.md` to guide agents toward maintainable Clean Architecture with vertical slices and CQRS-lite. Treat these conventions as simplification rules, not ceremony.

## Architecture Conventions

Use Clean Architecture as the default pattern. Prefer simple vertical slices and CQRS-lite over tactical DDD ceremony.

### Layer Ownership

- `Domain`: deterministic rules, value objects, formatting/comparison rules, and small models with no IO dependencies.
- `Application`: user-visible use cases, workflows, commands, queries, handlers, results, and ports.
- `Infrastructure`: adapters for external systems such as file IO, databases, IIS, PowerShell, APIs, process/runtime access, and persistence.
- `Presentation`: UI, ViewModels, views, dialogs, keyboard/focus/event wiring, host composition, and display formatting.

Keep `Domain` intentionally small. Do not place DTOs, command/query results, display rows, or UI models in Domain just because they are important.

### Vertical Slices

Organize Application code around user-visible workflows, for example:

- `SaveConfigurationDocument`
- `ReloadConfigurationContent`
- `RestoreConfigurationBackup`
- `GetApplicationPoolStatus`
- `CreateApplicationPool`

A slice should be easy to explain from its filename and tests. Prefer one cohesive slice over broad reusable coordinators or generic service buckets.

### CQRS-Lite

Use CQRS-lite as vocabulary, not as a framework.

- Use named `Command` records for write/mutating use cases.
- Use named `Query` records for read-only use cases.
- Use `Handler` classes when there is a real workflow decision, side effect, or reusable host boundary.
- Use `Result` for the complete return payload.
- Use `Outcome` for the branch/category inside a result, such as `Created`, `Exists`, `Failed`, `Ready`, `NoChanges`, or `TargetError`.

Do not add mediator pipelines, marker interfaces, base handler classes, generic command/query abstractions, repositories, or domain events unless a concrete repeated need proves they reduce code.

Keep named `Command`/`Query` records even if they currently have one field when the handler is still a real Application boundary. Do not churn between `Handle(string)` and `Handle(SomeCommand)` purely because the request is small.

### Boundary Rules

- Application handlers should not create UI widgets, show dialogs, set focus, mutate toolbar text, or depend on UI frameworks.
- Presentation should translate UI intent into Application commands/queries and decide how results become dialogs, focus changes, visual state, or messages.
- Infrastructure should implement Application-facing ports for external systems.
- File, IIS, process, network, and PowerShell access should stay behind Infrastructure adapters or narrow Application ports.

### Naming

Prefer names that describe concrete behavior:

- Good: `Reader`, `Writer`, `Selector`, `Builder`, `Formatter`, `Handler`, `Interaction`, `Binding`.
- Avoid generic names unless earned: `Service`, `Manager`, `Coordinator`, `Factory`, `Planner`, `Workflow`.

Use `Factory` only when object creation itself is the meaningful abstraction. Use `Builder` when incrementally composing a complex object/configuration is the real job.

### Simplification Rules

Prefer the smallest behavior-preserving change that makes the code easier to explain.

Inline or delete helpers when they only forward to one handler or hide a simple call chain. Keep a helper only when it owns:

- a user-visible use case,
- a deterministic rule,
- an external adapter,
- reusable UI composition,
- or repeated behavior that is clearer with a name.

Do not split files just to reach one-type-per-file. Split when a file mixes enough behavior and plumbing that a human reader has to scroll past noise to understand the use case.

### Presentation Rules

Presentation code may contain UI-specific binding/composition classes. These do not need Domain/Application names.

Use MVVM for UI state management. ReactiveUI is the blessed ViewModel/state composition library unless a project has an established alternative that should be preserved. Prefer ReactiveUI ViewModels for UI state, derived state, and UI command transitions.

Keep UI-only behavior in Presentation:

- dialogs,
- keyboard shortcuts,
- focus changes,
- breadcrumbs,
- colour/scheme changes,
- toolbar state,
- framework-specific widget composition.

ViewModels own UI state and derived UI state. They should not parse files, call external APIs, or contain business workflow decisions except through Application handlers/services.

### Shared Code

Promote code into shared libraries only after a concrete consumer proves the boundary and the type name is clearly generic. Do not abstract project-specific concepts early just because they might be reusable later.

### Testing

Use tests as a pressure gauge.

- Add handler tests for Application workflow decisions and outcomes.
- Add ViewModel tests for UI state transitions.
- Add Infrastructure tests around adapter behavior when practical.
- Avoid test churn for pure renames unless the old name would keep misleading future maintainers.

### Migration Workflow

Use a read, simplify, verify, commit loop.

1. Pick one user-visible workflow or confusing file.
2. Identify whether the behavior belongs in Application, Infrastructure, Presentation, or Domain.
3. Move UI prompts and visual effects to Presentation.
4. Keep IO behind Infrastructure or Application ports.
5. Rename stale DDD-era helpers to concrete role names.
6. Run focused tests.
7. Commit one small slice with a meaningful message.

Architecture is a simplification tool, not a ceremony target.
