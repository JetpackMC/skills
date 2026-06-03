# Script Structure

## Files and modules

Each `.jet` file under `plugins/Jetpack/scripts/` is a module. A module's path is its file path relative to the scripts root, without the `.jet` suffix, with directory separators as dots. A file at `scripts/shared/utils.jet` has module path `shared.utils`.

A module's top level may contain: a `manifest` block, `using` imports, `@`-annotations, variable declarations, function/interval/listener/command declarations, deconstruction declarations, and statements.

Conventional layout of a file:

```text
manifest { ... }        // optional, at most one, at the top
using ...               // imports
<top-level declarations and statements>
```

## manifest

```text
manifest {
  name = "Example"
  version = "1.0.0"
  author = "Alex"
  description = "Example script"
}
```

- The block belongs at the top of the file and is read once at load time. It is metadata, not a runtime object.
- Keys are bare identifiers. Values are string, int, or float literals, or arrays of those.
- The keys surfaced as script metadata are `name`, `description`, `version`, and `author`. Other keys parse without error but are not read as metadata. A value given as an array is joined into a single string for metadata purposes.
- Access modifiers do not apply to `manifest`. At most one `manifest` per file.

## Annotations

An annotation is `@key "string"` on its own line. When one or more annotations immediately precede a `command` or `listener` declaration, they configure that declaration (see [commands.md](commands.md) and [listeners-and-events.md](listeners-and-events.md)). Annotations not attached to such a declaration are inert.

## using imports

```text
using math
using json as Json
using shared.utils
using shared.messages as messages
using .messages
using ..shared.utils
using shared.*
```

### Registered modules

A single-segment `using name` imports a registered module when `name` is one (`math`, `json`, `random`, `storage`, `time`, `regex`, `http`, `bukkit`, or a host-plugin-registered module). Registered module names are case-sensitive. The dynamic `bukkit` module also allows deeper paths and aliasing of a member, for example `using bukkit.Material as Material`.

### Script imports

A dotted path that is not a registered module imports another script module by its module path. The imported names available depend on the access modifiers in the imported script.

### Relative and recursive

- Leading dots make the path relative to the importing module's directory. One dot is the current directory; each additional dot moves one level up.
- A trailing `.*` is a recursive import that brings in every module under the prefix directory.
- A recursive import cannot be aliased.

### Aliases

`using path as alias` binds the import under `alias`. An alias that collides with a builtin global or another import is an error.

## Access and exports

An imported module exposes its top-level declarations according to their access modifier:

- `public`: visible to importers; writable where the underlying declaration is writable.
- `protected`: visible to importers as a read-only view; mutation through the import is blocked.
- `private`: not exported.

`const` declarations and `protected` declarations are read-only to importers. A `protected` value of a structural kind (list, object, command, listener, interval, nested module) is exposed through a wrapper that blocks mutation and lifecycle control from another file.

Reading an exported variable before the defining module has initialized it raises a runtime error. Function/interval/listener/command exports are available once declared.

## Load order

1. All scripts are parsed.
2. Imports are resolved for every module.
3. Each module is validated (name resolution, then type checking) after its dependencies, with circular imports reported as an error.
4. Validated modules load. Loading a module runs its top level: top-level function/interval/listener/command declarations are established first, then the remaining top-level statements execute in source order, importing dependencies on demand.

A module that fails validation or whose dependency fails does not load, and its entry points are not registered.
