# Server-side Steamworks

SwiftlyS2 exposes server bindings in `SwiftlyS2.Shared.SteamAPI`: `SteamGameServer`, `SteamGameServerUtils`, `SteamGameServerUGC` and `SteamGameServerStats`. Use them for Steam callbacks, server metadata, ownership checks or Workshop queries. Check the needed method and result type against the target package through MCP.

## Readiness and identity

Start Steam-dependent work after `Core.Event.OnSteamAPIActivated`. Keep a named listener and remove it when its owner stops. The event fires on activation and does not replay when a listener subscribes. When loading into a running server, check how the target runtime reports readiness; another activation event may not arrive. The host handles Steam initialization and callback pumping.

For a connected player's authenticated identity, use `OnClientSteamAuthorize` and `OnClientSteamAuthorizeFail` with [player state](player-state.md). A nonzero `SteamID` alone does not establish authorization under every server auth policy. Construct a `CSteamID` from the authorized ID; `IsValid()` and `BIndividualAccount()` check its representation and account type, not successful authentication. Bots do not have a persistent individual Steam account.

For manual ticket validation, pair a successful `SteamGameServer.BeginAuthSession` with `EndAuthSession` for the session it owns. A plugin observing CS2 player authorization should use the framework's signals.

## Callback ownership

| Operation | Managed owner |
| --- | --- |
| Receive repeated notifications of one result type | `Callback<T>.Create(Action<T>)` |
| Receive the result of one request returning `SteamAPICall_t` | `CallResult<T>.Create(call.m_SteamAPICall, Action<T, bool>)` |

Keep each registration in the service that owns it and dispose it when the operation stops or the plugin unloads. For concurrent results, keep one owner per outstanding request. Release the previous registration before replacing a field. `CallResult<T>` disposes itself after delivery, so create a new one for a later request. Handle the callback's `ioFailure` flag separately from the result struct's status fields, according to the operation.

Copy the values needed for deferred work. Apply game changes through the [thread-management skill](../../sws2-thread-management/SKILL.md) and resolve the intended session again on the game thread. A Steam callback registration does not own the player's connection or the map.

## Workshop and metadata

Use `SteamGameServerUGC` for server Workshop operations. Check the item state, start the required query or download, and match the result to the requested item. Read install information only after a successful result and check the method's success return. A download supplies server files; follow the [game-assets skill](../../sws2-game-assets/SKILL.md) for client asset delivery.

Use `SteamGameServer` metadata setters only for fields the feature is meant to control. For app IDs, callback struct fields and Workshop query handles, read the current declarations; select the release function specified for each acquired handle.

Reference: [Steamworks guide](https://swiftlys2.net/docs/development/steamworks).
