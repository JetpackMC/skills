---
name: jetpack
description: Authoritative reference for the Jetpack scripting language .jet files used by the Jetpack Paper/Minecraft plugin. Use when reading, writing, reviewing, or debugging .jet scripts, or when answering questions about Jetpack syntax, types, modules, commands, listeners, intervals, imports, or the Bukkit native bridge. Also covers host-plugin extensions including how a separate Paper plugin Java/Kotlin registers its own modules and global functions into the Jetpack runtime.
---

# Jetpack Language

Jetpack is a small, statically-checked scripting language that runs inside a Paper (Minecraft) server plugin, also called Jetpack. Its purpose is to let server operators add custom server behavior by writing short script files instead of building and compiling a Java plugin. A script reacts to player commands, responds to in-game events, runs work on a timer, talks to the Bukkit/Paper server API, and uses a small standard library for math, JSON, randomness, persistent storage, time, regular expressions, and HTTP.

Scripts use the `.jet` extension and live in `plugins/Jetpack/scripts/`. Each `.jet` file is a module: a self-contained unit that can declare commands, listeners, intervals, functions, and variables, and can import other modules. When the server loads, the plugin discovers every script, runs each one through a fixed pipeline, and wires its declarations into the running server.

The pipeline is: the lexer turns source text into tokens; the parser builds an abstract syntax tree; the name resolver checks that every identifier is defined and that declarations sit where the language allows; the type checker verifies that every operation is type-correct; and finally a tree-walking interpreter executes the module. The first four phases are static, meaning they run before any code executes. A script that fails any static phase is rejected and does not run, so most mistakes surface at load time rather than during play.

This skill is the language specification. It reflects the interpreter's actual behavior, which is the source of truth. Where the published documentation disagrees with the interpreter, follow the interpreter.

## How to use this skill

Load the reference file for the construct you are working with. Files are grouped under `reference/` by domain. Each file specifies grammar, semantics, constraints, and failure modes.

`reference/language/` — the core language:

- [lexical-structure.md](reference/language/lexical-structure.md): tokens, keywords, comments, literals, identifiers
- [types-and-values.md](reference/language/types-and-values.md): type system, `var`, `null` and nullability, coercion, equality, truthiness
- [variables-and-scope.md](reference/language/variables-and-scope.md): declarations, `const`, access modifiers, deconstruction, scoping
- [operators.md](reference/language/operators.md): arithmetic, comparison, logical, assignment, range, `in`, precedence
- [strings.md](reference/language/strings.md): string literals, escapes, interpolation, string methods
- [enums.md](reference/language/enums.md): file-scope enum declarations and enum member values
- [collections.md](reference/language/collections.md): list and object literals, indexing, iteration, methods
- [control-flow.md](reference/language/control-flow.md): `if`/`else`, `while`, `foreach`, `break`/`continue`, ternary
- [functions.md](reference/language/functions.md): function declarations, parameters, defaults, return, type rules
- [error-handling.md](reference/language/error-handling.md): `try`/`catch`/`finally`, exception hierarchy, exception object
- [concurrency.md](reference/language/concurrency.md): `thread` targets, semantics, restrictions

`reference/scripts/` — file-scope runtime constructs:

- [script-structure.md](reference/scripts/script-structure.md): file model, `manifest`, `using` imports, access/export visibility, load order
- [commands.md](reference/scripts/commands.md): `command`, sender, parameters, subcommands, `default`, annotations, runtime methods
- [listeners-and-events.md](reference/scripts/listeners-and-events.md): `listener`, event names, event object, cancellation, annotations, runtime methods
- [intervals.md](reference/scripts/intervals.md): `interval`, tick conversion, runtime methods

`reference/modules/` — libraries:

- [standard-modules.md](reference/modules/standard-modules.md): `math`, `json`, `random`, `storage`, `time`, `regex`, `http`
- [bukkit-and-native.md](reference/modules/bukkit-and-native.md): `bukkit` module, native reflection, properties, enums, construction, conversion

`reference/extending/` — host-plugin extensions (for Java/Kotlin plugin developers, not `.jet` authors):

- [host-plugin-extensions.md](reference/extending/host-plugin-extensions.md): registering modules and builtins from a Paper plugin through `JetpackPlugin`, type/signature construction, validation, and the registration lifecycle

Runnable programs are in [examples/](examples/). Examples demonstrate idioms and are not part of the specification.

## Rules that prevent silent breakage

These are the points most often gotten wrong. Each is covered in detail in its reference file.

1. Output reaches players or the console through a sender (`sender.sendMessage(...)`), an event entity (`event.player.sendMessage(...)`), or the Bukkit native API under `bukkit.*`. The one global function is `typeof`. See [reference/modules/bukkit-and-native.md](reference/modules/bukkit-and-native.md).
2. Object literal keys are quoted strings: `{ "name": "Alex" }`. `manifest` keys are the exception and are written as bare identifiers.
3. `catch` types use Jetpack exception names: `Exception`, `RuntimeException`, and the `RuntimeException` subtypes `TypeException`, `NameException`, `IndexException`, `KeyException`, `ArgumentException`, `ArithmeticException`, `StateException`, `PermissionException`, `NativeException`, `ModuleException`. See [reference/language/error-handling.md](reference/language/error-handling.md).
4. Every variable declaration includes an initializer.
5. Function parameters carry explicit type annotations.
6. `function`, `command`, `listener`, `interval`, `enum`, `manifest`, and access modifiers appear only at file scope, never inside a block.
7. The checker types `**` as `float`. Annotate the target `float` or `var` to hold a power result.
8. List and object transformation methods return new values. Reassign to keep the result.

## Execution model

- A `.jet` file is a module. Top-level `function`/`interval`/`listener`/`command` declarations are hoisted, then top-level statements execute top to bottom once at load time.
- `command`, `listener`, and `interval` are entry points wired to the server. Reusable logic belongs in `function`s called from entry points.
- The interpreter runs on Kotlin coroutines. `command`, `listener`, and `interval` bodies run on the main server thread dispatcher. `thread` and some module calls move work off the main thread.
- Static checking is per-module after import resolution. A type or name error in a module prevents that module from loading.
