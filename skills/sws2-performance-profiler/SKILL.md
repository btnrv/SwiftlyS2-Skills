---
name: sws2-performance-profiler
description: Diagnose SwiftlyS2 plugin performance with the built-in sw profiler, Core.Profiler recordings, EventPipe .nettrace captures, timing scopes, and before-and-after comparison of CPU, allocation and callback cost. Use when a plugin or server is slow, lags, drops ticks, spikes in frame time, uses high CPU or memory, triggers frequent garbage collection, or when the user asks to profile, benchmark or optimize SwiftlyS2 code.
---

Identify the slow action, player/entity load, and capture interval. Verify the deployed SwiftlyS2 version and the server-console `sw profiler` help before choosing commands.

Read [capture and interpretation](references/profiler.md) for the supported command sequence, output files, duration units, and tracking limits. Capture the smallest representative workload, inspect the summary and raw trace, and connect the result to the reported tick or frame spike. Add narrow timing scopes when the trace needs more attribution.

Keep native operations on their required thread. Compare equivalent workloads before and after a targeted change, and stop profiling after the capture is saved. Use the batching skill when the user explicitly requests that optimization.
