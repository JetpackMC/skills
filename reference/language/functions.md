# Functions

Functions group reusable logic behind a name, typed parameters, and an optional return type.

## Declaration

```text
[access] function name(params): returnType { body }
```

- `function` declarations appear only at file scope.
- `returnType` is optional. Omit it for a function that produces no meaningful value.
- An access modifier controls export visibility ([script-structure.md](../scripts/script-structure.md)); the default is `private`.
- Declarations are hoisted within their module, so a function may be called before its textual position, and functions may be mutually recursive.

## Parameters

Each parameter carries an explicit type annotation:

```text
type name
type name = defaultExpr
```

- A type annotation on every parameter is required; an untyped parameter is a name-resolution error.
- A parameter type is one of the declarable types, including `var`, `object`, and `list<T>`.
- A default makes the parameter optional. The required count is the number of parameters without a default. A call supplies between the required count and the total count of arguments.
- A default expression is evaluated at call time, in the function's defining scope, each time the default is used.

## Return

- `return expr` exits with a value; `return` alone exits with `null`.
- When a return type is declared (and is not `var`), every path through the body must return a value, and each returned value must match the declared type. Flow analysis treats `while`/`foreach` as non-terminating for this purpose, so a function that returns only inside a loop still needs a terminating `return` after it.
- The return value is coerced to the declared return type (numeric coercion applies; see [types-and-values.md](types-and-values.md)).

## Calls and runtime checks

- Argument count is checked against the required/total range; a mismatch is `ArgumentException`.
- Each argument is coerced to its parameter type, then checked; a value that the parameter type does not accept is `ArgumentException`.
- A missing required argument with no usable default is `ArgumentException`.
- Calling a value that is not callable is `TypeException`.

Functions are the unit of reuse: commands, listeners, and intervals are entry points that call functions, and exported functions are how one script offers behavior to another.
