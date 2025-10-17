The `exit` statement immediately terminates the execution of all the
blocks currently executing: the current `action` (if invoked within an
`action`), the current `control`, and all its callers. `exit` statements
are not allowed within parsers or functions.

Any copy-out behavior due to direction `out` or `inout` parameters of
the enclosing `action` or `control`, and all of its callers, are still
performed after the execution of the `exit` statement. See Section
\[\#sec-calling-convention\] for details on copy-out behavior.

\~ Begin P4Grammar \[INCLUDE=grammar.mdk:exitStatement\] \~ End
P4Grammar

There are some expressions whose evaluation might cause an `exit`
statement to be executed. Examples include:

  - `table.apply().action_run`, which can only appear as the expression
    of a `switch` statement (see Section \[\#sec-switch-stmt\]), and
    when it appears there, it must be the entire expression.
  - Any expression containing `table.apply().hit` or
    `table.apply().miss` (see Section \[\#sec-invoke-mau\]), which can
    be part of arbitrarily complex expressions in many places of a P4
    program, such as:
      - the expression in an `if` statement.
      - the expression `e1` in a conditional expression `e1 ? e2 : e3`.
      - in an assignment statement, in the left and/or right hand sides.
      - an argument passed to a function or method call.
      - an expression to calculate a table key (see Section
        \[\#sec-mau-semantics\]).

This list is not intended to be exhaustive.

If applying the table causes an action to be executed, which in turn
causes an `exit` statement to be executed, then evaluation of the
expression ends immediately, and the rest of the current expression or
statement does not complete its execution. See Section
\[\#sec-expr-eval-order\] for the order of evaluation of the parts of an
expression. For the examples listed above, it also means the following
behavior after the expression evaluation is interrupted.

  - For a `switch` statement, if `table.apply()` exits, then none of the
    blocks in the `switch` statement are executed.
  - If `table.apply().hit` or `table.apply().miss` cause an exit during
    the evaluation of an expression:
      - If it is the expression of an `if` statement, then neither the
        ‘then’ nor ‘else’ branches of the `if` statement are executed.
      - If it is the expression `e1` in a conditional expression `e1 ?
        e2 : e3`, then neither expression `e2` nor `e3` are evaluated.
      - If the expression is the right hand side of an assignment
        statement, or part of the calculation of the L-value on the left
        hand side (e.g. the index expression of a header stack
        reference), then no assignment occurs.
      - If the expression is an argument passed to a function or method
        call, then the function/method call does not occur.
      - If the expression is a table key, then the table is not applied.
