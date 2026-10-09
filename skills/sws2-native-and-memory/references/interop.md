# Gamedata and native calls

Match the native return type, calling convention, argument layout, and explicit `this` pointer in the delegate. Borrowed entity, user-command, and KeyValues pointers retain their engine lifetime. Establish whether a call borrows, copies, or owns its argument before allocating or freeing it.

The template includes `resources/gamedata/{signatures,offsets,patches}.jsonc`. Each file is optional; signature and offset names missing from plugin data resolve against framework gamedata. Signature entries name a library and platform byte patterns; offset entries contain platform numbers. Inspect the resolver for the target build. Schema byte offsets, vtable slots, and instruction-relative addresses have separate meanings.

Each file is a JSONC object keyed by the name passed to `Core.GameData`:

| File | Fields within each named entry |
| --- | --- |
| `signatures.jsonc` | `lib` (for example `server`), `windows` and `linux` pattern strings |
| `offsets.jsonc` | `windows` and `linux` integer offsets or vtable slots |
| `patches.jsonc` | `signature` naming the target signature, `windows` and `linux` byte strings |

Use values recovered for the supported binaries and record their source. The resolver checks plugin entries before framework gamedata. Namespace plugin-specific keys to avoid unintentionally replacing a framework lookup. A byte offset is added to an address; a vtable slot goes to the vtable API without multiplying it by pointer size.

Establish the target ABI before adapting the template example. This fragment assumes an `int(int, int)` function:

```csharp
delegate int VerifiedFunction(int first, int second);

if (!Core.GameData.TryGetSignature("VerifiedFunction", out var address))
{
    throw new InvalidOperationException("Required function signature is unavailable.");
}
var function = Core.Memory.GetUnmanagedFunctionByAddress<VerifiedFunction>(address);
var hookId = function.AddHook(next => (first, second) => next()(first, second));
var result = function.Call(1, 2);
// Retain function and hookId. In Unload: function.RemoveHook(hookId);
```

`next()` continues the hook chain. `CallOriginal` bypasses SwiftlyS2 hooks; use it when that behavior is required. Match `Alloc` and `Free` for plugin-owned memory and keep borrowed addresses within their owner's lifetime.

For a real offset example, inspect `GameHooks/Hooks/RunCommand.cs`: it resolves `CPlayer_MovementServices::RunCommand` with `Core.GameData.GetOffset`, obtains the class vtable with `Core.Memory.GetVTableAddress`, and passes the slot to `GetUnmanagedFunctionByVTable<CPlayerMovementServicesRunCommand>`. The delegate is `nint(nint pMovementServices, nint pUserCmd)`. Plugin features can use the public RunCommand hook; this implementation demonstrates offset resolution and ABI. Its numeric slots belong to that build.

For an uncovered field, establish the target binary's layout, store a named platform byte offset, and apply it to a valid object of the required type. For legacy KeyValues inspect `Natives/Structs/KeyValues.cs`; its unsafe pointers differ from disposable entity spawn values.

Reference: [Native guide](https://swiftlys2.net/docs/development/native-functions-and-hooks).
