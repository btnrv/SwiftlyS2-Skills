# Entity lifetime and spawn values

An entity wrapper borrows engine memory. `IsValid` is a point-in-time check. `CHandle<T>` stores an index and serial so it can resolve the entity after a tick; an index alone may refer to a replacement.

Run this route on the game thread after the entity system becomes available:

```csharp
using SwiftlyS2.Shared.EntitySystem;
using SwiftlyS2.Shared.SchemaDefinitions;

var relay = Core.EntitySystem.CreateEntityByDesignerName<CBaseEntity>("logic_relay");
using var values = new CEntityKeyValues();
values.SetString("targetname", "example_relay");
values.SetBool("StartDisabled", false);
relay.DispatchSpawn(values);
var handle = Core.EntitySystem.GetRefEHandle(relay);
Core.Scheduler.NextTick(() =>
{
    if (handle.IsValid && handle.Value is { } current)
    {
        current.Despawn();
    }
});
```

`CEntityKeyValues` owns a native allocation and implements `IDisposable`. Typed setters support strings, numbers, vectors, angles, colors, tokens, and pointers. Keep the container alive until `DispatchSpawnAsync` completes when using that method.

Legacy `SwiftlyS2.Shared.Natives.KeyValues` is a separate unsafe engine struct. Use `CEntityKeyValues` for entity spawn parameters and verify the engine ownership contract for legacy KeyValues pointers. Asset KV3 and plugin JSON configuration have their own formats.

For schema mutation, inspect the generated setter or `ref` accessor and its change-notification implementation. Apply the field's matching network notification when required. Spawn keys and input/output names come from entity datamaps; schema fields describe memory.

References: [Entity guide](https://swiftlys2.net/docs/development/entity), [spawn values](https://swiftlys2.net/docs/development/entitykeyvalues).
