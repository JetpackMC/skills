# Variables and Scope

## Declaration

```text
[access] [const] <type|var> name = initializer
```

- The initializer is mandatory; a declaration always assigns an initial value.
- `type` is `int`, `float`, `string`, `bool`, `object`, `list<T>`, or `var`.
- `access` (`public`/`protected`/`private`) is allowed only on top-level declarations.
- The declared type is enforced and applied as coercion on assignment (see [types-and-values.md](types-and-values.md)).

Redeclaring a name in the same scope is an error. A name that collides with a builtin global or an imported module name cannot be declared.

## const

`const` forbids reassignment after initialization; reassigning, compound-assigning, or `++`/`--` on a `const` is an error (`PermissionException` at runtime). It may combine with an access modifier. A `const` integer participates in compile-time constant folding, so it can be used where a constant is required.

## Access modifiers and exports

Top-level variable, function, command, listener, interval, and deconstruction declarations may carry a modifier; the default is `private`. Modifiers cannot appear inside any block.

- `public`: visible to importing scripts, read/write where the declaration is writable.
- `protected`: visible to importing scripts as a read-only view; mutation through the import is blocked.
- `private`: not exported; usable only within the defining script.

`const` and `protected` declarations are read-only to importers. Full import/export behavior is in [script-structure.md](../scripts/script-structure.md).

## Deconstruction

Binds positions of a list to variables. Forms:

```text
var (a, b) = listExpr        declare with inferred types
const (a, b) = listExpr      declare as constants
(int a, string b) = listExpr declare with explicit per-binding types
(a, b) = listExpr            assign to already-declared variables
```

Rules:

- The right-hand side must evaluate to a list; a non-list is `TypeException`.
- Each binding reads its position; a missing position is `IndexException`.
- `_` skips a position without declaring or assigning.
- A `var (...)` pattern cannot specify element types; a typed pattern must type every non-`_` binding.
- The assignment form requires the names to exist already and cannot carry an access modifier.
- Declaration forms may carry an access modifier and export their bound names.

## Scope

Lexical with nested child scopes. Each function, command, listener, interval, loop body, conditional branch, `try`/`catch`/`finally` block, and explicit `{ ... }` block is a child scope.

- Lookup walks outward to the module scope.
- A child scope may shadow an outer name; redeclaration within the same scope cannot.
- `foreach` item variables and `catch` variables are read-only; reassigning them is an error.
- Prefer the smallest scope that works.
