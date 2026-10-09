# Dependency injection

Create the plugin's service collection in `Load`. Initialize its configuration first, then call `AddSwiftly(Core)`. That extension registers the core and framework services, typed loggers and the plugin configuration for constructor injection. The current template already references `Microsoft.Extensions.DependencyInjection` with the framework runtime exclusion.

This composition fragment assumes a `PluginConfig` model, a `FeatureService` class, and a packaged `resources/templates/config.jsonc` containing the `Main` section:

```csharp
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;

private ServiceProvider? services;

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
    collection.AddSingleton<FeatureService>();
    services = collection.BuildServiceProvider();
    Core.Registrator.Register(services.GetRequiredService<FeatureService>());
}

public override void Unload()
{
    services?.Dispose();
}
```

`AddSingleton<FeatureService>` registers construction; `GetRequiredService<FeatureService>` creates or retrieves the instance. Its constructor can take `ISwiftlyCore`, `ILogger<FeatureService>` and `IOptionsMonitor<PluginConfig>` directly. Use `CurrentValue` when a handler should see reloaded settings, and keep an `OnChange` subscription only when the feature needs work on a change.

Register each attribute-bearing instance once. The example registers at the composition root; a service may instead register itself through an injected core if the project consistently uses that convention. SwiftlyS2 registers the plugin entrypoint separately.

Keep plugin-owned service lifetimes explicit. Container-created disposable services are disposed with the provider. A service that owns subscriptions, timers or other resources releases them in its disposal path; the provider does not infer how to undo arbitrary event subscriptions. Long-running work still needs the thread-management skill's lifetime rules.

`AddSwiftly` installs a custom options factory: a configured list or dictionary replaces its code defaults. Configuration registration also depends on `Core.Configuration.BasePathExists`, which initialization establishes. Apply those details when choosing between a packaged template and typed defaults.

Sources: [dependency injection guide](https://swiftlys2.net/docs/guides/dependency-injection), configuration guide in the bundled docs, and `managed/src/SwiftlyS2.Shared/SwiftlyCoreInjection.cs` at [78b4c89](https://github.com/swiftly-solution/swiftlys2/blob/78b4c89a6e21de7b6a4d9485295b6448f58e26d9/managed/src/SwiftlyS2.Shared/SwiftlyCoreInjection.cs).
