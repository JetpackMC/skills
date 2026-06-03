# Control Flow

Control flow decides which statements run and how often. Jetpack provides conditional branching with `if`/`else`, two loops (`while` and `foreach`), loop control with `break` and `continue`, and a ternary expression for choosing between two values. A branch or loop condition is read by truthiness and cannot be `null`.

## Block form

A block body is either `{ ... }` or a single statement without braces. A control-flow header followed by one statement needs no braces; use braces for multiple statements. Each block is a child scope.

## if / else if / else

```
if (condition) <block>
else if (condition) <block>
else <block>
```

- The condition is an expression used by truthiness; it cannot be `null` (compile-time error, and `TypeException` at runtime if it occurs dynamically).
- Any number of `else if` clauses; an optional final `else`.
- `if`/`else` conditions narrow nullable variables for the matching branch (see [types-and-values.md](types-and-values.md)).

## while

```
while (condition) <block>
```

Repeats while the condition is truthy. The condition cannot be `null`.

## foreach

```
foreach (item in iterable) <block>
foreach (<type> item in iterable) <block>
```

- `iterable` may be a `list`, a `string`, or an `object`.
  - list: `item` is each element.
  - string: `item` is each character as a single-character string.
  - object: `item` is a read-only `{ "key": ..., "value": ... }` object per field.
- `item` is read-only; reassigning it is an error.
- An optional element type annotation is checked against the iterable's element type.
- Iterating a non-iterable value is `TypeException`.

## break / continue

`break` exits the nearest enclosing loop; `continue` skips to the next iteration. Both are valid only inside a `while` or `foreach`; using them elsewhere is an error. They cannot cross a `thread` boundary.

## return

`return [expr]` exits the enclosing function. It is valid only inside a function body (a `command`/`listener`/`interval` body counts as a function-like body for the purpose of allowing `return`, which there simply ends the body). It cannot cross a `thread` boundary. Return-type rules are in [functions.md](functions.md).

## Ternary expression

```
condition ? thenExpr : elseExpr
```

An expression, not a statement. The condition cannot be `null`. The result type is the common type of the two branches.
