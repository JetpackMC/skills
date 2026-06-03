# Commands

`command` declares a server command handled by a script. Commands are file-scope entry points registered with the server's command map.

## Form

```text
[annotations]
[access] command name([object sender,] params) {
  preamble statements
  command sub(...) { ... }
  default { ... }
}
```

## Sender

A root command may declare the command sender as its first parameter, written `object sender`. It represents a player, the console, or another command source, and is available to the body and to subcommands. At runtime it is a reflected native `CommandSender` object with an extra `rawtext` field holding the full raw command line. Run permission and capability checks before player-only behavior.

## Parameters

- Parameters follow the sender. Each has an explicit type and must be a scalar: `int`, `float`, `bool`, or `string`. `list`, `object`, and `var` are rejected, because command input arrives as text parsed per type.
- A default value, when present, must be a constant expression.
- Conversion before the body runs: `int`/`float` parse the token; `bool` accepts only `true`/`false`; `string` is verbatim. A conversion failure is `ArgumentException`.
- Too few arguments for the required parameters, or unexpected extra arguments, is `ArgumentException`.

## Subcommands and default

- A `command` nested in a body is a subcommand, selected by an extra argument after the parent's parameters, matched case-insensitively by name. Subcommand names are unique within a body.
- A `default` block runs when no subcommand is selected; at most one per body.
- Plain statements in the body form a preamble that runs before the matched subcommand or `default`; a `return` in the preamble stops dispatch.
- A body with only plain statements (no subcommand, no `default`) treats those statements as the default behavior.

## Annotations

On the lines immediately before a root `command`:

```text
@description "text"          command description
@usage "text"                usage text
@permission "node"           permission required to run
@permission_message "text"   shown when permission is missing
@aliases "a,b"               comma-separated aliases, written as one string
```

## Runtime methods

The command declaration produces a command value:

```text
activate() -> bool     registers/activates; true if the state changed
deactivate() -> bool   deactivates; true if the state changed
destroy() -> bool      permanently removes the command for this runtime
trigger(...) -> null   invokes from script code; if a sender is declared, pass it first, then args; null/absent sender uses the console sender
isActive() -> bool
```

Subcommand values are reached by member access (`cmd.sub`) and expose the same methods plus deeper subcommands.
