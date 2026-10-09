# Server resources and core operations

Paths below are relative to `game/csgo` with the default SwiftlyS2 root. Read the deployed version's help before using console commands. Keep configuration changes in the intended server's tracked configuration and apply them through the repository's deployment tools.

## Startup options

| Option | Purpose and values |
| --- | --- |
| `-sw_path addons/swiftlys2` | Framework root relative to `game/csgo`; this is the default. |
| `-sw_logpath addons/swiftlys2/logs` | Log location relative to `game/csgo`; this is the default. |
| `-sw_hide_logs_in_console 1` | Hides plugin console logs; accepts `1`, `TRUE`, `YES` case-insensitively. File logging remains available. |
| `-sw_loglevel WARNING` | Console minimum: `DEBUG`, `INFO`, `WARNING`, `ERROR`, `OFF`. |

These are server launch arguments. The resource page describes the default log level as all; the checked managed logger defaults to Information. Verify the deployed logger when debugging missing messages. Console visibility, filtering, and file logging have separate settings.

The launcher forwards `-sw_loglevel` and `-sw_hide_logs_in_console` through `SWIFTLY_LOG_LEVEL` and `SWIFTLY_HIDE_LOG_IN_CONSOLE`, which the managed logger reads.

## Command permission overrides

`addons/swiftlys2/configs/command_overrides.jsonc` maps exact command names to required permissions:

```jsonc
{
    "CommandOverrides": {
        "Permissions": {
            "example": "example.permission",
            "sw_example": "example.permission"
        }
    }
}
```

Use lowercase keys for the invoked command name. For a normal `[Command("example")]`, chat `!example` or `/example` uses the key `example`, while the player's console command `sw_example` uses `sw_example`. Configure both forms to apply the same override to both routes, and include the corresponding forms of each alias. The dispatcher records `originalCommandName` before adding its automatic `sw_` prefix. Server-console invocation bypasses this player permission check.

The file provider has `reloadOnChange: true` in the checked source. An empty permission value removes that managed permission requirement, so set it only when public access is intended. Overrides change the framework permission check; handler-specific checks and console-only guards still apply. Verify the intended player access through both chat and client-console routes after a policy edit.

## Console filter

The default path is `addons/swiftlys2/configs/confilter.jsonc`. Entries are rule-name/string pairs compiled as PCRE2 regular expressions:

```jsonc
{
    "example_noise": "^Example recurring console line"
}
```

Escape regex metacharacters when matching literal text, and account for JSON string escaping. `sw confilter reload` recompiles rules and clears filter counters. `sw confilter status` prints enable state and counters. `sw confilter enable` and `sw confilter disable` change runtime filtering. These are server-console commands. Preserve relevant diagnostics when choosing rules; temporarily disable filtering when investigating a suppressed line, then restore the prior state.

## Core configuration

`addons/swiftlys2/configs/core.jsonc` stores the following keys at the JSON root; nested paths below represent nested objects. Missing keys are written with defaults when native configuration loads. Use a server restart for launch/core changes unless the specific subsystem exposes a verified reload route.

| Settings | Default and practical use |
| --- | --- |
| `CommandPrefixes`, `CommandSilentPrefixes` | `["!"]`, `["/"]`: recognized normal and silent chat prefixes. |
| `AutoHotReload` | `true`: watches plugin DLL changes. |
| `ManualLoadPlugins`, `PluginLoadOrder` | `false`, `[]`: the checked loader automatically enumerates plugins when manual mode is false. With manual mode true it initially lists plugins as unloaded, then loads the configured plugin IDs or folder names in order. |
| `Language`, `UsePlayerLanguage` | `"en"`, `true`: server default language and use of available player language. |
| `ProfilerLevel` | `0` disabled, `1` EventPipe in checked source. See the [profiler skill](../../sws2-performance-profiler/SKILL.md) for bounded capture and saving. |
| `ConsoleFilter` | `true`: initial console-filter enable state. |
| `PatchesToPerform` | `[]`: startup patch identifiers; choose verified patches for the target build. |
| `FollowCS2ServerGuidelines` | `true`: CS2 server-guideline behavior; the game-specific key follows the current game name. |
| `Unlocker.Convars`, `Unlocker.ConCommands` | `false`, `false`: expose restricted engine variables or commands when required. |
| `DotnetCrashTracerLevel`, `WindowsFullDump` | `0`, `false`: crash tracer level (0/1) and Windows full memory dumps. |
| `SteamAuth.Mode`, `SteamAuth.AvailableModes` | `"flexible"`, `["flexible", "strict"]`: auth policy. In checked player source, an unauthorized player may expose an unverified ID in flexible mode; strict mode returns 0 until authorized. Use authorization state when verified identity is required. |

`AutoHotReload` controls the watcher independently of manual startup loading. For controlled plugin replacement, verify the current watcher behavior and use the explicit plugin commands.

`ConsoleLogger.Enable` and `ConsoleLogger.ManagedEnable` default to `true`; `WriteIntervalMs` is `2000`. Under `ConsoleLogger.Rotation`, `Enable` is `true`, `Mode` is `"file_count"`, `AvailableModes` is `["file_count", "time_interval"]`, `MaximumFiles` is `60`, and `DeleteOlderThanHours` is `168`. Choose the mode and retention for the deployment's log use.

`Menu.InputMode` defaults to `"button"`, with available modes `"button"` and `"wasd"`. Menu input, paging, buttons, navigation prefix, and sound defaults are covered in [built-in menus](../../sws2-menus/references/built-in.md).

## Core operator commands

These routes use the server console and are guarded against player execution in the checked source:

| Command | Use |
| --- | --- |
| `sw plugins list [page]` | Plugin state, location, and load errors; 20 plugins per page. |
| `sw plugins load <dllName>` | Load the named plugin. |
| `sw plugins unload <dllName>` | Unload the named plugin and invoke its lifecycle cleanup. |
| `sw plugins reload <dllName>` | Reload the named plugin. |
| `sw translations reload` | Regenerate/reload plugin translations. |
| `sw cmds [page]` | Inspect registered command names and declared permissions. |

Use the plugin assembly/folder name for `<dllName>`; the checked resolver accepts a trailing `.dll` case-insensitively. `PluginLoadOrder` accepts plugin metadata IDs or folder names. Confirm status afterward. `sw cmds` lists declared permissions; verify the effective permission override separately.

## Evidence

Checked 2026-10-09 against MCP docs discovery and the full content of [CLI options](https://swiftlys2.net/docs/resources/cli-options), [command overrides](https://swiftlys2.net/docs/resources/command-overrides), [console filter](https://swiftlys2.net/docs/resources/console-filter), and [core configuration](https://swiftlys2.net/docs/resources/core-config). Source commit `78b4c89a6e21de7b6a4d9485295b6448f58e26d9`: `src/core/entrypoint.cpp`, `src/server/configuration/configuration.cpp`, `src/server/players/player.cpp`, `src/engine/consoleoutput/consoleoutput.cpp`, and `managed/src/SwiftlyS2.Core/{Bootstrap.cs,Misc/SwiftlyLogger.cs,Services/CoreCommandService.cs,Modules/Commands/CommandCallback.cs,Modules/Plugins/PluginManager.cs}`. Source resolves the filter-path typo, log-level default, console guard, reload behavior, and plugin-loading semantics.
