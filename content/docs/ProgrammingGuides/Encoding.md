+++
title = "编码"
date = 2024-11-17T09:35:36+08:00
weight = 60
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

> 原文：[https://protobuf.dev/programming-guides/encoding/](https://protobuf.dev/programming-guides/encoding/)
>
> 收录该文档的时间：`2024-11-17T09:35:36+08:00`

# Encoding - 编码

Explains how Protocol Buffers encodes data to files or to the wire.

​	解释 Protocol Buffers 如何将数据编码到文件或网络传输中。

This document describes the protocol buffer *wire format*, which defines the details of how your message is sent on the wire and how much space it consumes on disk. You probably don’t need to understand this to use protocol buffers in your application, but it’s useful information for doing optimizations.

​	本文档描述了 protocol buffer 的*线格式*（wire format），它定义了消息如何通过网络发送以及在磁盘上占用的空间大小。您可能不需要了解这些细节即可在应用程序中使用 protocol buffers，但在进行优化时这些信息会很有用。

If you already know the concepts but want a reference, skip to the [Condensed reference card](https://protobuf.dev/programming-guides/encoding/#cheat-sheet) section.

​	如果您已经了解了相关概念并需要参考，可以跳转到[简化参考卡片](https://protobuf.dev/programming-guides/encoding/#cheat-sheet)部分。

[Protoscope](https://github.com/protocolbuffers/protoscope) is a very simple language for describing snippets of the low-level wire format, which we’ll use to provide a visual reference for the encoding of various messages. Protoscope’s syntax consists of a sequence of *tokens* that each encode down to a specific byte sequence.

​	[Protoscope](https://github.com/protocolbuffers/protoscope) 是一种非常简单的语言，用于描述低级线格式片段。我们将使用它提供各种消息编码的可视化参考。Protoscope 的语法由一系列*标记*组成，每个标记都编码为特定的字节序列。

For example, backticks denote a raw hex literal, like ``70726f746f6275660a``. This encodes into the exact bytes denoted as hex in the literal. Quotes denote UTF-8 strings, like `"Hello, Protobuf!"`. This literal is synonymous with ``48656c6c6f2c2050726f746f62756621`` (which, if you observe closely, is composed of ASCII bytes). We’ll introduce more of the Protoscope language as we discuss aspects of the wire format.

​	例如，反引号表示原始十六进制字面值，例如 `70726f746f6275660a`。它直接编码为十六进制字面值表示的确切字节。引号表示 UTF-8 字符串，例如 `"Hello, Protobuf!"`。此字面值等价于 `48656c6c6f2c2050726f746f62756621`（如果仔细观察，这是由 ASCII 字节组成的）。在讨论线格式的各个方面时，我们将进一步介绍 Protoscope 语言。

The Protoscope tool can also dump encoded protocol buffers as text. See https://github.com/protocolbuffers/protoscope/tree/main/testdata for examples.

​	Protoscope 工具还可以将编码的 protocol buffers 转储为文本。请参阅 https://github.com/protocolbuffers/protoscope/tree/main/testdata 了解示例。

## 一个简单的消息 A Simple Message

Let’s say you have the following very simple message definition:

​	假设您有以下非常简单的消息定义：

```proto
message Test1 {
  optional int32 a = 1;
}
```

In an application, you create a `Test1` message and set `a` to 150. You then serialize the message to an output stream. If you were able to examine the encoded message, you’d see three bytes:

​	在应用程序中，您创建了一个 `Test1` 消息并将 `a` 设置为 150。然后您将该消息序列化为输出流。如果检查编码后的消息，您会看到三个字节：

```proto
08 96 01
```

So far, so small and numeric – but what does it mean? If you use the Protoscope tool to dump those bytes, you’d get something like `1: 150`. How does it know this is the contents of the message?

​	看起来既小又数字化——但这是什么意思呢？如果您使用 Protoscope 工具转储这些字节，您会得到类似 `1: 150` 的结果。那么它是如何知道这是消息的内容的呢？

## 基于 128 的 Varint 编码 Base 128 Varints

Variable-width integers, or *varints*, are at the core of the wire format. They allow encoding unsigned 64-bit integers using anywhere between one and ten bytes, with small values using fewer bytes.

​	可变宽度整数（*varints*）是线格式的核心。它们允许使用 1 到 10 个字节之间的任意字节数来编码无符号 64 位整数，小值使用更少的字节。

Each byte in the varint has a *continuation bit* that indicates if the byte that follows it is part of the varint. This is the *most significant bit* (MSB) of the byte (sometimes also called the *sign bit*). The lower 7 bits are a payload; the resulting integer is built by appending together the 7-bit payloads of its constituent bytes.

​	Varint 中的每个字节都有一个*继续位*（continuation bit），用来指示后续的字节是否仍是 Varint 的一部分。此位是字节的*最高有效位*（MSB，有时也称为*符号位*）。其余的 7 位是有效负载；通过将组成字节的 7 位有效负载依次附加在一起构建出结果整数。

So, for example, here is the number 1, encoded as ``01`` – it’s a single byte, so the MSB is not set:

​	例如，以下是数字 1 的编码形式为 `01`——它是单字节编码，因此 MSB 未设置：

```proto
0000 0001
^ msb
```

And here is 150, encoded as ``9601`` – this is a bit more complicated:

​	而以下是数字 150 的编码形式为 `9601`——稍微复杂一些：

```proto
10010110 00000001
^ msb    ^ msb
```

How do you figure out that this is 150? First you drop the MSB from each byte, as this is just there to tell us whether we’ve reached the end of the number (as you can see, it’s set in the first byte as there is more than one byte in the varint). These 7-bit payloads are in little-endian order. Convert to big-endian order, concatenate, and interpret as an unsigned 64-bit integer:

​	如何确定这表示 150？首先从每个字节中删除 MSB，因为它仅用于告诉我们数字是否结束（正如您所见，第一个字节的 MSB 被设置，因为 Varint 包含多个字节）。这些 7 位有效负载是小端序的。将其转换为大端序，连接起来，并解释为无符号 64 位整数：

```proto
10010110 00000001        // Original inputs. 原始输入。
 0010110  0000001        // Drop continuation bits. 去掉继续位。
 0000001  0010110        // Convert to big-endian. 转换为大端序。
   00000010010110        // Concatenate. 连接。
 128 + 16 + 4 + 2 = 150  // Interpret as an unsigned 64-bit integer. 解释为无符号 64 位整数。
```

Because varints are so crucial to protocol buffers, in protoscope syntax, we refer to them as plain integers. `150` is the same as ``9601``.

​	由于 Varint 对于 protocol buffers 来说至关重要，在 Protoscope 语法中，我们将其称为普通整数。`150` 等价于 `9601`。

## 消息结构 Message Structure

A protocol buffer message is a series of key-value pairs. The binary version of a message just uses the field’s number as the key – the name and declared type for each field can only be determined on the decoding end by referencing the message type’s definition (i.e. the `.proto` file). Protoscope does not have access to this information, so it can only provide the field numbers.

​	Protocol buffer 消息是一系列键值对。消息的二进制版本仅使用字段编号作为键——每个字段的名称和声明类型只能通过引用消息类型的定义（即 `.proto` 文件）在解码端确定。Protoscope 无法访问这些信息，因此它只能提供字段编号。

When a message is encoded, each key-value pair is turned into a *record* consisting of the field number, a wire type and a payload. The wire type tells the parser how big the payload after it is. This allows old parsers to skip over new fields they don’t understand. This type of scheme is sometimes called [Tag-Length-Value](https://en.wikipedia.org/wiki/Type–length–value), or TLV.

​	当消息被编码时，每个键值对都会变成一个*记录*（record），由字段编号、线格式类型和有效负载组成。线格式类型告诉解析器之后的有效负载有多大。这允许旧的解析器跳过它们不理解的新字段。这种方案有时被称为 [标签-长度-值](https://en.wikipedia.org/wiki/Type–length–value)（Tag-Length-Value，TLV）。

There are six wire types: `VARINT`, `I64`, `LEN`, `SGROUP`, `EGROUP`, and `I32`

​	线格式类型有六种：`VARINT`、`I64`、`LEN`、`SGROUP`、`EGROUP` 和 `I32`。

| ID   | Name   | Used For                                                 |
| ---- | ------ | -------------------------------------------------------- |
| 0    | VARINT | int32, int64, uint32, uint64, sint32, sint64, bool, enum |
| 1    | I64    | fixed64, sfixed64, double                                |
| 2    | LEN    | string, bytes, embedded messages, packed repeated fields |
| 3    | SGROUP | group start (deprecated)                                 |
| 4    | EGROUP | group end (deprecated)                                   |
| 5    | I32    | fixed32, sfixed32, float                                 |

The “tag” of a record is encoded as a varint formed from the field number and the wire type via the formula `(field_number << 3) | wire_type`. In other words, after decoding the varint representing a field, the low 3 bits tell us the wire type, and the rest of the integer tells us the field number.

​	记录的“标签”（tag）被编码为一个由字段编号和线格式类型通过公式 `(field_number << 3) | wire_type` 形成的 Varint。换句话说，在解码表示字段的 Varint 后，最低的 3 位告诉我们线格式类型，其余的整数表示字段编号。

Now let’s look at our simple example again. You now know that the first number in the stream is always a varint key, and here it’s ``08``, or (dropping the MSB):

​	现在让我们再次看一下之前的简单示例。您已经知道数据流中的第一个数字始终是一个 Varint 键，这里是 `08`，或者去掉 MSB 后为：

```proto
000 1000
```

You take the last three bits to get the wire type (0) and then right-shift by three to get the field number (1). Protoscope represents a tag as an integer followed by a colon and the wire type, so we can write the above bytes as `1:VARINT`.

​	您取最后三位得到线格式类型（0），然后右移三位得到字段编号（1）。Protoscope 将标签表示为一个整数后跟一个冒号和线格式类型，因此我们可以将上述字节写为 `1:VARINT`。

Because the wire type is 0, or `VARINT`, we know that we need to decode a varint to get the payload. As we saw above, the bytes ``9601`` varint-decode to 150, giving us our record. We can write it in Protoscope as `1:VARINT 150`.

​	由于线格式类型为 0 或 `VARINT`，我们知道需要解码一个 Varint 来获得有效负载。如上所述，字节 `9601` 被解码为 150，从而生成我们的记录。我们可以在 Protoscope 中将其写为 `1:VARINT 150`。

Protoscope can infer the type for a tag if there is whitespace after the `:`. It does so by looking ahead at the next token and guessing what you meant (the rules are documented in detail in [Protoscope’s language.txt](https://github.com/protocolbuffers/protoscope/blob/main/language.txt)). For example, in `1: 150`, there is a varint immediately after the untyped tag, so Protoscope infers its type to be `VARINT`. If you wrote `2: {}`, it would see the `{` and guess `LEN`; if you wrote `3: 5i32` it would guess `I32`, and so on.

​	如果标签后面有空白，Protoscope 可以推断出标签的类型。它通过查看下一个标记并猜测您的意图（规则详见 [Protoscope 的 language.txt](https://github.com/protocolbuffers/protoscope/blob/main/language.txt)）来完成推断。例如，在 `1: 150` 中，未指定类型的标签后面紧跟一个 Varint，因此 Protoscope 推断其类型为 `VARINT`。如果您写的是 `2: {}`，它会看到 `{` 并猜测为 `LEN`；如果您写的是 `3: 5i32`，它会猜测为 `I32`。

## 更多整数类型 More Integer Types

### 布尔值和枚举 Bools and Enums

Bools and enums are both encoded as if they were `int32`s. Bools, in particular, always encode as either ``00`` or ``01``. In Protoscope, `false` and `true` are aliases for these byte strings.

​	布尔值和枚举的编码方式与 `int32` 相同。特别是布尔值，总是编码为 `00` 或 `01`。在 Protoscope 中，`false` 和 `true` 是这些字节串的别名。

### 有符号整数 Signed Integers

As you saw in the previous section, all the protocol buffer types associated with wire type 0 are encoded as varints. However, varints are unsigned, so the different signed types, `sint32` and `sint64` vs `int32` or `int64`, encode negative integers differently.

​	正如前面提到的，所有与线格式类型 0 相关的 protocol buffer 类型都以 Varint 编码。然而，Varint 是无符号的，因此不同的有符号类型（如 `sint32` 和 `sint64` 对比 `int32` 或 `int64`）对负数的编码方式有所不同。

The `intN` types encode negative numbers as two’s complement, which means that, as unsigned, 64-bit integers, they have their highest bit set. As a result, this means that *all ten bytes* must be used. For example, `-2` is converted by protoscope into

​	`intN` 类型使用二进制补码（two’s complement）表示负数。这意味着它们作为无符号 64 位整数时，最高位会被设置。因此，编码时需要使用*全部十个字节*。例如，`-2` 在 Protoscope 中被转换为：

```proto
11111110 11111111 11111111 11111111 11111111
11111111 11111111 11111111 11111111 00000001
```

This is the *two’s complement* of 2, defined in unsigned arithmetic as `~0 - 2 + 1`, where `~0` is the all-ones 64-bit integer. It is a useful exercise to understand why this produces so many ones.

​	这是数字 2 的*二进制补码*，定义为无符号运算中的 `~0 - 2 + 1`，其中 `~0` 是全 1 的 64 位整数。理解为什么这会产生这么多 1 是一个有益的练习。

`sintN` uses the “ZigZag” encoding instead of two’s complement to encode negative integers. Positive integers `p` are encoded as `2 * p` (the even numbers), while negative integers `n` are encoded as `2 * |n| - 1` (the odd numbers). The encoding thus “zig-zags” between positive and negative numbers. For example:

​	`sintN` 类型使用“ZigZag 编码”而非二进制补码来对负数进行编码。正整数 `p` 被编码为 `2 * p`（偶数），而负整数 `n` 被编码为 `2 * |n| - 1`（奇数）。这种编码方式在正负数之间“之字形”变化。例如：

| Signed Original | Encoded As |
| --------------- | ---------- |
| 0               | 0          |
| -1              | 1          |
| 1               | 2          |
| -2              | 3          |
| …               | …          |
| 0x7fffffff      | 0xfffffffe |
| -0x80000000     | 0xffffffff |

In other words, each value `n` is encoded using

​	换句话说，每个值 `n` 被编码为：

```fallback
(n << 1) ^ (n >> 31)
```

for `sint32`s, or

​	（适用于 `sint32`），或

```fallback
(n << 1) ^ (n >> 63)
```

for the 64-bit version.

​	（适用于 `sint64`）。

When the `sint32` or `sint64` is parsed, its value is decoded back to the original, signed version.

​	当解析 `sint32` 或 `sint64` 时，其值会被解码回原始的有符号值。

In protoscope, suffixing an integer with a `z` will make it encode as ZigZag. For example, `-500z` is the same as the varint `999`.

​	在 Protoscope 中，为整数添加后缀 `z` 会使其以 ZigZag 编码。例如，`-500z` 等价于 Varint 编码值 `999`。

### 非 Varint 类型的数字 Non-varint Numbers

Non-varint numeric types are simple – `double` and `fixed64` have wire type `I64`, which tells the parser to expect a fixed eight-byte lump of data. We can specify a `double` record by writing `5: 25.4`, or a `fixed64` record with `6: 200i64`. In both cases, omitting an explicit wire type implies the `I64` wire type.

​	非 Varint 的数值类型比较简单：`double` 和 `fixed64` 使用线格式类型 `I64`，指示解析器应期望接收一个固定的八字节数据块。我们可以通过写 `5: 25.4` 表示一个 `double` 记录，或者用 `6: 200i64` 表示一个 `fixed64` 记录。在这两种情况下，省略显式的线格式类型会默认采用 `I64` 类型。

Similarly `float` and `fixed32` have wire type `I32`, which tells it to expect four bytes instead. The syntax for these consists of adding an `i32` suffix. `25.4i32` will emit four bytes, as will `200i32`. Tag types are inferred as `I32`.

​	同样，`float` 和 `fixed32` 使用线格式类型 `I32`，表示接收四字节的数据块。这些值的语法通过添加 `i32` 后缀实现，例如 `25.4i32` 和 `200i32` 都会产生四字节的数据。标签类型会自动推断为 `I32`。

## 长度限定记录 Length-Delimited Records

*Length prefixes* are another major concept in the wire format. The `LEN` wire type has a dynamic length, specified by a varint immediately after the tag, which is followed by the payload as usual.

​	*长度前缀* 是线格式中的另一个重要概念。线格式类型 `LEN` 的长度是动态的，由标签后紧跟的 Varint 指定，然后按常规处理负载数据。

Consider this message schema:

​	以下是一个消息结构的示例：

```proto
message Test2 {
  optional string b = 2;
}
```

A record for the field `b` is a string, and strings are `LEN`-encoded. If we set `b` to `"testing"`, we encoded as a `LEN` record with field number 2 containing the ASCII string `"testing"`. The result is ``120774657374696e67``. Breaking up the bytes,

​	字段 `b` 是一个字符串，字符串使用 `LEN` 编码。如果我们将 `b` 设置为 `"testing"`，它会被编码为一个带有字段编号 2 的 `LEN` 类型记录，包含 ASCII 字符串 `"testing"`。结果是：`120774657374696e67`。将这些字节分解后为：

```proto
12 07 [74 65 73 74 69 6e 67]
```

we see that the tag, ``12``, is `00010 010`, or `2:LEN`. The byte that follows is the int32 varint `7`, and the next seven bytes are the UTF-8 encoding of `"testing"`. The int32 varint means that the max length of a string is 2GB.

​	可以看到，标签部分是 `12`，即 `00010 010`，表示 `2:LEN`。接下来的字节是 int32 类型的 Varint `7`，后面七个字节是 `"testing"` 的 UTF-8 编码。int32 的 Varint 表示字符串的最大长度为 2GB。

In Protoscope, this is written as `2:LEN 7 "testing"`. However, it can be inconvenient to repeat the length of the string (which, in Protoscope text, is already quote-delimited). Wrapping Protoscope content in braces will generate a length prefix for it: `{"testing"}` is a shorthand for `7 "testing"`. `{}` is always inferred by fields to be a `LEN` record, so we can write this record simply as `2: {"testing"}`.

​	在 Protoscope 中，这段内容可以写为 `2:LEN 7 "testing"`。但是，重复写字符串长度可能会显得麻烦（而且在 Protoscope 文本中，字符串已经由引号分隔）。通过用大括号包裹内容，可以生成长度前缀：`{"testing"}` 是 `7 "testing"` 的简写。对于字段来说，大括号 `{}` 总是被推断为 `LEN` 记录，因此我们可以简单地写成 `2: {"testing"}`。

`bytes` fields are encoded in the same way.

​	`bytes` 类型的字段编码方式与此相同。

### 子消息 Submessages

Submessage fields also use the `LEN` wire type. Here’s a message definition with an embedded message of our original example message, `Test1`:

​	子消息字段同样使用线格式类型 `LEN`。以下是一个嵌套了我们最初示例消息 `Test1` 的消息定义：

```proto
message Test3 {
  optional Test1 c = 3;
}
```

If `Test1`’s `a` field (i.e., `Test3`’s `c.a` field) is set to 150, we get ```1a03089601```. Breaking it up:

​	如果 `Test1` 的字段 `a`（即 `Test3` 的 `c.a` 字段）设置为 150，则会生成 `1a03089601`。将其分解为：

```proto
 1a 03 [08 96 01]
```

The last three bytes (in `[]`) are exactly the same ones from our [very first example](https://protobuf.dev/programming-guides/encoding/#simple). These bytes are preceded by a `LEN`-typed tag, and a length of 3, exactly the same way as strings are encoded.

​	最后三个字节（`[]` 内）正是我们[第一个示例](https://protobuf.dev/programming-guides/encoding/#simple)中的内容。这些字节之前是一个 `LEN` 类型的标签，以及长度 `3`，与字符串的编码方式完全相同。

In Protoscope, submessages are quite succinct. ```1a03089601``` can be written as `3: {1: 150}`.

​	在 Protoscope 中，子消息的表示非常简洁。`1a03089601` 可以写作 `3: {1: 150}`。

## 可选字段和重复字段 Optional and Repeated Elements

Missing `optional` fields are easy to encode: we just leave out the record if it’s not present. This means that “huge” protos with only a few fields set are quite sparse.

​	省略的 `optional` 字段的编码很简单：如果字段不存在，就直接省略对应的记录。这意味着即使消息结构庞大，但只设置了少量字段时，编码仍然非常稀疏。

`repeated` fields are a bit more complicated. Ordinary (not [packed](https://protobuf.dev/programming-guides/encoding/#packed)) repeated fields emit one record for every element of the field. Thus, if we have

​	`repeated` 字段稍微复杂一些。普通的（非 [packed](https://protobuf.dev/programming-guides/encoding/#packed)）`repeated` 字段为字段中的每个元素生成一个记录。例如：

```proto
message Test4 {
  optional string d = 4;
  repeated int32 e = 5;
}
```

and we construct a `Test4` message with `d` set to `"hello"`, and `e` set to `1`, `2`, and `3`, this *could* be encoded as ``220568656c6c6f280128022803``, or written out as Protoscope,

​	如果我们构造一个 `Test4` 消息，将 `d` 设置为 `"hello"`，并将 `e` 设置为 `1`、`2` 和 `3`，其编码结果可能是 `220568656c6c6f280128022803`，在 Protoscope 中表示为：

```proto
4: {"hello"}
5: 1
5: 2
5: 3
```

However, records for `e` do not need to appear consecutively, and can be interleaved with other fields; only the order of records for the same field with respect to each other is preserved. Thus, this could also have been encoded as

​	然而，`e` 的记录并不需要连续出现，可以与其他字段交错；唯一需要保留的顺序是同一字段的记录彼此之间的顺序。因此，这也可以被编码为：

```proto
5: 1
5: 2
4: {"hello"}
5: 3
```

### Oneofs

[`Oneof` fields]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2#oneof" >}}) are encoded the same as if the fields were not in a `oneof`. The rules that apply to `oneofs` are independent of how they are represented on the wire.

​	[`Oneof` 字段]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2#oneof" >}}) 的编码方式与它们不在 `oneof` 中时完全相同。`oneof` 的规则与它们在线格式上的表示方式是独立的。

### 后写覆盖规则 Last One Wins

Normally, an encoded message would never have more than one instance of a non-`repeated` field. However, parsers are expected to handle the case in which they do. For numeric types and strings, if the same field appears multiple times, the parser accepts the *last* value it sees. For embedded message fields, the parser merges multiple instances of the same field, as if with the `Message::MergeFrom` method – that is, all singular scalar fields in the latter instance replace those in the former, singular embedded messages are merged, and `repeated` fields are concatenated. The effect of these rules is that parsing the concatenation of two encoded messages produces exactly the same result as if you had parsed the two messages separately and merged the resulting objects. That is, this:

​	通常情况下，编码消息不会包含多于一个实例的非 `repeated` 字段。然而，解析器需要能够处理这种情况。对于数值类型和字符串类型，如果同一个字段出现多次，解析器会接受它所遇到的*最后一个值*。对于嵌套消息字段，解析器会合并相同字段的多个实例，就像使用 `Message::MergeFrom` 方法一样——即，后一个实例的所有单个标量字段替换前一个实例的对应字段，单个嵌套消息会被合并，而 `repeated` 字段会被拼接。这种规则的作用是：解析两个编码消息的连接体，得到的结果与分别解析两个消息并合并结果对象完全一致。例如，以下代码：

```cpp
MyMessage message;
message.ParseFromString(str1 + str2);
```

is equivalent to this:

​	等价于：

```cpp
MyMessage message, message2;
message.ParseFromString(str1);
message2.ParseFromString(str2);
message.MergeFrom(message2);
```

This property is occasionally useful, as it allows you to merge two messages (by concatenation) even if you do not know their types.

​	这种特性偶尔会非常有用，因为即使不知道消息类型，也可以通过连接合并两个消息。

### 打包的重复字段（ Packed Repeated Fields

Starting in v2.1.0, `repeated` fields of a primitive type (any [scalar type]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2#scalar" >}}) that is not `string` or `bytes`) can be declared as “packed”. In proto2 this is done using the field option `[packed=true]`. In proto3 it is the default.

​	从 v2.1.0 开始，`repeated` 字段的基本类型（任何非 `string` 或 `bytes` 的[标量类型]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2#scalar" >}})）可以声明为“打包”类型。在 proto2 中，这通过字段选项 `[packed=true]` 来实现。在 proto3 中，打包是默认行为。

Instead of being encoded as one record per entry, they are encoded as a single `LEN` record that contains each element concatenated. To decode, elements are decoded from the `LEN` record one by one until the payload is exhausted. The start of the next element is determined by the length of the previous, which itself depends on the type of the field.

​	与为每个条目生成一个记录不同，打包的重复字段会被编码为单个 `LEN` 记录，其中包含所有元素的连接体。解码时，元素会一个接一个地从 `LEN` 记录中解码，直到负载被耗尽。下一个元素的开始位置由上一个元素的长度决定，而该长度取决于字段类型。

For example, imagine you have the message type:

​	例如，假设有以下消息类型：

```proto
message Test5 {
  repeated int32 f = 6 [packed=true];
}
```

Now let’s say you construct a `Test5`, providing the values 3, 270, and 86942 for the repeated field `f`. Encoded, this gives us ``3206038e029ea705``, or as Protoscope text,

​	如果我们构造一个 `Test5`，将重复字段 `f` 设置为值 3、270 和 86942，编码结果是：`3206038e029ea705`，在 Protoscope 文本中表示为：

```proto
6: {3 270 86942}
```

Only repeated fields of primitive numeric types can be declared “packed”. These are types that would normally use the `VARINT`, `I32`, or `I64` wire types.

​	只有基本数值类型的重复字段可以声明为“打包”。这些类型通常使用 `VARINT`、`I32` 或 `I64` 线格式类型。

Note that although there’s usually no reason to encode more than one key-value pair for a packed repeated field, parsers must be prepared to accept multiple key-value pairs. In this case, the payloads should be concatenated. Each pair must contain a whole number of elements. The following is a valid encoding of the same message above that parsers must accept:

​	注意，尽管通常没有理由为打包的重复字段编码多个键值对，但解析器必须能够接受多个键值对的情况。在这种情况下，负载应被拼接。每对必须包含完整数量的元素。以下是解析器必须接受的同一消息的有效编码：

```proto
6: {3 270}
6: {86942}
```

Protocol buffer parsers must be able to parse repeated fields that were compiled as `packed` as if they were not packed, and vice versa. This permits adding `[packed=true]` to existing fields in a forward- and backward-compatible way.

​	协议缓冲区解析器必须能够将被编译为 `packed` 的重复字段解析为未打包格式，反之亦然。这使得在向现有字段添加 `[packed=true]` 时能够保持向前和向后兼容。

### Maps

Map fields are just a shorthand for a special kind of repeated field. If we have

​	映射字段实际上是一种特殊的重复字段的简写。例如：

```proto
message Test6 {
  map<string, int32> g = 7;
}
```

this is actually the same as

​	这实际上等同于：

```proto
message Test6 {
  message g_Entry {
    optional string key = 1;
    optional int32 value = 2;
  }
  repeated g_Entry g = 7;
}
```

Thus, maps are encoded exactly like a `repeated` message field: as a sequence of `LEN`-typed records, with two fields each.

​	因此，映射字段的编码方式与 `repeated` 消息字段完全相同：作为 `LEN` 类型记录的序列，其中每条记录包含两个字段。

## 分组 Groups

Groups are a deprecated feature that should not be used, but they remain in the wire format, and deserve a passing mention.

​	分组是一个已弃用的特性，但它仍然存在于线格式中，因此值得简要提及。

A group is a bit like a submessage, but it is delimited by special tags rather than by a `LEN` prefix. Each group in a message has a field number, which is used on these special tags.

​	分组有点类似子消息，但它通过特殊标签而非 `LEN` 前缀来分隔。在消息中，每个分组都有一个字段编号，用于这些特殊标签。

A group with field number `8` begins with an `8:SGROUP` tag. `SGROUP` records have empty payloads, so all this does is denote the start of the group. Once all the fields in the group are listed, a corresponding `8:EGROUP` tag denotes its end. `EGROUP` records also have no payload, so `8:EGROUP` is the entire record. Group field numbers need to match up. If we encounter `7:EGROUP` where we expect `8:EGROUP`, the message is mal-formed.

​	字段编号为 `8` 的分组以 `8:SGROUP` 标签开始。`SGROUP` 记录的负载为空，它仅表示分组的开始。当分组中的所有字段都列出后，一个对应的 `8:EGROUP` 标签表示分组结束。`EGROUP` 记录同样没有负载，因此 `8:EGROUP` 即为整个记录。分组的字段编号必须匹配。如果在预期 `8:EGROUP` 的地方遇到 `7:EGROUP`，消息即为格式错误。

Protoscope provides a convenient syntax for writing groups. Instead of writing

​	Protoscope 提供了一个便捷的语法来编写分组。例如，以下内容：

```proto
8:SGROUP
  1: 2
  3: {"foo"}
8:EGROUP
```

Protoscope allows

​	可以简写为：

```proto
8: !{
  1: 2
  3: {"foo"}
}
```

This will generate the appropriate start and end group markers. The `!{}` syntax can only occur immediately after an un-typed tag expression, like `8:`.

​	这将生成适当的开始和结束分组标记。`!{}` 语法只能紧跟在未定义类型的标签表达式之后，例如 `8:`。

### 字段顺序 Field Order

Field numbers may be declared in any order in a `.proto` file. The order chosen has no effect on how the messages are serialized.

​	字段编号在 `.proto` 文件中可以按任意顺序声明。其顺序不会影响消息的序列化方式。

When a message is serialized, there is no guaranteed order for how its known or [unknown fields]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2#updating" >}}) will be written. Serialization order is an implementation detail, and the details of any particular implementation may change in the future. Therefore, protocol buffer parsers must be able to parse fields in any order.

​	消息序列化时，其已知或[未知字段]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2#updating" >}})的书写顺序没有保证。序列化顺序是实现细节，任何特定实现的细节可能在未来发生变化。因此，协议缓冲区解析器必须能够按任意顺序解析字段。

### 影响 Implications

- Do not assume the byte output of a serialized message is stable. This is especially true for messages with transitive bytes fields representing other serialized protocol buffer messages.
  - **不要假设序列化消息的字节输出是稳定的。** 特别是当消息包含其他序列化协议缓冲区消息的字节字段时。

- By default, repeated invocations of serialization methods on the same protocol buffer message instance may not produce the same byte output. That is, the default serialization is not deterministic. 默认情况下，对同一个协议缓冲区消息实例的重复序列化方法调用可能不会产生相同的字节输出。即，默认序列化不是确定性的。

  - Deterministic serialization only guarantees the same byte output for a particular binary. The byte output may change across different versions of the binary.
    - 确定性序列化仅保证特定二进制文件的相同字节输出。字节输出可能会因二进制文件版本不同而变化。

- The following checks may fail for a protocol buffer message instance `foo`: 以下对消息实例 `foo` 的检查可能失败：

  - `foo.SerializeAsString() == foo.SerializeAsString()`
  - `Hash(foo.SerializeAsString()) == Hash(foo.SerializeAsString())`
  - `CRC(foo.SerializeAsString()) == CRC(foo.SerializeAsString())`
  - `FingerPrint(foo.SerializeAsString()) == FingerPrint(foo.SerializeAsString())`

- Here are a few example scenarios where logically equivalent protocol buffer messages `foo` and  `bar` may serialize to different byte outputs: 以下场景中，逻辑上等价的消息 `foo` 和 `bar` 可能会序列化为不同的字节输出：

  - `bar` is serialized by an old server that treats some fields as unknown.
    - `bar` 由一个旧服务器序列化，该服务器将某些字段视为未知字段。
  - `bar` is serialized by a server that is implemented in a different programming language and serializes fields in different order.
    - `bar` 由以不同编程语言实现的服务器序列化，字段序列化顺序不同。
  - `bar` has a field that serializes in a non-deterministic manner.
    - `bar` 包含以非确定性方式序列化的字段。
  - `bar` has a field that stores a serialized byte output of a protocol buffer message which is serialized differently.
    - `bar` 包含存储了以不同方式序列化的协议缓冲区消息的字节字段。
  - `bar` is serialized by a new server that serializes fields in a different order due to an implementation change.
    - `bar` 由一个新的服务器序列化，该服务器由于实现更改以不同顺序序列化字段。
  - `foo` and `bar` are concatenations of the same individual messages in a different order.
    - `foo` 和 `bar` 是相同个体消息以不同顺序拼接的结果。

## 编码的 Proto 大小限制 Encoded Proto Size Limitations

Protos must be smaller than 2 GiB when serialized. Many proto implementations will refuse to serialize or parse messages that exceed this limit.

​	序列化的 Proto 消息必须小于 2 GiB。许多 Proto 实现会拒绝序列化或解析超过此限制的消息。

## 精简参考卡片 Condensed Reference Card

The following provides the most prominent parts of the wire format in an easy-to-reference format.

​	以下内容简明地展示了线格式的主要部分，方便参考。

```gdscript3
message    := (tag value)*

tag        := (field << 3) bit-or wire_type;
                encoded as uint32 varint
value      := varint      for wire_type == VARINT,
              i32         for wire_type == I32,
              i64         for wire_type == I64,
              len-prefix  for wire_type == LEN,
              <empty>     for wire_type == SGROUP or EGROUP

varint     := int32 | int64 | uint32 | uint64 | bool | enum | sint32 | sint64;
                encoded as varints (sintN are ZigZag-encoded first)
i32        := sfixed32 | fixed32 | float;
                encoded as 4-byte little-endian;
                memcpy of the equivalent C types (u?int32_t, float)
i64        := sfixed64 | fixed64 | double;
                encoded as 8-byte little-endian;
                memcpy of the equivalent C types (u?int64_t, double)

len-prefix := size (message | string | bytes | packed);
                size encoded as int32 varint
string     := valid UTF-8 string (e.g. ASCII);
                max 2GB of bytes
bytes      := any sequence of 8-bit bytes;
                max 2GB of bytes
packed     := varint* | i32* | i64*,
                consecutive values of the type specified in `.proto`
```

See also the [Protoscope Language Reference](https://github.com/protocolbuffers/protoscope/blob/main/language.txt).

​	更多信息参见 [Protoscope 语言参考](https://github.com/protocolbuffers/protoscope/blob/main/language.txt)。

### Key

- `message := (tag value)*`

  A message is encoded as a sequence of zero or more pairs of tags and values.

  ​	一条消息被编码为由零个或多个标签和值对组成的序列。

- `tag := (field << 3) bit-or wire_type`

  A tag is a combination of a `wire_type`, stored in the least significant three bits, and the field number that is defined in the `.proto` file.

  ​	标签是 `wire_type` 和 `.proto` 文件中定义的字段编号的组合，其中 `wire_type` 存储在最低有效的三位比特中。

- `value := varint for wire_type == VARINT, ...`

  A value is stored differently depending on the `wire_type` specified in the tag.

  ​	根据标签中指定的 `wire_type`，值以不同的方式存储。

- `varint := int32 | int64 | uint32 | uint64 | bool | enum | sint32 | sint64`

  You can use varint to store any of the listed data types.

  ​	可以使用 varint 来存储上述列出的任意数据类型。

- `i32 := sfixed32 | fixed32 | float`

  You can use fixed32 to store any of the listed data types.

  ​	可以使用 fixed32 来存储上述列出的数据类型。

- `i64 := sfixed64 | fixed64 | double`

  You can use fixed64 to store any of the listed data types.

  ​	可以使用 fixed64 来存储上述列出的数据类型。

- `len-prefix := size (message | string | bytes | packed)`

  A length-prefixed value is stored as a length (encoded as a varint), and then one of the listed data types.

  ​	长度前缀值被存储为长度（以 varint 编码）以及其中列出的一种数据类型。

- `string := valid UTF-8 string (e.g. ASCII)`

  As described, a string must use UTF-8 character encoding. A string cannot exceed 2GB.

  ​	如描述，字符串必须使用有效的 UTF-8 字符编码（例如 ASCII）。字符串长度不能超过 2GB。

- `bytes := any sequence of 8-bit bytes`

  As described, bytes can store custom data types, up to 2GB in size.

  ​	如描述，字节可以存储自定义数据类型，大小上限为 2GB。

- `packed := varint* | i32* | i64*`

  Use the `packed` data type when you are storing consecutive values of the type described in the protocol definition. The tag is dropped for values after the first, which amortizes the costs of tags to one per field, rather than per element.
  
  ​	当存储协议定义中描述的连续值时，可使用 `packed` 数据类型。对于第一个值之后的值，标签被省略，从而将标签的开销分摊到每个字段，而不是每个元素。
