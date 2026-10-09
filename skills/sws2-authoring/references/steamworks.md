# Server-side Steamworks

Use this reference for a feature that needs Steam callbacks, server metadata, ownership checks or Workshop queries. SwiftlyS2 exposes server bindings in `SwiftlyS2.Shared.SteamAPI`: `SteamGameServer`, `SteamGameServerUtils`, `SteamGameServerUGC` and `SteamGameServerStats`. Confirm the needed method and result type against the target package through MCP.

## Readiness and identity

Start Steam-dependent work after `Core.Event.OnSteamAPIActivated`. Keep a named listener and remove it when its owner stops. This is an activation notification, not a replay-on-subscribe readiness service. When loading into an already running server, establish the supported readiness check from the matching runtime before assuming another activation event will arrive. The host owns Steam initialization and callback pumping.

For a connected player's authenticated identity, use `OnClientSteamAuthorize` and `OnClientSteamAuthorizeFail` with [player state](player-state.md). A nonzero `SteamID` alone does not establish authorization under every server auth policy. Construct a `CSteamID` from the authorized ID; `IsValid()` and `BIndividualAccount()` check its representation/account type, not successful authentication. Bots do not have a persistent individual Steam account.

Manual ticket validation is a separate feature: pair a successful `SteamGameServer.BeginAuthSession` with `EndAuthSession` for the session it owns. A normal plugin observing CS2 player authorization should use the framework's signals.

## Callback ownership

| Operation | Managed owner |
| --- | --- |
| Receive repeated notifications of one result type | `Callback<T>.Create(Action<T>)` |
| Receive the result of one request returning `SteamAPICall_t` | `CallResult<T>.Create(call.m_SteamAPICall, Action<T, bool>)` |

Retain each registration in its owning service and dispose it when that operation stops or the plugin unloads. Keep one owner per outstanding request if concurrent results are required; replacing a field must release the prior registration. `CallResult<T>` disposes itself after delivery, so create a new one for a later request. The callback's `ioFailure` flag and the result struct's status fields are separate outcomes to handle according to the operation.

Copy the values needed for deferred work. Apply game changes through the [thread-management skill](../../sws2-thread-management/SKILL.md), re-resolving the intended session on the game thread. A Steam callback registration does not own the player's connection or the map.

## Workshop and metadata

Use `SteamGameServerUGC` for server Workshop operations. Check item state, initiate the required query/download, and match completion to the requested item. Read install information only after a successful result and check the method's success return. A download supplies server files; client asset delivery still follows the [game-assets skill](../../sws2-game-assets/SKILL.md).

Use `SteamGameServer` metadata setters only for fields the feature is meant to control. For app IDs, callback struct fields and Workshop query handles, read the current declarations; select the release function specified for each acquired handle.

Reference: [Steamworks guide](https://swiftlys2.net/docs/development/steamworks).
