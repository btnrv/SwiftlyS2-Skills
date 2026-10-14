# Dispatch and lifetime

A continuation after `await` may execute outside the game thread. Schema access and native calls keep their thread requirements inside an async method.

An `async` method runs on its caller's thread until it reaches the first `await` on a task that hasn't completed. Awaiting I/O from a handler doesn't leave the game thread by itself: everything before that point stalls the tick. The bundled SQLite driver, System.Data.SQLite, has no real async methods. `OpenAsync` and Dapper's `QueryAsync` run the whole query synchronously and return a completed task, so the code after `await` stays on the game thread too. MySqlConnector and Npgsql do real async I/O. Start I/O with `Task.Run` so it leaves the game thread whichever driver is configured.

Don't block the game thread on a task (`.Result`, `.Wait()`, `GetAwaiter().GetResult()`). The server stops ticking until the task finishes. If the task awaits `NextTickAsync` or a SwiftlyS2 `*Async` API from a background thread, it never finishes, because the tick loop it is blocking is what completes those tasks.

`CommandListener`, the `Core.Event` delegates, and the `Action` overloads of `Delay`, `DelayAndRepeat`, `Repeat` and their `BySeconds` forms all return `void`, so an `async` lambda passed to them becomes `async void`. Handlers that return `HookResult` (game events, net messages, client command and chat hooks) and `AddTimer` reject async lambdas at compile time. In an `async void` handler, code after `await` runs on a thread-pool thread. Any exception it throws, even one thrown before the first `await`, skips SwiftlyS2's handler try/catch and terminates the server process. Keep handlers synchronous and catch exceptions inside the work they start:

```csharp
Core.Command.RegisterCommand("rank", ctx =>
{
    if (ctx.Sender is not { } player) return;
    var sessionId = player.SessionId;
    var steamId = player.SteamID;

    _ = Task.Run(async () =>
    {
        try
        {
            var rank = await LoadRankAsync(steamId);
            Core.Scheduler.NextTick(() =>
            {
                Core.PlayerManager.GetPlayerFromSessionId(sessionId)
                    ?.SendMessage(MessageType.Chat, $"Rank: {rank}");
            });
        }
        catch (Exception ex)
        {
            Core.Logger.LogError(ex, "Rank lookup failed");
        }
    });
});
```

The player can disconnect during the query. SwiftlyS2 disposes their `IPlayer`, including `ctx.Sender`, as soon as the `OnClientDisconnected` handlers return. From then on `SteamID`, `Name`, `Controller`, `SendMessage` and `SendMessageAsync` throw `ObjectDisposedException`. Copy values before the I/O starts and resolve the player again on the game thread. `IsValid` reads entity state, so check it in the callback, not before scheduling.

`NextTick(Action)` and `NextWorldUpdate(Action)` queue synchronous work on the corresponding engine loop. `NextTickAsync(Action)` and the synchronous `Func<T>` variant return tasks that complete when that work has run. Callback overloads accepting `Func<Task>` are obsolete and throw. Use synchronous callbacks. From code already off the game thread, such as inside `Task.Run`:

```csharp
var result = await LoadManagedDataAsync();
await Core.Scheduler.NextTickAsync(() =>
{
    ApplyManagedDataToCurrentGameState(result);
});
```

These application methods stand for the feature's I/O and the step that applies its result to game state. Copy plain data and stable identifiers before I/O; resolve current targets when applying the result. Check connection identity as well as a reusable slot when the operation belongs to one session.

Code after `await NextTickAsync(...)` usually resumes inline on the game thread while the tick queue drains, because the scheduler completes the task there without `RunContinuationsAsynchronously`. Treat this as an implementation detail. Don't count on it for thread-unsafe calls, and don't run I/O right after it, because a SQLite query there runs inside the tick.

`SendMessageAsync`, `ReplyAsync`, `TeleportAsync` and other `*Async` methods that call `QueueOrNow` run inline when called on the game thread, so their exceptions throw there. Off the game thread they queue the call and return a task that faults on the next tick. `KickAsync` always waits for the next tick. A fire-and-forget `_ = ...Async()` fault is reported only when the garbage collector finalizes the task, so await these tasks inside a try/catch.

Some off-thread calls appear to work and still race the engine. Synchronous `SendMessage` calls the native function with no thread check. A schema field write lands in memory immediately while its network update waits for the next world update. Off the game thread, `NetMessage.Send()` queues the send. If the message is disposed first (`using var msg`), the queued send fails with `ObjectDisposedException` on a discarded task, so the message is dropped without a log entry.

Borrowed callback pointers stay within their documented lifetime. A game event object is disposed when its handler returns; reading it from `Task.Run` or a later tick throws `The event is already disposed.` Copy values for deferred work; retain an entity handle when the entity must be resolved again on another tick.

Timers return `CancellationTokenSource`. Register map-bound timers with `StopOnMapChange` and cancel plugin-owned timers on unload. One-shot queues provide no public cancellation handle. Synchronous `NextTick`/`NextWorldUpdate` callbacks carry the scheduler service's plugin lifecycle token. `NextTickAsync`/`NextWorldUpdateAsync` callbacks carry no such token and may still run after unload; check the plugin lifetime inside callbacks that can outlive it. Apply a map/session lifetime check when queued work belongs to that map or connection. Choose tick or world-update scheduling according to the engine operation and verify hibernation behavior when it matters.

References: [Thread safety](https://swiftlys2.net/docs/development/thread-safety), [scheduler](https://swiftlys2.net/docs/development/scheduler).
