P4 provides a rich assortment of types. Base types include bit-strings,
numbers, and errors. There are also built-in types for representing
constructs such as parsers, pipelines, actions, and tables. Users can
construct new types based on these: structures, enumerations, headers,
header stacks, header unions, etc.

In this document we adopt the following conventions:

  - Built-in types are written with lowercase characters—e.g.,
    `int<20>`,
  - User-defined types are capitalized—e.g., `IPv4Address`,
  - Type variables are always uppercase—e.g., `parser P<H, IH>()`,
  - Variables are uncapitalized— e.g., `ipv4header`,
  - Constants are written with uppercase characters—e.g., `CPU_PORT`,
    and
  - Errors and enumerations are written in camel-case—
    e.g. `PacketTooShort`.
