# Agent Standards

Attach only the guidance files needed for the task. These standards are split by intent so agents can carry the right context without loading one large migration guide.

## Routing

- Clean layer ownership or dependency direction: `Architecture.md`.
- Vertical slice structure or CQRS-lite shape: `VerticalSlicesAndCqrsLite.md`.
- Naming, file shape, extension methods, or generic folder cleanup: `NamingAndFileShape.md`.
- Encoding, BOM cleanup, line endings, or editor encoding metadata: `EncodingAndLineEndings.md`.
- Test project naming or moving manual/dependency-heavy tests: `Testing.md`.
- Blazor, ReactiveUI, Terminal TUI, or other host presentation work: `Presentation.md`.
- Common `Apis.*` extraction or product/common ownership decisions: `SharedApisOwnership.md`.
- Compatibility migration, shims, checklists, PR preparation, commit order, or commit messages: `MigrationWorkflow.md`.

## Related Templates

- Agent instruction template: `../Template/AGENTS.md`.
- Test project structure template: `../Template/TestProjectStructureTemplate.md`.
