# Appendix D P4 core library


The P4 core library contains declarations that are useful to most
programs.

For example, the core library includes the declarations of the
predefined `packet_in` and `packet_out` extern objects, used in parsers
and deparsers to access packet data.

\~ Begin P4Example /// Standard error codes. New error codes can be
declared by users. error { NoError, /// No error. PacketTooShort, ///
Not enough bits in packet for ‘extract’. NoMatch, /// ‘select’
expression has no matches. StackOutOfBounds, /// Reference to invalid
element of a header stack. HeaderTooShort, /// Extracting too many bits
into a varbit field. ParserTimeout, /// Parser execution time limit
exceeded. ParserInvalidArgument /// Parser operation was called with a
value /// not supported by the implementation. } extern packet\_in { ///
Read a header from the packet into a fixed-sized header @hdr /// and
advance the cursor. /// May trigger error PacketTooShort or
StackOutOfBounds. /// @T must be a fixed-size header type void
extract<T>(out T hdr); /// Read bits from the packet into a
variable-sized header @variableSizeHeader /// and advance the cursor.
/// @T must be a header containing exactly 1 varbit field. /// May
trigger errors PacketTooShort, StackOutOfBounds, or HeaderTooShort. void
extract<T>(out T variableSizeHeader, in bit\<32\>
variableFieldSizeInBits); /// Read bits from the packet without
advancing the cursor. /// @returns: the bits read from the packet. /// T
may be an arbitrary fixed-size type. T lookahead<T>(); /// Advance the
packet cursor by the specified number of bits. void advance(in bit\<32\>
sizeInBits); /// @return packet length in bytes. This method may be
unavailable on /// some target architectures. bit\<32\> length(); }
extern packet\_out { /// Write @data into the output packet, skipping
invalid headers /// and advancing the cursor /// @T can be a header
type, a header stack, a header\_union, or a struct /// containing fields
with such types. void emit<T>(in T data); } action NoAction() {} ///
Standard match kinds for table key fields. /// Some architectures may
not support all these match kinds. /// Architectures can declare
additional match kinds. match\_kind { /// Match bits exactly. exact, ///
Ternary match, using a mask. ternary, /// Longest-prefix match. lpm }

/// Static assert evaluates a boolean expression /// at compilation
time. If the expression evaluates to /// false, compilation is stopped
and the corresponding message is printed. /// The function returns a
boolean, so that it can be used /// as a global constant value in a
program, e.g.: /// const version = static\_assert(V1MODEL\_VERSION \>
20180000, “Expected a v1 model version \>= 20180000”); extern bool
static\_assert(bool check, string message);

/// Like the above but using a default message. extern bool
static\_assert(bool check);

\~ End P4Example

