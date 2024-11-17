+++
title = "Text Format Language Specification"
date = 2024-11-17T12:07:24+08:00
weight = 30
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文：[https://protobuf.dev/reference/protobuf/textformat-spec/](https://protobuf.dev/reference/protobuf/textformat-spec/)
>
> 收录该文档的时间：`2024-11-17T12:07:24+08:00`

# Text Format Language Specification

The protocol buffer Text Format Language specifies a syntax for representation of protobuf data in text form, which is often useful for configurations or tests.



This format is distinct from the format of text within a `.proto` schema, for example. This document contains reference documentation using the syntax specified in [ISO/IEC 14977 EBNF](https://en.wikipedia.org/wiki/Extended_Backus–Naur_Form).

#### Note

This is a draft spec reverse-engineered from the C++ text format [implementation](https://github.com/protocolbuffers/protobuf/blob/master/src/google/protobuf/text_format.cc) and may change based on further discussion and review. While an effort has been made to keep text formats consistent across supported languages, incompatibilities are likely to exist.

## Example

```fallback
convolution_benchmark {
  label: "NHWC_128x20x20x56x160"
  input {
    dimension: [128, 56, 20, 20]
    data_type: DATA_HALF
    format: TENSOR_NHWC
  }
}
```

## Parsing Overview

The language elements in this spec are split into lexical and syntactic categories. Lexical elements must match the input text exactly as described, but syntactic elements may be separated by optional `WHITESPACE` and `COMMENT` tokens.

For example, a signed floating point value comprises two syntactic elements: the sign (`-`) and the `FLOAT` literal. Optional whitespace and comments may exist between the sign and the number, but not within the number. Example:

```fallback
value: -2.0   # Valid: no additional whitespace.
value: - 2.0  # Valid: whitespace between '-' and '2.0'.
value: -
  # comment
  2.0         # Valid: whitespace and comments between '-' and '2.0'.
value: 2 . 0  # Invalid: the floating point period is part of the lexical
              # element, so no additional whitespace is allowed.
```

There is one edge case that requires special attention: a number token (`FLOAT`, `DEC_INT`, `OCT_INT`, or `HEX_INT`) may not be immediately followed by an `IDENT` token. Example:

```fallback
foo: 10 bar: 20           # Valid: whitespace separates '10' and 'bar'
foo: 10,bar: 20           # Valid: ',' separates '10' and 'bar'
foo: 10[com.foo.ext]: 20  # Valid: '10' is followed immediately by '[', which is
                          # not an identifier.
foo: 10bar: 20            # Invalid: no space between '10' and identifier 'bar'.
```

## Lexical Elements

The lexical elements described below fall into two categories: uppercase primary elements and lowercase fragments. Only primary elements are included in the output stream of tokens used during syntactic analysis; fragments exist only to simplify construction of primary elements.

When parsing input text, the longest matching primary element wins. Example:

```fallback
value: 10   # '10' is parsed as a DEC_INT token.
value: 10f  # '10f' is parsed as a FLOAT token, despite containing '10' which
            # would also match DEC_INT. In this case, FLOAT matches a longer
            # subsequence of the input.
```

### Characters

```fallback
char    = ? Any non-NUL unicode character ? ;
newline = ? ASCII #10 (line feed) ? ;

letter = "A" | "B" | "C" | "D" | "E" | "F" | "G" | "H" | "I" | "J" | "K" | "L" | "M"
       | "N" | "O" | "P" | "Q" | "R" | "S" | "T" | "U" | "V" | "W" | "X" | "Y" | "Z"
       | "a" | "b" | "c" | "d" | "e" | "f" | "g" | "h" | "i" | "j" | "k" | "l" | "m"
       | "n" | "o" | "p" | "q" | "r" | "s" | "t" | "u" | "v" | "w" | "x" | "y" | "z"
       | "_" ;

oct = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" ;
dec = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9" ;
hex = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9"
    | "A" | "B" | "C" | "D" | "E" | "F"
    | "a" | "b" | "c" | "d" | "e" | "f" ;
```

### Whitespace and Comments

```fallback
COMMENT    = "#", { char - newline }, [ newline ] ;
WHITESPACE = " "
           | newline
           | ? ASCII #9  (horizontal tab) ?
           | ? ASCII #11 (vertical tab) ?
           | ? ASCII #12 (form feed) ?
           | ? ASCII #13 (carriage return) ? ;
```

### Identifiers

```fallback
IDENT = letter, { letter | dec } ;
```

### Numeric Literals

```fallback
dec_lit   = "0"
          | ( dec - "0" ), { dec } ;
float_lit = ".", dec, { dec }, [ exp ]
          | dec_lit, ".", { dec }, [ exp ]
          | dec_lit, exp ;
exp       = ( "E" | "e" ), [ "+" | "-" ], dec, { dec } ;

DEC_INT   = dec_lit
OCT_INT   = "0", oct, { oct } ;
HEX_INT   = "0", ( "X" | "x" ), hex, { hex } ;
FLOAT     = float_lit, [ "F" | "f" ]
          | dec_lit,   ( "F" | "f" ) ;
```

Decimal integers can be cast as floating-point values by using the `F` and `f` suffixes. Example:

```fallback
foo: 10    # This is an integer value.
foo: 10f   # This is a floating-point value.
foo: 1.0f  # Also optional for floating-point literals.
```

### String Literals

```fallback
STRING = single_string | double_string ;
single_string = "'", { escape | char - "'" - newline - "\" }, "'" ;
double_string = '"', { escape | char - '"' - newline - "\" }, '"' ;

escape = "\a"                        (* ASCII #7  (bell)                 *)
       | "\b"                        (* ASCII #8  (backspace)            *)
       | "\f"                        (* ASCII #12 (form feed)            *)
       | "\n"                        (* ASCII #10 (line feed)            *)
       | "\r"                        (* ASCII #13 (carriage return)      *)
       | "\t"                        (* ASCII #9  (horizontal tab)       *)
       | "\v"                        (* ASCII #11 (vertical tab)         *)
       | "\?"                        (* ASCII #63 (question mark)        *)
       | "\\"                        (* ASCII #92 (backslash)            *)
       | "\'"                        (* ASCII #39 (apostrophe)           *)
       | '\"'                        (* ASCII #34 (quote)                *)
       | "\", oct, [ oct, [ oct ] ]  (* octal escaped byte value         *)
       | "\x", hex, [ hex ]          (* hexadecimal escaped byte value   *)
       | "\u", hex, hex, hex, hex    (* Unicode code point up to 0xffff  *)
       | "\U000",
         hex, hex, hex, hex, hex     (* Unicode code point up to 0xfffff *)
       | "\U0010",
         hex, hex, hex, hex ;        (* Unicode code point between 0x100000 and 0x10ffff *)
```

Octal escape sequences consume up to three octal digits. Additional digits are passed through without escaping. For example, when unescaping the input `\1234`, the parser consumes three octal digits (123) to unescape the byte value 0x83 (ASCII ‘S’) and the subsequent ‘4’ passes through as the byte value 0x34 (ASCII ‘4’). To ensure correct parsing, express octal escape sequences with 3 octal digits, using leading zeros as needed, such as: `\000`, `\001`, `\063`, `\377`. Fewer than three digits are consumed when a non-numeric character follows the numeric characters, such as `\5Hello`.

Hexadecimal escape sequences consume up to two hexadecimal digits. For example, when unescaping `\x213`, the parser consumes only the first two digits (21) to unescape the byte value 0x21 (ASCII ‘!’). To ensure correct parsing, express hexadecimal escape sequences with 2 hexadecimal digits, using leading zeros as needed, such as: `\x00`, `\x01`, `\xFF`. Fewer than two digits are consumed when a non-hexadecimal character follows the numeric character, such as `\xFHello` or `\x3world`.

Use byte-wise escaping only for fields with type `bytes`. While it is possible to use byte-wise escaping in fields with type `string`, those escape sequences are required to form valid UTF-8 sequences. Using byte-wise escaping to express UTF-8 sequences is error-prone. Prefer unicode escape sequences for unprintable characters and line-breaking characters in literals for `string`-type fields.

Longer strings can be broken into several quoted strings on successive lines. For example:

```proto
  quote:
      "When we got into office, the thing that surprised me most was to find "
      "that things were just as bad as we'd been saying they were.\n\n"
      "  -- John F. Kennedy"
```

Unicode code points are interpreted per [Unicode 13 Table A-1 Extended BNF](https://www.unicode.org/versions/Unicode13.0.0/appA.pdf#page=5) and are encoded as UTF-8.

#### Warning

The C++ implementation currently interprets escaped high-surrogate code points as UTF-16 code units, and expects a `\uHHHH` low-surrogate code point to immediately follow, without any split across separate quoted strings. In addition, unpaired surrogates will be rendered directly into also-invalid UTF-8. These are both non-conforming behaviors[^surrogates] and should not be relied on.

## Syntax Elements

### Message

A message is a collection of fields. A text format file is a single Message.

```fallback
Message = { Field } ;
```

### Literals

Field literal values can be numbers, strings, or identifiers such as `true` or enum values.

```fallback
String             = STRING, { STRING } ;
Float              = [ "-" ], FLOAT ;
Identifier         = IDENT ;
SignedIdentifier   = "-", IDENT ;   (* For example, "-inf" *)
DecSignedInteger   = "-", DEC_INT ;
OctSignedInteger   = "-", OCT_INT ;
HexSignedInteger   = "-", HEX_INT ;
DecUnsignedInteger = DEC_INT ;
OctUnsignedInteger = OCT_INT ;
HexUnsignedInteger = HEX_INT ;
```

A single string value can comprise multiple quoted parts separated by optional whitespace. Example:

```fallback
a_string: "first part" 'second part'
          "third part"
no_whitespace: "first""second"'third''fourth'
```

### Field Names

Fields that are part of the containing message use simple `Identifiers` as names. [`Extension`](https://protobuf.dev/programming-guides/proto2#extensions) and [`Any`](https://protobuf.dev/programming-guides/proto3#any) field names are wrapped in square brackets and fully-qualified. `Any` field names are prefixed with a qualifying domain name, such as `type.googleapis.com/`.

```fallback
FieldName     = ExtensionName | AnyName | IDENT ;
ExtensionName = "[", TypeName, "]" ;
AnyName       = "[", Domain, "/", TypeName, "]" ;
TypeName      = IDENT, { ".", IDENT } ;
Domain        = IDENT, { ".", IDENT } ;
```

Regular fields and extension fields can have scalar or message values. `Any` fields are always messages. Example:

```fallback
reg_scalar: 10
reg_message { foo: "bar" }

[com.foo.ext.scalar]: 10
[com.foo.ext.message] { foo: "bar" }

any_value {
  [type.googleapis.com/com.foo.any] { foo: "bar" }
}
```

#### Unknown Fields

Text format parsers cannot support unknown fields represented as raw field numbers in place of field names because three of the six wire types are represented in the same way in textformat. Some text-format serializer implementations encode unknown fields with a format that uses a field number and a numeric representation of the value, but this is inherently lossy because the wire-type information is ignored. For comparison, wire-format is non-lossy because it includes the wire-type in each field tag as `(field_number << 3) | wire_type`. For more information on encoding, see the [Encoding](https://protobuf.dev/programming-guides/encoding.md) topic.

Without information about the field type from the message schema, the value cannot be correctly encoded into a wire-format proto message.

### Fields

Field values can be literals (strings, numbers, or identifiers), or nested messages.

```fallback
Field        = ScalarField | MessageField ;
MessageField = FieldName, [ ":" ], ( MessageValue | MessageList ) [ ";" | "," ];
ScalarField  = FieldName, ":",     ( ScalarValue  | ScalarList  ) [ ";" | "," ];
MessageList  = "[", [ MessageValue, { ",", MessageValue } ], "]" ;
ScalarList   = "[", [ ScalarValue,  { ",", ScalarValue  } ], "]" ;
MessageValue = "{", Message, "}" | "<", Message, ">" ;
ScalarValue  = String
             | Float
             | Identifier
             | SignedIdentifier
             | DecSignedInteger
             | OctSignedInteger
             | HexSignedInteger
             | DecUnsignedInteger
             | OctUnsignedInteger
             | HexUnsignedInteger ;
```

The `:` delimiter between the field name and value is required for scalar fields but optional for message fields (including lists). Example:

```fallback
scalar: 10          # Valid
scalar  10          # Invalid
scalars: [1, 2, 3]  # Valid
scalars  [1, 2, 3]  # Invalid
message: {}         # Valid
message  {}         # Valid
messages: [{}, {}]  # Valid
messages  [{}, {}]  # Valid
```

Values of message fields can be surrounded by curly brackets or angle brackets:

```fallback
message: { foo: "bar" }
message: < foo: "bar" >
```

Fields marked `repeated` can have multiple values specified by repeating the field, using the special `[]` list syntax, or some combination of both. The order of values is maintained. Example:

```fallback
repeated_field: 1
repeated_field: 2
repeated_field: [3, 4, 5]
repeated_field: 6
repeated_field: [7, 8, 9]
```

Non-`repeated` fields cannot use the list syntax. For example, `[0]` is not valid for `optional` or `required` fields. Fields marked `optional` can be omitted or specified once. Fields marked `required` must be specified exactly once.

Fields not specified in the associated *.proto* message are not allowed unless the field name is present in the message’s `reserved` field list. `reserved` fields, if present in any form (scalar, list, message), are simply ignored by text format.

## Value Types

When a field’s associated *.proto* value type is known, the following value descriptions and constraints apply. For the purposes of this section, we declare the following container elements:

```fallback
signedInteger   = DecSignedInteger | OctSignedInteger | HexSignedInteger ;
unsignedInteger = DecUnsignedInteger | OctUnsignedInteger | HexUnsignedInteger ;
integer         = signedInteger | unsignedInteger ;
```

|        **.proto Type**        |                          **Values**                          |
| :---------------------------: | :----------------------------------------------------------: |
|       `float`, `double`       | A `Float`, `DecSignedInteger`, or `DecUnsignedInteger` element, or an `Identifier` or `SignedIdentifier` element whose `IDENT` portion is equal to *"inf"*, *"infinity"*, or *"nan"* (case-insensitive). Overflows are treated as infinity or -infinity. Octal and hexadecimal values are not valid.Note: *"nan"* should be interpreted as [Quiet NaN](https://en.wikipedia.org/wiki/NaN#Quiet_NaN) |
| `int32`, `sint32`, `sfixed32` | Any of the `integer` elements in the range *-0x80000000* to *0x7FFFFFFF*. |
| `int64`, `sint64`, `sfixed64` | Any of the `integer` elements in the range *-0x8000000000000000* to *0x7FFFFFFFFFFFFFFF*. |
|      `uint32`, `fixed32`      | Any of the `unsignedInteger` elements in the range *0* to *0xFFFFFFFF*. Note that signed values (*-0*) are not valid. |
|      `uint64`, `fixed64`      | Any of the `unsignedInteger` elements in the range *0* to *0xFFFFFFFFFFFFFFFF*. Note that signed values (*-0*) are not valid. |
|           `string`            | A `String` element containing valid UTF-8 data. Any escape sequences must form valid UTF-8 byte sequences when unescaped. |
|            `bytes`            | A `String` element, possibly including invalid UTF-8 escape sequences. |
|            `bool`             | An `Identifier` element or any of the `unsignedInteger` elements matching one of the following values. **True values:** *"True"*, *"true"*, *"t"*, *1* **False values:** *"False"*, *"false"*, *"f"*, *0* Any unsigned integer representation of *0* or *1* is permitted: *00*, *0x0*, *01*, *0x1*, etc. |
|         *enum values*         | An `Identifier` element containing an enum value name, or any of the `integer` elements in the range *-0x80000000* to *0x7FFFFFFF* containing an enum value number. It is not valid to specify a name that is not a member of the field's `enum` type definition. Depending on the particular protobuf runtime implementation, it may or may not be valid to specify a number that is not a member of the field's `enum` type definition. Text format processors not tied to a particular runtime implementation (such as IDE support) may choose to issue a warning when a provided number value is not a valid member. Note that certain names that are valid keywords in other contexts, such as *"true"* or *"infinity"*, are also valid enum value names. |
|       *message values*        |                  A `MessageValue` element.                   |

## Extension Fields

Extension fields are specified using their qualified names. Example:

```fallback
local_field: 10
[com.example.ext_field]: 20
```

Extension fields are generally defined in other *.proto* files. The text format language does not provide a mechanism for specifying the locations of files that define extension fields; instead, the parser must have prior knowledge of their locations.

## `Any` Fields

Text format supports an expanded form of the [`google.protobuf.Any`](https://protobuf.dev/programming-guides/proto3#any) well-known type using a special syntax resembling extension fields. Example:

```fallback
local_field: 10

# An Any value using regular fields.
any_value {
  type_url: "type.googleapis.com/com.example.SomeType"
  value: "\x0a\x05hello"  # serialized bytes of com.example.SomeType
}

# The same value using Any expansion
any_value {
  [type.googleapis.com/com.example.SomeType] {
    field1: "hello"
  }
}
```

In this example, `any_value` is a field of type `google.protobuf.Any`, and it stores a serialized `com.example.SomeType` message containing `field1: hello`.

## `group` Fields

In text format, a `group` field uses a normal `MessageValue` element as its value, but is specified using the capitalized group name rather than the implicit lowercased field name. Example:

```proto
message MessageWithGroup {
  optional group MyGroup = 1 {
    optional int32 my_value = 1;
  }
}
```

With the above *.proto* definition, the following text format is a valid `MessageWithGroup`:

```fallback
MyGroup {
  my_value: 1
}
```

Similar to Message fields, the `:` delimiter between the group name and value is optional.

## `map` Fields

Text format does not provide a custom syntax for specifying map field entries. When a [`map`](https://protobuf.dev/programming-guides/proto2#maps) field is defined in a *.proto* file, an implicit `Entry` message is defined containing `key` and `value` fields. Map fields are always repeated, accepting multiple key/value entries. Example:

```proto
message MessageWithMap {
  map<string, int32> my_map = 1;
}
```

With the above *.proto* definition, the following text format is a valid `MessageWithMap`:

```fallback
my_map { key: "entry1" value: 1 }
my_map { key: "entry2" value: 2 }

# You can also use the list syntax
my_map: [
  { key: "entry3" value: 3 },
  { key: "entry4" value: 4 }
]
```

Both the `key` and `value` fields are optional and default to the zero value of their respective types if unspecified. If a key is duplicated, only the last-specified value will be retained in a parsed map.

The order of maps is not maintained in textprotos.

## `oneof` Fields

While there is no special syntax related to `oneof` fields in text format, only one `oneof` member may be specified at a time. Specifying multiple members concurrently is not valid. Example:

```proto
message OneofExample {
  message MessageWithOneof {
    optional string not_part_of_oneof = 1;
    oneof Example {
      string first_oneof_field = 2;
      string second_oneof_field = 3;
    }
  }
  repeated MessageWithOneof message = 1;
}
```

The above *.proto* definition results in the following text format behavior:

```fallback
# Valid: only one field from the Example oneof is set.
message {
  not_part_of_oneof: "always valid"
  first_oneof_field: "valid by itself"
}

# Valid: the other oneof field is set.
message {
  not_part_of_oneof: "always valid"
  second_oneof_field: "valid by itself"
}

# Invalid: multiple fields from the Example oneof are set.
message {
  not_part_of_oneof: "always valid"
  first_oneof_field: "not valid"
  second_oneof_field: "not valid"
}
```

## Text Format Files

A text format file uses the `.txtpb` filename suffix and contains a single `Message`. Text format files are UTF-8 encoded. An example textproto file is provided below.

#### Important

`.txtpb` is the canonical text format file extension and should be preferred to the alternatives. This suffix is preferred for its brevity and consistency with the official wire-format file extension `.binpb`. The legacy canonical extension `.textproto` still has widespread usage and tooling support. Some tooling also supports the legacy extensions `.textpb` and `.pbtxt`. All other extensions besides the above are **strongly** discouraged; in particular, extensions such as `.protoascii` wrongly imply that text format is ascii-only, and others like `.pb.txt` are not recognized by common tooling.

```fallback
# This is an example of Protocol Buffer's text format.
# Unlike .proto files, only shell-style line comments are supported.

name: "John Smith"

pet {
  kind: DOG
  name: "Fluffy"
  tail_wagginess: 0.65f
}

pet <
  kind: LIZARD
  name: "Lizzy"
  legs: 4
>

string_value_with_escape: "valid \n escape"
repeated_values: [ "one", "two", "three" ]
```

### Header

The header comments `proto-file` and `proto-message` inform developer tools of the schema, so they may provide various features.

```fallback
# proto-file: some/proto/my_file.proto
# proto-message: MyMessage
```

## Working with the Format Programmatically

Due to how individual Protocol Buffer implementations emit neither a consistent nor canonical text format, tools or libraries that modify TextProto files or emit TextProto output must explicitly use https://github.com/protocolbuffers/txtpbfmt to format their output.
