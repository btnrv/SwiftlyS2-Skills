# Framework and game terms

Use these terms precisely in code names, API searches and explanations. Keep identifiers distinct when crossing an event, entity or database boundary.

| Term | Meaning and use |
| --- | --- |
| Managed | The .NET runtime and C# plugin side. Garbage collection manages managed objects; a wrapper can still borrow native memory with a shorter lifetime. |
| Native | The C++ engine/framework side, including pointers, native allocations and ABI-bound calls. Native ownership and threading requirements remain in effect when accessed from C#. |
| Controller | `CCSPlayerController`, the server's player identity and control entity. Name, account identity and team/control state belong here. Resolve the current pawn for character operations. |
| Pawn | The player's in-world character, normally `CCSPlayerPawn`. Position, velocity, health, model and weapons belong to the pawn and its services. Death, respawn and observer transitions can replace it. |
| Slot / PlayerID | The current server connection slot. SwiftlyS2 `IPlayer.PlayerID` equals `Slot`; the controller entity index is normally slot + 1. A later connection can reuse the slot. |
| SteamID | Account identity used for persistent player data. It is distinct from a slot or entity index. |
| Player object | SwiftlyS2's `IPlayer` facade over the connected player, controller, pawn and convenience operations. Its `PlayerID` is not a persistent account identifier. |
| Entity index | A position in the engine's entity system. An index can be reused after removal; derive current entities through the entity API. |
| Entity handle | `CHandle<T>`, a packed entity index and serial number. `Value` resolves the current matching entity, and `IsValid` checks that identity. It does not own the entity or keep it alive. |
| Temporary entity | An entity whose intended lifetime follows a round or shorter feature operation. Clean it up at that boundary. |
| Permanent entity | In this terminology, an entity intended to survive round resets within the map. Map change still ends its lifetime. Verify the entity's persistence rules when creating it. |

The guide illustrates controllers at indices 1 through 64 and other entities above that range. Use the runtime's player capacity and entity APIs to enumerate them; a numeric range is not a type check. For event fields named `userid` or `playerid`, inspect the generated event declaration and its player-resolution contract before converting a number. A field name alone does not establish whether it is a slot, handle or another engine identifier.

For aiming, distinguish the living pawn's eyes, an observer pawn's state, an observed target's eyes and the rendered client camera. Use [sws2-traceray](../../sws2-traceray/SKILL.md) to select the view that the feature promises.

Sources: [terminologies guide](https://swiftlys2.net/docs/guides/terminologies); `managed/src/SwiftlyS2.Core/Modules/Players/Player.cs` (`PlayerID => Slot`) and `managed/src/SwiftlyS2.Shared/Natives/Structs/CHandle.cs` at [78b4c89](https://github.com/swiftly-solution/swiftlys2/tree/78b4c89a6e21de7b6a4d9485295b6448f58e26d9). The handle definition follows the implementation's index-and-serial representation.
