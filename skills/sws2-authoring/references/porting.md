# Port an existing plugin

Use the old plugin to establish required behavior: commands and access, settings and saved data, player-visible feedback, lifecycle and integration with other plugins. Preserve the operator's current file structure and working rules. Adopt the official template's required build and resource settings without reorganizing unrelated code.

Choose one complete route, such as an authorized command changing a player's state and reporting the result. Capture its intended behavior in the available test setup or a reproducible server scenario, then port and validate that route. Repeat for the next feature. Keep compatibility required by active callers or persisted data in scope; add adapters only when that requirement is established.

## Resolve each framework boundary

| Existing concern | SwiftlyS2 owner and reference |
| --- | --- |
| Plugin metadata, SDK/package and resource output | Official template and [layout](../../sws2-code-architecture/references/layout.md); pin the deployed package |
| Global API access or static plugin singleton | Constructor-injected `ISwiftlyCore` and the project's service composition |
| Commands, command context, flags and permissions | [Commands and permissions](commands-permissions.md); verify `ICommandContext.Args`, player/console routes and permission semantics |
| Lifecycle listeners versus game events | [Events and hooks](events-hooks.md); confirm payload, registration owner and phase |
| CounterStrikeSharp `DynamicHook` or old hook-style core events | Prefer the matching typed `Core.GameHooks` contract; use [native interop](../../sws2-native-and-memory/SKILL.md) for uncovered operations |
| Controller/slot/pawn assumptions or delayed player work | [Player state](player-state.md); match connection identity and resolve the current pawn |
| Schema writes and `SetStateChanged` | [Entity management](../../sws2-entity-management/SKILL.md); verify the field's generated notification mechanism |
| User messages and raw protobuf access | [Network messages](netmessages.md); verify generated type, recipients and borrowed lifetime |
| Timers and deferred callbacks | [Thread management](../../sws2-thread-management/SKILL.md); migrate dispatch and cancellation semantics |
| Configuration, database and cross-plugin capabilities | [Integration](../../sws2-code-architecture/references/integration.md); preserve settings/data contracts and use the framework services |
| Menus, translations and custom content | Existing [menu](../../sws2-menus/SKILL.md), [text](../../sws2-text-styling/SKILL.md) and [asset](../../sws2-game-assets/SKILL.md) owners |

Search for the current declaration through [sws2-api-mcp](../../sws2-api-mcp/SKILL.md) before translating a call. Similar names do not prove matching cancellation, threading, ownership or identifier semantics. Port signatures and gamedata for the supported binaries; existing byte patterns are evidence only for their original build.

Build and inspect the published archive, then compare the required behavior on the intended server. Exercise the lifecycle the feature uses, including hot reload when supported. Report any remaining behavior differences and checks that could not run. Remove superseded registrations and paths as each route is replaced so one action has one owner.

Reference: [CounterStrikeSharp porting guide](https://swiftlys2.net/docs/guides/porting-from-css). Preserve this skillset's constructor injection convention when adapting the guide's static `Core` example.
