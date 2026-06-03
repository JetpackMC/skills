# Examples

Runnable `.jet` programs that demonstrate idiomatic Jetpack. They illustrate the rules in `../reference/`; they are not the specification. Do not treat an example's incidental style as a rule.

- [types_and_collections.jet](types_and_collections.jet): variables, `var`, lists, objects, deconstruction, `foreach`, string methods, ranges.
- [hello_command.jet](hello_command.jet): a command with a sender, parameters, subcommands, `default`, and annotations.
- [protect_blocks_listener.jet](protect_blocks_listener.jet): a listener on a cancellable event, with `@priority` and event cancellation.
- [greet_join_listener.jet](greet_join_listener.jet): a listener on `PlayerJoinEvent` using the event object.
- [heartbeat_interval.jet](heartbeat_interval.jet): an interval that logs and broadcasts through the `bukkit` API.
- [modules_and_errors.jet](modules_and_errors.jet): `math`, `random`, `json`, `storage`, `try`/`catch`, and a threaded HTTP call.
- [shared/strings_util.jet](shared/strings_util.jet) with [use_import.jet](use_import.jet): exporting and importing across modules.
