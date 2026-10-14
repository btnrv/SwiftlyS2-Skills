# Layout and packaging

## Feature folders and ownership

```text
PluginName/
  PluginName.csproj
  src/
    PluginName.cs
    Configuration/PluginConfig.cs
    Features/
      Announcements/
        AnnouncementCommands.cs
        AnnouncementLifecycle.cs
    Integrations/SharedContract.cs
    Library/<Concern>/<Helper>.cs
  resources/
    templates/config.jsonc
    translations/en.jsonc
    gamedata/
    exports/PluginName.Contract.dll
  build/
```

Begin with the entrypoint and one working feature. Create folders as you add configuration, integrations, shared helpers or resources. Preserve an established repository layout; for new plugins, put gameplay under `src/Features/<Feature>/<Subfeature>/`. Choose file boundaries by responsibility, regardless of line count. A small feature can stay in one file. Larger features can separate commands, state transitions, menus and lifecycle handlers.

The entrypoint holds metadata, the `BasePlugin` declaration, constructor and configuration, and coordinates the plugin's lifecycle. Gameplay files declare the same `public partial class PluginName` in the same namespace and use its inherited `Core`. Declare the base type and plugin metadata only in the entrypoint. All partial files form one type and share its instance and lifetime; they are not independent services.

Keep each feature's state, registration, handlers and cleanup in its folder, even though other partial files can access those members. Have features coordinate through methods that manage their state changes. Keep members private unless a public contract requires them. Folders do not require separate namespaces or interfaces.

Use separate types for configuration models, data records, shared contracts, external integrations and components with their own ownership. Keep helpers local until multiple features need them, then place shared helpers under `Library/<Concern>/`. Follow [framework reuse](framework-reuse.md) before adding a helper; it must not duplicate a SwiftlyS2 operation.

## Complete partial-class example

Use these three C# files and one translation file in an official template project, replacing the generated example entrypoint. Keep the template project settings and pin the framework package to the deployment. The command responds to `sw_announcement` in a player's console or the server console and counts requests since plugin load or the last map load.

`src/PluginName.cs`:

```csharp
using SwiftlyS2.Shared;
using SwiftlyS2.Shared.Plugins;

namespace ExamplePlugin;

[PluginMetadata(
    Id = "ExamplePlugin",
    Name = "Example Plugin",
    Version = "1.0.0",
    Author = "Example",
    Description = "Localized announcement example.")]
public partial class PluginName : BasePlugin
{
    public PluginName(ISwiftlyCore core) : base(core)
    {
    }

    public override void Load(bool hotReload)
    {
        RegisterAnnouncements();
    }

    public override void Unload()
    {
        StopAnnouncements();
    }
}
```

`src/Features/Announcements/AnnouncementCommands.cs`:

```csharp
using SwiftlyS2.Shared.Commands;

namespace ExamplePlugin;

public partial class PluginName
{
    private void OnAnnouncement(ICommandContext context)
    {
        var localizer = context.Sender is { } player
            ? Core.Translation.GetPlayerLocalizer(player)
            : Core.Localizer;

        _announcementCount++;
        context.Reply(localizer["announcement.message", _announcementCount]);
    }
}
```

`src/Features/Announcements/AnnouncementLifecycle.cs`:

```csharp
using System;
using SwiftlyS2.Shared.Events;

namespace ExamplePlugin;

public partial class PluginName
{
    private Guid _announcementCommandId;
    private int _announcementCount;

    private void RegisterAnnouncements()
    {
        _announcementCount = 0;
        _announcementCommandId = Core.Command.RegisterCommand(
            "announcement", OnAnnouncement,
            helpText: "Show a localized announcement.");
        Core.Event.OnMapLoad += OnAnnouncementMapLoad;
    }

    private void OnAnnouncementMapLoad(IOnMapLoadEvent @event)
    {
        _announcementCount = 0;
    }

    private void StopAnnouncements()
    {
        Core.Command.UnregisterCommand(_announcementCommandId);
        Core.Event.OnMapLoad -= OnAnnouncementMapLoad;
        _announcementCount = 0;
    }
}
```

`resources/translations/en.jsonc`:

```jsonc
{
    "announcement.message": "Announcement {0}: welcome to the server."
}
```

The entrypoint starts and stops the feature once. The feature owns its command ID, counter and map subscription. SwiftlyS2 registers the plugin entrypoint automatically, so do not register it again through `Core.Registrator`. Do not add a `[Command]` attribute to this programmatically registered handler. Fixed commands can use attribute discovery as described in [commands](../commands-permissions.md).

SwiftlyS2 clears its tracked command/event registrations on plugin unload. The stop method also supports stopping the feature earlier and resets its state. Use the framework's cleanup APIs and avoid a second registration registry. Define lifetimes for timers, entities, native hooks and background work when a feature uses them. Follow the contracts in [events](../events-hooks.md) and [thread management](../../../sws2-thread-management/SKILL.md).

The counter resets on hot reload. Features that depend on connected players or existing game state must reconstruct that state during load. Only those features need state reconstruction on reload.

Invoke `sw_announcement` twice and check for replies numbered 1 and 2. The next reply after a map load or plugin reload should be numbered 1. After plugin unload, the command should no longer invoke the feature. For any additional supported language, add a translation file and check player-language selection. These behaviors need server checks; compilation alone cannot verify them.

## Template and distribution

The current template targets `net10.0`, enables nullable references and implicit usings, and excludes `examples/**/*.cs` from compilation. Preserve its package runtime exclusions: the server supplies SwiftlyS2 and the listed framework dependencies. Pin `SwiftlyS2.CS2` to the deployed runtime's compatible version for a reproducible build.

The template copies `resources/gamedata`, `resources/templates` and `resources/translations`. Add packaging rules for other required resources, such as contract exports. Exclude any contracts project inside the plugin directory from the plugin's default recursive compile glob.

`dotnet publish -c Release` runs the template's `CreateZip` target. The documented distribution paths are `build/publish/<PluginId>/` and `build/<AssemblyName>.zip`. Inspect the generated project's evaluated `PublishDir`, `OutputPath` and zip target when the template version or command-line output overrides differ. The zip must contain `<PluginId>/<PluginId>.dll` plus its resources and required private dependencies. Deploy that folder under `addons/swiftlys2/plugins/`.

Keep contract source in a separate `PluginName.Contract` project containing public interfaces and value types. Deploy the resulting DLL at `plugins/<Provider>/resources/exports/PluginName.Contract.dll`. SwiftlyS2 scans these export folders before loading plugins and shares their types across plugin load contexts. Consumers reference the same contract assembly identity. Replacing an exported contract DLL requires a server restart with this loader: exports load during framework initialization into a non-collectible context, and plugin reload does not reload them.

Reference: [Plugin template](https://github.com/swiftly-solution/swiftlys2/tree/master/managed/SwiftlyS2.PluginTemplate/templates). Use the generated project's packaging rules for the chosen template version.
