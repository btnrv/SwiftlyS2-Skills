---
name: sws2-traceray
description: Implement SwiftlyS2 collision traces and aim queries using the appropriate player or observer origin, shape, masks, and result checks.
---

Establish which view the feature traces from: living player, dead player's observer pawn, or spectator camera. Verify the schema fields and trace APIs through SwiftlyS2 MCP for the installed runtime.

Read [origins and trace results](references/traces.md) for the aim route and current API. Run traces on the game thread. Choose masks, ignored entities, and line or hull geometry from the feature's collision requirements. Use hit and solid-state results to decide whether the destination or target satisfies the feature.

Build and exercise required alive and observer modes on a live map, including misses and relevant collision geometry.
