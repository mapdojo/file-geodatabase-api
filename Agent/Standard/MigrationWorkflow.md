# Migration Workflow

## Start Every Migration With A Checklist

Create a small markdown checklist in the repo before or during the first commit of the migration. Keep it updated as the work proceeds so reviewers can see what changed, what was verified, and what remains.

Recommended locations:

```text
Agent/Initiative/<InitiativeName>/Checklist.md
Agent/Initiative/<InitiativeName>/<SliceName>Checklist.md
```

Recommended checklist shape:

```markdown
# <Project or Slice> migration checklist

## Intent

Short description of what is moving and what compatibility must be preserved.

## Compatibility contracts

- [ ] Public cmdlet/API/script/UI names remain unchanged.
- [ ] Existing parameters, output types, and pipeline behavior remain unchanged.
- [ ] Manual/live workflows remain granular where operators currently inspect between steps.

## Change checklist

- [ ] Move application use cases into vertical slices.
- [ ] Move infrastructure implementations behind application ports.
- [ ] Update host adapters to call handlers.
- [ ] Remove or document legacy shims.
- [ ] Update architecture documentation.

## Verification checklist

- [ ] Unit tests for migrated handlers.
- [ ] Host adapter tests.
- [ ] Build affected host.
- [ ] Manual smoke tests, if external systems are required.
- [ ] Commit slice.
```

## Agent Work Loop

Use this loop for migration and architecture work:

1. Inspect the existing code and current git status before planning edits.
2. Identify the compatibility contracts that must remain stable.
3. Create or update the initiative checklist before broad changes.
4. Make the smallest coherent slice of changes.
5. Prefer existing project patterns over new abstractions.
6. Do not create new `Helpers` or generic buckets while migrating toward vertical slices.
7. Keep temporary seams documented with why they exist and what removes them.
8. Run targeted verification before each commit.
9. Commit the slice with a specific conventional-style message.
10. Update the checklist or plan with what was proven and what remains.

If unexpected local changes are already present, work with them and do not revert them unless explicitly asked.

## Compatibility Migration Pattern

When existing users depend on the current surface, preserve the surface and change internals progressively.

Preferred sequence:

1. Identify the public contract.
   Public contracts include cmdlet names, parameters, output types, pipeline behavior, scripts, API routes, DTOs, UI flows, scheduled tasks, file formats, and database tables.

2. Add or identify the application slice.
   Create command/query/handler/result files under the owning vertical slice.

3. Add ports for external work.
   The application handler depends on interfaces for persistence, HTTP, filesystem, process execution, email, IIS, or other external boundaries.

4. Move concrete work to infrastructure.
   Infrastructure implements the ports and registers them in infrastructure composition.

5. Keep the host surface stable.
   PowerShell cmdlets, endpoints, jobs, or UI actions call the handler and keep their existing behavior.

6. Use existing Helpers only as temporary shims.
   Do not create new Helpers. Existing Helpers may remain briefly when they preserve compatibility, but document them and remove them in a later cleanup once all callers use handlers.

7. Test the handler without external dependencies.
   Unit tests should verify handler behavior and port calls. External live checks belong in `ManualTests` or smoke checklists.

8. Commit the slice.
   Keep each commit reviewable, even when the PR is intentionally large.

## Compatibility Contract Checklist

Before changing internals, identify the public surface that must keep working.

Check every applicable contract:

- [ ] PowerShell cmdlet names, aliases, parameters, parameter sets, pipeline behavior, and output types.
- [ ] PowerShell script names, script parameters, scheduled task entry points, and manual step-by-step workflows.
- [ ] API routes, request DTOs, response DTOs, status codes, auth requirements, and ServiceStack metadata.
- [ ] Background Job names, arguments, scheduling assumptions, retry behavior, and progress/log visibility.
- [ ] Blazor, TUI, CLI, or other UI flows that users already rely on.
- [ ] Database table names, column names, migrations, log table shape, and data compatibility.
- [ ] File formats, appsettings shape, web.config shape, environment variable names, and generated output paths.
- [ ] External service contracts such as HTTP URLs, query parameters, email payloads, OGR/FME invocation arguments, and filesystem layout.
- [ ] Manual inspection points where operators intentionally pause between granular steps.

When compatibility cannot be preserved, document the breaking change, migration path, and release note before implementing it.

## Background Jobs And Alternate Hosts

Do not make ServiceStack Background Jobs call PowerShell commands or legacy Helpers.

Target shape:

```text
PowerShell cmdlet  \
ServiceStack API    > Application command/query handler -- Application ports -- Infrastructure adapters
Background Job     /
```

For long-running operational workflows, preserve the same granularity users rely on today. If operators currently run `Order`, inspect, then `Download`, inspect, then `Update`, keep those as separate commands/jobs. Do not hide the workflow inside one large opaque job.

## PR Review Checklist

Before review, check:

- [ ] No application project references the main infrastructure project unless explicitly documented as a temporary project-graph seam.
- [ ] No application handler creates concrete infrastructure clients directly.
- [ ] No new `Helpers`, `Services`, `Models`, `UseCases`, `Commands`, `Queries`, or `Handlers` buckets were added.
- [ ] Each active CQRS-lite command/query/handler/result is in its own file.
- [ ] Public host contracts are unchanged or deliberately versioned.
- [ ] Product-neutral code is in `Apis.*`; product-specific code stays in product projects.
- [ ] Unit tests do not require external dependencies.
- [ ] Manual smoke steps exist for external dependencies.
- [ ] Architecture docs and migration checklist match the code.

## Known Acceptable Temporary Seams

Some seams may remain during a large migration. They are acceptable only when documented with a cleanup path.

Examples:

- A product application project still references persistence while database ports are being introduced.
- A compatibility project still holds shared base types while new clean projects are being introduced.
- A PowerShell host still contains composition code while cmdlets are kept stable.
- Existing Helpers remain as shims while callers migrate to application handlers.

Every temporary seam should answer:

- Why is this still here?
- What compatibility does it protect?
- What would remove it?
- Which tests or smoke checks prove it still works?

## Commit Message Standard

Use short conventional-style commit messages so history can be scanned quickly.

Format:

```text
type: imperative summary
```

Examples:

- `docs: organize agent guidance by intent`
- `feat: migrate datashare order handlers`
- `fix: preserve datashare order history parsing`
- `refactor: move hive web config adapters`
- `test: split manual hive application tests`

Use these common types:

- `docs`: documentation, plans, checklists, templates, or standards.
- `feat`: new user-visible behavior or additive capability.
- `fix`: bug fix or compatibility regression fix.
- `refactor`: internal structure change without intended behavior change.
- `test`: test-only changes.
- `build`: project files, package references, solution entries, or build scripts.
- `ci`: workflow or automation changes.
- `chore`: low-risk maintenance that does not fit the above.

Keep the summary imperative and specific. Prefer `docs: organize agent guidance by intent` over `updated docs` or `agent stuff`.

## Submodule Workflow

When a change touches a submodule and the parent repository, commit in dependency order.

Use this sequence:

1. Make and verify the submodule change inside the submodule working tree.
2. Commit the submodule first with a message that describes the submodule change.
3. Return to the parent repository.
4. Stage the submodule pointer update with any parent-repo integration changes.
5. Commit the parent repository with a message that describes the integration.

Keep related submodule and parent commits close together in time and intent. If the submodule change is only preparation for the parent change, say so in the commit summary or body.

Do not mix unrelated parent repository cleanup into a submodule pointer commit. Pointer commits should be easy to review and easy to bisect.

## Submodule Agent Context

When parent repository work depends on a submodule change, the submodule should carry its own `Agent/` context for the work.

Use this structure inside the submodule when it does not already exist:

```text
Agent/
  Archive/
  Initiative/
  Standard/
```

Add the smallest useful documentation inside the submodule so future consumers can understand the change without reading the parent repository history.

Use the submodule `Agent/` folder for:

- Initiative plans or checklists for submodule-owned migrations.
- Decision logs for submodule architecture choices.
- Archive notes for superseded submodule migration documents.
- References from active submodule README files to archived historical context.

Do not rely only on the parent repository `Agent/` folder for submodule decisions. The parent can reference the submodule work, but the reason for the submodule change should be discoverable from inside the submodule itself.

## Rename And Move Checklist

For large folder, namespace, or project moves:

- [ ] Use `git mv` for deliberate file moves when practical.
- [ ] Move files by domain concept or vertical slice, not by generic technical bucket.
- [ ] Update namespaces, project files, solution entries, and package references.
- [ ] Update documentation links and agent guidance references.
- [ ] Avoid unrelated formatting, line-ending, or using cleanup in the same commit.
- [ ] Run targeted searches for old paths, namespaces, and class names.
- [ ] Run the smallest useful build or test command that proves the move.
- [ ] Commit the move separately from behavior changes when possible.

If a move also requires behavior changes, prefer one commit for the mechanical move and a later commit for the behavior change.

## Decision Log Standard

For architecture choices that affect future agents or other repositories, add a `DecisionLog.md` inside the initiative folder.

Use a decision log when the choice changes:

- Layer ownership.
- Common versus product-specific ownership.
- Public compatibility expectations.
- Naming or folder standards.
- Test project classification.
- Submodule boundaries.
- Migration order or rollback strategy.

Recommended entry shape:

```markdown
## YYYY-MM-DD - Short Decision Name

Decision:

Short statement of the chosen direction.

Context:

Why the decision was needed and what constraints mattered.

Consequences:

What this enables, what tradeoffs it creates, and what follow-up work remains.
```

Keep decision entries short. Link to plans, checklists, commits, or PRs for detailed implementation history.

## Suggested Commit Order For Large Migrations

Even when the PR is intentionally large, make history readable:

1. Commit architecture documentation and checklist.
2. Commit project skeletons and solution entries.
3. Commit one migrated feature family or vertical slice.
4. Commit tests for that slice.
5. Commit host adapter rewiring.
6. Commit deletion of duplicated legacy implementation.
7. Commit documentation updates that record what was proven.
8. Repeat for the next slice.

For submodules, commit the submodule first, then commit the parent repo pointer and host changes.
