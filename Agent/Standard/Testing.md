# Testing Standard

## Test Project Rules

Use names that tell maintainers whether tests are safe to run without external dependencies.

- `UnitTests`: no live databases, IIS, network, credentials, installed SDKs, or hard-coded local paths.
- `IntegrationTests`: may use controlled infrastructure such as test containers, in-memory servers, or local test databases when setup is automated.
- `ManualTests`: live/manual probes, hard-coded operator configuration, credentials, IIS, external services, long-running workflows, or tests that require inspection.

Move existing hard-coded development probes out of `UnitTests` and into `ManualTests`.

## Unit Test Shape

Unit tests should verify behavior, handler decisions, and application port calls without requiring live infrastructure.

For migrated vertical slices:

- Test command/query handlers directly.
- Use in-memory fakes or mocks for application ports.
- Keep host adapter tests focused on mapping public inputs to application requests.
- Do not place live IIS, filesystem, external service, credential, or operator-inspection tests in `UnitTests`.

## Related Template

Use `../Template/TestProjectStructureTemplate.md` when creating or renaming test projects.
