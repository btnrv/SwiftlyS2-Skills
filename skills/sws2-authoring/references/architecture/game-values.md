# Game values and domain types

Use existing API, schema and protocol types and their named members for CS2 states, flags and identifiers. Avoid magic numbers, literal identifiers and locally recreated definitions. Preserve domain types throughout gameplay logic so the compiler can check compatibility. Convert a named value to a primitive only when the API requires primitive storage, matching the declaration's width and signedness. Do not use casts to bypass type checking.

Use [sws2-api-mcp](../../../sws2-api-mcp/SKILL.md) to resolve the managed member declaration and related game definitions before using them. Search for related definitions even when the field is primitive. If an index omits the enum, inspect the matching package or source. An empty search result alone does not justify a local replacement.

Use a framework operation when it already covers the behavior. For example, `IPlayer.IsAlive` and `Core.PlayerManager.GetAlive()` handle ordinary alive-player checks and selection. Do not recreate them from schema fields. Use raw domain comparisons when a feature needs a state the higher-level operation does not expose.

For discrete states, compare against the named state or explicit set of states the behavior requires. Being unequal to one state does not establish another. For bit flags, use the declared named masks and choose the bitwise test for whether any or all requested flags must be present. Follow these rules for all API and game-data values with existing domain definitions.

Given a `LifeState_t state` value, these comparisons distinguish dying from dead using the existing [LifeState_t](https://swiftlys2.net/schema-viewer/cs2/client/LifeState_t) enum. Place them inside the method:

```csharp
bool isDying = state == LifeState_t.LIFE_DYING;
bool isDead = state == LifeState_t.LIFE_DEAD;
```

The enum is in `SwiftlyS2.Shared.SchemaDefinitions`. Avoid `state != 0`. If the requirement is "not alive," use `state != LifeState_t.LIFE_ALIVE`; that also includes dying and respawn states, so it is not equivalent to `isDead`. Verify the managed property's declared type before applying this comparison to a schema wrapper.

For flags, use the existing [ConvarFlags](https://swiftlys2.net/api-docs/stable/convars/convarflags) type and named members. This fragment assumes a `ConvarFlags flags` value; place the `using` directive at file scope and the comparisons inside the method:

```csharp
using SwiftlyS2.Shared.Convars;

ConvarFlags requested = ConvarFlags.REPLICATED | ConvarFlags.NOTIFY;

bool hasAny = (flags & requested) != ConvarFlags.NONE;
bool hasAll = (flags & requested) == requested;
```

Choose the any/all test the behavior requires. Comparing `flags == requested` also requires every other flag to be absent. Keep these expressions inline without adding a generic flag-helper layer. Reuse both enums. Define local enums only for plugin-owned states with no existing framework or game definition.
