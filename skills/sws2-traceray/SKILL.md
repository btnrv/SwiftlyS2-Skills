---
name: sws2-traceray
description: Implement SwiftlyS2 collision traces and aim queries with Core.Trace, including line and hull traces, eye position and view angles for living players, dead players' observer pawns and spectator cameras, collision masks, ignored entities and hit results. Use whenever a plugin needs a raycast, line of sight, what a player is aiming or looking at, wall or ground checks, teleport-to-aim, hit position or surface normal, or distance to a surface.
---

Establish which view the feature traces from: living player, dead player's observer pawn, or spectator camera. Verify the schema fields and trace APIs through SwiftlyS2 MCP for the installed runtime.

Read [origins and trace results](references/traces.md) for the aim route and current API. Run traces on the game thread. Choose masks, ignored entities, and line or hull geometry from the feature's collision requirements. Use hit and solid-state results to decide whether the destination or target satisfies the feature.

Build and exercise required alive and observer modes on a live map, including misses and relevant collision geometry.
