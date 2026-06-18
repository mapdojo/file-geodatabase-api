# Shared Apis Ownership

## Common Versus Product-specific Ownership

Move reusable behavior into common `Apis.*` projects only after the boundary is product-neutral enough to share.

Use this split:

- `Apis.Standard.*`: broadly reusable primitives and utilities.
- `Apis.Terminal.*`: product-neutral terminal mechanics only.
- `Apis.Hive.Domain/Application/Infrastructure`: product-neutral Hive concepts and workflows.
- Product projects such as `Flux.*`: product-specific hosts, public cmdlets, scripts, labels, paths, release conventions, appsettings specializations, and compatibility adapters.

Do not move product-specific behavior into `Apis` just because another product may need something similar later. First separate the reusable behavior from product names, release paths, labels, and assumptions.

## Promotion To Apis Checklist

Before moving code into `Apis.*`, check:

- [ ] Names are product-neutral and do not include Flux-specific terminology unless Flux is genuinely part of the shared domain.
- [ ] No Flux-specific paths, labels, release conventions, appsettings names, or environment assumptions remain.
- [ ] No PowerShell, ServiceStack, Blazor, TUI, scheduler, or product-host type leaks into shared application or domain code.
- [ ] External work is behind application ports with infrastructure adapters.
- [ ] The shared behavior is useful to at least one plausible second product or host without copy/paste.
- [ ] Product-specific customizations are passed in through models, options, derived types, or host composition.
- [ ] Unit tests cover the product-neutral behavior without live external dependencies.
- [ ] The parent product continues to preserve its public compatibility surface after the move.

If most checklist items fail, keep the code in the product project and extract only the smaller reusable concept.

## Common Hive Direction

Shared Hive manager functionality should converge in `Apis.Hive.*` when it is common across products. Product projects should provide the product-specific pieces such as `HivePath`, `HiveEnvironment`, `AppSettings`, labels, release paths, and host compatibility.

PowerShell, ServiceStack APIs, Background Jobs, Blazor, Terminal TUI, and scheduled scripts should reuse the same application classes where practical instead of reinventing product-specific implementations.
