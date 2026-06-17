# Lexical Structure

How source text becomes tokens. A lexing error stops the script before parsing.

## Comments

- `// ...` line comment, to end of line.
- `/* ... */` block comment. Must be closed; an unclosed block comment is a lexer error. Block comments cannot nest, because they end at the first `*/`.

## Whitespace and separators

Spaces, tabs, and carriage returns are insignificant between tokens. Newlines and `;` separate statements; consecutive ones collapse. A statement normally ends at a newline, with `;` available to end or separate explicitly. Inside `(...)`, `[...]`, and argument lists, newlines are skipped, so an expression may wrap across lines there.

## Keywords

Reserved; not usable as identifiers.

```text
int float string bool list object var null
true false
function return const
public private protected
if else while foreach in break continue
try catch finally
thread to until
enum
interval listener command default
using as manifest
```

`true`, `false`, `null` are literals; the rest are syntactic keywords.

## Identifiers

Starts with a letter or `_`, continues with letters, digits, or `_`. Case-sensitive. `_` is a normal identifier except as the skip placeholder inside a deconstruction pattern. Builtin global names (currently `typeof`) and imported module names are reserved at name resolution and cannot be redeclared.

## Literals

Integer: digits, stored as 32-bit signed. A literal outside `-2147483648..2147483647` is a parse error.

Float: digits, `.`, digits, with a digit required on both sides of the `.`, for example `3.14`.

String: delimited by `"` or `'`. A raw newline inside a literal is an error. Escapes: `\n` `\t` `\r` `\\` `\'` `\"`. Any other escape is a lexer error.

Interpolated string: begins with `$"`; see [strings.md](strings.md). Inside it the recognized escapes are `\n` `\t` `\r` `\\` `\"` (not `\'`).

Boolean and null: `true`, `false`, `null`.

## Operator and punctuation tokens

```text
+  -  *  /  %  **
+= -= *= /= %= **=
++ --
!  !=
=  ==
<  <=  >  >=
&&  ||
?  :
.  ,  ;
(  )  {  }  [  ]
@
```

A lone `&` or `|` is a lexer error; only `&&` and `||` exist.
