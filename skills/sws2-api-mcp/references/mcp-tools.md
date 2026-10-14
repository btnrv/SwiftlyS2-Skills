# MCP tools

The server uses Streamable HTTP at `https://swiftlys2.net/api/mcp`. A client may expose tool names with a server prefix; use the names it advertises. The documentation and API tools belong to this same endpoint.

```json
{
    "mcpServers": {
        "swiftlys2": {
            "url": "https://swiftlys2.net/api/mcp"
        }
    }
}
```

The [main skill's tool table](../SKILL.md#mcp-tool-table) is the tool inventory. Use the sections below for [search filters](#narrow-searches), [request examples](#request-examples), [domain definitions](#resolve-domain-definitions) and [missing-capability checks](#check-a-missing-capability).

## Versions and source selection

API `branch` is `stable` by default or `beta`. Select it explicitly to match the project's release or prerelease package. Their source links follow moving `master` and `beta` branches; resolve discrepancies against the installed package and matching source revision. Game-data, Panorama and site-search tools accept `game: "cs2"`; the API branch selector does not pin game-data dumps to an old CS2 build.

Use documentation for the workflow, API declarations for managed calls, and game-data lookups for engine details. Search snippets alone are insufficient. Read the full relevant documentation section and inspect the owning API type. A field in an engine dump does not establish a managed convenience method with a similar name.

## Narrow searches

Text queries generally use substring matching; exact filters are noted below. Read `total` and `truncated` when a response supplies them, as `entity_list` does. Other tools return bare arrays or grouped results without those markers. Narrow a broad query before treating its results as complete; absence from a capped search does not establish absence from the API.

| Tool | Additional filters |
| --- | --- |
| `schema_search` | `q`, `field`, `type`, `offset`, `enumvalue`, `networked` |
| `entity_search` | `q`, `kind: input/output/member`, `field` |
| `protobuf_search` | `q`, `kind: message/enum`, `file`, `module` |
| `gameevent_search` | `q`, `field`, `file` |
| `convar_search` | `q`, `kind: all/convar/concommand`, `modulesInclude`, `modulesExclude`, `flagsInclude`, `flagsExclude`, `attrsInclude`, `attrsExclude` |

All filters in this table are optional. `q` is required for `docs_search`, `apidocs_search`, `panorama_search` and `site_search`.

- Schema `offset` and `enumvalue` take exact values as decimal or `0x`-hex strings; `networked` is a boolean. `schema_search` searches across projects and has no `project` argument. Use `project` on `schema_list` and `schema_lookup`.
- Protobuf `file` is a substring filter in searches and an exact file name in lists/lookups; search `module` is exact.
- Convar include/exclude filters take arrays. Includes require all listed flags or attributes; excludes reject entries with any listed flag or attribute. Module filters select or exclude the listed modules. Engine flags use lowercase names such as `cheat` and `replicated`. Attribute values are `has_default`, `has_min`, `has_max`, `has_callback` and `has_completion_callback`.
- Lists browse categories or containers: API categories, schema projects, protobuf modules/files, event files and convar modules. Entity lists use a case-insensitive `prefix`; an unfiltered entity list returns only the first page. Schema name lists are also capped. Use narrower searches for discovery.

## Exact lookups

Use exact names returned by discovery. Schema lookups accept raw and C# names; entity lookups take `className`, not `name`. Protobuf dots may appear as underscores in C# wrapper names. Specify the returned project, module or file when a name is ambiguous.

For `apidocs_lookup` member filtering, methods use the displayed signature (`SendChat(string)`, `Unload()`), while properties use their names (`PlayerLanguage`). A bare method-name miss can still belong to a valid type; inspect the whole type. `protobuf_search.q` also accepts a numeric network-message ID, and `gameevent_search.q` accepts a hexadecimal event hash.

Generic type names can use CLR arity in the index: look up ``CHandle`1`` for `CHandle<T>`. Copy the name returned by `apidocs_search` when a familiar C# spelling misses.

## Request examples

Pass JSON objects to the advertised tool, with the branch chosen for the actual package. These API examples use `beta`; use `stable` for a release package.

Find the precache workflow with `docs_search`:

```json
{"q":"precache"}
```

Read the returned documentation section, then find the API with `apidocs_search`:

```json
{"q":"precache","branch":"beta"}
```

Copy the owning type name from the results into `apidocs_lookup`, for example:

```json
{"name":"IOnPrecacheResourceEvent","branch":"beta"}
```

For entity input discovery, call `entity_search`:

```json
{"q":"CCSPlayerPawn","kind":"input","game":"cs2"}
```

Resolve a returned class with `entity_lookup`:

```json
{"className":"CCSPlayerPawn","game":"cs2"}
```

Separately search `AcceptInput` with `apidocs_search`, then retrieve its owning type and displayed overload with `apidocs_lookup`. Use the verified managed API to invoke the documented input.

For a menu, search the requested behavior, inspect the returned menu API, manager and option types, and build the plugin's menu using those verified components. Apply the missing-capability checks if they cannot provide the requested behavior.

Panorama's index covers CSS properties. It does not establish XML panel types, JavaScript functions, asset delivery or a server-to-client bridge. Read the Panorama skill for those parts.

## Resolve domain definitions

Search both the game concept and raw engine field/type name in the API and relevant game-data indexes. Follow fields and parameters to enums, flags and named constants even when the managed member stores a primitive number.

If a schema lookup misses in one project, omit `project` or use `schema_search` across projects. A related enum may be indexed separately from the field that uses it. Retrieve the complete matching declarations with `apidocs_lookup`, `schema_lookup` or `protobuf_lookup`.

When the index does not expose a definition, inspect the generated wrapper and definitions in the matching package or source revision. Record the managed member type, related domain type, named members, underlying representation and whether values are discrete states or bit flags. Reuse existing definitions instead of recreating enums or embedding raw values.

## Check a missing capability

Before concluding that SwiftlyS2 lacks a capability:

1. Search the intended behavior and related terminology with `docs_search` and `apidocs_search`, not only a guessed function name. Browse the relevant API category when needed; use `site_search` if the source category is unclear.
2. Retrieve full candidate type declarations. If member narrowing misses, remove `member`; copy exact names and displayed signatures from discovery. Narrow capped searches and follow related types and domain definitions.
3. Inspect the matching installed package or source revision for relevant services, helpers, generated wrappers and definitions. Resolve version disagreements and compile any proposed API usage against that package before treating it as usable.

Do not write a replacement service, local enum or native workaround merely because discovery was unsuccessful. If these checks confirm a gap, apply the main skill's planning/implementation constraint rule. If access to MCP, the matching source or the package prevents verification, report the missing evidence without claiming the capability is absent. Bundled documentation can supply prose context but cannot satisfy the mandatory MCP API discovery step.

Reference: [SwiftlyS2 AI tools](https://swiftlys2.net/ai).
