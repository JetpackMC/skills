# Types and Values

Jetpack is statically type-checked. Every expression has a static type; the checker rejects incompatible operations before the script runs. At runtime values carry their own type.

## Type set

Declarable types:

- `int`: 32-bit signed integer
- `float`: 64-bit double
- `string`: text, immutable
- `bool`: `true` / `false`
- `object`: string-keyed map with a dynamic field set
- `list<T>`: ordered sequence of `T`; element type is tracked
- `var`: inferred from the initializer; internally the "unknown" type
- `null`: the single null value

These value kinds can be held in variables but cannot be used as a declared type annotation: `function`, `interval`, `schedule`, `listener`, `command`, `module`. `typeof` reports `"function"` for both script functions and builtins.

## var and inference

`var` takes its type from the initializer. Internally it is the "unknown" type: assignable to and from anything, and it suppresses most type errors on that value. Use concrete types for exports, parameters, and return types; use `var` for locals whose type is obvious or genuinely dynamic (JSON, storage, native results).

## Nullability

Nullability is a property the type checker infers from how a value is used rather than something written in a declaration.

- A variable initialized with `null` has an inferred nullable type.
- The checker rejects most operations on a nullable value (member access, indexing, arithmetic, comparison, method calls) until it is narrowed.
- Narrowing happens through `x != null` / `x == null` tests in `if` conditions and `&&` / `||` chains. Inside the branch where the value is known non-null, it is usable.

A condition cannot be `null`: a `null`-typed expression as an `if`/`while`/ternary condition or a `&&`/`||` operand is a static error, and `TypeException` if it occurs dynamically.

## Numeric coercion on assignment

When a value is stored into a target with a declared numeric type:

- `float` into an `int` target truncates toward zero, silently.
- `int` into a `float` target widens.
- Assigning into `list<int>` or `list<float>` coerces each element.

A value assigned into a `var` target is stored unchanged.

## Equality (`==`, `!=`)

Value-based, always allowed by the checker, always yields `bool`:

- `int`/`float` compare numerically across kinds: `1 == 1.0` is true.
- `string` by content; `bool` by value.
- `list` element-by-element, recursively (equal length, equal elements).
- `bool` versus a number compares the bool against the number's truthiness: `true == 1` is true, `false == 0` is true.
- `null == null` is true; `null` versus any non-null is false.
- Anything else is unequal.

## Truthiness

Used by conditions and `&&`/`||`. A `null` condition is an error. For non-null values:

- `bool`: its value
- `int`: `!= 0`
- `float`: `!= 0.0`
- `string`: non-empty
- `list`, `object`, other reference kinds: always truthy

## Checker notes

- A list literal's element type is inferred; elements with no common type are an error. All-`int` elements infer `list<int>`; mixed numeric infer `list<float>`.
- Reading a member/method on a type that has none is an error, except on `object` and `var`, which allow dynamic access.
- A builtin method must be called: referencing `s.length` instead of `s.length()` is an error.

See [operators.md](operators.md) for per-operator typing, [collections.md](collections.md) for lists/objects, and [error-handling.md](error-handling.md) for the runtime exception each failure produces.
