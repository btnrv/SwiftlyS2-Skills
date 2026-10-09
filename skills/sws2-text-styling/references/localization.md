# Localization and escaping

Keep chat, console, plain center, CenterHTML, and custom HUD messages in separate keys when their presentation differs. The translation loader applies `Helper.Colored()` to every string before the destination is known. Lowercase chat tokens become raw control bytes at that point. Native console `ClearColors` removes bracket tokens, so it cannot remove bytes already introduced during loading. Sending a chat translation to another surface can therefore leak control characters into its text.

Use lowercase tokens in chat translations for that managed first pass. Configuration prefixes receive native parsing only at send time. `[teamcolor]` and `[newline]` remain tokens until the native chat path processes them.

An existing `resources/translations` folder requires `en.jsonc`. Other languages fall back to English for missing entries. A key missing from both selected and English resources renders as `<language>.<key>`. Keep the same keys and placeholder order across language files. `GetPlayerLocalizer` honors core `UsePlayerLanguage`; when disabled, it uses core `Language`.

Place values in numbered format placeholders. Escape literal braces in a formatted resource as `{{` and `}}`; braces inside an argument are ordinary argument text. Separate `chat.status.ready` and `chat.status.waiting` (or corresponding CenterHTML keys) let C# select semantic state while resources own color and markup.

## Player-controlled values

For CenterHTML text nodes, encode values with `System.Net.WebUtility.HtmlEncode` before passing them to the localizer. This escapes `&`, `<`, `>`, and quotes. Keep values in text placeholders; trusted resource content defines attributes and classes.

For chat, HTML encoding does not neutralize bracket tokens. Apply a display-text policy to player-supplied names or messages before interpolation: remove engine control characters and line breaks, and replace ASCII `[`/`]` with ordinary parentheses when arbitrary token-looking text should remain literal. Then append the intended reset token in the translation. A player's `[red]` or `[newline]` text otherwise reaches the native parser as presentation instructions.

For custom HUD dialog variables, pass plain text using the HUD API. Its XML and CSS presentation remain authored resources. Check the target client's parsing rules when adding a new text surface.

References: [TranslationService](https://github.com/swiftly-solution/swiftlys2/blob/master/managed/src/SwiftlyS2.Core/Modules/Translations/TranslationService.cs), [Localizer](https://github.com/swiftly-solution/swiftlys2/blob/master/managed/src/SwiftlyS2.Core/Modules/Translations/Localizer.cs), [native color parsing](https://github.com/swiftly-solution/swiftlys2/blob/master/src/api/shared/string.cpp).
