# Dispatch and lifetime

A continuation after `await` may execute outside the game thread. Schema access and native calls retain their thread requirements in an async method.

`NextTick(Action)` and `NextWorldUpdate(Action)` queue synchronous work on the corresponding engine loop. `NextTickAsync(Action)` and the synchronous `Func<T>` variant return tasks for completion of that work. Callback overloads accepting `Func<Task>` are obsolete and throw. Use synchronous callbacks:

```csharp
var result = await LoadManagedDataAsync();
await Core.Scheduler.NextTickAsync(() =>
{
    ApplyManagedDataToCurrentGameState(result);
});
```

These application methods represent the feature's I/O and game-state application. Copy plain data and stable identifiers before I/O; resolve current targets when applying the result. Check connection identity as well as a reusable slot when the operation belongs to one session.

Borrowed callback pointers stay within their documented lifetime. Copy values for deferred work; retain an entity handle when the entity must be resolved again on another tick.

Timers return `CancellationTokenSource`. Register map-bound timers with `StopOnMapChange` and cancel plugin-owned timers on unload. One-shot queues provide no public cancellation handle. Synchronous `NextTick`/`NextWorldUpdate` callbacks carry the scheduler service's plugin lifecycle token. `NextTickAsync`/`NextWorldUpdateAsync` callbacks carry no such token and may still run after unload; check the plugin lifetime inside callbacks that can outlive it. Apply a map/session lifetime check when queued work belongs to that map or connection. Choose tick or world-update scheduling according to the engine operation and verify hibernation behavior when it matters.

References: [Thread safety](https://swiftlys2.net/docs/development/thread-safety), [scheduler](https://swiftlys2.net/docs/development/scheduler).
