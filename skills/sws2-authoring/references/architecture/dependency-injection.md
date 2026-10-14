# Dependency injection

Partial feature files share the entrypoint's injected `Core` without needing a container or a service per file. Give a separate component its own type when it has an independent responsibility, dependency set or contract, and pass its dependencies through the constructor. Use a service collection to compose those components or bind options. Keep the provider at the composition root.

Initialize configuration before `AddSwiftly(Core)`. The extension registers the core and framework services, typed loggers and plugin configuration; do not register duplicates. The template references `Microsoft.Extensions.DependencyInjection` with the framework runtime exclusion.

## Configuration and a separate command bundle

This optional status-command bundle uses constructor injection and attribute registration. The partial feature in [layout](layout.md) can keep its existing structure.

Put this composition fragment inside an existing `BasePlugin` entrypoint, with its `using` directives at file scope. Replace the entrypoint's `Load` and `Unload` methods when trying the example.

```csharp
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using SwiftlyS2.Shared;

private ServiceProvider? _services;

public override void Load(bool hotReload)
{
    Core.Configuration
        .InitializeWithTemplate("config.jsonc", "config.jsonc")
        .Configure(builder => builder.AddJsonFile(
            Core.Configuration.GetConfigPath("config.jsonc"),
            optional: false,
            reloadOnChange: true));

    var collection = new ServiceCollection();
    collection.AddSwiftly(Core);
    collection.AddOptions<PluginConfig>().BindConfiguration("Main");
    collection.AddSingleton<StatusCommands>();
    _services = collection.BuildServiceProvider();
    Core.Registrator.Register(_services.GetRequiredService<StatusCommands>());
}

public override void Unload()
{
    _services?.Dispose();
}
```

`src/Configuration/PluginConfig.cs`:

```csharp
namespace ExamplePlugin;

public sealed class PluginConfig
{
    public string ServerLabel { get; set; } = "Community";
}
```

`src/Features/Status/StatusCommands.cs`:

```csharp
using Microsoft.Extensions.Options;
using SwiftlyS2.Shared;
using SwiftlyS2.Shared.Commands;

namespace ExamplePlugin;

public sealed class StatusCommands
{
    private readonly ISwiftlyCore _core;
    private readonly IOptionsMonitor<PluginConfig> _config;

    public StatusCommands(ISwiftlyCore core, IOptionsMonitor<PluginConfig> config)
    {
        _core = core;
        _config = config;
    }

    [Command("status", helpText: "Show this server's label.")]
    public void OnStatus(ICommandContext context)
    {
        var localizer = context.Sender is { } player
            ? _core.Translation.GetPlayerLocalizer(player)
            : _core.Localizer;
        context.Reply(localizer["status.message", _config.CurrentValue.ServerLabel]);
    }
}
```

`resources/templates/config.jsonc`:

```jsonc
{
    "Main": {
        "ServerLabel": "Community"
    }
}
```

Add this key to `resources/translations/en.jsonc`:

```jsonc
{
    "status.message": "Server: {0}."
}
```

`AddSingleton<StatusCommands>` registers construction; `GetRequiredService<StatusCommands>` creates or retrieves the instance. Inject `ILogger<StatusCommands>` if the component needs logging. Read `IOptionsMonitor<T>.CurrentValue` when handlers need reloaded settings, and subscribe to `OnChange` only when a change must trigger work.

## Registration and lifetime

Register each instance with attribute handlers once, either at the composition root as shown or inside the component if that is the project's convention. SwiftlyS2 registers the plugin entrypoint automatically. Features that can stop independently need programmatic registrations and matching release APIs because the public registrator has no object-level `Unregister` counterpart.

The provider disposes disposable components it creates. Each component must release its owned resources during disposal; the provider cannot infer how to cancel tasks or detach subscriptions to another plugin's events. Use the framework's registration cleanup and release owned work when its lifetime ends. Follow [thread management](../../../sws2-thread-management/SKILL.md) for asynchronous lifetimes.

`AddSwiftly` installs a custom options factory, so a configured list or dictionary replaces its code defaults. Initialize configuration to establish `Core.Configuration.BasePathExists` before registration. See [integration](integration.md) for configuration, database and shared-interface rules.

References: [Dependency injection guide](https://swiftlys2.net/docs/guides/dependency-injection), [SwiftlyCoreInjection](https://github.com/swiftly-solution/swiftlys2/blob/master/managed/src/SwiftlyS2.Shared/SwiftlyCoreInjection.cs).
