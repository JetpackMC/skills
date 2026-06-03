# Operators

Operators combine values into expressions. Every operator is statically typed: the type checker determines the result type and rejects operand types an operator does not accept, so an invalid combination is reported before the script runs. Most arithmetic and comparison operators require numeric operands, while a few are overloaded to also work on strings and lists.

## Precedence

From lowest to highest binding:

1. Assignment: `=` `+=` `-=` `*=` `/=` `%=` `**=` (statement-level only; see below)
2. Ternary: `? :`
3. Logical OR: `||`
4. Logical AND: `&&`
5. Equality: `==` `!=`
6. Comparison: `<` `<=` `>` `>=` and `in`
7. Range: `to` `until`
8. Additive: `+` `-`
9. Multiplicative: `*` `/` `%`
10. Power: `**` (right-associative)
11. Unary prefix: `!` `-` `++` `--`
12. Postfix: `.member` `[index]` `call(...)` `++` `--`

## Arithmetic `+ - * / % **`

Operands must be numeric (or `var`). Result type:

- `int op int` is `int` for `+ - * / %`.
- Any `float` operand makes the result `float`.
- `**` is always typed `float` by the checker, even for two `int` operands. To hold `2 ** 3` in an `int` you must use a `var` or `float` target.

Runtime numeric behavior:

- `int + - *` use wrapping 32-bit arithmetic; overflow wraps silently without throwing.
- `int / int` and `int % int` truncate toward zero; a zero divisor throws `ArithmeticException`.
- `float` division/modulo by zero throws `ArithmeticException`.
- `**` with two ints and a non-negative exponent yields an `int` and throws `ArithmeticException` on overflow; a negative integer exponent yields a `float`; any float operand yields a `float`.

`-x` (unary minus) negates a numeric value.

## String and list operators

- `string + string` concatenates.
- `string * int` and `int * string` repeat the string. A negative count throws `ArgumentException`.
- `list + list` concatenates, producing a new list. Element types must be compatible; incompatible element types are an error (checker) and `TypeException` (runtime).

`+=` and `*=` follow the same rules for their target types (string concat, string repeat, list concat, or numeric).

## Comparison `< <= > >=`

Numeric operands only (or `var`). Result is `bool`. Nullable operands are rejected.

## Equality `== !=`

Allowed on any operands; result `bool`. See value-equality rules in [types-and-values.md](types-and-values.md). `true == 1` is `true`; `1 == 1.0` is `true`.

## Logical `&& ||`

Short-circuit. Operands are evaluated as conditions (a `null` operand is an error). Result is `bool`. The right operand benefits from null-narrowing established by the left operand.

## `in`

`left in right`:

- `right` is a list: membership test using value equality.
- `right` is an object: `left` must be a string; tests key presence.
- `right` is a string: `left` must be a string; substring test.

Other right-hand types are an error. Result is `bool`.

## Range `to` / `until`

Produces a `list<int>`.

- `a to b` is inclusive of `b`.
- `a until b` is exclusive of `b`.
- `a` and `b` must be `int` (non-nullable). Descending ranges are allowed when `a > b`.
- Ranges cannot be chained (`a to b to c` is an error).
- The resulting list size cannot exceed the `int` maximum; exceeding it throws `ArithmeticException`.

## Increment / decrement `++ --`

Prefix and postfix. The target must be an assignable numeric (`int`/`float`) variable, field, or index. Prefix returns the new value; postfix returns the old value. Not allowed on `const`, on read-only items, or on a threaded result.

## Assignment

`=` and the compound forms are statements/expressions, not part of general expression precedence. The target must be an identifier, a member access (`a.b`), or an index access (`a[i]`). Assigning to anything else, or to a threaded result, is an error. Compound assignment cannot be applied to a nullable target.
