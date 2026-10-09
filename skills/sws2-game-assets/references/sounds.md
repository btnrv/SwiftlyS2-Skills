# Emit sound events

Use `SwiftlyS2.Shared.Sounds.SoundEvent` for a named game sound event. For asset compilation, mounting, delivery and precaching, see [resource formats and precache](assets.md). An event name such as `Weapon_AK47.Single` identifies a sound event, not a raw audio filename. Check names and custom parameters in the target game's resources.

Run this on the game thread, with `player` as the current intended recipient:

```csharp
using SwiftlyS2.Shared.Sounds;

using var sound = new SoundEvent("Weapon_AK47.Single", volume: 0.6f);
sound.Recipients.AddRecipient(player.PlayerID);
sound.SourceEntityIndex = -1;
uint soundGuid = sound.Emit();
```

The default recipient filter is empty. Add individual slots for targeted feedback or call `AddAllPlayers()` for a broadcast. If reusing a filter, clear its previous recipients before selecting a new audience. With source index `-1`, sound originates at the recipient; `SetSourceEntity(entity)` uses a current entity's index. `SetFloat3` can set a position parameter when the sound definition supports it. Set typed fields only when their names and types are known for that event.

`SoundEvent` owns a native message after emission and implements `IDisposable`. Keep it in a `using` scope. `Emit()` requires the game thread; `EmitAsync()` schedules emission and must be awaited before leaving that scope. Resolve any source entity and current recipients on the game thread according to [thread management](../../sws2-thread-management/SKILL.md). Disposal releases the message allocation; it does not imply that an already playing sound stops.

With the content mounted, check that the selected clients hear the event at the expected location and volume. A successful send or returned GUID does not prove the client has the sound assets.

Reference: [Sound events guide](https://swiftlys2.net/docs/development/soundevents).
