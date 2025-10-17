P4 provides multiple constructs for writing modular programs: extern
methods, parsers, controls, actions. All these constructs behave
similarly to procedures in standard general-purpose programming
languages:

  - They have named and typed parameters.
  - They introduce a new local scope for parameters and local variables.
  - They allow arguments to be passed by binding them to their
    parameters.

Invocations are executed using copy-in/copy-out semantics.

Each parameter may be labeled with a direction:

  - `in` parameters are read-only. It is an error to use an `in`
    parameter on the left-hand side of an assignment or to pass it to a
    callee as a non-`in` argument. `in` parameters are initialized by
    copying the value of the corresponding argument when the invocation
    is executed.
  - `out` parameters are, with a few exceptions listed below,
    uninitialized and are treated as l-values (See Section
    \[\#sec-lvalues\]) within the body of the method or function. An
    argument passed as an `out` parameter must be an l-value; after the
    execution of the call, the value of the parameter is copied to the
    corresponding storage location for that l-value.
  - `inout` parameters behave like a combination of `in` and `out`
    parameters simultaneously: On entry the value of the arguments is
    copied to the parameters. On return the value of the parameters is
    copied back to the arguments. In consequence, an argument passed as
    an `inout` parameter must be an l-value.
  - The meaning of parameters with no direction depends upon the kind of
    entity the parameter is for:
      - For anything other than an action, e.g. a control, parser, or
        function, a directionless parameter means that the value
        supplied as an argument in a call must be a compile-time known
        value (see Section \[\#sec-compile-time-known\]).
      - For an action, a directionless parameter indicates that it is
        “action data”. See Section \[\#sec-actions\] for the meaning
        of action data, but its meaning includes the following
        possibilities:
          - The parameter’s value is provided in the P4 program. In this
            case, the parameter behaves as if the direction were `in`.
            Such an argument expression need not be a compile-time known
            value.
          - The parameter’s value is provided by the control plane
            software when an entry is added to a table that uses that
            action. See Section \[\#sec-actions\].

A directionless parameter of extern object type is passed by reference.

Direction `out` parameters are always initialized at the beginning of
execution of the portion of the program that has the `out` parameters,
e.g. `control`, `parser`, `action`, function, etc. This initialization
is not performed for parameters with any direction that is not `out`.

  - If a direction `out` parameter is of type `header` or
    `header_union`, it is set to “invalid”.
  - If a direction `out` parameter is of type header stack, all elements
    of the header stack are set to “invalid”, and its `nextIndex` field
    is initialized to 0 (see Section \[\#sec-expr-hs\]).
  - If a direction `out` parameter is a compound type, e.g. a struct or
    tuple, other than one of the types listed above, then apply these
    rules recursively to its members.
  - If a direction `out` parameter has any other type, e.g. `bit<W>`, an
    implementation need not initialize it to any predictable value.

<!-- end list -->

  - For example, if a direction `out` parameter has type `s2_t` named
    `p`:  
    Begin P4Example header h1\_t { bit\<8\> f1; bit\<8\> f2; } struct
    s1\_t { h1\_t h1a; bit\<3\> a; bit\<7\> b; } struct s2\_t { h1\_t
    h1b; s1\_t s1; bit\<5\> c; }
    
    End P4Example

then at the beginning of execution of the part of the program that has
the `out` parameter `p`, it must be initialized so that `p.h1b` and and
`p.s1.h1a` are invalid. No other parts of `p` are required to be
initialized.

Arguments are evaluated from left to right prior to the invocation of
the function itself. The order of evaluation is important when the
expression supplied for an argument can have side-effects. Consider the
following example:

\~ Begin P4Example extern void f(inout bit x, in bit y); extern bit
g(inout bit z); bit a; f(a, g(a)); \~ End P4Example

Note that the evaluation of `g` may mutate its argument `a`, so the
compiler has to ensure that the value passed to `f` for its first
parameter is not changed by the evaluation of the second argument. The
semantics for evaluating a function call is given by the following
algorithm (implementations can be different as long as they provide the
same result):

1.  Arguments are evaluated from left to right as they appear in the
    function call expression.

2.  ``` 
     If a parameter has a default value and no corresponding argument is
    ```
    
    supplied, the default value is used as an argument.

3.  For each `out` and `inout` argument the corresponding l-value is
    saved (so it cannot be changed by the evaluation of the following
    arguments). This is important if the argument contains indexing
    operations into a header stack.

4.  The value of each argument is saved into a temporary.

5.  The function is invoked with the temporaries as arguments. We are
    guaranteed that the temporaries that are passed as arguments are
    never aliased to each other, so this “generated” function call can
    be implemented using call-by-reference if supported by the
    architecture.

6.  On function return, the temporaries that correspond to `out` or
    `inout` arguments are copied in order from left to right into the
    l-values saved in Step 3.

According to this algorithm, the previous function call is equivalent to
the following sequence of statements:

\~ Begin P4Example bit tmp1 = a; // evaluate a; save result bit tmp2 =
g(a); // evaluate g(a); save result; modifies a f(tmp1, tmp2); //
evaluate f; modifies tmp1 a = tmp1; // copy inout result back into a \~
End P4Example

To see why Step 3 in the above algorithm is important, consider the
following example:

\~ Begin P4Example header H { bit z; } H\[2\] s; f(s\[a\].z, g(a)); \~
End P4Example

The evaluation of this call is equivalent to the following sequence of
statements:

\~ Begin P4Example bit tmp1 = a; // save the value of a bit tmp2 =
s\[tmp1\].z; // evaluate first argument bit tmp3 = g(a); // evaluate
second argument; modifies a f(tmp2, tmp3); // evaluate f; modifies tmp2
s\[tmp1\].z = tmp2; // copy inout result back; dest is not s\[a\].z \~
End P4Example

When used as arguments, `extern` objects can only be passed as
directionless parameters—e.g., see the packet argument in the very
simple switch example.

### Justification

The main reason for using copy-in/copy-out semantics (instead of the
more common call-by-reference semantics) is for controlling the
side-effects of `extern` functions and methods. `extern` methods and
functions are the main mechanism by which a P4 program communicates with
its environment. With copy-in/copy-out semantics `extern` functions
cannot hold references to P4 program objects; this enables the compiler
to limit the side-effects that `extern` functions may have on the P4
program both in space (they can only affect `out` parameters) and in
time (side-effects can only occur at function call time).

In general, `extern` functions are arbitrarily powerful: they can store
information in global storage, spawn separate threads, “collude” with
each other to share information — but they cannot access any variable in
a P4 program. With copy-in/copy-out semantics the compiler can still
reason about P4 programs that invoke `extern` functions.

There are additional benefits of using copy-in copy-out semantics:

  - It enables P4 to be compiled for architectures that do not support
    references (e.g., where all data is allocated to named registers.
    Such architectures may require indices into header stacks that
    appear in a program to be compile-time known values.)
  - It simplifies some compiler analyses, since function parameters can
    never alias to each other within the function body.

\~ Begin P4Grammar \[INCLUDE=grammar.mdk:parameterList\]

\[INCLUDE=grammar.mdk:nonEmptyParameterList\]

\[INCLUDE=grammar.mdk:parameter\]

  - \[INCLUDE=grammar.mdk:direction\]  
    End P4Grammar

Following is a summary of the constraints imposed by the parameter
directions:

  - When used as arguments, extern objects can only be passed as
    directionless parameters.
  - All constructor parameters are evaluated at compilation-time, and in
    consequence they must all be directionless (they cannot be `in`,
    `out`, or `inout`); this applies to `package`, `control`, `parser`,
    and `extern` objects. Expressions for these parameters must be
    supplied at compile-time, and they must evaluate to compile-time
    known values. See Section \[\#sec-parameterization\] for further
    details.
  - For actions all directionless parameters must be at the end of the
    parameter list. When an action appears in a `table`’s `actions`
    list, only the parameters with a direction must be bound. See
    Section \[\#sec-actions\] for further details.
  - Actions can also be explicitly invoked using function call syntax,
    either from a control block or from another action. In this case,
    values for all action parameters must be supplied explicitly,
    including values for the directionless parameters. The directionless
    parameters in this case behave like `in` parameters. See Section
    \[\#sec-invoke-actions\] for further details.
  - Default expressions are only allowed for ‘in’ or direction-less
    parameters, and the expressions supplied as defaults must be
    compile-time known values.
  - If parameters with default values do not appear at the end of the
    list of parameters, invocations that use the default values must use
    named arguments, as in the following example:

\~ Begin P4Example extern void f(in bit a, in bit\<3\> b = 2, in
bit\<5\> c);

void g() { f(a = 1, b = 2, c = 3); // ok f(a = 1, c = 3); // ok,
equivalent to the previous call, b uses default value f(1, 2, 3); // ok,
equivalent to the previous call // f(1, 3); // illegal, since the
parameter b is not the last in the list } \~ End P4Example

### Optional parameters

A parameter that is annotated with the `@optional` annotation is
optional: the user may omit the value for that parameter in an
invocation. Optional parameters can only appear for arguments of:
packages, parser types, control types, extern functions, extern methods,
and extern object constructors. Optional parameters cannot have default
values. If a procedure-like construct has both optional parameters and
default values then it can only be called using named arguments. It is
recommended, but not mandatory, for all optional parameters to be at the
end of a parameter list.

The implementation of such objects is not expressed in P4, so the
meaning and implementation of optional parameters should be specified by
the target architecture. For example, we can imagine a two-stage switch
architecture where the second stage is optional. This could be declared
as a package with an optional parameter:

\~Begin P4Example package pipeline(/\* parameters omitted \*/); package
switch(pipeline first, @optional pipeline second);

pipeline(/\* arguments omitted \*/) ingress; switch(ingress) main; // a
switch with a single-stage pipeline \~End P4Example

Here the target architecture could implement the elided optional
argument using an empty pipeline.

The following example shows optional parameters and parameters with
default values.

\~Begin P4Example extern void h(in bit\<32\> a, in bool b = true); //
default value

// function calls h(10); // same as h(10, true); h(a = 10); // same as
h(10, true); h(a = 10, b = true);

struct Empty {} control nothing(inout Empty h, inout Empty m) { apply {}
}

parser parserProto\<H, M\>(packet\_in p, out H h, inout M m); control
controlProto\<H, M\>(inout H h, inout M m);

package pack\<HP, MP, HC, MC\>( @optional parserProto\<HP, MP\>
\_parser, // optional parameter controlProto\<HC, MC\> \_control =
nothing()); // default parameter value

pack() main; // No value for \_parser, \_control is an instance of
nothing() \~End P4Example
