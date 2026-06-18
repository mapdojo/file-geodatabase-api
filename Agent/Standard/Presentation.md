# Presentation And Host Adapter Standard

## Host Adapter Rule

Presentation and host projects own user interaction and public compatibility surfaces.

Put here:

- PowerShell cmdlets and scripts.
- ServiceStack endpoints, Background Job classes, and request/response DTO mapping.
- Blazor pages/components and ReactiveUI view models.
- Terminal TUI screens, dialogs, keyboard bindings, prompts, and display state.
- Host composition and product-specific wiring.

Host adapters should translate user input into application commands/queries, call handlers, and translate results back to the public surface. They should not contain the workflow itself.

## Blazor And ReactiveUI Rule

Co-locate Blazor views, code-behind files, and ReactiveUI view models by feature or page.

Good:

```text
Pages/
  DataShare/
    Download/
      DownloadView.razor
      DownloadView.razor.cs
      DownloadViewModel.cs
```

Avoid:

```text
Views/
ViewModels/
Pages/
  DataShare/
    DownloadView.razor
```

Use `.razor.cs` for component code-behind files. Do not create generic `Views` or `ViewModels` folders when the files can be co-located with the page, component, vertical slice, or domain concept.

## Terminal TUI Rule

Terminal UI code belongs in presentation or host projects unless it is product-neutral terminal mechanics that can move to `Apis.Terminal.*`.

Keep product-specific labels, paths, release rules, and Hive specializations in product projects. Move only product-neutral terminal application, infrastructure, domain, or presentation mechanics into common `Apis.Terminal.*` projects.
