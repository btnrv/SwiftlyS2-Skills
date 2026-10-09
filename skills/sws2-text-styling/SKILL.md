---
name: sws2-text-styling
description: Style localized SwiftlyS2 chat, prefixes and CenterHTML messages with the actual CS2 palette, supported markup and typography, and editable presentation resources.
---

# Text styling

Put the plugin prefix in its JSONC configuration and complete messages in `resources/translations/<language>.jsonc`. Store color tokens in those files so operators can change the appearance without recompiling. C# selects translation keys and supplies values; color constants and styled literals belong in the data files.

Use `Core.Translation.GetPlayerLocalizer(player)` for player output and `Core.Localizer` for server-language output. Keep an `en.jsonc` baseline and stable numbered placeholders across languages. The core `UsePlayerLanguage` setting controls whether player output follows the recipient's language. Localize broadcasts per recipient when needed.

SwiftlyS2 chat uses bracket tokens such as `[white]` and `[green]`. Use a neutral base and highlight the words that carry meaning. Reset the message color after the configurable prefix and after highlighted values. Read the [exact palette and message example](references/palette.md) before choosing a token.

For CenterHTML tags, fonts, sizing, escaping, and duration, read [CenterHTML](references/center-html.md). Keep surface-specific translation keys: chat color tokens become control bytes when translations load. Read [localization and escaping](references/localization.md) when inserting player text or sharing messages across destinations.

For a custom XML HUD with its own CSS and buttons, use [sws2-panorama-authoring](../sws2-panorama-authoring/SKILL.md). Confirm send APIs and thread requirements through [sws2-api-mcp](../sws2-api-mcp/SKILL.md).
