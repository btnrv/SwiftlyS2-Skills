# Reuse SwiftlyS2 before writing code

Never reimplement a capability supplied by the target SwiftlyS2 package. Use its services, extension methods, typed wrappers, enums and constants. Performance preferences, familiarity with another framework or a shorter native implementation do not justify replacing a supported API.

## Resolve the operation first

1. Describe the required behavior, then use [sws2-api-mcp](../../../sws2-api-mcp/SKILL.md) to find the documented workflow and declarations for the target version.
2. Search the owning service, the object receiving the operation and related extensions. Search by behavior as well as the suspected method name. One empty lookup does not establish that the capability is missing.
3. Verify arguments, return types, side effects, registration ownership and threading. Use the complete framework operation when it meets the requirement; do not reconstruct it from lower-level fields.
4. Add the plugin-specific rules and coordination. If the framework leaves a gap, identify and implement only the missing behavior. Inspect the matching package or source definitions before using native interop.

Make these checks before introducing a helper, wrapper, cache, registry, parser, scheduler or native call. A wrapper should enforce plugin policy or combine operations into a feature. Forwarding a call under a new name adds no behavior. Keep helpers local until multiple features need them.

## Where to look

| Need | Reuse |
| --- | --- |
| Player state and selection | `IPlayer` properties and `Core.PlayerManager` queries, including `IsAlive` and `GetAlive()` |
| Commands, aliases, caller replies and permissions | `Core.Command`, `ICommandContext.Reply` and declared command permissions; `Core.Permission` for other entrypoints |
| Engine convar access | `Core.ConVar` and its typed/string wrappers; see [convars](convars.md) |
| Game states, flags and identifiers | Existing API/schema/protobuf definitions; see [game values](game-values.md) |
| Delayed or game-thread work | `Core.Scheduler` and the documented cancellation/map-lifetime operations |
| Configuration, translation and database connections | `Core.Configuration`, `Core.Translation`, `Core.Localizer` and `Core.Database` |
| Entities, messages, hooks and menus | The corresponding typed framework service and generated definitions |

The table lists starting points for API discovery. Verify each operation against the plugin's package and check for capabilities beyond those listed here.

With an `ISwiftlyCore core`, this fragment runs inside a method on the game thread:

```csharp
foreach (var player in core.PlayerManager.GetAlive())
{
    var localizer = core.Translation.GetPlayerLocalizer(player);
    player.SendChat(localizer["announcement.ready"]);
}
```

Add `announcement.ready` to the plugin's translation resources. The framework handles alive-player selection, language lookup and chat delivery; do not duplicate its filters or translation loader. For a single player, use `player.IsAlive`. Compare raw life-state enums only when the behavior needs those distinct states.

## Know every member of the objects you hold

Reuse applies to properties, fields and events as much as to methods. Each SwiftlyS2 object a plugin receives (`IPlayer`, a command context, event or hook arguments, a schema entity, a menu, a convar, a service) already exposes state through its members. Before reading that state another way, look up the complete type with `apidocs_lookup` and go through all of its members, not only the method you came for. For schema classes, also check `schema_lookup` and the base classes. Keep the member list of each type you work with in mind for the rest of the task, so later code reuses it.

Read a value from the object that already has it. Do not derive it from lower-level fields, track it in a plugin-side copy, or hand it around as a separate parameter when the object you already pass carries it. This applies only to SwiftlyS2 types and the game types it wraps. It is not a reason to inspect every .NET type in the project.

`IPlayer` illustrates the point. Among its properties are `IsFakeClient`, `IsAuthorized`, `IsFirstSpawn`, `PlayerLanguage`, `PressedButtons`, `SteamID`, `Slot`, `Controller`, `Pawn`, `PlayerPawn` and `RequiredPlayerPawn`. A plugin that only knew `SendChat` would end up checking SteamIDs to find bots, reading movement services for buttons and resolving the pawn handle from the controller. This fragment uses the properties instead:

```csharp
using SwiftlyS2.Shared.Events;

foreach (var player in core.PlayerManager.GetAlive())
{
    if (player.IsFakeClient)
        continue;

    if ((player.PressedButtons & GameButtonFlags.E) == GameButtonFlags.None)
        continue;

    var pawn = player.PlayerPawn;
    // Gameplay rule using the pawn.
}
```

`PlayerPawn` is nullable; use `RequiredPlayerPawn` only where a missing pawn is a bug, because it throws. The same check covers events and hooks: read the payload and argument members the callback provides before querying the same data through another service.

## Review custom code

Before delivery, review new helpers, direct engine-field access and native operations. Replace duplicated framework operations, properties, enums, permission lookups, command parsers and lifecycle mechanisms with the verified APIs. Keep custom gameplay rules and required ownership checks. Add no infrastructure for hypothetical future failures.

Public references: [Swiftly core](https://swiftlys2.net/docs/development/swiftly-core), [player API](https://swiftlys2.net/api-docs/stable/players/iplayer), [player manager](https://swiftlys2.net/api-docs/stable/players/iplayermanagerservice).
