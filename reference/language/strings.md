# Strings

`string` values are immutable. Every transforming method returns a new string.

## Literals

Delimited by `"` or `'`. Escapes are in [lexical-structure.md](lexical-structure.md). A raw newline inside a literal is an error; use `\n`.

## Interpolation

An interpolated string starts with `$"` and ends with `"`.

- `{ expr }` embeds the value of `expr`. The expression is parsed and type-checked like any other and may contain strings, calls, indexing, and nested braces/parentheses/brackets, and may span multiple lines.
- `{{` and `}}` produce literal `{` and `}`. A lone `}` outside an interpolation is an error.
- The embedded value uses standard string rendering (same as concatenation/`toString`): `object` and `list` render in a structured JSON-like form; `float` renders as plain decimal.
- Recognized escapes in the literal portion are `\n` `\t` `\r` `\\` `\"` (not `\'`).

Use interpolation when a message embeds several values; `+` concatenation is fine for one or two parts.

## Methods

Called with parentheses. Indices are zero-based.

```text
length() -> int                number of characters
contains(s) -> bool            whether s occurs
replace(old, new) -> string    replaces every literal occurrence
lower() -> string
upper() -> string
trim() -> string               removes leading/trailing whitespace
substring(from, to) -> string  substring [from, to); out of range or from > to errors
split(sep) -> list<string>     splits on literal sep; empty sep errors
indexOf(s) -> int              first index, or -1
indexOf(s, from) -> int        first index at or after from, or -1
lastIndexOf(s) -> int          last index, or -1
count(s) -> int                non-overlapping occurrences; empty s errors
startsWith(s) -> bool
endsWith(s) -> bool
```

Builtin string-method failures (bad index, empty delimiter) surface as `RuntimeException`.

## Indexing and iteration

- `s[i]` returns the single-character string at index `i`. A non-`int` index is `TypeException`; out of range is `IndexException`.
- `foreach (c in s)` iterates characters as single-character strings.

For pattern matching rather than literal substrings, use the `regex` module ([standard-modules.md](../modules/standard-modules.md)).
