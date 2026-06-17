# Enums

`enum` declares a file-scope read-only group of named constant values.

## Form

```text
[access] enum Name {
  MEMBER,
  OTHER = "value"
}
```

Enum members are accessed through member syntax:

```jet
enum STATUS {
  READY,
  FAILED = "failed",
  RETRY = 10
}

int ready = STATUS.READY
string failed = STATUS.FAILED
int retry = STATUS.RETRY
```

## Values

- Explicit enum values may be `string`, `int`, `float`, or `bool` literals.
- A member without an explicit value receives the next integer value.
- The first implicit value is `0`.
- After an explicit integer value, the next implicit value is that integer plus one.
- After an explicit non-integer value, the following member must provide an explicit value.

```jet
enum CODE {
  OK = 200,
  NOT_FOUND
}
```

`CODE.NOT_FOUND` is `201`.

```jet
enum INVALID {
  NAME = "custom",
  NEXT
}
```

This is invalid because `NEXT` has no integer value to continue from.

## Access and imports

`enum` supports `public`, `protected`, and `private` like other file-scope declarations. A public enum can be imported and read from another module. Enum members are read-only.
