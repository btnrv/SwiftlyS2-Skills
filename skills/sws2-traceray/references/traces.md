# Origins and trace results

For a living player, resolve the valid `PlayerPawn` and use `EyePosition` and `EyeAngles`. Tracing from the pawn's feet changes the hit surface.

For dead players and spectators, resolve `Controller.ObserverPawn.Value`. The minimal branch below retrieves the observer pawn's origin and view angle. This establishes observer-state access; match it to the camera modes the feature supports.

```csharp
using SwiftlyS2.Shared.Natives;
using SwiftlyS2.Shared.Players;
using SwiftlyS2.Shared.SchemaDefinitions;

private static bool TryGetTraceView(
    IPlayer player,
    out Vector position,
    out QAngle angles,
    out CBaseEntity? ignoredEntity)
{
    position = Vector.Zero;
    angles = default;
    ignoredEntity = null;
    if (!player.IsValid) return false;
    if (player.IsAlive)
    {
        var pawn = player.PlayerPawn;
        if (pawn is null || !pawn.IsValid || pawn.EyePosition is not { } eye)
            return false;
        position = eye;
        angles = pawn.EyeAngles;
        ignoredEntity = pawn.As<CBaseEntity>();
        return true;
    }
    var observer = player.Controller.ObserverPawn.Value;
    if (observer is null || !observer.IsValid ||
        observer.CBodyComponent?.SceneNode?.AbsOrigin is not { } origin)
        return false;
    position = origin;
    angles = observer.V_angle;
    ignoredEntity = observer.As<CBaseEntity>();
    return true;
}
```

Verify roaming, first-person, and chase views required by the feature against the target schema and live server. Use the observed target's eyes when the query explicitly follows that player's view. The helper does not establish that observer origin and `V_angle` equal every rendered client camera; document the supported camera policy.

Read `observer.ObserverServices?.ObserverMode` and `ObserverTarget` to choose that policy. Resolve `ObserverMode_t` through the schema MCP. For roaming, the helper exposes the observer pawn state to validate. For an in-eye view, resolve the observed target pawn and its `EyePosition`/`EyeAngles`. In chase mode those target eyes represent the watched pawn's aim; a query from the rendered chase camera needs its own verified camera position.

On the game thread, use the view result with the current beta trace API:

```csharp
using SwiftlyS2.Shared.Natives;
using SwiftlyS2.Shared.Trace;

var parameters = new TraceParamsBuilder()
    .WithLineRay()
    .WithObjectQuery(RnQueryObjectSet.Static | RnQueryObjectSet.Dynamic)
    .WithInteraction(MaskTrace.Solid | MaskTrace.Player)
    .WithCollisionGroup(CollisionGroup.Player)
    .Build();
if (TryGetTraceView(player, out var eyePosition, out var eyeAngles, out var ignored))
{
    if (ignored is not null) parameters.EntitiesToIgnore.Add(ignored);
    var hit = Core.Trace.TraceShapeAngle(eyePosition, eyeAngles, 8192f, parameters);
    // Apply the feature's hit and placement checks here.
}
```

The mask includes players; choose the intended surfaces for the feature. `SimpleTrace` is obsolete in the verified beta API. Select the equivalent supported method when targeting another runtime.

`DidHit` includes `StartInSolid`; inspect both for destination placement. A world hit can have a null `Entity`. `EndPos`, `HitPoint`, and `HitNormal` have separate contracts. A teleport requiring player clearance needs a player-sized hull or bounding-box query under its movement requirements. A ray hit establishes the aimed point.

Beta MCP `ITraceManager`; schema MCP `CCSObserverPawn` (inherits `CCSPlayerPawnBase`). Source: `managed/src/SwiftlyS2.Shared/Modules/Trace/{ITraceManager,TraceParamsBuilder,TraceResult}.cs`. Origin example read 2026-10-09: `plugins/internal/Jailbreak/src/Library/Admin/AdminCommandHelpers.cs`, `TryGetAimBringDestination`. Its range and destination adjustment belong to the command.

Sources checked 2026-10-09. SwiftlyS2 source commit `78b4c89a6e21de7b6a4d9485295b6448f58e26d9`; verify APIs and build-specific data against the deployed runtime before use.
