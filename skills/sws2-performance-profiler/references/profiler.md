# Capture and interpretation

## Server-console route

The core command is console-only. Player permissions and command overrides do not enable player access. Run `sw version`, then `sw profiler` to check the installed command set. The commands are:

| Command | Effect |
| --- | --- |
| `sw profiler status` | Prints Disabled or EventPipe. |
| `sw profiler enable 1` | Starts EventPipe recording; omitted level also defaults to 1. |
| `sw profiler save` | Saves the active trace and generates a summary when the analyzer is installed. |
| `sw profiler save nosummary` | Saves the raw trace while skipping summary generation. |
| `sw profiler disable` | Stops recording. |

Enable, reproduce the workload, and save. Wait for the logged saved path before disabling: `save` runs asynchronously and starts a fresh recording session after saving while EventPipe remains enabled. Saving requires an active session; disabling first makes it unavailable for `save`. The supported export route is `save`; use the installed help for any version-specific verbs.

Output is under the SwiftlyS2 root's `profilers/<GUID>/<UTC timestamp>.nettrace`, with a sibling `.summary.txt` when analysis succeeds. The raw trace remains available if summary conversion fails. Use an EventPipe-capable trace viewer for stack and timeline analysis. Record the capture duration, server/framework version, scenario, and load beside the result.

Profiler status reports the configured level. Confirm the actual saved artifact: session creation can fail while the level still reads EventPipe. Keep captures bounded; runtime event tracking and trace writing add overhead and disk use. Compare measurements with the same profiling setup and disable tracking afterward.

## What the report means

The summary combines sampled managed CPU, runtime GC/allocation and exception events, and explicit `Core.Profiler` timings. `inc%` includes callees; `exc%` represents self cost. `ms/t` is a capture average derived using a hard-coded 64 Hz tick count in the analyzer. It is not a direct worst-tick measurement. Interpret totals with call frequency and capture length, and inspect the timeline or server tick evidence for a reported spike. Inclusive costs can overlap across parent and child scopes.

Custom timings show elapsed milliseconds per call and p50/p75/p95/p99. `First(ms)` and `Last(ms)` are event timestamps within the trace. `ExcBudget` counts samples exceeding 15.625 ms, the analyzer's 64 Hz budget. A timing that spans I/O includes waiting; its elapsed duration does not establish CPU cost. Check event loss and capture length before interpreting percentiles or absence of samples. Native engine work requires matching server evidence when managed attribution is incomplete.

## Focused instrumentation

Pair timing scopes with `finally`:

```csharp
Core.Profiler.StartRecording("Menu.Build");
try
{
    BuildMenu(); // the feature's operation
}
finally
{
    Core.Profiler.StopRecording("Menu.Build");
}
```

The service prefixes scope names with the plugin identifier. Active recordings use one dictionary entry per plugin/name: overlapping starts with the same name overwrite its start timestamp. Use separate names for nested operations and measured durations for concurrent operations.

`RecordTime` uses milliseconds, as do start/stop timings and the analyzer. Pass `stopwatch.Elapsed.TotalMilliseconds`; using microseconds scales results by 1,000.

Reference: [Profiler documentation](https://swiftlys2.net/docs/development/profiler).
