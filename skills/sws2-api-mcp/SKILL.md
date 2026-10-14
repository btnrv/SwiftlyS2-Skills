---
name: sws2-api-mcp
description: Look up SwiftlyS2 documentation, C# API declarations (methods, properties, events, enums, overloads, namespaces) and CS2 game data (schema fields, entity inputs and outputs, protobuf net messages, game event payloads, convars, Panorama CSS) through the SwiftlyS2 MCP. Use whenever creating, changing, reviewing or debugging a SwiftlyS2 plugin, before choosing any framework call, property, enum or game identifier, when an API name or signature is unknown or a build fails on one, and when the user asks whether SwiftlyS2 has something or mentions swiftlys2.net, apidocs, the schema viewer or the SwiftlyS2 MCP.
---

# SwiftlyS2 API and documentation

You must use the SwiftlyS2 MCP during planning and implementation before choosing framework APIs or writing framework-dependent behavior. Find and use the existing API functions, services, enums and named constants. Do not guess declarations or independently reimplement capabilities that SwiftlyS2 already provides. Plugin-specific application logic may compose verified APIs.

## MCP tool table

Use the tool names advertised by the connected client; they may include a server prefix. The notation below shows named arguments, not positional calls. `?` marks optional arguments; search filters beyond `q` are detailed in the linked reference.

| Purpose | Discover or browse | Resolve and use |
| --- | --- | --- |
| Development workflow | `docs_search(q)`, `docs_list()` | Read the full returned page or matching bundled section; no `docs_lookup` exists. |
| C# API functions, services and types | `apidocs_search(q, branch?)`, `apidocs_list(category?, branch?)` | `apidocs_lookup(name, member?, branch?)`: verify the declaration and overload before using it. |
| Engine fields and enums | `schema_search(q?, ...)`, `schema_list(project?)` | `schema_lookup(name, project?)`: inspect fields, types and enum members; verify the C# wrapper separately. |
| Entity inputs, outputs and datamaps | `entity_search(q?, ...)`, `entity_list(prefix?)` | `entity_lookup(className)`: confirm the input/output/member and its managed API. |
| Network messages and protobuf enums | `protobuf_search(q?, ...)`, `protobuf_list(module?, file?)` | `protobuf_lookup(name, file?)`: inspect fields and values, then resolve the managed wrapper. |
| Game event payloads | `gameevent_search(q?, ...)`, `gameevent_list(file?)` | `gameevent_lookup(name)`: inspect payload fields and verify the managed event type. |
| Console variables and commands | `convar_search(q?, ...)`, `convar_list(module?)` | `convar_lookup(name, module?)`: inspect flags and metadata; look up the C# access API. |
| Panorama CSS properties | `panorama_search(q)`, `panorama_list()` | `panorama_lookup(name)`: read the property contract; this index does not describe XML or JavaScript APIs. |
| Unknown source category | `site_search(q)` | Route results to the appropriate lookup or full documentation page. |

API tools accept `branch: "stable"` (default) or `"beta"`. Game-data, Panorama and site-search tools accept `game: "cs2"`. Read [MCP tools](references/mcp-tools.md) for connection details, complete search filters, request examples and lookup troubleshooting.

## Required lookup workflow

1. Read the project's SwiftlyS2 package and deployed runtime versions. Select `stable` for release packages or `beta` for prereleases. Their source links follow `master` and `beta`, respectively. Both indexes move over time; resolve version differences against the source tag and package used by the project.
2. Find the relevant development page with `docs_search` or `docs_list`. Read its full section through the returned URL or the bundled documentation snapshot. Search results contain excerpts, not the whole contract. Bundled prose does not replace MCP API discovery.
3. Search the intended behavior with `apidocs_search` to find existing functions and their owning types, then use `apidocs_lookup` for the declarations and overloads. Confirm the namespace, arguments, return value, ownership, threading requirements and cleanup mechanism before using them. Follow linked documentation or matching source when those requirements are not in the declaration. For member narrowing, copy the displayed signature, such as `SendChat(string)`; properties use bare names. If a member lookup misses, retrieve the complete type before concluding it is absent.
4. Resolve engine details through the matching data tool: schema fields, entity inputs, protobuf fields, event payloads, convars or Panorama properties. Verify the corresponding C# wrapper separately.
5. Keep the selected declarations and their version available while writing and reviewing code. Recheck when a new subsystem, native operation or compiler error introduces another API assumption. Reuse declarations within the same version and task.

When documentation and the installed package disagree, inspect the implementation at the matching revision and compile against that package. Report the specific missing evidence when it changes what can be implemented. A guessed name, ported API or empty search result is not a declaration.

## Resolve related domain definitions

Follow fields and parameters to their domain definitions, including existing enums, flag types and named constants. A primitive storage type does not establish that no corresponding domain definition exists. Search the game concept and raw engine name in both API and game-data indexes, then retrieve the full matching declarations with `apidocs_lookup`, `schema_lookup` or `protobuf_lookup`.

If a lookup misses, follow the [domain-definition procedure](references/mcp-tools.md#resolve-domain-definitions) before creating local enums, flags or constants. Use the existing definitions when available.

## Missing capability or unavailable MCP

An empty search result does not prove that an API is missing. Complete the [missing-capability checks](references/mcp-tools.md#check-a-missing-capability) before declaring a gap. If MCP access is unavailable, identify that as an access constraint, not evidence that the API does not exist.

- During planning, ask the user how to handle a confirmed capability gap or unavailable MCP access before finalizing the affected approach.
- During implementation, do not silently build a substitute or bypass the MCP requirement. Continue independent work and raise the constraint in the conclusion: describe the requested behavior, checks performed, missing evidence or capability, and work left incomplete. Follow an already agreed resolution if one exists.

[llms.txt](references/llms.txt) is the compact page index. Search [llms-full.txt](references/llms-full.txt) for the relevant page heading and read that section; load the whole snapshot only when the task requires it.
