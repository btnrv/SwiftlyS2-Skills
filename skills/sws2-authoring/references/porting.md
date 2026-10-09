# Port an existing plugin

Use the old plugin to identify the behavior to preserve: commands and access, settings and saved data, player-visible feedback, lifecycle and integration with other plugins. Apply the official template's required build and resource settings.

Port one feature at a time, such as an authorized command that changes a player's state and reports the result. Capture its behavior in a test or a reproducible server scenario, then port it and run the test or scenario. Add compatibility adapters where active callers or persisted data require them.

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
| Menus, translations and custom content | [Menu](../../sws2-menus/SKILL.md), [text](../../sws2-text-styling/SKILL.md) and [asset](../../sws2-game-assets/SKILL.md) skills |

Read the current declaration through [sws2-api-mcp](../../sws2-api-mcp/SKILL.md) before translating a call. Methods with similar names can differ in cancellation, threading, ownership or identifier semantics. When porting signatures and gamedata, check existing byte patterns against the supported binaries.

Build and inspect the published archive, then compare behavior on the intended server. Test the lifecycle events the feature uses, including hot reload when supported. Report any remaining behavior differences and checks that could not run. Remove old registrations and code as each feature is ported so each action has one owner.

Reference: [CounterStrikeSharp porting guide](https://swiftlys2.net/docs/guides/porting-from-css). Use constructor injection when adapting the guide's static `Core` example.
