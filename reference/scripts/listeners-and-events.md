# Listeners and Events

`listener` registers a handler for a server (Bukkit/Paper) event. Listeners are file-scope entry points.

## Form

```text
[annotations]
[access] listener EventName name(object event) {
  body
}
```

- `EventName` must exactly match a known event name (case-sensitive); an unknown name fails name resolution.
- `name` is the listener's identifier in the module.
- The parameter is optional. When present it is written `object event` and binds the event object, conventionally named `event`.

## Event object

The event object is the reflected native event, exposed as a read-only object. Its fields and methods come from the native event class (see [bukkit-and-native.md](../modules/bukkit-and-native.md)), and native enum values on it are exposed as their string names. For a cancellable event, a `cancel()` method is injected on the event object; calling it cancels the event.

## Annotations

On the lines immediately before the `listener`:

```text
@priority "VALUE"          event priority
@ignoreCancelled "true"    when "true", skip events already cancelled by earlier handlers
```

Priority values, earliest to latest: `LOWEST`, `LOW`, `NORMAL` (default), `HIGH`, `HIGHEST`, `MONITOR` (observation only; do not modify the event at this priority). An unknown value fails name resolution.

## Runtime methods

The listener declaration produces a listener value:

```text
activate() -> bool        activates; true if the state changed
deactivate() -> bool      deactivates; true if the state changed
destroy() -> bool         permanently removes the listener for this runtime
trigger(sender?) -> null  invokes the body from script code; null/absent value uses the console sender
isActive() -> bool
```
