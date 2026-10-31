# Trust framework guarantees

Write a check only when its answer can be "no" at that line. A check the framework already answered costs reads on every call, buries the conditions that do matter, and makes readers wonder what case it handles. Before adding or keeping a check, sort it into one of these cases:

1. **Framework guarantee.** The API that produced the value already ensured it. Remove the check.
2. **Same synchronous path.** The value was validated earlier in the same method or tick, and nothing in between can change it: no kill, respawn, team change, disconnect or entity removal, and no `await`, scheduler, menu or event callback boundary. Remove the check.
3. **Game rule.** The check is redundant only because of CS2 rules such as rounds, warmup, respawn, team joining or convars. That is a project decision, not a framework fact. Decide it deliberately and keep the rule visible in the code or the review.
4. **No guarantee.** Keep the check. This covers data that crosses an `await`, `NextTick`, timer, menu or event boundary; players from unfiltered lookups (`GetAllPlayers`, `GetPlayer`, `GetPlayerFromSteamId`, `GetPlayerFromSessionId`); players and entities supplied by events and hooks; and entities stored in fields.

## Guarantees

These were verified in the SwiftlyS2 1.5.1 beta source (`PlayerManagerService.cs`, `Player.cs`, `CHandle.cs`, `EntityManager.cs`, `CommandContext.cs`, `GameEventAccessor.cs`, `EntitySystem.cs`, `SchedulerService.cs`, `TranslationFactory.cs`, `PlayerListComponent.cs`). Recheck them with [sws2-api-mcp](../../../sws2-api-mcp/SKILL.md) or the package source when the target version changes.

| API | Guarantee | Remove |
| --- | --- | --- |
| `GetAllValidPlayers`, `GetAlive`, `GetCT`, `GetT`, `GetInTeam`, `GetTAlive`, `GetCTAlive`, `GetBots`, `GetSpectators` | Every result passes `IPlayer.IsValid`: a connected, non-HLTV controller with a valid `Pawn` | `IsValid`, `Controller.IsValid`, `Pawn` null and `IsValid` checks on the results |
| `GetAlive`, `GetTAlive`, `GetCTAlive` | `Pawn.LifeState == LIFE_ALIVE`, the same test as `IsAlive` | Alive and life-state checks |
| `GetCT`, `GetT`, `GetInTeam`, `GetTAlive`, `GetCTAlive` | `Pawn.TeamNum` equals the team (the pawn's team, which normally matches the controller's) | The matching team check |
| `FindTargettedPlayers` | Results are valid players with a valid `Pawn`. `Alive`, `Dead`, `TeamOnly`, `OppositeTeamOnly` and `NoBots` are enforced. `NoMultipleTargets` returns the first match instead of rejecting ambiguous input | `IsValid`, `Pawn` checks and any condition a flag in the search mode already covers |
| `IPlayer.IsAlive` | `IsValid && Pawn.LifeState == LIFE_ALIVE` | `IsValid` next to `IsAlive`. `!IsValid \|\| IsAlive` means something else; keep it |
| `RequiredController`, `RequiredPawn`, `RequiredPlayerPawn` | Throw when invalid | Null and `IsValid` checks on the result |
| `CHandle<T>.Value` | Null unless the handle resolves to a live entity. The manager updates its slot table and address map together, so a non-null `Value` is valid in the same tick. `IPlayer.Pawn`, `IPlayer.PlayerPawn`, `ActiveWeapon.Value` and `OwnerEntity.Value` all come from `Value` | `handle.IsValid` before `Value`, and `IsValid` on a `Value` result read in the same synchronous code |
| `IPlayer.Controller` | Never null; an empty slot yields a dummy controller at address 0 | `!= null` and `?.`. Keep `Controller.IsValid` unless `IsValid`, `IsAlive` or a query above already covered the player |
| `ICommandContext.Sender` | Null exactly when `IsSentByPlayer` is false | `!context.IsSentByPlayer \|\|` before a `Sender` pattern |
| Game event `UserIdController` and other controller accessors | Never null; a dummy when unresolved | Null checks. Check `IsValid` instead |
| `CreateEntityByDesignerName`, `CreateEntity` | Return the entity or throw | Null checks on the result |
| `ISchemaClass.As<T>()` | Never null | Null checks on the result |
| Entity-created event `Entity` | Never null | Null checks |
| `Core.Scheduler` timers and next-tick work | Cancelled when the plugin unloads. `Unload()` runs, then the plugin's core and scheduler are disposed | Cancel and dispose calls for scheduler timers in `Unload()`. Keep them where the timer is also stopped on other paths |
| Translation values | Colorized when the translation file loads | `.Colored()` on localizer results, unless a format argument carries color markup on purpose |
| `PlayerListComponent.OnPlayerSelected` | The target is checked for validity and list membership before the callback runs | `ctx.Target.IsValid`. Keep alive and team checks: the list refreshes once a second |

## Read a property once

Player and entity properties are computed on every read. `IPlayer.Controller` is a locked slot lookup. `Pawn` and `PlayerPawn` look up the controller and resolve a handle. `IsValid` takes several lookups, `IsAlive` adds a pawn read, and `SteamID` is a native call. Network message field reads go through native reflection.

When one method or lambda reads the same property more than once in the same tick, store it in a local on first use and reuse it. Re-read after an `await` or callback boundary. Cache the wrapper, not schema field values the code changes or expects to be current: `pawn.Health` is a live memory read.

```csharp
var steamId = player.SteamID;
var pawn = player.PlayerPawn;
if (pawn is null)
{
    return;
}

_lastSeen[steamId] = DateTime.UtcNow;
pawn.Teleport(_spawnPosition, null, Vector.Zero);
```

## Compare domain values

Compare teams with the `Team` enum: `player.Controller.Team == Team.CT`, `team is Team.T or Team.CT`. `entity.Team` is `(Team)TeamNum`. Read the controller's team for player checks, because team changes act on the controller. Keep numbers only where the value is stored or passed to an API that takes one. See [game values](game-values.md).

## Shape of the checks that remain

- Use guard clauses with an early `return` or `continue` instead of nested blocks. Ask one question per `if`.
- Keep side effects out of conditions. Put the call on its own line and test a local: `var added = seen.Add(id); if (!added) { … }`. Split `a || seen.Add(id)` into separate guards. Exception: keep out-parameter calls such as `TryGetValue`, `TryParse` and `Remove(key, out var value)` inline; moving them out loses the compiler's nullable flow analysis.
- Don't wrap a single framework call in a renamed helper. See [framework reuse](framework-reuse.md).
- In hot paths such as per-shot, per-sound, movement or transmit hooks, test the cheapest condition that rejects the most calls first, before any player or entity lookup.

This loop over `GetTAlive()` drops checks the query already answered and reads each property once:

```csharp
// Before
foreach (var player in Core.PlayerManager.GetTAlive())
{
    if (player.IsValid && player.IsAlive && player.Controller.TeamNum == 2)
    {
        if (player.PlayerPawn is { IsValid: true } pawn)
        {
            _slain.Add(player.SteamID);
            pawn.CommitSuicide(false, true);
        }
    }
}

// After
foreach (var player in Core.PlayerManager.GetTAlive())
{
    if (player.PlayerPawn is not { } pawn)
    {
        continue;
    }

    _slain.Add(player.SteamID);
    pawn.CommitSuicide(false, true);
}
```

The remaining `PlayerPawn` null check is a game-rule case: a living player's pawn and player-pawn handles normally point to the same entity, but no framework query promises it. Other common game-rule cases are "an alive player is on T or CT" and "an authorized player has a nonzero SteamID". Treat each as a project decision.
