---
name: sws2-performance-profiler
description: Diagnose SwiftlyS2 plugin performance with the built-in sw profiler, EventPipe captures, timing scopes, and measured comparison of CPU, allocation, and callback cost.
---

Identify the slow action, player/entity load, and capture interval. Verify the deployed SwiftlyS2 version and the server-console `sw profiler` help before choosing commands.

Read [capture and interpretation](references/profiler.md) for the supported command sequence, output files, duration units, and tracking limits. Capture the smallest representative workload, inspect the summary and raw trace, and connect the result to the reported tick or frame spike. Add narrow timing scopes when the trace needs more attribution.

Keep native operations on their required thread. Compare equivalent workloads before and after a targeted change, and stop profiling after the capture is saved. Use the batching skill when the user explicitly requests that optimization.
