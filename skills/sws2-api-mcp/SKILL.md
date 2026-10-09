---
name: sws2-api-mcp
description: Ground SwiftlyS2 plugin development in live documentation, C# API declarations and CS2 data. Use whenever creating, changing, reviewing or debugging a SwiftlyS2 plugin, before choosing framework calls or game identifiers.
---

# SwiftlyS2 API and documentation

Use the SwiftlyS2 MCP throughout implementation. Confirm the documented workflow before choosing an approach, then look up the declarations and game data used by each change. Keep those verified details available while writing and reviewing the code.

1. Read the project's SwiftlyS2 package and deployed runtime versions. Select `stable` for release packages or `beta` for prereleases. Their source links follow `master` and `beta`, respectively. Both indexes move over time; resolve version differences against the source tag and package used by the project.
2. Find the relevant development page with `docs_search` or `docs_list`. Read its full section through the returned URL or the bundled documentation snapshot. Search results contain excerpts, not the whole contract.
3. Use `apidocs_search` to find a type, then `apidocs_lookup` for its declaration, overloads, ownership and threading requirements. Confirm the namespace, arguments, return value and cleanup mechanism before using it. For member narrowing, copy the full method name shown by the index, such as `SendChat(string)`; properties use bare names. If a member lookup misses, retrieve the complete type before concluding it is absent.
4. Resolve engine details through the matching data tool: schema fields, entity inputs, protobuf fields, event payloads, convars or Panorama properties. Verify the corresponding C# wrapper separately.
5. Recheck when a new subsystem, native operation or compiler error introduces an unverified assumption. Reuse an already checked declaration within the same version and task.

When documentation and the installed package disagree, inspect the implementation at the matching revision and compile against that package. Report the specific missing evidence when it changes what can be implemented. A guessed name, ported API or empty search result is not a declaration.

Read [MCP tools](references/mcp-tools.md) for connection details, tool selection and query examples. [llms.txt](references/llms.txt) is the compact page index. Search [llms-full.txt](references/llms-full.txt) for the relevant page heading and read that section; load the whole snapshot only when the task requires it. [Snapshot provenance](references/snapshots.md) records their source and checksums.
