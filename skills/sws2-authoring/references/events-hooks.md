# Events, hooks and registration ownership

| Required behavior | Entry point |
| --- | --- |
| Player connection, map, entity or engine-loop lifecycle | `Core.Event` and the matching `IEventSubscriber` event |
| A generated CS2 event notification | `Core.GameEvent` and its generated event type |
| Intercept an engine operation already covered by a typed hook | `Core.GameHooks`, with its operation-specific `Pre`/`Post` context |
| A required engine operation absent from the managed API | [sws2-native-and-memory](../../sws2-native-and-memory/SKILL.md) |

Some legacy game events no longer fire in current CS2 builds. Check for an equivalent core event or typed game hook, and verify the chosen event on the target server. Replace obsolete `Core.Event` hook members with their documented `Core.GameHooks` equivalent when implementing that behavior; for example, damage interception uses `Core.GameHooks.Entities.TakeDamage`.

## Fixed and dynamic handlers

Handlers belong in `src/<Module>/<Type>.cs`. The framework discovers attributes on the plugin entrypoint automatically. Register a service instance created through dependency injection once with `Core.Registrator.Register(instance)` when it contains fixed attribute handlers. Registering the same instance again duplicates callbacks. The public registrator exposes `Register` but has no object-level `Unregister` counterpart.

| Surface | Fixed handler attribute | Registration owned by a service that can stop early |
| --- | --- | --- |
| Core event | `[EventListener<EventDelegates.OnClientConnected>]`, using the exact delegate signature | `Core.Event.OnClientConnected += handler`, then `-= handler` |
| Game event | `[GameEventHandler(HookMode.Pre)]` or `Post`; returns `HookResult` | `HookPre<T>(handler)` / `HookPost<T>(handler)` return a `Guid`; retain it and call `Core.GameEvent.Unhook(id)` |
| Typed game hook | `[GameHookHandler(HookMode.Pre)]` or `Post`; signature follows the selected context | Subscribe to the hook's `Pre` or `Post` event with `+=`, then remove the same handler with `-=` |

Use either attribute discovery or programmatic registration for each subscription. The framework clears plugin listeners on unload; a service that stops earlier must remove its listeners itself. Preserve the delegate when using a lambda. Type-wide game-event unhook methods remove all matching registrations in that plugin service, so use the returned ID to remove one owner's callback.

## Phase and cancellation

A game-event pre-hook runs before the event is dispatched. Returning `HookResult.Stop` blocks the event; it does not undo the gameplay operation that produced it. Use the relevant game hook to prevent damage, item acquisition or another engine operation. A post-hook observes the event after dispatch; return `Continue` for ordinary observation.

Typed game hooks use `void` handlers taking their context by `ref`, such as `void OnDamagePre(ref TakeDamageEntityPreContext ctx)`. Set the result through `ctx.SetHookResult(...)`. For the damage hook, `Stop` or `CancelOriginal` in `Pre` prevents the native damage call and its `Post` callback. Post runs after the original call and cannot cancel that completed operation. Inspect the selected hook before relying on parameter mutation, a replacement return value or later-listener suppression; these are operation-specific contracts.

Some core events expose a mutable `Result`; ordinary lifecycle notifications do not. The event interface determines whether a callback accepts or returns `HookResult`.

## Payload lifetime and firing

Game-event objects and their accessors are borrowed for the callback. Typed game-hook contexts are `ref struct` values; their borrowed parameters, native references and user-command payloads stay in the current call. Copy the values needed for later work and resolve current game objects when that work runs. Use [sws2-thread-management](../../sws2-thread-management/SKILL.md) for dispatch and session/map lifetimes.

For generated game events, `Fire<T>` broadcasts, `FireToPlayer<T>` targets a slot, and `FireToServer<T>` stays server-side. Their `Async` counterparts marshal off-thread calls. Configuration callbacks also receive temporary event objects. Prefer generated properties; use `Accessor` for a declared field without a generated property. `DontBroadcast` controls client broadcasting of an intercepted event.

Further API detail: [Using Attributes](https://swiftlys2.net/docs/development/using-attributes), [Core Events](https://swiftlys2.net/docs/development/core-events), [Game Events](https://swiftlys2.net/docs/development/game-events), and [typed game hooks](https://swiftlys2.net/docs/development/native-functions-and-hooks#higher-level-alternative-game-hooks).
