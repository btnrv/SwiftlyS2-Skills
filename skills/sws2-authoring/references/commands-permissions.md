# Commands and permissions

Command handlers belong in their feature's `src/<Module>/<Type>.cs` and receive dependencies through constructors.

## Fixed and runtime registration

For a fixed entrypoint, put `[Command]` on a method with the signature `void Handler(ICommandContext context)`. The entrypoint instance is discovered automatically; register another handler instance once through `Core.Registrator.Register(instance)`. [`CommandAlias("shortname")`](https://swiftlys2.net/api-docs/stable/commands/commandalias) belongs on the same method and can be repeated.

```csharp
using SwiftlyS2.Shared.Commands;

public sealed class EchoCommands
{
    [Command("echo", permission: "example.echo", helpText: "Repeat one argument.")]
    [CommandAlias("repeat")]
    public void Echo(ICommandContext context)
    {
        if (context.Args.Length != 1)
        {
            context.Reply("Usage: echo <text>");
            return;
        }

        context.Reply(context.Args[0]);
    }
}
```

For names selected by configuration or an independently enabled feature, use `Core.Command.RegisterCommand(name, handler, registerRaw: false, permission: permission, helpText: helpText)`. Save its `Guid` in the owning service and call `UnregisterCommand(guid)` when that service stops. The string overload unregisters all that service's listeners with the matching name; the `Guid` identifies one registration.

By default `echo` registers as console command `sw_echo`; `registerRaw: true` omits this prefix. Normal chat prefixes come from core configuration. `RegisterCommandAlias(commandName, alias, registerRaw: false)` adds an alternate name; aliases need not be abbreviations. Register the target command first. An alias has its own raw-prefix choice.

Plugin unload cleans up its registered commands, aliases and client hooks. There is no individual alias-unregistration method: aliases are released when the plugin command service is disposed. `UnregisterCommand(guid)` removes the command listener, not its aliases. If independently removable aliases are required, register the alternate names as commands and own their returned handles.

## Context and access

`Args` is a `string[]` containing arguments, so use `Length` and index from zero. `CommandName` holds the command name. Parse and validate the arguments before passing them to the feature service.

Use `context.Reply(...)` to respond to the caller. Server console has `IsSentByPlayer == false` and `Sender == null`. A player-only action checks `IsSentByPlayer` before using `Sender`; a command intended for both routes handles each explicitly. The API spelling is `IsSlient`. Keep the registered handler `void`; coordinate any asynchronous work through the owner and [thread-management guidance](../../sws2-thread-management/SKILL.md).

The `permission` argument checks player access before the handler runs. Server-console calls bypass this check, so permission alone does not make a command player-only. See [server resources](server-resources.md#command-permission-overrides) for command overrides and their chat, console and alias keys.

`Core.Permission` uses SteamID64 (`player.SteamID`), not a slot. `PlayerHasPermission(steamId, key)` checks one key; `PlayerHasPermissions(steamId, keys)` requires all keys. Use these checks for other entrypoints, such as a menu action or client hook. A command's declared permission usually makes an identical check in its handler unnecessary.

The global `addons/swiftlys2/configs/permissions.jsonc` maps players to permissions or groups:

```jsonc
{
  "Permissions": {
    "Players": { "76561198000000000": ["example.admins"] },
    "PermissionGroups": {
      "__default": ["example.echo"],
      "example.admins": ["example.admin.*"]
    }
  }
}
```

`__default` applies to everyone. Groups can include other groups, and grants support `*` and prefix wildcards such as `example.admin.*`. `GetPlayerPermissions(steamId)` includes inherited entries and leaves wildcards unexpanded. `AddSubPermission(parent, child)` introduces a runtime implication; remove an owned implication with `RemoveSubPermission` when it is no longer needed.

Runtime `AddPermission` grants belong to the shared permission manager and are not scoped automatically to a plugin. If the feature grants temporary access, define its expiry and remove only the grant it owns. `RemovePermission` also removes the matching configured direct grant from the current in-memory player state. `ClearPermissions` removes all direct base and temporary grants for that player; defaults still apply. Reserve it for administrative operations because plugin cleanup could remove another feature's grants. Use the plural name; `ClearPermission` is obsolete. These calls do not write the permission configuration file.

## Intercepting client commands

`[ClientCommandHookHandler]` or `Core.Command.HookClientCommand(handler)` receives `HookResult Handler(int playerId, string commandLine)` for client console commands, including commands the plugin did not register. This requires no `[Command]` registration; `registerRaw` only controls names for registered commands.

Filter by the command token before resolving players or doing feature work. A raw `StartsWith("jointeam")` also matches longer unrelated names; parse the name and arguments needed by the feature. Return `HookResult.Continue` to let processing continue or `HookResult.Stop` to block the command. Hooks have no declared command permission, so check access in the handler.

Save a programmatic hook's `Guid` and call `UnhookClientCommand(guid)` in its owner's shutdown. Chat interception is separate: `HookClientChat` takes `(int playerId, string text, bool teamOnly)` and pairs with `UnhookClientChat`. Register a given callback through one path so the same event is not processed twice.

`IsCommandRegistered`, `GetAllCommandsInfo`, and `GetCommandsByPlugin` expose registered names, declared permissions and help text. Test access through the intended player and console routes, including aliases and feature unload/reload.

API details: [command service](https://swiftlys2.net/api-docs/stable/commands/icommandservice), [command context](https://swiftlys2.net/api-docs/stable/commands/icommandcontext), and [permission manager](https://swiftlys2.net/api-docs/stable/permissions/ipermissionmanager).
