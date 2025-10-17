Certain expressions in a P4 program have the property that their value
can be determined at compile time. Moreover, for some of these
expressions, their value can be determined only using information in the
current scope. We call these *compile-time known values* and *local
compile-time known values* respectively.

The following are local compile-time known values:

  - Integer literals, Boolean literals, and string literals.
  - Identifiers declared as constants using the `const` keyword.
  - Identifiers declared in an `error`, `enum`, or `match_kind`
    declaration.
  - The `default` identifier.
  - The `size` field of a value with type header stack.
  - The `_` identifier when used as a `select` expression label
  - The expression `{#}` representing an invalid header or header union
    value.
  - Instances constructed by instance declarations (Section
    \[\#sec-instantiations\]) and constructor invocations.
  - Identifiers that represent declared types, actions, functions,
    tables, parsers, controls, or packages.
  - Tuple expression where all components are local compile-time known
    values.
  - Structure-valued expressions, where all fields are local
    compile-time known values.
  - Expressions evaluating to a list type, where all elements are local
    compile-time known values.
  - Legal casts applied to local compile-time known values.
  - The following expressions (`+`, `-`, `|+|`, `|-|`, `*`, `/`, `%`,
    `!`, `&`, `|`, `^`, `&&`, `||`, `<<`, `>>`, `~`, `/`, `>`, `<`,
    `==`, `!=`, `<=`, `>=`, `++`, `[:]`, `?:`) when their operands are
    all local compile-time known values.
  - Expressions of the form `e.minSizeInBits()`, `e.minSizeInBytes()`,
    `e.maxSizeInBits()` and `e.maxSizeInBytes()` where the type of `e`
    is not generic.

The following are compile-time known values:

  - All local compile-time known values.
  - Constructor parameters (i.e., the declared parameters for a
    `parser`, `control`, etc.)
  - Tuple expression where all components are compile-time known values.
  - Expressions evaluating to a list type, where all elements are
    compile-time known values.
  - Structure-valued expressions, where all fields are compile-time
    known values.
  - Expressions evaluating to a list type, where all elements are
    compile-time known values.
  - Legal casts applied to compile-time known values.
  - The following expressions (`+`, `-`, `|+|`, `|-|`, `*`, `/`, `%`,
    `cast`, `!`, `&`, `|`, `^`, `&&`, `||`, `<<`, `>>`, `~`, `/`, `>`,
    `<`, `==`, `!=`, `<=`, `>=`, `++`, `[:]`, `?:`) when their operands
    are all compile-time known values.
  - Expressions of the form `e.minSizeInBits()`, `e.minSizeInBytes()`,
    `e.maxSizeInBits()` and `e.maxSizeInBytes()` where the the type of
    `e` is generic.

Intuitively, the main difference between *compile-time known values* and
*local compile-time known values* is that the former also contains
constructor parameters. The distinction is important when it comes to
defining the meaning of features like types. For example, in the type
`bit<e>`, the expression `e` must be a local compile-time known value.
Suppose instead that `e` were a constructor parameter—i.e., merely a
compile-time known value. In this situation, it would be impossible to
resolve `bit<e>` to a concrete type using purely local information—we
would have to wait until the constructor was instantiated and the value
of `e` known.
