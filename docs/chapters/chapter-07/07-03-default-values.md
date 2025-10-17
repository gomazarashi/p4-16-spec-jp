Some P4 types define a “default value,” which can be used to
automatically initialize values of that type. The default values are as
follows:

  - For `int`, `bit<N>` and `int<N>` types the default value is 0.
  - For `bool` the default value is `false`.
  - For `error` the default value is `error.NoError` (defined in
    core.p4)
  - For `string` the default value is the empty string `""`
  - For `varbit<N>` the default value is a string of zero bits (there is
    currently no P4 literal to represent such a value).
  - For `enum` values with an underlying type the default value is 0,
    even if 0 is actually not one of the named values in the enum.
  - For `enum` values without an underlying type the default value is
    the first value that appears in the `enum` type declaration.
  - For `header` types the default value is `invalid`.
  - For header stacks the default value is that all elements are invalid
    and the `nextIndex` is 0.
  - For `header_union` values the default value is that all union
    elements are invalid.
  - For `struct` types the default value is a `struct` where each field
    has the default value of the suitable field type – if all such
    default values are defined.
  - For a `tuple` type the default value is a `tuple` where each field
    has the default value of the suitable type – if all such default
    values are defined.

Note that some types do not have default values, e.g., `match_kind`, set
types, function types, extern types, parser types, control types,
package types.
