To aid composition of programs from multiple source files P4 compilers
should support the following subset of the C preprocessor functionality:

  - `#define` for defining macros (without arguments)
  - `#undef`
  - `#if #else #endif #ifdef #ifndef #elif`
  - `#include`

The preprocessor should also remove the sequence backslash newline
(ASCII codes 92, 10) to facilitate splitting content across multiple
lines when convenient for formatting.

Additional C preprocessor capabilities may be supported, but are not
guaranteed—e.g., macros with arguments. Similar to C, `#include` can
specify a file name either within double quotes or within `<>`.

\~ Begin P4Example \# include <system_file> \# include “user\_file” \~
End P4Example

The difference between the two forms is the order in which the
preprocessor searches for header files when the path is incompletely
specified.

P4 compilers should correctly handle `#line` directives that may be
generated during preprocessing. This functionality allows P4 programs to
be built from multiple source files, potentially produced by different
programmers at different times:

  - the P4 core library, defined in this document,
  - the architecture, defining data plane interfaces and extern blocks,
  - user-defined libraries of useful components (e.g. standard protocol
    header definitions), and
  - the P4 programs that specify the behavior of each programmable
    block.
