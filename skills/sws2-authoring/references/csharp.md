# C# conventions

Follow the repository's `.editorconfig`. For a new plugin, use four spaces per indentation level, a tab width of four, file-scoped namespaces and braces for control-flow blocks. Use PascalCase for types and public members, camelCase for parameters and local variables, and one consistent convention for private fields.

Keep `using` directives at the top. Let nullable annotations describe absence and narrow values before dereferencing them. Use `var` when the right-hand side makes the type clear; state the type where it conveys a domain distinction. Comments explain engine constraints or ownership that the code cannot express.

Keep handlers short enough to expose their event contract. Extract a method when it names a meaningful operation or isolates a different lifetime. Constructor injection makes dependencies visible; a service provider belongs at the composition root.

## Performance

Measure relevant code with [sws2-performance-profiler](../../sws2-performance-profiler/SKILL.md) and server observations before choosing an optimization. Compare both frame spikes and total work.

- Prefer framework services and typed operations before introducing native calls or custom data structures.
- Keep per-tick and transmit handlers synchronous and bounded. Move database and network I/O to asynchronous work, then return engine operations to the game thread.
- Cache stable metadata and configuration-derived values at the lifetime where they remain valid. Cache an entity identity or handle when needed and resolve its current object before use.
- Use `foreach` and direct lookups where they express hot-path work clearly. Avoid repeated LINQ materialization, captured delegates and formatted logging in a measured hot loop.
- Use collections suited to access patterns. A slot-indexed array needs connection-lifetime invalidation; a reconnect can reuse the same slot.
- Use spans for synchronous managed buffers when they simplify ownership. Pool buffers only when profiling justifies the extra lifetime management, returning each rented buffer once after its last use.
- Use `Task` for ordinary asynchronous APIs. Select `ValueTask` only when the API contract and measured allocation pattern justify its consumption rules.

Copy native-backed values into owned data before leaving their valid scope. A `ref`, span or wrapper around engine memory does not become safe because the C# type is convenient. The thread-management skill defines the handoff boundary.

References: [Thread safety](https://swiftlys2.net/docs/development/thread-safety), [core events](https://swiftlys2.net/docs/development/core-events), [profiler](https://swiftlys2.net/docs/development/profiler), [dependency injection](https://swiftlys2.net/docs/guides/dependency-injection).
