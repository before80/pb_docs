+++
title = "文本格式语言规范"
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

# Text Format Language Specification - 文本格式语言规范

The protocol buffer Text Format Language specifies a syntax for representation of protobuf data in text form, which is often useful for configurations or tests.

​	协议缓冲区文本格式语言指定了一种用于以文本形式表示 protobuf 数据的语法，这在配置或测试中通常很有用。

This format is distinct from the format of text within a `.proto` schema, for example. This document contains reference documentation using the syntax specified in [ISO/IEC 14977 EBNF](https://en.wikipedia.org/wiki/Extended_Backus–Naur_Form).

​	此格式与 `.proto` 模式中的文本格式不同。本文档包含参考文档，使用[ISO/IEC 14977 EBNF](https://en.wikipedia.org/wiki/Extended_Backus–Naur_Form)中指定的语法。

> Note
>
> This is a draft spec reverse-engineered from the C++ text format [implementation](https://github.com/protocolbuffers/protobuf/blob/master/src/google/protobuf/text_format.cc) and may change based on further discussion and review. While an effort has been made to keep text formats consistent across supported languages, incompatibilities are likely to exist.
>
> ​	这是从 C++ 文本格式[实现](https://github.com/protocolbuffers/protobuf/blob/master/src/google/protobuf/text_format.cc)中逆向设计的草稿规范，可能会根据进一步的讨论和审查而更改。尽管尽力在受支持的语言之间保持文本格式的一致性，但可能存在不兼容性。

## 示例 Example

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

## 解析概述 Parsing Overview

The language elements in this spec are split into lexical and syntactic categories. Lexical elements must match the input text exactly as described, but syntactic elements may be separated by optional `WHITESPACE` and `COMMENT` tokens.

​	此规范中的语言元素分为词法和句法类别。词法元素必须与输入文本完全匹配，而句法元素可以由可选的 `WHITESPACE` 和 `COMMENT` 标记分隔。

For example, a signed floating point value comprises two syntactic elements: the sign (`-`) and the `FLOAT` literal. Optional whitespace and comments may exist between the sign and the number, but not within the number. Example:

​	例如，一个带符号的浮点值由两个句法元素组成：符号 (`-`) 和 `FLOAT` 字面量。符号和数字之间可以存在可选的空白和注释，但数字内部不能有空白。例如：

```fallback
value: -2.0   # Valid: no additional whitespace. 有效：没有额外的空格。
value: - 2.0  # Valid: whitespace between '-' and '2.0'. 有效：符号 `-` 和 `2.0` 之间有空格。
value: -
  # comment
  2.0         # Valid: whitespace and comments between '-' and '2.0'. 有效：符号 `-` 和 `2.0` 之间有空格和注释。
value: 2 . 0  # Invalid: the floating point period is part of the lexical
              # element, so no additional whitespace is allowed. 无效：浮点数的小数点是词法元素的一部分，因此不能有额外空格。
```

There is one edge case that requires special attention: a number token (`FLOAT`, `DEC_INT`, `OCT_INT`, or `HEX_INT`) may not be immediately followed by an `IDENT` token. Example:

​	有一个需要特别注意的边界情况：数字标记（`FLOAT`、`DEC_INT`、`OCT_INT` 或 `HEX_INT`）后不能紧跟 `IDENT` 标记。例如：

```fallback
foo: 10 bar: 20           # Valid: whitespace separates '10' and 'bar' 有效：空格分隔 `10` 和 `bar`。
foo: 10,bar: 20           # Valid: ',' separates '10' and 'bar' 有效：逗号分隔 `10` 和 `bar`。
foo: 10[com.foo.ext]: 20  # Valid: '10' is followed immediately by '[', which is
                          # not an identifier. 有效：`10` 紧跟着 `[`, 而不是标识符。
foo: 10bar: 20            # Invalid: no space between '10' and identifier 'bar'. 无效：`10` 和标识符 `bar` 之间没有空格。
```

## 词法元素 Lexical Elements

The lexical elements described below fall into two categories: uppercase primary elements and lowercase fragments. Only primary elements are included in the output stream of tokens used during syntactic analysis; fragments exist only to simplify construction of primary elements.

​	以下描述的词法元素分为两类：大写的主要元素和小写的片段。只有主要元素包含在句法分析期间使用的输出标记流中；片段仅用于简化主要元素的构造。

When parsing input text, the longest matching primary element wins. Example:

​	在解析输入文本时，匹配最长的主要元素优先。例如：

```fallback
value: 10   # '10' is parsed as a DEC_INT token. `10` 被解析为 DEC_INT 标记。
value: 10f  # '10f' is parsed as a FLOAT token, despite containing '10' which
            # would also match DEC_INT. In this case, FLOAT matches a longer
            # subsequence of the input. `10f` 被解析为 FLOAT 标记，尽管它包含 `10`，也匹配 DEC_INT。在这种情况下，FLOAT 匹配输入的更长子序列。
```

### 字符 Characters

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

### 空白和注释 Whitespace and Comments

```fallback
COMMENT    = "#", { char - newline }, [ newline ] ;
WHITESPACE = " "
           | newline
           | ? ASCII #9  (horizontal tab) ?
           | ? ASCII #11 (vertical tab) ?
           | ? ASCII #12 (form feed) ?
           | ? ASCII #13 (carriage return) ? ;
```

### 标识符 Identifiers

```fallback
IDENT = letter, { letter | dec } ;
```

### 数字字面量 Numeric Literals

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

​	十进制整数可以通过使用 `F` 和 `f` 后缀转换为浮点值。例如：

```fallback
foo: 10    # This is an integer value. 这是一个整数值。
foo: 10f   # This is a floating-point value. 这是一个浮点值。
foo: 1.0f  # Also optional for floating-point literals. 对浮点字面量同样有效。
```

### 字符串字面量 String Literals

```fallback
STRING = single_string | double_string ;
single_string = "'", { escape | char - "'" - newline - "\" }, "'" ;
double_string = '"', { escape | char - '"' - newline - "\" }, '"' ;

escape = "\a"                        (* ASCII #7  (bell 响铃)                 *)
       | "\b"                        (* ASCII #8  (backspace 退格)            *)
       | "\f"                        (* ASCII #12 (form feed 换页)            *)
       | "\n"                        (* ASCII #10 (line feed 换行)            *)
       | "\r"                        (* ASCII #13 (carriage return 回车)      *)
       | "\t"                        (* ASCII #9  (horizontal tab 水平制表符)       *)
       | "\v"                        (* ASCII #11 (vertical tab 垂直制表符)         *)
       | "\?"                        (* ASCII #63 (question mark 问号)        *)
       | "\\"                        (* ASCII #92 (backslash 反斜杠)            *)
       | "\'"                        (* ASCII #39 (apostrophe 单引号)           *)
       | '\"'                        (* ASCII #34 (quote 双引号)                *)
       | "\", oct, [ oct, [ oct ] ]  (* octal escaped byte value 八进制转义的字节值        *)
       | "\x", hex, [ hex ]          (* hexadecimal escaped byte value 十六进制转义的字节值  *)
       | "\u", hex, hex, hex, hex    (* Unicode code point up to 0xffff 最高至 0xffff 的 Unicode 码点 *)
       | "\U000",
         hex, hex, hex, hex, hex     (* Unicode code point up to 0xfffff 最高至 0xfffff 的 Unicode 码点 *)
       | "\U0010",
         hex, hex, hex, hex ;        (* Unicode code point between 0x100000 and 0x10ffff 范围在 0x100000 至 0x10ffff 的 Unicode 码点 *)
```

Octal escape sequences consume up to three octal digits. Additional digits are passed through without escaping. For example, when unescaping the input `\1234`, the parser consumes three octal digits (123) to unescape the byte value 0x83 (ASCII ‘S’) and the subsequent ‘4’ passes through as the byte value 0x34 (ASCII ‘4’). To ensure correct parsing, express octal escape sequences with 3 octal digits, using leading zeros as needed, such as: `\000`, `\001`, `\063`, `\377`. Fewer than three digits are consumed when a non-numeric character follows the numeric characters, such as `\5Hello`.

​	八进制转义序列最多消耗三个八进制数字。未被消耗的额外数字将直接通过，不进行转义。例如，在解析输入 `\1234` 时，解析器会消耗三个八进制数字（123）并将其转义为字节值 `0x83`（ASCII ‘S’），后续的 `4` 会作为字节值 `0x34`（ASCII ‘4’）传递。为了确保正确解析，请始终以三位八进制数字表示八进制转义序列，例如：`\000`、`\001`、`\063`、`\377`。

Hexadecimal escape sequences consume up to two hexadecimal digits. For example, when unescaping `\x213`, the parser consumes only the first two digits (21) to unescape the byte value 0x21 (ASCII ‘!’). To ensure correct parsing, express hexadecimal escape sequences with 2 hexadecimal digits, using leading zeros as needed, such as: `\x00`, `\x01`, `\xFF`. Fewer than two digits are consumed when a non-hexadecimal character follows the numeric character, such as `\xFHello` or `\x3world`.

​	十六进制转义序列最多消耗两个十六进制数字。例如，在解析 `\x213` 时，解析器仅消耗前两个数字（21），转义为字节值 `0x21`（ASCII ‘!’）。为了确保正确解析，请始终以两位十六进制数字表示十六进制转义序列，例如：`\x00`、`\x01`、`\xFF`。

Use byte-wise escaping only for fields with type `bytes`. While it is possible to use byte-wise escaping in fields with type `string`, those escape sequences are required to form valid UTF-8 sequences. Using byte-wise escaping to express UTF-8 sequences is error-prone. Prefer unicode escape sequences for unprintable characters and line-breaking characters in literals for `string`-type fields.

​	对于类型为 `bytes` 的字段，请仅使用字节级转义。对于类型为 `string` 的字段，尽管可以使用字节级转义，但这些转义序列必须形成有效的 UTF-8 序列。建议对字符串字段中的不可打印字符和换行字符优先使用 Unicode 转义序列。

Longer strings can be broken into several quoted strings on successive lines. For example:

​	较长的字符串可以分解为多行的多个引用字符串。例如：

```proto
  quote:
      "When we got into office, the thing that surprised me most was to find "
      "that things were just as bad as we'd been saying they were.\n\n"
      "  -- John F. Kennedy"
```

Unicode code points are interpreted per [Unicode 13 Table A-1 Extended BNF](https://www.unicode.org/versions/Unicode13.0.0/appA.pdf#page=5) and are encoded as UTF-8.

​	Unicode 代码点按照 [Unicode 13 Table A-1 Extended BNF](https://www.unicode.org/versions/Unicode13.0.0/appA.pdf#page=5) 的规范解释，并编码为 UTF-8。

> Warning
>
> The C++ implementation currently interprets escaped high-surrogate code points as UTF-16 code units, and expects a `\uHHHH` low-surrogate code point to immediately follow, without any split across separate quoted strings. In addition, unpaired surrogates will be rendered directly into also-invalid UTF-8. These are both non-conforming behaviors[^surrogates] and should not be relied on.
>
> ​	C++ 实现当前将转义的高代理代码点解释为 UTF-16 代码单元，并期望紧跟一个 `\uHHHH` 低代理代码点，且不能跨多个引用字符串拆分。此外，未配对的代理将直接呈现为无效的 UTF-8。这些都是不符合规范的行为[^surrogates]，不应依赖。

## 语法元素 Syntax Elements

### 消息 Message

A message is a collection of fields. A text format file is a single Message.

​	消息是字段的集合。一个文本格式文件就是一个消息。

```fallback
Message = { Field } ;
```

### 字面量 Literals

Field literal values can be numbers, strings, or identifiers such as `true` or enum values.

​	字段字面值可以是数字、字符串或标识符（如 `true` 或枚举值）。

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

​	单个字符串值可以由多个带引号的部分组成，这些部分之间可以有可选的空白。例如：

```fallback
a_string: "first part" 'second part'
          "third part"
no_whitespace: "first""second"'third''fourth'
```

### 字段名称 Field Names

Fields that are part of the containing message use simple `Identifiers` as names. [`Extension`]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2#extensions" >}}) and [`Any`]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto3#any" >}}) field names are wrapped in square brackets and fully-qualified. `Any` field names are prefixed with a qualifying domain name, such as `type.googleapis.com/`.

​	包含消息的字段使用简单的 `Identifier` 作为名称。[`扩展字段`]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2#extensions" >}})和[`Any`]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto3#any" >}}) 字段名称用方括号括起并使用全限定名。`Any` 字段名称以限定的域名前缀开头，例如 `type.googleapis.com/`。

```fallback
FieldName     = ExtensionName | AnyName | IDENT ;
ExtensionName = "[", TypeName, "]" ;
AnyName       = "[", Domain, "/", TypeName, "]" ;
TypeName      = IDENT, { ".", IDENT } ;
Domain        = IDENT, { ".", IDENT } ;
```

Regular fields and extension fields can have scalar or message values. `Any` fields are always messages. Example:

​	常规字段和扩展字段可以是标量或消息值。`Any` 字段始终是消息。例如：

```fallback
reg_scalar: 10
reg_message { foo: "bar" }

[com.foo.ext.scalar]: 10
[com.foo.ext.message] { foo: "bar" }

any_value {
  [type.googleapis.com/com.foo.any] { foo: "bar" }
}
```

#### 未知字段 Unknown Fields

Text format parsers cannot support unknown fields represented as raw field numbers in place of field names because three of the six wire types are represented in the same way in textformat. Some text-format serializer implementations encode unknown fields with a format that uses a field number and a numeric representation of the value, but this is inherently lossy because the wire-type information is ignored. For comparison, wire-format is non-lossy because it includes the wire-type in each field tag as `(field_number << 3) | wire_type`. For more information on encoding, see the [Encoding](https://protobuf.dev/programming-guides/encoding.md) topic.

​	文本格式解析器无法支持以原始字段编号表示未知字段，因为文本格式中的三种主要线型使用相同的表示形式。一些文本格式的序列化实现将未知字段编码为使用字段编号和数值表示值的格式，但这是有损的，因为忽略了线型信息。相比之下，线格式是无损的，因为它在每个字段标签中包含线型信息 `(field_number << 3) | wire_type`。有关编码的更多信息，请参见[编码](https://protobuf.dev/programming-guides/encoding.md)主题。

Without information about the field type from the message schema, the value cannot be correctly encoded into a wire-format proto message.

​	如果没有来自消息模式的字段类型信息，则无法正确地将值编码为线格式的 proto 消息。

### 字段 Fields

Field values can be literals (strings, numbers, or identifiers), or nested messages.

​	字段值可以是字面量（字符串、数字或标识符），也可以是嵌套消息。

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

​	标量字段的字段名称和值之间必须有 `:` 分隔符，而消息字段（包括列表）的分隔符是可选的。例如：

```fallback
scalar: 10          # Valid 有效
scalar  10          # Invalid 无效
scalars: [1, 2, 3]  # Valid
scalars  [1, 2, 3]  # Invalid
message: {}         # Valid
message  {}         # Valid
messages: [{}, {}]  # Valid
messages  [{}, {}]  # Valid
```

Values of message fields can be surrounded by curly brackets or angle brackets:

​	消息字段的值可以用大括号或尖括号括起：

```fallback
message: { foo: "bar" }
message: < foo: "bar" >
```

Fields marked `repeated` can have multiple values specified by repeating the field, using the special `[]` list syntax, or some combination of both. The order of values is maintained. Example:

​	`repeated` 字段可以通过重复字段、使用特殊的 `[]` 列表语法或两者的组合来指定多个值。值的顺序会被保留。例如：

```fallback
repeated_field: 1
repeated_field: 2
repeated_field: [3, 4, 5]
repeated_field: 6
repeated_field: [7, 8, 9]
```

Non-`repeated` fields cannot use the list syntax. For example, `[0]` is not valid for `optional` or `required` fields. Fields marked `optional` can be omitted or specified once. Fields marked `required` must be specified exactly once.

​	非 `repeated` 字段不能使用列表语法。例如，`[0]` 对于 `optional` 或 `required` 字段无效。标记为 `optional` 的字段可以省略或指定一次。标记为 `required` 的字段必须仅指定一次。

Fields not specified in the associated *.proto* message are not allowed unless the field name is present in the message’s `reserved` field list. `reserved` fields, if present in any form (scalar, list, message), are simply ignored by text format.

​	未在相关 *.proto* 消息中指定的字段不被允许，除非字段名称在消息的 `reserved` 字段列表中。如果 `reserved` 字段以任何形式出现（标量、列表或消息），文本格式将忽略它们。

## 值类型 Value Types

When a field’s associated *.proto* value type is known, the following value descriptions and constraints apply. For the purposes of this section, we declare the following container elements:

​	当字段的 *.proto* 关联值类型已知时，适用以下值描述和约束。为便于说明，我们声明以下容器元素：

```fallback
signedInteger   = DecSignedInteger | OctSignedInteger | HexSignedInteger ;
unsignedInteger = DecUnsignedInteger | OctUnsignedInteger | HexUnsignedInteger ;
integer         = signedInteger | unsignedInteger ;
```

|        **.proto Type**        |                          **Values**                          |
| :---------------------------: | :----------------------------------------------------------: |
|       `float`, `double`       | 一个 `Float`、`DecSignedInteger` 或 `DecUnsignedInteger` 元素，或者 `Identifier` 或 `SignedIdentifier` 元素，其 `IDENT` 部分等于 *"inf"*、*"infinity"* 或 *"nan"*（不区分大小写）。溢出会被视为正无穷或负无穷。八进制和十六进制值无效。注意：*"nan"* 应解释为 [Quiet NaN](https://en.wikipedia.org/wiki/NaN#Quiet_NaN)。  A `Float`, `DecSignedInteger`, or `DecUnsignedInteger` element, or an `Identifier` or `SignedIdentifier` element whose `IDENT` portion is equal to *"inf"*, *"infinity"*, or *"nan"* (case-insensitive). Overflows are treated as infinity or -infinity. Octal and hexadecimal values are not valid.Note: *"nan"* should be interpreted as [Quiet NaN](https://en.wikipedia.org/wiki/NaN#Quiet_NaN) |
| `int32`, `sint32`, `sfixed32` | 范围为 *-0x80000000* 至 *0x7FFFFFFF* 的任何 `integer` 元素。Any of the `integer` elements in the range *-0x80000000* to *0x7FFFFFFF*. |
| `int64`, `sint64`, `sfixed64` | 范围为 *-0x8000000000000000* 至 *0x7FFFFFFFFFFFFFFF* 的任何 `integer` 元素。 Any of the `integer` elements in the range *-0x8000000000000000* to *0x7FFFFFFFFFFFFFFF*. |
|      `uint32`, `fixed32`      | 范围为 *0* 至 *0xFFFFFFFF* 的任何 `unsignedInteger` 元素。注意，签名值（如 *-0*）无效。 Any of the `unsignedInteger` elements in the range *0* to *0xFFFFFFFF*. Note that signed values (*-0*) are not valid. |
|      `uint64`, `fixed64`      | 范围为 *0* 至 *0xFFFFFFFFFFFFFFFF* 的任何 `unsignedInteger` 元素。注意，签名值（如 *-0*）无效。 Any of the `unsignedInteger` elements in the range *0* to *0xFFFFFFFFFFFFFFFF*. Note that signed values (*-0*) are not valid. |
|           `string`            | 包含有效 UTF-8 数据的 `String` 元素。任何转义序列在解除转义时必须形成有效的 UTF-8 字节序列。 A `String` element containing valid UTF-8 data. Any escape sequences must form valid UTF-8 byte sequences when unescaped. |
|            `bytes`            | 一个 `String` 元素，可能包括无效的 UTF-8 转义序列。 A `String` element, possibly including invalid UTF-8 escape sequences. |
|            `bool`             | 一个 `Identifier` 元素或匹配以下值之一的 `unsignedInteger` 元素。<br/>**True 值：** *"True"*、*"true"*、*"t"*、*1*<br/>**False 值：** *"False"*、*"false"*、*"f"*、*0*<br/>任何 *0* 或 *1* 的无符号整数表示形式都是允许的：*00*、*0x0*、*01*、*0x1* 等。 An `Identifier` element or any of the `unsignedInteger` elements matching one of the following values. **True values:** *"True"*, *"true"*, *"t"*, *1* **False values:** *"False"*, *"false"*, *"f"*, *0* Any unsigned integer representation of *0* or *1* is permitted: *00*, *0x0*, *01*, *0x1*, etc. |
|         *enum values*         | 一个包含枚举值名称的 `Identifier` 元素，或者范围为 *-0x80000000* 到 *0x7FFFFFFF* 的任何 `integer` 元素，其内容为枚举值编号。指定一个不属于字段 `enum` 类型定义成员的名称是无效的。根据具体的 protobuf 运行时实现，可能允许或不允许指定一个不属于字段 `enum` 类型定义成员的编号。与特定运行时实现无关的文本格式处理器（例如 IDE 支持）可以选择在提供的编号值不是有效成员时发出警告。请注意，在其他上下文中有效的某些名称，例如 *"true"* 或 *"infinity"*，在这里也是有效的枚举值名称。 An `Identifier` element containing an enum value name, or any of the `integer` elements in the range *-0x80000000* to *0x7FFFFFFF* containing an enum value number. It is not valid to specify a name that is not a member of the field's `enum` type definition. Depending on the particular protobuf runtime implementation, it may or may not be valid to specify a number that is not a member of the field's `enum` type definition. Text format processors not tied to a particular runtime implementation (such as IDE support) may choose to issue a warning when a provided number value is not a valid member. Note that certain names that are valid keywords in other contexts, such as *"true"* or *"infinity"*, are also valid enum value names. |
|       *message values*        |     一个 `MessageValue` 元素。 A `MessageValue` element.     |

## 扩展字段 Extension Fields

Extension fields are specified using their qualified names. Example:

​	扩展字段使用其限定名称指定。例如：

```fallback
local_field: 10
[com.example.ext_field]: 20
```

Extension fields are generally defined in other *.proto* files. The text format language does not provide a mechanism for specifying the locations of files that define extension fields; instead, the parser must have prior knowledge of their locations.

​	扩展字段通常在其他 *.proto* 文件中定义。文本格式语言不提供指定定义扩展字段的文件位置的机制；相反，解析器必须事先知道这些文件的位置。

## `Any` Fields

Text format supports an expanded form of the [`google.protobuf.Any`]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto3#any" >}}) well-known type using a special syntax resembling extension fields. Example:

​	文本格式支持扩展形式的 [`google.protobuf.Any`]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto3#any" >}}) 类型，使用类似扩展字段的特殊语法。例如：

```fallback
local_field: 10

# An Any value using regular fields.
# 使用常规字段的 Any 值
any_value {
  type_url: "type.googleapis.com/com.example.SomeType"
  value: "\x0a\x05hello"  # serialized bytes of com.example.SomeType
}

# The same value using Any expansion
# 使用 Any 扩展的相同值
any_value {
  [type.googleapis.com/com.example.SomeType] {
    field1: "hello"
  }
}
```

In this example, `any_value` is a field of type `google.protobuf.Any`, and it stores a serialized `com.example.SomeType` message containing `field1: hello`.

​	在此示例中，`any_value` 是类型为 `google.protobuf.Any` 的字段，并存储包含 `field1: hello` 的序列化 `com.example.SomeType` 消息。

## `group` Fields

In text format, a `group` field uses a normal `MessageValue` element as its value, but is specified using the capitalized group name rather than the implicit lowercased field name. Example:

​	在文本格式中，`group` 字段使用常规的 `MessageValue` 元素作为其值，但以大写的组名而非隐式小写字段名称指定。例如：

```proto
message MessageWithGroup {
  optional group MyGroup = 1 {
    optional int32 my_value = 1;
  }
}
```

With the above *.proto* definition, the following text format is a valid `MessageWithGroup`:

​	根据以上 *.proto* 定义，以下文本格式是有效的 `MessageWithGroup`：

```fallback
MyGroup {
  my_value: 1
}
```

Similar to Message fields, the `:` delimiter between the group name and value is optional.

​	与消息字段类似，组名称和值之间的 `:` 分隔符是可选的。

## `map` Fields

Text format does not provide a custom syntax for specifying map field entries. When a [`map`]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2#maps" >}}) field is defined in a *.proto* file, an implicit `Entry` message is defined containing `key` and `value` fields. Map fields are always repeated, accepting multiple key/value entries. Example:

​	文本格式没有提供用于指定映射字段条目的自定义语法。当在 *.proto* 文件中定义了[`map`]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2#maps" >}})字段时，会隐式定义一个包含 `key` 和 `value` 字段的 `Entry` 消息。映射字段始终是 `repeated` 的，可以接受多个键/值条目。例如：

```proto
message MessageWithMap {
  map<string, int32> my_map = 1;
}
```

With the above *.proto* definition, the following text format is a valid `MessageWithMap`:

​	根据以上 *.proto* 定义，以下文本格式是有效的 `MessageWithMap`：

```fallback
my_map { key: "entry1" value: 1 }
my_map { key: "entry2" value: 2 }

# You can also use the list syntax
# 也可以使用列表语法
my_map: [
  { key: "entry3" value: 3 },
  { key: "entry4" value: 4 }
]
```

Both the `key` and `value` fields are optional and default to the zero value of their respective types if unspecified. If a key is duplicated, only the last-specified value will be retained in a parsed map.

​	`key` 和 `value` 字段都是可选的，若未指定则默认为其相应类型的零值。如果键重复，只有最后指定的值会保留在解析后的映射中。

The order of maps is not maintained in textprotos.

​	在文本格式的 `proto` 中，映射的顺序不会保留。

## `oneof` Fields

While there is no special syntax related to `oneof` fields in text format, only one `oneof` member may be specified at a time. Specifying multiple members concurrently is not valid. Example:

​	虽然文本格式中与 `oneof` 字段相关没有特殊语法，但一次只能指定一个 `oneof` 成员。同时指定多个成员是无效的。例如：

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

​	根据以上 *.proto* 定义，以下文本格式行为如下：

```fallback
# Valid: only one field from the Example oneof is set.
# 有效：仅设置了 Example oneof 的一个字段。
message {
  not_part_of_oneof: "always valid"
  first_oneof_field: "valid by itself"
}

# Valid: the other oneof field is set.
# 有效：设置了另一个 oneof 字段。
message {
  not_part_of_oneof: "always valid"
  second_oneof_field: "valid by itself"
}

# Invalid: multiple fields from the Example oneof are set.
# 无效：同时设置了多个 Example oneof 字段。
message {
  not_part_of_oneof: "always valid"
  first_oneof_field: "not valid"
  second_oneof_field: "not valid"
}
```

## 文本格式文件 Text Format Files

A text format file uses the `.txtpb` filename suffix and contains a single `Message`. Text format files are UTF-8 encoded. An example textproto file is provided below.

​	文本格式文件使用 `.txtpb` 文件名后缀，并包含单个 `Message`。文本格式文件是 UTF-8 编码的。以下是一个文本格式的示例文件：

> Important
>
> `.txtpb` is the canonical text format file extension and should be preferred to the alternatives. This suffix is preferred for its brevity and consistency with the official wire-format file extension `.binpb`. The legacy canonical extension `.textproto` still has widespread usage and tooling support. Some tooling also supports the legacy extensions `.textpb` and `.pbtxt`. All other extensions besides the above are **strongly** discouraged; in particular, extensions such as `.protoascii` wrongly imply that text format is ascii-only, and others like `.pb.txt` are not recognized by common tooling.
>
> ​	`.txtpb` 是规范的文本格式文件扩展名，应优先使用。此后缀因其简洁性和与官方线格式扩展名 `.binpb` 的一致性而被优先选择。遗留的规范扩展名 `.textproto` 仍被广泛使用并获得工具支持。一些工具还支持遗留扩展名 `.textpb` 和 `.pbtxt`。强烈建议不要使用以上扩展名以外的其他扩展名，尤其是 `.protoascii`（错误地暗示文本格式仅限于 ASCII）和 `.pb.txt`（不被常见工具识别）。

```fallback
# This is an example of Protocol Buffer's text format.
# Unlike .proto files, only shell-style line comments are supported.
# 这是 Protocol Buffer 的文本格式示例。
# 与 .proto 文件不同，仅支持 shell 风格的行注释。

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

### 文件头 Header

The header comments `proto-file` and `proto-message` inform developer tools of the schema, so they may provide various features.

​	文件头注释 `proto-file` 和 `proto-message` 提供有关模式的信息，以便开发者工具能够提供各种功能。

```fallback
# proto-file: some/proto/my_file.proto
# proto-message: MyMessage
```

## 编程方式使用文本格式 Working with the Format Programmatically

Due to how individual Protocol Buffer implementations emit neither a consistent nor canonical text format, tools or libraries that modify TextProto files or emit TextProto output must explicitly use https://github.com/protocolbuffers/txtpbfmt to format their output.

​	由于各个协议缓冲区实现的文本格式输出不一致，也没有规范化，因此修改 TextProto 文件或生成 TextProto 输出的工具或库必须明确使用 https://github.com/protocolbuffers/txtpbfmt 格式化其输出。
