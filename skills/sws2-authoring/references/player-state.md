# Player state and connection lifetime

A `Dictionary<int, PlayerState>` keyed by `PlayerID` can hold a feature's state when handlers and state updates run on the game thread. Include the connection's `SessionId` when work crosses callbacks or an `await`. Keep related fields in one type in the feature's module.

## Choose the identity

| Identity | Use |
| --- | --- |
| `PlayerID` / `Slot` | Locate the current connection and its runtime state. Remove the entry on disconnect; another connection can reuse the slot. |
| `SessionId` | Identify one connection across delayed work. `Core.PlayerManager.GetPlayerFromSessionId(id)` resolves it or returns null after removal. Use it for runtime state, not as a database key. |
| Authorized `SteamID` | Key persistent account preferences or progress. A reconnect can have the same SteamID and a different session. |
| `CHandle<T>` | Retain the identity of one entity across callbacks. Resolve it before use; pawn replacement creates a different entity even within the same session. |

For features that store account data, wait for Steam authorization and exclude bots from account storage. `SteamID != 0` alone does not establish authorization: the core's flexible authentication mode can expose an unauthenticated ID. See [server authentication settings](server-resources.md) and [terminology](terminology.md).

## Initialize and release state

Choose the callback that supplies the feature's prerequisites:

| Boundary | State work |
| --- | --- |
| `OnClientConnected` | Start early connection work. The managed player object exists, but the pawn may not be ready and the account may not be authenticated. |
| `OnClientPutInServer` | Initialize gameplay session state. Use `Kind` or the current player to select humans or bots as the feature requires. Check pawn readiness before pawn operations. |
| `OnClientSteamAuthorize` | Resolve the player from `PlayerId`, confirm account eligibility, and attach persistent data. Arrange initialization so either authorization or session setup can happen first. |
| `OnClientDisconnected` | Remove slot state synchronously. Copy the stored account ID and values before any asynchronous persistence. The event exposes `PlayerId` and `Reason`; it has no `SteamID` property. |
| Late load / hot reload | Run the same initialization for `Core.PlayerManager.GetAllPlayers()`, including currently authorized accounts. Existing connections do not replay their join/auth callbacks for the new plugin instance. |
| Map change / unload | Reset state at the lifetime promised by the feature; cancel owned work and release owned resources. Session preferences and map-specific entity handles can require different boundaries. |

Make initialization safe to call again for the same `SessionId`, so the load scan and relevant events do not reset existing feature state or start duplicate loads. `GetAllValidPlayers()` filters through `IPlayer.IsValid`, which requires a connected controller and a valid pawn; use `GetAllPlayers()` for connection state that must exist before pawn readiness. Apply `IsAlive` or resolve `PlayerPawn` only when the operation requires it. See [entity lifetime](../../sws2-entity-management/references/entities.md) for pawn handles and schema changes.

## Return asynchronous results to the same connection

Capture value identifiers on the game thread before I/O. In this fragment, `player` is an authorized human, `unloadToken` belongs to the plugin and is canceled during unload, and `LoadPreferencesAsync`/`ApplyPreferences` are application methods:

```csharp
var sessionId = player.SessionId;
var steamId = player.SteamID;
var preferences = await LoadPreferencesAsync(steamId, unloadToken);

await Core.Scheduler.NextTickAsync(() =>
{
    if (unloadToken.IsCancellationRequested)
    {
        return;
    }

    var current = Core.PlayerManager.GetPlayerFromSessionId(sessionId);
    if (current is null)
    {
        return;
    }

    ApplyPreferences(current, preferences);
});
```

The session lookup prevents a completed request from updating a replacement connection, including a reconnect by the same Steam account. `ApplyPreferences` updates the plugin's state on the game thread and resolves any pawn it needs at that moment. Keep map-specific work within the lifetime of its map. Follow [thread management](../../sws2-thread-management/references/threads.md) for task ownership and scheduler lifetime; a concurrent collection does not make native player or entity access thread-safe.

Validate the feature's first join, an already-connected player at load, and disconnect/reconnect while its I/O is pending. Include pawn replacement or map change when the feature retains state across those boundaries.

References: [Core events](https://swiftlys2.net/docs/development/core-events), [Steam authorization](https://swiftlys2.net/docs/development/steamworks#authorization-signals).
