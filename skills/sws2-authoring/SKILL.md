---
name: sws2-authoring
description: Create and deliver SwiftlyS2 plugins with the official template, modern C# conventions, build and live-server validation, CI and the repository's branch workflow. Use for a new plugin or a substantial plugin feature.
---

# Author a SwiftlyS2 plugin

Read [sws2-api-mcp](../sws2-api-mcp/SKILL.md) before selecting framework APIs and [sws2-code-architecture](../sws2-code-architecture/SKILL.md) for the project structure. Use the [framework terminology](references/terminology.md) explicitly and keep controller, pawn, slot, SteamID and handle distinct. Establish the plugin's behavior, target runtime and one complete player-visible route first. Build that route before adding other features.

## Scaffold

With the .NET SDK required by the chosen template installed:

```text
dotnet new install SwiftlyS2.CS2.PluginTemplate
dotnet new swplugin -n PluginName --PluginName "Plugin Name" --PluginVersion "1.0.0" --PluginAuthor "Author" --PluginDescription "Plugin purpose"
```

For repeatable scaffolding, use the package's `::<version>` install syntax with a verified template version. Inspect the generated project and pin its SwiftlyS2 package to the deployment. Preserve resource copying, compile exclusions and the publish target. Keep implementation under `src/<Module>/<Type>.cs`.

## Implement

The plugin entrypoint receives `ISwiftlyCore`, implements `Load(bool hotReload)` and `Unload()`, and composes its services. Start resources when their dependencies are ready and dispose them with their owner. Reconstruct required state for connected players when loading into an existing match.

Use documented attributes for fixed command and event handlers, or programmatic registration when configuration determines registration at runtime. Register service instances containing attributes through `Core.Registrator`. Confirm callback signatures and event phases through MCP; a pre-hook, post-hook and asynchronous continuation have different lifetimes.

Player commands handle the server-console case through `ICommandContext.IsSentByPlayer`. Declare permission requirements using the command API. Choose config-driven aliases where the feature requires operator control. Use [sws2-text-styling](../sws2-text-styling/SKILL.md) for feedback and [sws2-menus](../sws2-menus/SKILL.md) for selection flows.

Read [C# conventions](references/csharp.md) for syntax and performance decisions. Load the entity, thread, native, assets, tracing or Panorama skill when the feature reaches that subsystem. Native batching is a separate, explicit user-requested optimization.

## Validate and deliver

Validate through `dotnet build -c Release`, `dotnet publish -c Release`, archive inspection and observation on the intended CS2 server. Keep automated test projects and test runners out of the plugin scaffold and CI. Compile success establishes type compatibility; the live server establishes engine behavior.

Exercise the feature's complete route, relevant alive/dead/spectator states, and resource cleanup across the lifecycle it uses. Check logs and the profiler when timing or native work matters. Report which checks ran and which need a running client or server.

Read [server resources](references/server-resources.md) when changing launch options, command permissions, console filtering or core configuration. For performance investigations, use [sws2-performance-profiler](../sws2-performance-profiler/SKILL.md). Follow [CI and source control](references/delivery.md) for build artifacts and branches. Keep commit, push, tag and release actions within the user's authorization and the repository's workflow.
