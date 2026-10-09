---
name: sws2-code-architecture
description: Structure SwiftlyS2 plugins with the official template, src/module/code.cs layout, constructor injection, shared contracts and the framework database provider. Use when creating a plugin or changing its module boundaries, dependencies or packaging.
---

# Plugin architecture

For a new plugin, start with the official `swplugin` template through [sws2-authoring](../sws2-authoring/SKILL.md). Preserve an existing project's layout and operator-defined boundaries when extending it. Use [sws2-api-mcp](../sws2-api-mcp/SKILL.md) to verify each framework boundary.

Keep the plugin entrypoint in `src/PluginName.cs`. It composes services and owns `Load(bool hotReload)` and `Unload()`. Put feature code in `src/<Module>/<Type>.cs`, with folders created when they gain a responsibility. Keep related state and behavior together; introduce another service when it has a separate owner or dependency.

Pass `ISwiftlyCore`, configuration and dependencies through constructors. SwiftlyS2 supplies `AddSwiftly(Core)` for a plugin's service collection. Register attribute-bearing service instances with `Core.Registrator.Register(instance)`; the plugin entrypoint is registered by the framework. Use a partial class only to split one cohesive type.

Give timers, native hooks, subscriptions and asynchronous work an owner with a matching shutdown path. Keep engine objects on the game thread and pass value snapshots to background work. Read [sws2-thread-management](../sws2-thread-management/SKILL.md) when introducing asynchronous boundaries.

Store operator settings in the plugin configuration service and package defaults in `resources/templates`. Keep player text in `resources/translations`. Database configuration selects a connection name from `Core.Database`; shared credentials belong to SwiftlyS2's global database configuration.

Read [layout and packaging](references/layout.md) for the directory tree and template outputs. Read [dependency injection](references/dependency-injection.md) when composing services and [configuration, database and shared contracts](references/integration.md) when adding those dependencies.

Read [convars](references/convars.md) when the feature exposes engine console settings, replicates a value to clients or queries a client convar. Keep plugin JSON configuration and engine convars distinct.
