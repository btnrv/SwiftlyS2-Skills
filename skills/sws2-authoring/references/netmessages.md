# Typed network messages

Use `Core.NetMessage` with the generated interfaces under `SwiftlyS2.Shared.ProtobufDefinitions`. Resolve the payload through MCP `protobuf_lookup` and check the generated C# interface against the project's package. A protobuf definition alone does not establish that a message is supported or delivered on the current server.

## Sending and ownership

`Send<T>(configureMessage)` constructs and sends a message using the recipient filter set in the callback. Set recipients with `message.Recipients.AddAllPlayers()` or `AddRecipient(playerId)`, and keep the callback's wrapper local.

`Create<T>()` returns an owned, disposable message for explicit lifetime control or repeated sends from one configuration. `Send()` uses its recipient filter; `SendToPlayer(playerId)` replaces that filter with one slot; `SendToAllPlayers()` selects all players. Keep allocation, mutation, send and disposal in one synchronous game-thread operation:

```csharp
using SwiftlyS2.Shared;
using SwiftlyS2.Shared.ProtobufDefinitions;

public sealed class ShakeMessages(ISwiftlyCore core)
{
    public void SendToPlayer(int playerId)
    {
        using var message = core.NetMessage.Create<CUserMessageShake>();
        message.Command = 0;
        message.Amplitude = 0.5f;
        message.Frequency = 2.0f;
        message.Duration = 1.0f;
        message.SendToPlayer(playerId);
    }
}
```

`Send()` queues work when called off-thread. Disposing the message immediately afterward can release it before the send runs. Dispatch the whole operation using [sws2-thread-management](../../sws2-thread-management/SKILL.md). Use `Create<T>()` with `using` when deterministic release is needed.

## Hook direction and removal

| Traffic | Registration | Handler shape |
| --- | --- | --- |
| Client to server | `HookClientMessage<T>` / `[ClientNetMessageHandler]` | `HookResult Handler(T message, int playerId)` |
| Server to clients | `HookServerMessage<T>` / `[ServerNetMessageHandler]` | `HookResult Handler(T message)` |
| Internal server to one client | `HookServerMessageInternal<T>` / `[ServerNetMessageInternalHandler]` | `HookResult Handler(T message, int playerId)` |

Only some outbound messages use the internal pipeline. Check the message's route before choosing a hook. Return `Continue` to allow delivery or `Stop` to block that pipeline. Blocking delivery does not reverse the gameplay that caused the message. A regular server hook exposes the recipient filter and writes its changes back to the outgoing recipient mask.

Each programmatic hook returns a `Guid`; keep it with the owning service and call `Core.NetMessage.Unhook(id)` when that registration ends. The type-wide unhook methods remove all matching hooks in that plugin's service. Attribute discovery and dynamic service lifetimes follow [events and registration ownership](events-hooks.md).

Hook payloads and their accessors borrow engine memory. Read or change declared typed fields during the callback; copy plain values for deferred work. Sending methods require a message allocated through the service, so a received wrapper cannot be resent directly. Construct a new owned message when forwarding is part of the feature. Use `Accessor` for declared fields missing from the generated interface or for field-level diagnostics.

Further API detail: [Network Messages](https://swiftlys2.net/docs/development/netmessages) and the [CUserMessageShake payload](https://swiftlys2.net/protobuf-viewer/cs2/usermessages.proto/CUserMessageShake).
