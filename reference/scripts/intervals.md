# Intervals

`interval` runs a body repeatedly on a timer. Intervals are file-scope entry points.

## Form

```text
[access] interval name(ms) { body }
```

`ms` is a positive integer literal (the period in milliseconds). A non-positive or non-literal period is a parse error.

The period is converted to server ticks as `((ms + 25) / 50)` with a minimum of 1 tick (one tick is 50 ms), so very small periods run once per tick. The body runs on the main server thread dispatcher.

## Runtime methods

The interval declaration produces an interval value:

```text
activate() -> bool      activates the interval; true if the state changed
deactivate() -> bool    deactivates the interval; true if the state changed
destroy() -> bool       permanently removes the interval; true if destroyed
trigger() -> null       runs the body once from script code
isActive() -> bool      whether the interval is currently active
```
