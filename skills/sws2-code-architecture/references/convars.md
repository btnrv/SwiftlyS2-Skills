# Console variables

Use `Core.ConVar` for values that operators need to inspect or change through the console. Keep structured settings in the existing configuration service. Create or resolve a feature's convars in its owning module once its constructor-supplied core is ready; this does not require another module layout or partial entrypoint.

## Resolve the type and ownership

- `Find<T>(name)` returns `IConVar<T>?`; the type must match the engine variable. For an unknown type, use `FindAsString(name)` and `IConVar.ValueAsString`.
- `Create<T>(name, helpMessage, defaultValue, flags)` requires a new name and throws if it exists.
- `CreateOrFind<T>(...)` deliberately reuses an existing name, useful across reloads. Existing values, defaults, bounds and flags are retained; creation arguments do not reconfigure an existing variable.

Namespace plugin-owned names, for example `sw_example_max_uses`. Both creation methods support `bool`, the signed and unsigned 16/32/64-bit integer types, `float`, `double`, `string`, `Color`, `QAngle`, `Vector`, `Vector2D`, and `Vector4D`. Match the generic type to the default (`1.0f` for `float`).

For a numeric range, the overload order is `defaultValue, minValue, maxValue, flags`; either nullable bound can be omitted with `null`. Although the signature permits `T : unmanaged`, [range creation](https://github.com/swiftly-solution/swiftlys2/blob/78b4c89a6e21de7b6a4d9485295b6448f58e26d9/managed/src/SwiftlyS2.Core/Modules/Convars/ConVarService.cs#L128) only supports `short`, `ushort`, `int`, `uint`, `long`, `ulong`, `float`, and `double`. It rejects ranged `bool`, vector, angle and color creation.

Inside the owning feature's initialization, with constructor-supplied `core`:

```csharp
using SwiftlyS2.Shared.Convars;

IConVar<int> maxUses = core.ConVar.CreateOrFind<int>(
    "sw_example_max_uses", "Uses allowed per round.",
    3, 0, 10, ConvarFlags.NONE);
```

Choose defaults, bounds and any sentinel meanings from the feature contract. Read optional metadata with `TryGetMinValue`, `TryGetMaxValue`, and `TryGetDefaultValue`; the non-generic equivalents end in `AsString`.

## Server values and client values

| Operation | Effect |
| --- | --- |
| `convar.Value = value` / `ValueAsString = text` | Queues a server value change; automatically replicates if the convar is replicated. |
| `convar.SetInternal(value)` / `SetInternalAsString(text)` | Writes internally without client replication. Use when the value must apply immediately within the current hook. |
| `convar.ReplicateToClient(playerId, value)` | Sends that value to one client without changing the stored server value. The non-generic method is `ReplicateToClientAsString`. |
| `core.ConVar.ReplicateToClient(playerId, name, text)` / `ReplicateToAll(name, text)` | Sends a named client value; the name need not exist on the server. |
| `convar.QueryClient(playerId, callback)` / `core.ConVar.QueryClient(playerId, name, callback)` | Requests a client value and later calls `Action<string>`, including for a typed wrapper. |

Replication is a network operation, not proof that a particular client variable accepts the change. Verify the target name/flags with the convar MCP tools and test the intended client effect. If the feature temporarily overrides a client value, give restoring that value a defined lifecycle too.

Client query callbacks have no cancellation handle, timeout or success-status argument. [Pending callbacks](https://github.com/swiftly-solution/swiftlys2/blob/78b4c89a6e21de7b6a4d9485295b6448f58e26d9/managed/src/SwiftlyS2.Core/Modules/Convars/ConVarService.cs#L306) are kept in a static queue per `(playerId, name)` and removed when a reply arrives; there is no plugin-unload cleanup in that path. Do not model this as guaranteed completion. When a reply drives live player state, confirm that the owner is still active and that the current player's identity matches the request. Keep captured state small; stopping the owner does not remove a pending callback from the framework.

For change-driven behavior, subscribe to `Core.Event.OnConVarValueChanged`, filter by the exact owned `ConVarName`, and remove the same delegate in the owner's shutdown. The event exposes old/new values as strings. `IConVarService` has no public destroy/unregister operation for plugin variables; do not invent a disposal method for the wrapper.

## Flag semantics

`ConvarFlags` describes engine behavior and is not the SwiftlyS2 permission system. Start with `NONE` when no flag behavior is required.

| Flag | Meaning relevant to authoring |
| --- | --- |
| `REPLICATED` | Enforces the server setting on clients through normal replicated writes. |
| `NOTIFY` | Announces value changes to players. |
| `CHEAT` | Restricts use to the engine's cheat/debug conditions. |
| `PROTECTED` | Masks the value sent for sensitive variables; it is not an admin access check. |
| `SERVER_CAN_EXECUTE` | Allows the server to execute the command on clients; it does not grant administrator-only modification. |
| `SERVER_CANNOT_QUERY` | Prevents the server from querying the client variable. |

Apply command permissions at the controlling command or access layer. Runtime value changes do not update the plugin's JSON configuration; document an operator's required persistence route only when the feature needs one.

API details: [convar service](https://swiftlys2.net/api-docs/stable/convars/iconvarservice), [typed convars](https://swiftlys2.net/api-docs/stable/convars/iconvar-1), and [flags](https://swiftlys2.net/api-docs/stable/convars/convarflags).
