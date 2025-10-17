Each P4 compiler implementation can define additional annotations
specific to the target of the compiler. The syntax of the annotations
should conform to the above description. The semantics of such
annotations is target-specific. They could be used in a similar way to
pragmas in other languages.

The P4 compiler should provide:

  - Errors when annotations are used incorrectly (e.g., an annotation
    expecting a parameter but used without arguments, or with arguments
    of the wrong type)
  - Warnings for unknown annotations.
