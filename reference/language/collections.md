# Collections: list and object

## list

### Literals and element type

A list literal is `[a, b, c]`. The static element type is inferred; a literal whose elements share no common type is an error. All-`int` elements infer `list<int>`; mixed numeric infer `list<float>`. Declare a concrete `list<T>` for API surfaces.

### Indexing

- `xs[i]` reads the element at zero-based index `i`. Non-`int` index is `TypeException`; out of range is `IndexException`.
- `xs[i] = v` assigns. The value must stay element-compatible (`TypeException` otherwise). Assigning into a read-only list (a `protected` import or a range result) is `PermissionException`.

### List methods

Transforming methods return a new list and do not mutate the receiver; reassign to keep the result.

```text
length() -> int                element count
contains(v) -> bool            value-equality membership
append(v) -> list<T>           new list with v added; v must be element-compatible
remove(i) -> list<T>           new list without index i; out of range errors
ascend() -> list<T>            new list sorted ascending; elements must be all int/float/string
descend() -> list<T>           new list sorted descending; same element constraint
reverse() -> list<T>           new reversed list
slice(from, to) -> list<T>     new list for [from, to); out of range or from > to errors
first() -> T                   first element; empty list errors
last() -> T                    last element; empty list errors
indexOf(v) -> int              first index, or -1
indexOf(v, from) -> int        first index at or after from, or -1
lastIndexOf(v) -> int          last index, or -1
join(sep) -> string            joins element renderings with sep
```

`ascend`/`descend` on a list whose static element type is not `int`/`float`/`string` is rejected by the checker.

### Iteration and concatenation

- `foreach (item in xs)` binds each element to a read-only `item`.
- `var (a, b) = xs` and related forms bind positions; see [variables-and-scope.md](variables-and-scope.md).
- `xs + ys` concatenates; element types must be compatible.

## object

`object` is a string-keyed map with a dynamic field set, used for records, JSON-like data, and native results.

### Literals

```text
{ "key": value, "key2": value2 }
```

Keys are string literals; unquoted keys are a parse error. Values are any expression.

### Field access

- `obj.name` and `obj["name"]` read a field; the bracket form is needed for a computed key. A missing field is `KeyException`; a non-string bracket key is `TypeException`.
- `obj.name = v` and `obj["name"] = v` write. Writing to a read-only object (a `protected` import or a reflected native/event object) raises a runtime error.

### Object methods

```text
keys() -> list<string>         the object's keys
has(key) -> bool               whether key exists
length() -> int                number of fields
get(key) -> value              field value, or null if absent
set(key, value) -> bool        writes a field, returns true; fails on a read-only object
remove(key) -> bool            removes a key, returns whether something was removed; fails on a non-mutable object
append(other) -> object        new object merging both; keys in other win
```

### Iteration and membership

- `foreach (entry in obj)` yields read-only `{ "key": ..., "value": ... }` objects, one per field.
- `"key" in obj` tests key presence (left operand must be a string).

Objects do not support deconstruction. Read named fields explicitly, and validate keys from JSON, storage, or native APIs before use.
