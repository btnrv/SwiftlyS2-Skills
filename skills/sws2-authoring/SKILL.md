---
name: sws2-authoring
description: Create, extend, port or structure SwiftlyS2 C# plugins for Counter-Strike 2 (CS2). Covers the swplugin template, reusing existing framework methods and object properties, feature folders and partial classes, commands and permissions, events and game hooks, per-player state, network messages, convars, JSONC configuration, databases, shared plugin APIs, dependency injection, Steamworks, tests, build and publish. Use for any SwiftlyS2 or CS2 server plugin work, including porting from CounterStrikeSharp and reviewing plugin code for redundant `IsValid`/`IsAlive`/team checks, even if the user only says "make a plugin", "add a command", "clean this up" or describes a gameplay feature.
---

# Author a SwiftlyS2 plugin

Never reimplement functionality SwiftlyS2 already provides. Before writing custom helpers, use [sws2-api-mcp](../sws2-api-mcp/SKILL.md) to verify the services, extensions and domain types available in the target version. Write plugin code for gameplay rules and coordination. Add custom infrastructure only for a verified framework gap.

Before creating a plugin or changing its architecture, read:

- [Framework reuse](references/architecture/framework-reuse.md): discovery before implementation and direct use of existing capabilities.
- [Framework guarantees](references/architecture/guarantees.md): checks the framework already answered, reading properties once, and the shape of the checks that remain.
- [C# conventions](references/architecture/csharp.md): syntax, naming, typing and focused methods.
- [Layout and packaging](references/architecture/layout.md): feature folders, partial classes and a complete command example.

Use the [framework terminology](references/terminology.md) and distinguish controller, pawn, slot, SteamID and handle. Define the plugin's behavior and target runtime, then build one complete player-visible route before adding features. Include cleanup from the start. Validate only what neither the framework nor the current code path already guarantees. Avoid speculative exception handlers, retries, fallbacks and abstractions, and harden only against demonstrated failures.

## Scaffold

Install the .NET SDK required by the chosen template, then run:

```text
dotnet new install SwiftlyS2.CS2.PluginTemplate
dotnet new swplugin -n PluginName --PluginName "Plugin Name" --PluginVersion "1.0.0" --PluginAuthor "Author" --PluginDescription "Plugin purpose"
```

For repeatable scaffolding, pin the template with `SwiftlyS2.CS2.PluginTemplate@<version>`. Inspect the generated project and pin its SwiftlyS2 package to the deployment. Preserve resource copying, compile exclusions and the publish target. Keep feature code under `src/Features/<Feature>/<Subfeature>/` and create folders as needed.

## Implement

The plugin entrypoint receives `ISwiftlyCore` and coordinates configuration, feature registration, `Load(bool hotReload)` and `Unload()`. Prefer focused partial-class files for gameplay, with each feature's state, handlers and cleanup together. Use separate types for models, records, integrations and components with their own ownership. Pass their dependencies through constructors. A feature does not need a service layer just because it has its own folder.

Follow the framework's registration and cleanup contracts. Start resources when their dependencies are ready and stop plugin-owned work when its owner stops. The framework already stops what it owns, such as scheduler timers when the plugin unloads; don't cancel those again in `Unload()`. When loading into an existing match, reconstruct any required state for connected players.

Know the full member list of every SwiftlyS2 object the plugin handles, not just its methods. Properties, fields and events on `IPlayer`, event and hook arguments, schema entities and services often already hold the value a feature needs. Look up the complete type, keep its members in mind for the rest of the task and read state from the object that has it instead of deriving or duplicating it. See [object members](references/architecture/framework-reuse.md#know-every-member-of-the-objects-you-hold).

Use existing API and schema types and their named members for game states, flags and identifiers, including the `Team` enum for team checks. Read [game values and domain types](references/architecture/game-values.md) before using those values.

Trust what a framework call already established. Players from `GetAlive()`, `GetCT()` and the other filtered queries are valid (and alive or on that team where the query says so). `FindTargettedPlayers` enforces its search-mode flags. `IsAlive` includes `IsValid`, and a non-null `CHandle.Value` is a live entity. Do not re-check these in the same tick. Do re-check after an `await`, timer or callback. Read a player or entity property once per method; `SteamID` is a native call and `PlayerPawn` resolves a handle on every read. The full list is in [framework guarantees](references/architecture/guarantees.md).

| Task | Reference |
| --- | --- |
| Port an existing plugin | [Porting](references/porting.md) |
| Commands, arguments, permissions, aliases or client interception | [Commands and permissions](references/commands-permissions.md) |
| Core events, game events or typed gameplay hooks | [Events and hooks](references/events-hooks.md) |
| Per-player state, authorization, reconnect or hot reload | [Player state](references/player-state.md) |
| Typed network messages, recipients or message hooks | [Network messages](references/netmessages.md) |
| Engine convars, replication or client queries | [Convars](references/architecture/convars.md) |
| Configuration, database access or shared contracts | [Integration](references/architecture/integration.md) |
| Separately owned components or options binding | [Dependency injection](references/architecture/dependency-injection.md) |
| Steam callbacks, server metadata or Workshop queries | [Steamworks](references/steamworks.md) |

Use [sws2-text-styling](../sws2-text-styling/SKILL.md) for feedback and [sws2-menus](../sws2-menus/SKILL.md) for selection flows. Read the entity, thread, assets, tracing or Panorama skill when the feature uses that subsystem. Use [native interop](../sws2-native-and-memory/SKILL.md) only for a verified managed-API gap. Investigate native batching only when the user explicitly requests it.

## Validate and deliver

Write a test for the intended behavior, implement the change, then run the test and fix failures. For a bug fix, start with a test that reproduces the bug. Test managed logic with the repository's existing setup and reproduce engine behavior on the server. Add only the test scaffolding the change needs.

Run `dotnet build -c Release` and `dotnet publish -c Release`, inspect the archive, and check behavior on the intended CS2 server. A successful build verifies type compatibility. Engine behavior requires a live-server check.

Exercise the feature's complete route, relevant alive/dead/spectator states, and cleanup at the lifecycle boundaries it uses. Review custom helpers against the framework APIs and remove duplicated functionality, renamed framework calls and checks the framework already guarantees. Check logs, then report which checks ran and which still need a running client or server.

Read [server resources](references/server-resources.md) when changing launch options, command permissions, console filtering or core configuration. For performance investigations, use [sws2-performance-profiler](../sws2-performance-profiler/SKILL.md). Follow [CI and source control](references/delivery.md) for build artifacts and branches. Keep commit, push, tag and release actions within the user's authorization and the repository's workflow.
