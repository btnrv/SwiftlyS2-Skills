---
name: sws2-authoring
description: Create, extend or port SwiftlyS2 plugins with the official template, command, event and network APIs, build and live-server validation, and the repository's workflow.
---

# Author a SwiftlyS2 plugin

Read [sws2-api-mcp](../sws2-api-mcp/SKILL.md) before selecting framework APIs and [sws2-code-architecture](../sws2-code-architecture/SKILL.md) for the project structure. Use the [framework terminology](references/terminology.md) explicitly and keep controller, pawn, slot, SteamID and handle distinct. Establish the plugin's behavior, target runtime and one complete player-visible route first. Build that route before adding other features.

Apply the operator's instructions and repository conventions before reference examples. Extend the existing layout and services; the scaffold below is for a new plugin. Add validation required by the behavior as part of that route. Add recovery paths or abstractions when requirements or observed failures justify them. For a framework migration, read [porting](references/porting.md).

## Scaffold

With the .NET SDK required by the chosen template installed:

```text
dotnet new install SwiftlyS2.CS2.PluginTemplate
dotnet new swplugin -n PluginName --PluginName "Plugin Name" --PluginVersion "1.0.0" --PluginAuthor "Author" --PluginDescription "Plugin purpose"
```

For repeatable scaffolding, use the package's `::<version>` install syntax with the chosen template version. Inspect the generated project and pin its SwiftlyS2 package to the deployment. Preserve resource copying, compile exclusions and the publish target. Keep implementation under `src/<Module>/<Type>.cs`.

## Implement

The plugin entrypoint receives `ISwiftlyCore`, implements `Load(bool hotReload)` and `Unload()`, and composes its services. Start resources when their dependencies are ready and dispose them with their owner. Reconstruct required state for connected players when loading into an existing match.

Read only the reference needed by the feature:

| Task | Reference |
| --- | --- |
| Commands, arguments, permissions, aliases or client interception | [Commands and permissions](references/commands-permissions.md) |
| Core events, game events or typed gameplay hooks | [Events and hooks](references/events-hooks.md) |
| Per-player state, authorization, reconnect or hot reload | [Player state](references/player-state.md) |
| Typed network messages, recipients or message hooks | [Network messages](references/netmessages.md) |
| Engine convars, replication or client queries | [Convars](../sws2-code-architecture/references/convars.md) |
| Steam callbacks, server metadata or Workshop queries | [Steamworks](references/steamworks.md) |

Use [sws2-text-styling](../sws2-text-styling/SKILL.md) for feedback and [sws2-menus](../sws2-menus/SKILL.md) for selection flows. Configuration, database access and shared interfaces belong to the architecture skill.

Read [C# conventions](references/csharp.md) for syntax and performance decisions. Load the entity, thread, native, assets, tracing or Panorama skill when the feature reaches that subsystem. Native batching is a separate, explicit user-requested optimization.

## Validate and deliver

Follow the operator's and repository's test policy. Work in focused increments: express the intended behavior in a test, implement it, run it and fix demonstrated failures. Reproduce a reported bug before fixing it. Use the existing test setup for managed behavior; for engine-dependent behavior, specify and exercise a reproducible server scenario. Keep added test scaffolding proportional to the task.

Validate through `dotnet build -c Release`, `dotnet publish -c Release`, archive inspection and observation on the intended CS2 server. Compile success establishes type compatibility; the live server establishes engine behavior.

Exercise the feature's complete route, relevant alive/dead/spectator states, and resource cleanup across the lifecycle it uses. Check logs and the profiler when timing or native work matters. Report which checks ran and which need a running client or server.

Read [server resources](references/server-resources.md) when changing launch options, command permissions, console filtering or core configuration. For performance investigations, use [sws2-performance-profiler](../sws2-performance-profiler/SKILL.md). Follow [CI and source control](references/delivery.md) for build artifacts and branches. Keep commit, push, tag and release actions within the user's authorization and the repository's workflow.
