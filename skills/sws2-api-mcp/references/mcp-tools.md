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

## Choose a source

| Need | Discover | Resolve |
| --- | --- | --- |
| Development workflow | `docs_search(q)`, `docs_list()` | Read the returned page; there is no `docs_lookup` |
| C# service, type, method or overload | `apidocs_search(q, branch)`, `apidocs_list(category?, branch)` | `apidocs_lookup(name, member?, branch)` |
| Engine field or enum | `schema_search(...)`, `schema_list(project?)` | `schema_lookup(name, project?)` |
| Entity input, output or datamap | `entity_search(...)`, `entity_list(prefix?)` | `entity_lookup(className)` |
| Net message or protobuf enum | `protobuf_search(...)`, `protobuf_list(module?, file?)` | `protobuf_lookup(name, file?)` |
| Game event payload | `gameevent_search(...)`, `gameevent_list(file?)` | `gameevent_lookup(name)` |
| Console variable or command | `convar_search(...)`, `convar_list(module?)` | `convar_lookup(name, module?)` |
| Panorama CSS property | `panorama_search(q)`, `panorama_list()` | `panorama_lookup(name)` |
| Unknown category | `site_search(q)` | Follow up with the domain-specific lookup |

`?` marks optional parameters. API `branch` is `stable` by default or `beta`. Game-data and Panorama tools accept `game: "cs2"`; the API branch selector does not pin game-data dumps to an old CS2 build.

## Narrow searches

All string searches use substring matching. Read `total` and `truncated` when a response supplies them, as `entity_list` does. Other tools return bare arrays or grouped results without those markers. Narrow a broad query before treating its results as complete; absence from a capped search does not establish absence from the API.

| Tool | Additional filters |
| --- | --- |
| `schema_search` | `q`, `field`, `type`, `offset`, `enumvalue`, `networked` |
| `entity_search` | `q`, `kind: input/output/member`, `field` |
| `protobuf_search` | `q`, `kind: message/enum`, `file`, `module` |
| `gameevent_search` | `q`, `field`, `file` |
| `convar_search` | `q`, `kind: all/convar/concommand`, `modulesInclude`, `modulesExclude`, `flagsInclude`, `flagsExclude`, `attrsInclude`, `attrsExclude` |

Use exact names returned by discovery. Schema lookups accept raw and C# names; entity lookups take `className`, not `name`. Protobuf dots may appear as underscores in C# wrapper names. Specify the returned project, module or file when a name is ambiguous.

For `apidocs_lookup` member filtering, methods use the displayed signature (`SendChat(string)`, `Unload()`), while properties use their names (`PlayerLanguage`). A bare method-name miss can still belong to a valid type; inspect the whole type. `protobuf_search.q` also accepts a numeric network-message ID, and `gameevent_search.q` accepts a hexadecimal event hash.

Generic type names can use CLR arity in the index: look up ``CHandle`1`` for `CHandle<T>`. Copy the name returned by `apidocs_search` when a familiar C# spelling misses.

Example request bodies, supplied to the named tool:

```json
{"q":"precache"}
```

Use that with `docs_search`, then inspect `IOnPrecacheResourceEvent` with `apidocs_lookup`:

```json
{"name":"IOnPrecacheResourceEvent","branch":"beta"}
```

For a menu, inspect `IMenuAPI`, `IMenuManagerAPI` and the option type used. For an entity input, inspect its datamap and the managed `AcceptInput` declaration. A field in an engine dump does not establish a managed convenience method with a similar name.

Panorama's index covers CSS properties. It does not establish XML panel types, JavaScript functions, asset delivery or a server-to-client bridge. Read the Panorama skill for those parts.

Reference: [SwiftlyS2 AI tools](https://swiftlys2.net/ai).
