# Console variables

Use `Core.ConVar` to retrieve and change engine convars or create plugin-owned console settings. Keep structured plugin settings in the configuration service. Partial features use the entrypoint's inherited `Core`; separate components receive the core through their constructors. Use this service wherever it covers the operation, without adding a convar registry or issuing console command strings.

## Retrieve an existing engine variable

Use [sws2-api-mcp](../../../sws2-api-mcp/SKILL.md) to resolve the convar's name, type and flags, then confirm the wrapper declaration for the target package. `Find<T>` requires the engine's exact type. A value printed as `0` or `1` is not necessarily an `int`.

This fragment uses an `ISwiftlyCore core` for a feature that requires `sv_cheats`, whose documented type is `bool`. Put the `using` directives at file scope and the remaining statements in the feature's initialization method:

```csharp
using System;
using SwiftlyS2.Shared.Convars;

IConVar<bool>? cheats = core.ConVar.Find<bool>("sv_cheats");
if (cheats is null)
{
    throw new InvalidOperationException("Required convar 'sv_cheats' is unavailable.");
}

bool cheatsEnabled = cheats.Value;
```

Read the current `Value` when the feature needs the live setting. Keep a resolved wrapper only while it is valid; a value copied during initialization will not track later changes. Handle a missing variable according to whether the feature requires it. Do not create a missing engine variable with `CreateOrFind` or silently substitute a default. Add retries only for a demonstrated lifecycle requirement.

For untyped or string access, choose `FindAsString(name)` and read `ValueAsString`. Choose this interface deliberately; do not use it as an automatic fallback when a typed lookup fails. Apply the same distinction to writes.

## Create plugin-owned variables

- `Create<T>(name, helpMessage, defaultValue, flags)` requires a new name and throws if it exists.
- `CreateOrFind<T>(...)` reuses an existing name across reloads and retains its values, defaults, bounds and flags. Creation arguments do not reconfigure an existing variable.

Namespace plugin-owned names, for example `sw_example_max_uses`. Both creation methods support `bool`, the signed and unsigned 16/32/64-bit integer types, `float`, `double`, `string`, `Color`, `QAngle`, `Vector`, `Vector2D`, and `Vector4D`. Match the generic type to the default (`1.0f` for `float`).

For a numeric range, the overload order is `defaultValue, minValue, maxValue, flags`; pass `null` to omit either bound. The bounds overload requires `T : unmanaged`, so strings use the overload without bounds. The [1.4.14 implementation](https://github.com/swiftly-solution/swiftlys2/blob/bfffbdb137b94eb7dc0e06786b499a185066e285/managed/src/SwiftlyS2.Core/Modules/Convars/ConVarService.cs#L74) forwards typed bounds to the engine. Check the engine's behavior for the chosen type; the generic constraint alone does not establish it.

With an `ISwiftlyCore core`, this fragment creates a plugin-owned variable inside a method:

```csharp
using SwiftlyS2.Shared.Convars;

IConVar<int> maxUses = core.ConVar.CreateOrFind<int>(
    "sw_example_max_uses", "Uses allowed per round.",
    3, 0, 10, ConvarFlags.NONE);
```

Define what the defaults, bounds and sentinel values mean for the feature. Read optional metadata with `TryGetMinValue`, `TryGetMaxValue`, and `TryGetDefaultValue`; the non-generic equivalents end in `AsString`.

## Server values and client values

| Operation | Effect |
| --- | --- |
| `convar.Value = value` / `ValueAsString = text` | Queues a server value change; automatically replicates if the convar is replicated. |
| `convar.SetInternal(value)` / `SetInternalAsString(text)` | Writes internally without client replication. Use when the value must apply immediately within the current hook. |
| `convar.ReplicateToClient(playerId, value)` | Sends that value to one client without changing the stored server value. The non-generic method is `ReplicateToClientAsString`. |
| `core.ConVar.ReplicateToClient(playerId, name, text)` / `ReplicateToAll(name, text)` | Sends a named client value; the name need not exist on the server. |
| `convar.QueryClient(playerId, callback)` / `core.ConVar.QueryClient(playerId, name, callback)` | Requests a client value and later calls `Action<string>`, including for a typed wrapper. |

Replication sends a network message; the client may still keep its existing value. Check the target name and flags with the convar MCP tools, then test the effect on the client. Define when to restore any client value the feature temporarily overrides.

Client query callbacks have no cancellation handle, timeout or success-status argument. A query may never complete. The framework stores [pending callbacks](https://github.com/swiftly-solution/swiftlys2/blob/bfffbdb137b94eb7dc0e06786b499a185066e285/managed/src/SwiftlyS2.Core/Modules/Convars/ConVarService.cs#L154) in a static queue per `(playerId, name)` and removes them when a reply arrives. They remain queued if the owner stops or the plugin unloads, so keep captured state small. Before applying a reply to live player state, check that the owner is active and the current player's identity matches the request.

Use `IPlayer.GetClientConvarValue` when the server's stored client userinfo meets the requirement. Use `QueryClient` when you need to query the client. Check which applies to the target convar.

Subscribe to `Core.Event.OnConVarValueChanged` to react to changes, filtering by the exact `ConVarName`. SwiftlyS2 clears plugin listeners on unload; remove the same delegate if a feature stops earlier. The event exposes old and new values as strings. `IConVarService` has no public destroy or unregister operation for plugin variables.

## Flag semantics

Use `ConvarFlags` for engine behavior and command permissions for access control. Start with `NONE` when no flag behavior is required.

| Flag | Effect |
| --- | --- |
| `REPLICATED` | Enforces the server setting on clients through normal replicated writes. |
| `NOTIFY` | Announces value changes to players. |
| `CHEAT` | Restricts use to the engine's cheat/debug conditions. |
| `PROTECTED` | Masks the value sent for sensitive variables; it is not an admin access check. |
| `SERVER_CAN_EXECUTE` | Allows the server to execute the command on clients; it does not grant administrator-only modification. |
| `SERVER_CANNOT_QUERY` | Prevents the server from querying the client variable. |

Apply permissions at the controlling command or access layer. Runtime value changes do not update the plugin's JSON configuration. When settings need to persist, document how operators should save them.

API details: [convar service](https://swiftlys2.net/api-docs/stable/convars/iconvarservice), [typed convars](https://swiftlys2.net/api-docs/stable/convars/iconvar-1), and [flags](https://swiftlys2.net/api-docs/stable/convars/convarflags).
