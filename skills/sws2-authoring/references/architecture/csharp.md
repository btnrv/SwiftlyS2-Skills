# C# conventions

Follow repository instructions, preserving the existing style when extending code. If a new plugin has no chosen style, use four spaces per indentation level and a tab width of four. Use file-scoped namespaces, put `using` directives at the top, and add braces to control-flow blocks, including early returns.

## Names and declarations

- Use PascalCase for types, methods, properties and constants; camelCase for parameters and locals; `_camelCase` for private fields in new code. Preserve the repository's existing field convention when extending it.
- State accessibility explicitly. Keep implementation methods and feature state private unless a caller or framework contract requires wider access.
- Use `readonly` for fields whose references do not change. The contents of a readonly collection can still change, so its state is not immutable.
- Use descriptive behavior names such as `RegisterAnnouncements`, `OnAnnouncement` and `StopAnnouncements`. Name files after the responsibility they contain.
- Keep one root namespace for the plugin's partial class. Feature folders organize files; changing a partial file's namespace creates a different type.

## Types and control flow

Use nullable annotations where values can be absent, and narrow them before dereferencing. Use `var` when the initializer makes the type clear; write the type explicitly when it conveys a domain distinction. Follow [game values](game-values.md) when using existing enums and named values. Preserve those distinctions in casts and avoid replacement enums.

Keep handlers short so their event contract is easy to see. Use guard clauses for required input, permission or state checks. Extract a method when it names a plugin behavior or manages a separate lifetime, and call existing framework helpers directly. Avoid generic helper layers, empty exception handlers and retries for failures that have not been demonstrated.

This partial file uses the entrypoint from [layout](layout.md) and requires a `welcome.message` translation key:

```csharp
using SwiftlyS2.Shared.Commands;

namespace ExamplePlugin;

public partial class PluginName
{
    private void ReplyWithWelcome(ICommandContext context)
    {
        if (context.Sender is not { } player)
        {
            return;
        }

        var localizer = Core.Translation.GetPlayerLocalizer(player);
        context.Reply(localizer["welcome.message"]);
    }
}
```

The syntax example leaves registration to the feature that needs it. Use comments to explain engine constraints, ownership or reasons the code cannot express, and omit comments that repeat a statement. Choose language features supported by the SDK and project settings. Do not add an abstraction just to use a language feature.

## Dependencies and asynchronous work

Partial feature files use the entrypoint's inherited `Core` and shared instance members. For a separate component, pass `ISwiftlyCore` and other dependencies through its constructor. Keep any service provider at the composition root; see [dependency injection](dependency-injection.md).

Use `Task` or `Task<T>` for plugin-owned asynchronous methods, with an `Async` suffix. Match framework callback signatures, including `ValueTask` where required. Keep synchronous command/event handlers free of untracked `async void` work; coordinate asynchronous work through its owner and the framework scheduler. Use the database or network library's asynchronous APIs and return engine operations to the game thread.

Copy native-backed values into owned data before leaving their valid scope. A `ref`, span or wrapper remains subject to the lifetime of the engine memory it references. Follow [thread management](../../../sws2-thread-management/SKILL.md) for handoffs, cancellation and shutdown. These rules are required for correctness. Keep framework APIs in place when considering speculative performance changes.

References: [Thread safety](https://swiftlys2.net/docs/development/thread-safety), [core events](https://swiftlys2.net/docs/development/core-events), [dependency injection](https://swiftlys2.net/docs/guides/dependency-injection).
