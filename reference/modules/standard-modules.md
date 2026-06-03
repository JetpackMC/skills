# Standard Modules

The standard library is a set of built-in modules that ship with the plugin and give scripts capabilities beyond the core language: numeric computation, JSON handling, randomness, persistent storage, time and date work, regular expressions, and HTTP requests. Each module is a namespace of related functions and constants. A script gains access to a module by importing it, after which its members are reached through the module name.

Import a module with `using name`, then access members as `name.member`. Module names are case-sensitive. The `bukkit` module, which exposes the Minecraft server API rather than a fixed function set, is covered in [bukkit-and-native.md](bukkit-and-native.md).

Signatures below read `member(params) -> returnType`. Numeric parameters accept `int` or `float` unless stated. A module member referenced without a call where a call is required is an error; a missing member is `ModuleException`.

## math

The `math` module provides numeric helpers that operate on `int` and `float` values: comparison and clamping, rounding, square and cube roots, powers, trigonometry, and logarithms. It also exposes common mathematical constants. It performs pure calculation and cannot read or change any game or server state.

Constants (`float`): `PI E LN2 LN10 LOG2E LOG10E SQRT1_2 SQRT2`.

```text
math.min(a, b) -> int if both int else float
math.max(a, b) -> int if both int else float
math.clamp(value, lo, hi) -> int if all int else float
math.abs(int) -> int
math.abs(float) -> float
math.round(x) -> int            rounds to nearest integer
math.round(x, digits) -> float  rounds to that many decimals (digits 0 yields int)
math.ceil(x) -> int
math.floor(x) -> int
math.sqrt(x) -> float
math.cbrt(x) -> float
math.exp(x) -> float
math.log(x) -> float            natural log; x > 0
math.log2(x) -> float           x > 0
math.log10(x) -> float          x > 0
math.sin(x) -> float
math.cos(x) -> float
math.tan(x) -> float
math.asin(x) -> float
math.acos(x) -> float
math.atan(x) -> float
math.atan2(y, x) -> float
math.hypot(a, b) -> float
```

`log`, `log2`, and `log10` cannot take a non-positive argument; doing so raises `RuntimeException`.

## json

The `json` module converts between JSON text and Jetpack values, which is the usual way to exchange structured data with HTTP services, configuration text, and stored records. `parse` reads text into values, `stringify` writes a value back to text, and `valid` reports whether text is parseable without raising.

```text
json.parse(text) -> value      parses JSON into int/float/string/bool/null/list/object
json.stringify(value) -> string
json.valid(text) -> bool
```

`parse` reads numbers containing a `.`, `e`, or `E` as `float`, otherwise `int`. `stringify` works only on JSON-compatible values, so it cannot serialize a function, module, interval, listener, or command, cannot serialize a structure that contains a cycle, and requires objects to be fully materialized rather than backed by native fields.

## random

The `random` module produces random numbers and makes random choices from a list. Use it for chance-based behavior such as rewards, sampling, or shuffling.

```text
random.decimal() -> float            in [0, 1)
random.decimal(min, max) -> float    in [min, max); min must not exceed max
random.integer(min, max) -> int      inclusive of both ends; min must not exceed max
random.select(list) -> element       errors on an empty list
random.shuffle(list) -> list         new shuffled list
```

`random.select` cannot choose from an empty list, and the range functions cannot take a `min` greater than `max`; both raise `RuntimeException`.

## storage

The `storage` module is a persistent, file-backed key-value store that survives restarts. Data is grouped into named stores, each of which holds string keys mapped to values. A store id is a path-like string (no leading `/`, no `..`, no drive part) and maps to a `.db` file on disk. Stored values are JSON-compatible (`null`, `bool`, `int`, `float`, `string`, `list`, `object`).

A store must be created before it can hold data. Operations against a store that has not been created return `false` or `null` rather than raising, so check the result or create the store first.

```text
storage.create(store) -> bool        false if it already exists
storage.destroy(store) -> bool
storage.exists(store) -> bool
storage.has(store, key) -> bool
storage.keys(store) -> list<string>  sorted; empty if the store is missing
storage.get(store) -> object         all entries, or null if the store is missing
storage.get(store, key) -> value     value, or null if absent
storage.set(store, key, value) -> bool   false if the store is missing
storage.remove(store, key) -> bool   true if a value was removed
storage.clear(store) -> bool         empties the store
```

`storage` cannot persist a value that is not JSON-compatible, such as a function or a runtime handle.

## time

The `time` module reads the current date and time, formats and parses date-time text, measures the difference between two moments, and pauses execution. Time values are read-only objects: `now` and `parse` return `{year, month, day, hour, minute, second, millisecond}`, and `diff` returns `{days, hours, minutes, seconds, milliseconds}` as an absolute difference.

```text
time.now() -> object
time.format(timeObj, pattern) -> string   pattern is a date-time pattern
time.parse(text, pattern) -> object
time.diff(a, b) -> object
time.delay(ms) -> null                     suspends for ms (>= 0)
```

`time.delay` suspends the current flow until the wait elapses, so it should run under `thread` or inside an entry point body where holding the flow is acceptable; placing it directly at module top level would hold up loading.

## regex

The `regex` module matches, extracts, replaces, and splits text using standard regular expressions. Use it when the work depends on a pattern rather than a fixed substring. Replacement text is treated literally, so regex metacharacters in a replacement have no special meaning.

```text
regex.test(text, pattern) -> bool
regex.find(text, pattern) -> string or null     first match
regex.findAll(text, pattern) -> list<string>
regex.groups(text, pattern) -> list             capture groups of the first match; null per unmatched group
regex.replace(text, pattern, repl) -> string    replaces the first match
regex.replaceAll(text, pattern, repl) -> string
regex.split(text, pattern) -> list<string>
regex.escape(text) -> string                    quotes regex metacharacters
```

An invalid pattern raises `RuntimeException`. Because `find` may return `null`, its result must be checked before being used as a `string`.

## http

The `http` module sends HTTP requests to external services and returns the response. Use it for webhooks, web APIs, and remote data. Each call returns an object `{status: int, body: string, ok: bool}`, where `ok` is true when `status` is in 200..299. A request body is given as an object and is serialized to JSON, with `Content-Type: application/json` set automatically; `headers` is an object whose string values become request headers.

```text
http.get(url, headers?) -> object
http.post(url, body, headers?) -> object
http.put(url, body, headers?) -> object
http.delete(url, headers?) -> object
```

A network failure raises `RuntimeException`. The request runs off the main thread internally, but the call still suspends the current flow until the response arrives, so wrap it in `thread` when the caller should keep working meanwhile. The module cannot send a request body that is not an object.
