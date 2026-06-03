# Concurrency: thread

`thread` moves work onto a background thread. It is for waits, delayed work, HTTP calls, and other blocking or long-running operations that should not occupy the main server thread.

## Targets

`thread` applies to exactly one of:

- A function call: `thread name(args)`
- A method call: `thread target.method(args)`
- An `if`, `while`, or `foreach` statement: `thread if (...) { ... }`, `thread while (...) { ... }`, `thread foreach (...) { ... }`

`thread` cannot wrap an arbitrary statement block. To run several statements together on a background thread, place them in a function and `thread` the call.

## Semantics

`thread` offloads its target to a background thread and suspends the current script flow until the target completes; the statement after a `thread` runs only after the threaded work finishes. While suspended, the main server thread is free to do other work, so the server is not blocked during the wait. A threaded call yields the call's result value; a threaded control-flow statement yields `null`.

Arguments to a threaded call are evaluated in the current scope before the work is dispatched. The body then runs on the background thread.

## Restrictions

- A `thread` cannot be directly nested inside another `thread`. Nested threaded execution at runtime is `StateException`.
- A threaded result cannot be an assignment target, and a threaded result target cannot be mutated.
- `return`, `break`, and `continue` cannot escape a threaded block to an outer function or loop; doing so is `StateException`. Keep threaded code self-contained and report results through state updates or messages.

Use `thread` for waits and blocking I/O such as `time.delay` and `http` requests. Avoid using it to mutate shared state from many places at once, because asynchronous ordering makes that hard to reason about.
