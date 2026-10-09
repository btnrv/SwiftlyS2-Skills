---
name: sws2-batching
description: Use only when explicitly requested to investigate or implement batching of SwiftlyS2 native operations for measured tick or frame spikes.
---

Apply this skill when the user explicitly asks to investigate or implement batching. Establish current cost and gameplay timing before changing execution.

Read [measuring and scheduling batches](references/batching.md) for the repository example and scheduler behavior. Keep native work on the game thread. When measurements justify batching, bound work per tick, preserve required ordering, and define completion and lifecycle behavior. Resolve current targets when their operation runs.

Compare spike duration and total completion time under equivalent load. Retain the simpler route when batching fails to improve the measured problem or changes required gameplay semantics.
