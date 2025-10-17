Evaluation of a program proceeds in order of declarations, starting in
the top-level namespace:

  - All declarations (e.g., `parsers`, `controls`, types, constants)
    evaluate to themselves.
  - Each `table` evaluates to a table instance.
  - Constructor invocations evaluate to stateful objects of the
    corresponding type. For this purpose, all constructor arguments are
    evaluated recursively and bound to the constructor parameters.
    Constructor arguments must be compile-time known values. The order
    of evaluation of the constructor arguments should be unimportant —
    all evaluation orders should produce the same results.
  - Instantiations evaluate to named stateful objects.
  - The instantiation of a `parser` or `control` block recursively
    evaluates all stateful instantiations declared in the block.
  - The result of the program’s evaluation is the value of the top-level
    `main` variable.

Note that all stateful values are instantiated at compilation time.

  - As an example, consider the following program fragment:  
    Begin P4Example // architecture declaration parser P(/\* parameters
    omitted */); control C(/* parameters omitted */); control D(/*
    parameters omitted \*/);

package Switch(P prs, C ctrl, D dep);

extern Checksum16 { /\* body omitted \*/}

// user code Checksum16() ck16; // checksum unit instance

parser TopParser(/\* parameters omitted */)(Checksum16 unit) { /* body
omitted */} control Pipe(/* parameters omitted */) { /* body omitted */}
control TopDeparser(/* parameters omitted */)(Checksum16 unit) { /* body
omitted \*/}

Switch(TopParser(ck16), Pipe(), TopDeparser(ck16)) main; \~ End
P4Example

The evaluation of this program proceeds as follows:

1.  The declarations of `P`, `C`, `D`, `Switch`, and `Checksum16` all
    evaluate to themselves.
2.  The `Checksum16() ck16` instantiation is evaluated and it produces
    an object named `ck16` with type `Checksum16`.
3.  The declarations for `TopParser`, `Pipe`, and `TopDeparser` evaluate
    as themselves.
4.  The `main` variable instantiation is evaluated:
5.  The arguments to the constructor are evaluated recursively
6.  `TopParser(ck16)` is a constructor invocation 1. Its argument is
    evaluated recursively; it evaluates to the `ck16` object
7.  The constructor itself is evaluated, leading to the instantiation of
    an object of type `TopParser`
8.  Similarly, `Pipe()` and `TopDeparser(ck16)` are evaluated as
    constructor calls.
9.  All the arguments of the `Switch` package constructor have been
    evaluated (they are an instance of `TopParser`, an instance of
    `Pipe`, and an instance of `TopDeparser`). Their signatures are
    matched with the `Switch` declaration.
10. Finally, the `Switch` constructor can be evaluated. The result is an
    instance of the `Switch` package (that contains a `TopParser` named
    `prs` the first parameter of the `Switch`; a `Pipe` named `ctrl`;
    and a `TopDeparser` named `dep`).
11. The result of the program evaluation is the value of the `main`
    variable, which is the above instance of the `Switch` `package`.

Figure \[\#fig-compileeval\] shows the result of the evaluation in a
graphical form. The result is always a graph of instances. There is only
one instance of `Checksum16`, called `ck16`, shared between the
`TopParser` and `TopDeparser`. Whether this is possible is
architecture-dependent. Specific target compilers may require distinct
checksum units to be used in distinct blocks.

\~ Figure { \#fig-compileeval; caption: “Evaluation result.” }
\[compileeval\] \~ \[compileeval\]: figs/compileeval.png { width: 5cm;
page-align: here }
