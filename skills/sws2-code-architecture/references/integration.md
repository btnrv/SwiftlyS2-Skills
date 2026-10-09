# Configuration, database and shared contracts

## Configuration

Initialize defaults through `Core.Configuration.InitializeWithTemplate("config.jsonc", "config.jsonc")`, or initialize a typed model with `InitializeJsonWithModel<T>(file, section)`. Use the full path from `GetConfigPath` when registering an `AddJsonFile` source. Initialize configuration before `AddSwiftly(Core)`: its configuration registration depends on the plugin config directory existing. Bind configuration through the plugin service collection and use `IOptionsMonitor<T>` when live reload is part of the feature.

Keep defaults in packaged JSONC when operators need to edit text or presentation without recompiling. Configured lists and dictionaries replace their code defaults through SwiftlyS2's options factory; write complete collection values in the configuration. Dispose change subscriptions with their owning service. Read [dependency injection](dependency-injection.md) for composition and registration.

## Database

The shared credentials provider is `Core.Database` (`IDatabaseService`). Global `addons/swiftlys2/configs/database.jsonc` holds named connections and `default_connection`. The plugin configuration needs only its selected connection name.

`GetConnection(name)` returns an `IDbConnection`; dispose it after the operation. `GetConnectionString(name)` supports an ORM that needs a connection string, and `GetConnectionInfo(name)` returns parsed driver metadata. These methods fall back to the global default for an unknown name, so configure and verify the intended connection during deployment.

Run database I/O asynchronously or on a worker, passing copied player IDs and values. Parameterize SQL. Schedule the result's engine changes back on the game thread after resolving the current player and plugin lifetime. Use the database library's actual asynchronous API; `IDbConnection` itself has no `OpenAsync` method. Connection strings and metadata contain credentials and belong outside logs and source-controlled plugin settings.

## Shared API lifecycle

Keep the shared interface in the contract assembly described in [layout](layout.md). Use a stable versioned key, such as `PluginName.Service.v1`.

| Callback | Work |
| --- | --- |
| `ConfigureSharedInterface(IInterfaceManager)` | Publish with `AddSharedInterface<TInterface, TImpl>(key, instance)` |
| `UseSharedInterface(IInterfaceManager)` | Resolve with `GetSharedInterface<T>(key)` for a required provider, or `TryGetSharedInterface<T>(key, out value)` for an optional one |
| `OnSharedInterfaceInjected(IInterfaceManager)` | Start work that requires completed dependency resolution |
| `OnAllPluginsLoaded()` | Run work after the shared-interface phases have completed across loaded plugins |

`Load(bool hotReload)` composes the plugin before the shared-interface phases. Resolve other plugins in `UseSharedInterface` or `OnSharedInterfaceInjected`, when the rebuilt registry is available. The framework runs each phase across the plugin set before the next phase and reruns integration when that set changes. The registry is rebuilt, so publish interfaces on every `ConfigureSharedInterface` call. Replace old provider references and detach old event subscriptions before subscribing to a replacement. Provider methods should define their threading and lifetime contract, especially when they expose engine operations.

References: [Configuration](https://swiftlys2.net/docs/development/configuration), [database](https://swiftlys2.net/docs/development/database), [shared API](https://swiftlys2.net/docs/development/shared-api), [dependency injection](https://swiftlys2.net/docs/guides/dependency-injection).
