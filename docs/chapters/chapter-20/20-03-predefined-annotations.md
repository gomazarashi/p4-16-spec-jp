Annotation names that start with lowercase letters are reserved for the
standard library and architecture. This document pre-defines a set of
“standard” annotations in Appendix \[\#sec-p4-reserved-annotations\].
We expect that this list will grow. We encourage custom architectures to
define annotations starting with a manufacturer prefix: e.g., an
organization named X would use annotations named like `@X_annotation`

### Optional parameter annotations

A parameter to a package, parser type, control type, extern method,
extern function or extern object constructor can be annotated with
`@optional` to indicate that the user does not need to provide a
corresponding argument for that parameter. The meaning of a parameter
with no supplied value is target-dependent.

### Annotations on the table action list

The following two annotations can be used to give additional information
to the compiler and control-plane about actions in a table. These
annotations have no bodies.

  - `@tableonly`: actions with this annotation can only appear within
    the table, and never as default action.
  - `@defaultonly`: actions with this annotation can only appear in the
    default action, and never in the table.

\~ Begin P4Example table t { actions = { a, // can appear anywhere
@tableonly b, // can only appear in the table @defaultonly c, // can
only appear in the default action } /\* body omitted \*/ } \~ End
P4Example

### Control-plane API annotations

The `@name` annotation directs the compiler to use a different local
name when generating the external APIs used to manipulate a language
element from the control plane. This annotation takes a string literal
body. In the following example, the fully-qualified name of the table is
`c_inst.t1`.

\~ Begin P4Example control c( /\* parameters omitted */ )() {
@name(“t1”) table t { /* body omitted */ } apply { /* body omitted
\*/ } } c() c\_inst; \~ End P4Example

The `@hidden` annotation hides a controllable entity, e.g. a table, key,
action, or extern, from the control plane. This effectively removes its
fully-qualified name (Section \[\#sec-cp-names\]). This annotation does
not have a body.

#### Restrictions

Each element may be annotated with at most one `@name` or `@hidden`
annotation, and each control plane name must refer to at most one
controllable entity. This is of special concern when using an absolute
`@name` annotation: if a type containing a `@name` annotation with an
absolute pathname (i.e., one starting with a dot) is instantiated more
than once, it will result in the same name referring to two controllable
entities.

\~ Begin P4Example control noargs(); package top(noargs c1, noargs c2);

control c() { @name(“.foo.bar”) table t { /\* body omitted */ } apply {
/* body omitted \*/ } } top(c(), c()) main; \~ End P4Example

Without the `@name` annotation, this program would produce two
controllable entities with fully-qualified names `main.c1.t` and
`main.c2.t`. However, the `@name(".foo.bar")` annotation renames table
`t` in both instances to `foo.bar`, resulting in one name that refers to
two controllable entities, which is illegal.

### Concurrency control annotations

The `@atomic` annotation, described in Section \[\#sec-concurrency\] can
be used to enforce the atomic execution of a code block.

### Value set annotations

The `@match` annotation, described in Section \[\#sec-select\], is used
to specify a `match_kind` value other than the default `match_kind` of
`exact` for a field of a `value_set`.

### Extern function/method annotations

Various annotations may appear on extern function and method
declarations to describe limitations on the behavior and interactions of
those functions. By default extern functions might have any effect on
the environment of the P4 program and might interact in non-trivial ways
(subject to a few limitations – see section
\[\#sec-calling-convention-justification\]). Since externs are
architecture-specific and their behavior is known to the architecture
definition, these annotations are not strictly necessary (an
implementation can have knowledge of how externs interact based on their
names built into it), but these annotations provide a uniform way of
describing certain well-defined interactions (or their absence),
allowing architecture-independent analysis of P4 programs.

  - `@pure` - Describes a function that depends solely on its `in`
    parameter values, and has no effect other than returning a value,
    and copy-out behavior on its `out` and `inout` parameters. No hidden
    state is recorded between calls, and its value does not depend on
    any hidden state that may be changed by other calls. An example is a
    `hash` function that computes a deterministic hash of its arguments,
    and its return value does not depend upon any control-plane writable
    seed or initialization vector value. A `@pure` function whose
    results are unused may be safely eliminated with no adverse effects,
    and multiple calls with identical arguments may be combined into a
    single call (subject to the limits imposed by copy-out behavior of
    `out` and `inout` parameters). `@pure` functions may also be
    reordered with respect to other computations that are not data
    dependent.

  - `@noSideEffects` - Weaker than `@pure` and describes a function that
    does not change any hidden state, but may depend on hidden state.
    One example is a `hash` function that computes a deterministic hash
    of its arguments, plus some internal state that can be modified via
    control plane API calls such as a seed or initialization vector.
    Another example is a read of one element of a register array extern
    object. Such a function may be dead code eliminated, and may be
    reordered or combined with other `@noSideEffects` or `@pure` calls
    (subject to the limits imposed by copy-out behavior of `out` and
    `inout` parameters), but not with other function calls that may have
    side effects that affect the function.

### Deprecated annotation

The `deprecated` annotation has a required string argument that is a
message that will be printed by a compiler when a program is using the
deprecated construct. This is mostly useful for annotating library
constructs, such as externs.

\~ Begin P4Example @deprecated(“Please use the ‘check’ function
instead”) extern Checker { /\* body omitted \*/ } \~ End P4Example

### No warnings annotation

The `noWarn` annotation has a required string argument that indicates a
compiler warning that will be inhibited. For example `@noWarn("unused")`
on a declaration will prevent a compiler warning if that declaration is
not used.
