# Bukkit and the Native Bridge

The `bukkit` module is the gateway to the live Minecraft server. Through it a script reaches the Paper/Bukkit Java API, which represents players, worlds, blocks, items, inventories, the scheduler, and everything else the server exposes. Unlike the standard library, the `bukkit` module is not a fixed list of functions; it mirrors the server's Java classes directly using reflection, so what a script can do is whatever the underlying API offers.

Import it with `using bukkit`. The same reflection bridge also wraps native values that arrive from the server, namely the event objects passed to listeners, the command sender passed to commands, and the return values of any native call, so the rules below apply to those values as well.

The bridge reaches classes under the `org.bukkit` package. Types from other server packages cannot be navigated from `bukkit`, but their values are still wrapped and usable when the server hands them to a listener or command.

## Navigating the API

`bukkit.X` resolves a member under the `org.bukkit` package:

- If `org.bukkit.X` is a class, `bukkit.X` is that class object.
- Otherwise `bukkit.X` is a subpackage you keep navigating, for example `bukkit.entity.Player` or `bukkit.inventory.ItemStack`.

You can import a specific class with an alias: `using bukkit.Material as Material`.

## Class members

A class object exposes:

- Static fields, read as `Class.FIELD`.
- Static methods, called as `Class.method(args)`.
- Enum constants by name, read as `EnumClass.CONSTANT`.

Construct an instance by calling the class object like a function: `bukkit.SomeClass(args)`.

## Instance members

A reflected instance is a read-only Jet object whose members come from the Java class:

- A getter `getX()` or boolean `isX()` with no parameters is read as the property `obj.x`.
- A public field is read as `obj.field`.
- A setter `setX(v)` or a writable public field is assigned as `obj.x = v`.
- Other methods are called as `obj.method(args)`.

A native call or member access that fails raises `NativeException`.

## Overload resolution and argument conversion

When several methods or constructors share a name, the bridge picks the best match by scoring argument conversions; an unresolvable tie raises an error, and no viable candidate raises `NativeException`. Jet values convert to Java parameter types as follows:

- `int` to `int`/`long`/`short`/`byte`/`float`/`double`/`Number` (range-checked where narrowing).
- `float` to `float`/`double` and to integral types when in range.
- `string` to `String`/`CharSequence`, and to `char` when length 1.
- `bool` to `boolean`.
- An enum parameter accepts the matching native enum object, or a `string` naming the constant (exact match preferred, then case-insensitive).
- `list` to a Java array or `Collection`.
- `object` to a `Map`.
- A native object passes through where the parameter type accepts it.
- `null` passes to any non-primitive parameter.

## Return value wrapping

Native results become Jet values:

- `null` to `null`; Java numbers to `int`/`float`; `boolean` to `bool`; `String`/`char` to `string`.
- Arrays and `Collection`s to `list`; `Map`s to a read-only `object`.
- Enums to a native enum object. In a `listener` event object specifically, enum fields are exposed as their `string` names.
- Other objects to a read-only Jet object with the member access described above.

## Output paths

Output to players and the console uses native members:

- In a command, the sender: `sender.sendMessage("text")`.
- In a listener, an entity on the event: `event.player.sendMessage("text")`.
- Server-wide or from any context, the `bukkit` API: for example `bukkit.Bukkit.broadcastMessage("text")` to broadcast, or `bukkit.Bukkit.getLogger().info("text")` to log to the console.
