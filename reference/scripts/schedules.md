# Schedules

`schedule` registers a cron-based file-scope entry point. It runs on the main server thread dispatcher when its cron expression matches.

## Form

```text
[access] schedule name("minute hour day-of-month month day-of-week") { body }
```

The cron expression must be a string literal with five fields. Supported field syntax:

- `*` for every value
- comma lists such as `1,2,3`
- ranges such as `9-17`
- steps such as `*/5` or `9-17/2`
- month names `JAN` through `DEC`
- day names `SUN` through `SAT`

Day-of-week accepts `0` or `7` for Sunday. If both day-of-month and day-of-week are restricted, either field may match.

```jet
schedule hourly("0 * * * *") {
  // runs at the start of every hour
}
```

## Runtime methods

The schedule declaration produces a schedule value:

```text
activate() -> bool      activates future cron executions; true if the state changed
deactivate() -> bool    pauses future cron executions; true if the state changed
destroy() -> bool       permanently removes the schedule for this runtime
trigger() -> null       runs the body once from script code
isActive() -> bool      whether the schedule is active
cron() -> string        the original cron expression
nextRun() -> object|null  next scheduled time object, or null if inactive
```

Schedules are active after registration. Use `thread` inside the body for waits, HTTP requests, or other long-running work.
