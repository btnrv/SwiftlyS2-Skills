# SwiftlyS2 skills

13 agent skills for SwiftlyS2 plugin development. Uses the [Agent Plugins](https://agent-plugins.org/specification) and [Agent Skills](https://agentskills.io/specification) formats, with Claude Code compatibility.

## Install

Codex:

```sh
codex plugin marketplace add btnrv/SwiftlyS2-Skills
```

Open the Plugins Directory, select SwiftlyS2 skills, and install `swiftlys2-skills`.

Claude Code:

```text
/plugin marketplace add btnrv/SwiftlyS2-Skills
/plugin install swiftlys2-skills@swiftlys2-skills
```

Other agents: copy the folders inside `skills/` into your agent's skills directory. Keep all folders together so cross-skill links work. Add `https://swiftlys2.net/api/mcp` as a Streamable HTTP MCP server.

Plugin installs include the MCP configuration. Start a new session after installing.

## Skills

| Skill | Use |
| --- | --- |
| [sws2-api-mcp](skills/sws2-api-mcp/SKILL.md) | Verify documentation, APIs and game data |
| [sws2-authoring](skills/sws2-authoring/SKILL.md) | Scaffold, build and deliver plugins |
| [sws2-code-architecture](skills/sws2-code-architecture/SKILL.md) | Modules, configuration and shared contracts |
| [sws2-text-styling](skills/sws2-text-styling/SKILL.md) | Translations, chat colors and CenterHTML |
| [sws2-entity-management](skills/sws2-entity-management/SKILL.md) | Entity creation and lifetime |
| [sws2-game-assets](skills/sws2-game-assets/SKILL.md) | Compile, deliver and precache assets |
| [sws2-thread-management](skills/sws2-thread-management/SKILL.md) | Game-thread work and async handoffs |
| [sws2-performance-profiler](skills/sws2-performance-profiler/SKILL.md) | Capture and diagnose performance |
| [sws2-menus](skills/sws2-menus/SKILL.md) | Player menus and custom renderers |
| [sws2-panorama-authoring](skills/sws2-panorama-authoring/SKILL.md) | Custom HUD layouts and clicks |
| [sws2-native-and-memory](skills/sws2-native-and-memory/SKILL.md) | Native calls, hooks and gamedata |
| [sws2-traceray](skills/sws2-traceray/SKILL.md) | Aim queries and collision traces |
| [sws2-batching](skills/sws2-batching/SKILL.md) | Batching when explicitly requested |

Start with `sws2-authoring` for a new plugin. Use live MCP declarations for the project's SwiftlyS2 version; bundled documentation is a snapshot.

## License

[GPL-3.0-only](LICENSE). Bundled SwiftlyS2 documentation is credited in [snapshot provenance](skills/sws2-api-mcp/references/snapshots.md).
