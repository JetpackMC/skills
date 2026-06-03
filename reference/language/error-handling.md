# Error Handling

`try`/`catch`/`finally` handles runtime errors. Use it where the script can recover, add context, convert an error into a user-facing message, or guarantee cleanup.

## Form

```text
try { ... }
catch (ExceptionType variable) { ... }
catch (ExceptionType) { ... }
catch (variable) { ... }
catch { ... }
finally { ... }
```

- A `try` requires at least one `catch` or a `finally`.
- A catch may declare a typed exception with or without a bound variable.
- A parenthesized catch with a single identifier and no type is a fallback catch that binds the error to that name (the identifier is the variable, not a type).
- A catch with no parentheses is a fallback catch. A fallback catch must be last; any catch after it is rejected.
- The caught exception variable is read-only.

## Matching

The first catch whose type matches runs. Matching walks the thrown type up its supertype chain, so `catch (RuntimeException)` matches any `RuntimeException` subtype. Order specific types before broader ones.

## Exception hierarchy

`Exception` is the root. `RuntimeException` extends `Exception`. These extend `RuntimeException`: `TypeException`, `NameException`, `IndexException`, `KeyException`, `ArgumentException`, `ArithmeticException`, `StateException`, `PermissionException`, `NativeException`, `ModuleException`. A catch type outside this set is a type-check error.

## Which operation raises which type

- `ArithmeticException`: division/modulo by zero; `**` integer overflow; range size overflow
- `IndexException`: list index out of range; deconstruction index out of range
- `KeyException`: reading a missing object field/key; failed object member assignment
- `TypeException`: non-numeric operand where numeric required; non-int index; non-string object key; null condition; iterating a non-iterable; calling a method on an unsupported type
- `NameException`: reading an undefined identifier
- `PermissionException`: assigning to a `const`, a read-only symbol, or a read-only list
- `ArgumentException`: wrong argument count; missing required parameter; command argument conversion failure; negative string repetition
- `StateException`: control flow escaping a `thread`; nested `thread` execution
- `NativeException`: a Bukkit/native call or member access failing
- `ModuleException`: a missing module member at runtime
- `RuntimeException`: builtin method failures such as numeric-parse failure, out-of-range `substring`/`slice`, empty delimiter, math domain errors, HTTP failures, JSON/storage serialization failures

Failures inside standard module and builtin method implementations are generally `RuntimeException`. Catch `RuntimeException` to cover those, or `Exception` for everything.

## Exception object

A bound catch variable is a read-only object with: `type` (string, the exception type name), `message` (string), `line` (int, the source line where it was raised).

Avoid sending raw exception messages to players; convert them to short user-facing text and keep detail where administrators can see it.

## finally

`finally` runs after `try` and any `catch`, including when an exception propagates. Control flow inside `finally` (`return`, `break`, `continue`) can replace the original outcome; keep it to simple cleanup.
