# Host-Plugin Extensions

This reference is for Java/Kotlin developers writing a Paper plugin that extends Jetpack, not for `.jet` script authors. It specifies the public API a host plugin uses to register its own modules and global functions into the Jetpack runtime, so scripts can call into it. The host plugin compiles against the Jetpack artifact (classes under `dev.jetpack`) with `compileOnly` and declares Jetpack in its `plugin.yml` (`depend: [Jetpack]`).

## Entry point

All registration goes through the running `JetpackPlugin`, obtained with `server.pluginManager.getPlugin("Jetpack") as JetpackPlugin` in `onEnable`. Three public methods:

- `registerModule(owner: Plugin, builtin: Builtin): RegistrationHandle` — registers global functions and value methods.
- `registerModule(owner: Plugin, name: String, value: JetValue.JModule, fields: Map<String, JetType>, dynamic: Boolean = false): RegistrationHandle` — registers a named module imported with `using name`.
- `batchUpdate(block: () -> Unit)` — groups registrations into a single script reload.

`owner` is always the host plugin (`this`). Registration records the owner so Jetpack can clean up automatically.

## Named modules

A named module appears to scripts exactly like a built-in module (`math`, `storage`). Build it from two aligned pieces:

- `value: JetValue.JModule` — the runtime members. Construct with `JModule(mutableMapOf(...))`. Members are `JetValue`s: constants (`JInt`, `JFloat`, `JString`, `JBool`, `JList`) and functions (`JBuiltin { args -> ... }`, wrapping a `suspend (List<JetValue>) -> JetValue`).
- `fields: Map<String, JetType>` — the public interface for the static checker. Constants map to their value type; functions map to `callable(returnType, ...signatures)`.

```kotlin
val value = JModule(mutableMapOf(
    "VERSION" to JInt(1),
    "greet" to JBuiltin { args -> JString("Hello, ${(args[0] as JString).value}!") },
))
val fields = mapOf(
    "VERSION" to JetType.TInt,
    "greet" to callable(JetType.TString, signature(JetType.TString)),
)
jetpack.registerModule(this, "example", value, fields)
```

Wrap the module in a class (constructor injection) when it needs collaborators; `StorageModule` is the reference pattern. `MathModule` is the reference for constants plus overloaded functions.

Scripts then use it:

```jetpack
using example
command hello(object sender) { sender.sendMessage(example.greet("Alex")) }
```

> Note: the `ModuleSpec`-based overload (`registerModule(owner, spec)`) is private to `JetpackPlugin`. Built-in modules use `ModuleSpec`/`spec()` internally, but a host plugin must call the four-argument overload directly with `name`/`value`/`fields`/`dynamic`.

## Static validation

For `dynamic = false`, `value` and `fields` must describe the same member set, or registration throws `IllegalArgumentException`:

- `fields` entry with no matching `value` member → missing member.
- `value` member not in `fields` → undeclared member.
- runtime value not satisfying the declared type → type mismatch (callable members must be `JBuiltin`).

The module name must not collide with another registered module or a built-in global name.

`dynamic = true` skips field validation; the checker types the import as `unknown` and resolves members at runtime (how `bukkit` exposes the whole Bukkit API). Prefer static modules when the member set is known, for load-time type checking in scripts.

## Builtins

Implement `Builtin` to add import-free globals or methods:

```kotlin
interface Builtin {
    fun resolveGlobal(name: String): JetFn?            // implementation
    fun resolveMethod(target: JetValue, method: String): JetFn?
    fun globalType(name: String): JetType? = null      // type metadata
    fun methodType(targetType: JetType, method: String): JetType? = null
    fun globalNames(): Set<String> = emptySet()
}
```

`JetFn` is `suspend (List<JetValue>) -> JetValue`. For each name in `globalNames`: `globalType` must return a callable and `resolveGlobal` must return an implementation, or registration throws `IllegalArgumentException`. A global name must not collide with an existing global or a registered module name. For methods, return non-null `methodType`/`resolveMethod` only for the target type and method you handle; return `null` otherwise so other builtins still resolve. Scope method matches tightly — adding a method to a built-in type affects every script.

## Types and signatures

From `dev.jetpack.engine.parser.ast`:

- `signature(vararg paramTypes, requiredCount = paramTypes.size, variadicType = null, returnType = null)` — one parameter list. Lower `requiredCount` for optional trailing params; set `variadicType` for variadic tails.
- `callable(returnType, vararg signatures)` — combines signatures; a signature without its own `returnType` inherits the callable's.

Type/value pairs: `TInt`/`JInt`, `TFloat`/`JFloat` (accepts `int`), `TString`/`JString`, `TBool`/`JBool`, `TList(e)`/`JList`, `TObject`/`JObject`, `callable(...)`/`JBuiltin`, `TNullable(t)`/value-or-`JNull`, `TUnknown`/any. When a member has a callable type with signatures, Jetpack validates argument types against the signatures before the call and the return value after (unless return is `TUnknown`), raising `RuntimeException` on mismatch — catchable in scripts as `RuntimeException`. A callable with no signatures skips argument validation.

## Lifecycle

- **Main thread**: `registerModule`, `batchUpdate`, and `RegistrationHandle.unregister` assert `server.isPrimaryThread` and throw `IllegalArgumentException` off-thread. Reschedule async-triggered registration onto the main thread.
- **Batching**: each registration reloads scripts by default; `batchUpdate` collapses a group into one reload and nests safely (reload on outermost exit).
- **Handles**: both overloads return `RegistrationHandle`; `unregister()` removes the extension and reloads, returning `true` if it removed something.
- **Automatic reload**: registering/unregistering re-parses and re-validates every script. Scripts importing a now-missing module, or incompatible with a changed `fields`, fail validation until fixed. Reload runs after initial server load; `onEnable` registrations are simply in place at first load.
- **Automatic cleanup**: on the host plugin's `PluginDisableEvent`, everything it registered is removed and scripts reload. No `onDisable` unregistration needed.

Normal pattern: obtain `JetpackPlugin` in `onEnable`, register once inside a single `batchUpdate`. Reserve `unregister` for genuinely dynamic extensions.
