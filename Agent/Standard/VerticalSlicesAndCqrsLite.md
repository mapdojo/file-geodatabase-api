# Vertical Slices And CQRS-lite

## Vertical Slice Folder Shape

Organize by feature and operation, not by technical type.

Good:

```text
Flux.Hive.Application/
  DataShare/
    Download/
      Ready/
        GetDataShareDownloadReadyQuery.cs
        GetDataShareDownloadReadyHandler.cs
        GetDataShareDownloadReadyResult.cs
        DataShareDownloadReadyItem.cs
      Log/
        AddDataShareDownloadLogCommand.cs
        AddDataShareDownloadLogHandler.cs
        AddDataShareDownloadLogResult.cs

Apis.Hive.Application/
  Hosting/
    ApplicationPool/
      Add/
        AddHiveApplicationPoolCommand.cs
        AddHiveApplicationPoolHandler.cs
        AddHiveApplicationPoolResult.cs
      Get/
        GetApplicationPoolQuery.cs
        GetApplicationPoolHandler.cs
        GetApplicationPoolResult.cs
  Configuration/
    WebConfig/
      Add/
        AddHiveApplicationWebConfigCommand.cs
        AddHiveApplicationWebConfigHandler.cs
        AddHiveApplicationWebConfigResult.cs
        IHiveApplicationWebConfigWriter.cs
```

Avoid:

```text
Application/
  Commands/
  Queries/
  Handlers/
  Models/
  Services/
  Helpers/
  UseCases.cs
```

Each CQRS-lite artifact belongs in its own file so the slice can be scanned without opening large grouped files.

## CQRS-lite Rules

Use CQRS-lite as vocabulary, not as a framework.

- Use `Command` for writes, mutations, and actions.
- Use `Query` for reads.
- Use `Handler` for the class that executes the use case.
- Use `Result` or `Outcome` when the boundary should return named information.
- Keep direct method calls when a wrapper would only call a constructor, static method, or one-line host API.
- Do not add MediatR, marker interfaces, generic command abstractions, repository ceremony, or pipelines unless the project already uses them and the concrete workflow earns the extra layer.

Good handler boundary:

```csharp
public sealed record AddHiveApplicationPoolCommand(
    HiveWebServer WebServer,
    HiveEnvironment HiveEnvironment);

public sealed class AddHiveApplicationPoolHandler(IWebServerActions webServerActions)
{
    public Task<AddHiveApplicationPoolResult> Handle(
        AddHiveApplicationPoolCommand command,
        CancellationToken cancellationToken = default)
    {
        webServerActions.AddApplicationPool(command.WebServer, command.HiveEnvironment);
        return Task.FromResult(new AddHiveApplicationPoolResult(true));
    }
}
```

Over-abstracted boundary:

```csharp
public sealed class GetHivePathHandler
{
    public HivePath Handle(GetHivePathQuery query)
    {
        return new HivePath(query.Root, query.EnvironmentName);
    }
}
```

If a cmdlet or endpoint can clearly call the constructor directly, do that instead.
