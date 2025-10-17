The `return` statement immediately terminates the execution of the
`action`, `function` or `control` containing it. `return` statements are
not allowed within parsers. `return` statements followed by an
expression are only allowed within functions that return values; in this
case the type of the expression must match the return type of the
function. Any copy-out behavior due to direction `out` or `inout`
parameters of the enclosing `action`, `function`, or `control` are still
performed after the execution of the `return` statement. See Section
\[\#sec-calling-convention\] for details on copy-out behavior.

\~ Begin P4Grammar \[INCLUDE=grammar.mdk:returnStatement\] \~ End
P4Grammar
