# Configuration, database and shared contracts

## Configuration

Initialize defaults with `Core.Configuration.InitializeWithTemplate("config.jsonc", "config.jsonc")`, or initialize a typed model with `InitializeJsonWithModel<T>(file, section)`. Use the full path from `GetConfigPath` when registering an `AddJsonFile` source. Configuration must be initialized before `AddSwiftly(Core)` because registration needs the plugin config directory to exist. Bind configuration through the plugin service collection and use `IOptionsMonitor<T>` for live reload.

Keep defaults in packaged JSONC when operators need to edit text or presentation without recompiling. Configured lists and dictionaries replace their code defaults through SwiftlyS2's options factory; write complete collection values in the configuration. Dispose change subscriptions with their owning service. Read [dependency injection](dependency-injection.md) for composition and registration.

## Database

Access shared database credentials through `Core.Database` (`IDatabaseService`). The global `addons/swiftlys2/configs/database.jsonc` holds named connections and `default_connection`. Store only the selected connection name in plugin configuration.

`GetConnection(name)` returns an `IDbConnection`; dispose it after the operation. `GetConnectionString(name)` supports an ORM that needs a connection string, and `GetConnectionInfo(name)` returns parsed driver metadata. These methods fall back to the global default for an unknown name, so configure and verify the intended connection during deployment.

Run database I/O asynchronously or on a worker, passing copies of player IDs and values. Parameterize SQL. Apply results on the game thread after resolving the current player and checking the plugin's lifetime. Use the database library's asynchronous API; `IDbConnection` has no `OpenAsync` method. Keep connection strings and metadata out of logs and source-controlled plugin settings because they contain credentials.

## Shared API lifecycle

Keep the shared interface in the contract assembly described in [layout](layout.md). Use a stable versioned key, such as `PluginName.Service.v1`.

| Callback | Work |
| --- | --- |
| `ConfigureSharedInterface(IInterfaceManager)` | Publish with `AddSharedInterface<TInterface, TImpl>(key, instance)` |
| `UseSharedInterface(IInterfaceManager)` | Resolve with `GetSharedInterface<T>(key)` for a required provider, or `TryGetSharedInterface<T>(key, out value)` for an optional one |
| `OnSharedInterfaceInjected(IInterfaceManager)` | Start work that requires completed dependency resolution |
| `OnAllPluginsLoaded()` | Run work after the shared-interface phases have completed across loaded plugins |

`Load(bool hotReload)` composes the plugin before the shared-interface phases. Resolve other plugins in `UseSharedInterface` or `OnSharedInterfaceInjected`, when the rebuilt registry is available. SwiftlyS2 completes each phase across the plugin set before starting the next, and reruns integration when the set changes. Publish interfaces on every `ConfigureSharedInterface` call because the registry is rebuilt. Replace old provider references and detach old event subscriptions before subscribing to the replacement. Document threading and lifetime requirements for provider methods, especially those that expose engine operations.

References: [Configuration](https://swiftlys2.net/docs/development/configuration), [database](https://swiftlys2.net/docs/development/database), [shared API](https://swiftlys2.net/docs/development/shared-api), [dependency injection](https://swiftlys2.net/docs/guides/dependency-injection).
