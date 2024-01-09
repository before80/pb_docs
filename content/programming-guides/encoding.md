+++
title = "Encoding"
weight = 60
description = "说明 Protocol Buffers 如何将数据编码到文件或网络中。"
type = "docs"

+++

## Encoding 编码

Explains how Protocol Buffers encodes data to files or to the wire.

​	说明 Protocol Buffers 如何将数据编码到文件或网络中。



This document describes the protocol buffer *wire format*, which defines the details of how your message is sent on the wire and how much space it consumes on disk. You probably don’t need to understand this to use protocol buffers in your application, but it’s useful information for doing optimizations.

​	此文档介绍协议缓冲区有线格式，该格式定义了您的消息在有线上发送的方式的详细信息以及它在磁盘上占用的空间。您可能不需要了解这一点即可在您的应用程序中使用协议缓冲区，但这是进行优化的有用信息。

If you already know the concepts but want a reference, skip to the [Condensed reference card](https://protobuf.dev/programming-guides/encoding/#cheat-sheet) section.

​	如果您已经了解这些概念但想要一个参考，请跳至简明参考卡部分。

[Protoscope](https://github.com/protocolbuffers/protoscope) is a very simple language for describing snippets of the low-level wire format, which we’ll use to provide a visual reference for the encoding of various messages. Protoscope’s syntax consists of a sequence of *tokens* that each encode down to a specific byte sequence.

​	Protoscope 是一种非常简单的语言，用于描述低级有线格式的片段，我们将使用它为各种消息的编码提供可视参考。Protoscope 的语法由一系列标记组成，每个标记都编码为特定的字节序列。

For example, backticks denote a raw hex literal, like ``70726f746f6275660a``. This encodes into the exact bytes denoted as hex in the literal. Quotes denote UTF-8 strings, like `"Hello, Protobuf!"`. This literal is synonymous with ``48656c6c6f2c2050726f746f62756621`` (which, if you observe closely, is composed of ASCII bytes). We’ll introduce more of the Protoscope language as we discuss aspects of the wire format.

​	例如，反引号表示原始十六进制文字，如 ``70726f746f6275660a`` 。这编码为文字中以十六进制表示的确切字节。引号表示 UTF-8 字符串，如 `"Hello, Protobuf!"` 。此文字与 ``48656c6c6f2c2050726f746f62756621`` 同义（如果您仔细观察，它由 ASCII 字节组成）。当我们讨论有线格式的各个方面时，我们将介绍更多 Protoscope 语言。

The Protoscope tool can also dump encoded protocol buffers as text. See https://github.com/protocolbuffers/protoscope/tree/main/testdata for examples.

​	Protoscope 工具还可以将编码的协议缓冲区转储为文本。有关示例，请参阅 https://github.com/protocolbuffers/protoscope/tree/main/testdata。

## 一个简单的消息 A Simple Message 

Let’s say you have the following very simple message definition:

​	假设您有以下非常简单的消息定义：

```proto
message Test1 {
  optional int32 a = 1;
}
```

In an application, you create a `Test1` message and set `a` to 150. You then serialize the message to an output stream. If you were able to examine the encoded message, you’d see three bytes:

​	在应用程序中，您创建一个 `Test1` 消息并将 `a` 设置为 150。然后，您将消息序列化到输出流。如果您能够检查编码的消息，您会看到三个字节：

```proto
08 96 01
```

So far, so small and numeric – but what does it mean? If you use the Protoscope tool to dump those bytes, you’d get something like `1: 150`. How does it know this is the contents of the message?

​	到目前为止，都很小且是数字 - 但它意味着什么？如果您使用 Protoscope 工具转储这些字节，您会得到类似 `1: 150` 的内容。它如何知道这是消息的内容？

## 基本 128 变体 Base 128 Varints 

Variable-width integers, or *varints*, are at the core of the wire format. They allow encoding unsigned 64-bit integers using anywhere between one and ten bytes, with small values using fewer bytes.

​	可变宽整数或变体是线格式的核心。它们允许使用一个到十个字节对无符号 64 位整数进行编码，而较小的值使用较少的字节。

Each byte in the varint has a *continuation bit* that indicates if the byte that follows it is part of the varint. This is the *most significant bit* (MSB) of the byte (sometimes also called the *sign bit*). The lower 7 bits are a payload; the resulting integer is built by appending together the 7-bit payloads of its constituent bytes.

​	变体中的每个字节都有一个延续位，指示其后的字节是否属于变体。这是字节的最重要位 (MSB)（有时也称为符号位）。较低的 7 位是有效负载；结果整数通过将组成字节的 7 位有效负载附加在一起而构建。

So, for example, here is the number 1, encoded as ``01`` – it’s a single byte, so the MSB is not set:

​	因此，例如，这里有数字 1，编码为 ``01`` – 它是一个字节，因此未设置 MSB：

```proto
0000 0001
^ msb
```

And here is 150, encoded as ``9601`` – this is a bit more complicated:

​	这里有 150，编码为 ``9601`` – 这有点复杂：

```proto
10010110 00000001
^ msb    ^ msb
```

How do you figure out that this is 150? First you drop the MSB from each byte, as this is just there to tell us whether we’ve reached the end of the number (as you can see, it’s set in the first byte as there is more than one byte in the varint). These 7-bit payloads are in little-endian order. Convert to big-endian order, concatenate, and interpret as an unsigned 64-bit integer:

​	您如何计算出这是 150？首先，您从每个字节中删除 MSB，因为这只是为了告诉我们是否已达到数字的末尾（如您所见，它在第一个字节中设置，因为 varint 中有多个字节）。这些 7 位有效负载按小端顺序排列。转换为大端顺序，连接并解释为无符号 64 位整数：

```proto
10010110 00000001        // Original inputs.
 0010110  0000001        // Drop continuation bits.
 0000001  0010110        // Convert to big-endian.
   00000010010110        // Concatenate.
 128 + 16 + 4 + 2 = 150  // Interpret as an unsigned 64-bit integer.
```

Because varints are so crucial to protocol buffers, in protoscope syntax, we refer to them as plain integers. `150` is the same as ``9601``.

​	由于 varint 对协议缓冲区非常重要，因此在 protoscope 语法中，我们将它们称为普通整数。 `150` 与 ``9601`` 相同。

## 消息结构 Message Structure 

A protocol buffer message is a series of key-value pairs. The binary version of a message just uses the field’s number as the key – the name and declared type for each field can only be determined on the decoding end by referencing the message type’s definition (i.e. the `.proto` file). Protoscope does not have access to this information, so it can only provide the field numbers.

​	协议缓冲区消息是一系列键值对。消息的二进制版本仅使用字段的编号作为键 – 每个字段的名称和声明的类型只能在解码端通过引用消息类型的定义（即 `.proto` 文件）来确定。Protoscope 无权访问此信息，因此它只能提供字段编号。

When a message is encoded, each key-value pair is turned into a *record* consisting of the field number, a wire type and a payload. The wire type tells the parser how big the payload after it is. This allows old parsers to skip over new fields they don’t understand. This type of scheme is sometimes called [Tag-Length-Value](https://en.wikipedia.org/wiki/Type–length–value), or TLV.

​	当对消息进行编码时，每个键值对都会变成一条记录，该记录由字段编号、一个线型和一个有效负载组成。线型告诉解析器其后的有效负载有多大。这允许旧解析器跳过它们不理解的新字段。这种类型的方案有时称为标记长度值或 TLV。

There are six wire types: `VARINT`, `I64`, `LEN`, `SGROUP`, `EGROUP`, and `I32`
有六种线型： `VARINT` 、 `I64` 、 `LEN` 、 `SGROUP` 、 `EGROUP` 和 `I32`

| ID   | Name 名称 | Used For 用途                                                |
| ---- | --------- | ------------------------------------------------------------ |
| 0    | VARINT    | int32, int64, uint32, uint64, sint32, sint64, bool, enum int32、int64、uint32、uint64、sint32、sint64、bool、enum |
| 1    | I64       | fixed64, sfixed64, double fixed64、sfixed64、double          |
| 2    | LEN       | string, bytes, embedded messages, packed repeated fields string、bytes、嵌入式消息、打包重复字段 |
| 3    | SGROUP    | group start (deprecated) 组开始（已弃用）                    |
| 4    | EGROUP    | group end (deprecated) 组结束（已弃用）                      |
| 5    | I32       | fixed32, sfixed32, float fixed32、sfixed32、float            |

The “tag” of a record is encoded as a varint formed from the field number and the wire type via the formula `(field_number << 3) | wire_type`. In other words, after decoding the varint representing a field, the low 3 bits tell us the wire type, and the rest of the integer tells us the field number.

​	记录的“标记”编码为通过公式 `(field_number << 3) | wire_type` 从字段编号和数据线类型形成的变长整数。换句话说，在解码表示字段的变长整数后，低 3 位告诉我们数据线类型，整数的其余部分告诉我们字段编号。

Now let’s look at our simple example again. You now know that the first number in the stream is always a varint key, and here it’s ``08``, or (dropping the MSB):

​	现在，我们再次看看我们的简单示例。您现在知道流中的第一个数字始终是变长整数键，此处为 ``08`` ，或（去掉 MSB）：

```proto
000 1000
```

You take the last three bits to get the wire type (0) and then right-shift by three to get the field number (1). Protoscope represents a tag as an integer followed by a colon and the wire type, so we can write the above bytes as `1:VARINT`.

​	获取最后三位以获取数据线类型 (0)，然后右移三位以获取字段编号 (1)。Protoscope 将标记表示为一个整数，后跟一个冒号和数据线类型，因此我们可以将上述字节写为 `1:VARINT` 。

Because the wire type is 0, or `VARINT`, we know that we need to decode a varint to get the payload. As we saw above, the bytes ``9601`` varint-decode to 150, giving us our record. We can write it in Protoscope as `1:VARINT 150`.

​	由于线类型为 0，或 `VARINT` ，我们知道我们需要解码一个变体来获取有效负载。如上所述，字节 ``9601`` 变体解码为 150，从而为我们提供了记录。我们可以用 Protoscope 将其写成 `1:VARINT 150` 。

Protoscope can infer the type for a tag if there is whitespace after the `:`. It does so by looking ahead at the next token and guessing what you meant (the rules are documented in detail in [Protoscope’s language.txt](https://github.com/protocolbuffers/protoscope/blob/main/language.txt)). For example, in `1: 150`, there is a varint immediately after the untyped tag, so Protoscope infers its type to be `VARINT`. If you wrote `2: {}`, it would see the `{` and guess `LEN`; if you wrote `3: 5i32` it would guess `I32`, and so on.

​	如果 `:` 后有空格，Protoscope 可以推断标签的类型。它通过查看下一个标记并猜测您的意思来做到这一点（规则在 Protoscope 的 language.txt 中有详细说明）。例如，在 `1: 150` 中，未类型化标签后紧跟一个变体，因此 Protoscope 推断其类型为 `VARINT` 。如果您编写 `2: {}` ，它会看到 `{` 并猜测 `LEN` ；如果您编写 `3: 5i32` ，它会猜测 `I32` ，依此类推。

## 更多整数类型 More Integer Types 

### 布尔值和枚举 Bools and Enums 

Bools and enums are both encoded as if they were `int32`s. Bools, in particular, always encode as either ``00`` or ``01``. In Protoscope, `false` and `true` are aliases for these byte strings.

​	布尔值和枚举都编码为 `int32` 。特别是布尔值始终编码为 ``00`` 或 ``01`` 。在 Protoscope 中， `false` 和 `true` 是这些字节字符串的别名。

### 有符号整数 Signed Integers 

As you saw in the previous section, all the protocol buffer types associated with wire type 0 are encoded as varints. However, varints are unsigned, so the different signed types, `sint32` and `sint64` vs `int32` or `int64`, encode negative integers differently.

​	如您在上一节中所见，所有与线类型 0 关联的协议缓冲区类型都编码为可变整数。但是，可变整数是无符号的，因此不同的有符号类型 `sint32` 和 `sint64` 与 `int32` 或 `int64` 编码负整数的方式不同。

The `intN` types encode negative numbers as two’s complement, which means that, as unsigned, 64-bit integers, they have their highest bit set. As a result, this means that *all ten bytes* must be used. For example, `-2` is converted by protoscope into

​	`intN` 类型将负数编码为二进制补码，这意味着作为无符号的 64 位整数，它们设置了最高位。因此，这意味着必须使用全部十个字节。例如，protoscope 将 `-2` 转换为

```proto
11111110 11111111 11111111 11111111 11111111
11111111 11111111 11111111 11111111 00000001
```

This is the *two’s complement* of 2, defined in unsigned arithmetic as `~0 - 2 + 1`, where `~0` is the all-ones 64-bit integer. It is a useful exercise to understand why this produces so many ones.

​	这是 2 的二进制补码，在无符号算术中定义为 `~0 - 2 + 1` ，其中 `~0` 是全 1 的 64 位整数。理解为什么这会产生如此多的 1 是一个有用的练习。

`sintN` uses the “ZigZag” encoding instead of two’s complement to encode negative integers. Positive integers `p` are encoded as `2 * p` (the even numbers), while negative integers `n` are encoded as `2 * |n| - 1` (the odd numbers). The encoding thus “zig-zags” between positive and negative numbers. For example:

​	`sintN` 使用“ZigZag”编码而不是二进制补码来编码负整数。正整数 `p` 编码为 `2 * p` （偶数），而负整数 `n` 编码为 `2 * |n| - 1` （奇数）。因此，编码在正数和负数之间“交替”。例如：

| Signed Original 有符号原值 | Encoded As 编码为 |
| -------------------------- | ----------------- |
| 0                          | 0                 |
| -1                         | 1                 |
| 1                          | 2                 |
| -2                         | 3                 |
| …                          | …                 |
| 0x7fffffff                 | 0xfffffffe        |
| -0x80000000                | 0xffffffff        |

In other words, each value `n` is encoded using

​	换句话说，每个值 `n` 使用

```fallback
(n << 1) ^ (n >> 31)
```

for `sint32`s, or

​	对 `sint32` 进行编码，或

```fallback
(n << 1) ^ (n >> 63)
```

for the 64-bit version.

​	对 64 位版本进行编码。

When the `sint32` or `sint64` is parsed, its value is decoded back to the original, signed version.

​	解析 `sint32` 或 `sint64` 时，其值会解码回原始的带符号版本。

In protoscope, suffixing an integer with a `z` will make it encode as ZigZag. For example, `-500z` is the same as the varint `999`.

​	在 protoscope 中，用 `z` 为整数添加后缀会使其编码为 ZigZag。例如， `-500z` 与变量 `999` 相同。

### 非变量数字 Non-varint Numbers 

Non-varint numeric types are simple – `double` and `fixed64` have wire type `I64`, which tells the parser to expect a fixed eight-byte lump of data. We can specify a `double` record by writing `5: 25.4`, or a `fixed64` record with `6: 200i64`. In both cases, omitting an explicit wire type implies the `I64` wire type.

​	非变量数字类型很简单—— `double` 和 `fixed64` 具有数据线类型 `I64` ，它告诉解析器期望一个固定八字节的数据块。我们可以通过编写 `5: 25.4` 来指定 `double` 记录，或通过 `6: 200i64` 来指定 `fixed64` 记录。在这两种情况下，省略显式数据线类型都意味着 `I64` 数据线类型。

Similarly `float` and `fixed32` have wire type `I32`, which tells it to expect four bytes instead. The syntax for these consists of adding an `i32` prefix. `25.4i32` will emit four bytes, as will `200i32`. Tag types are inferred as `I32`.

​	类似地， `float` 和 `fixed32` 具有数据线类型 `I32` ，它告诉解析器期望四个字节。这些的语法包括添加 `i32` 前缀。 `25.4i32` 将发出四个字节， `200i32` 也是如此。标签类型推断为 `I32` 。

## 长度定界记录 Length-Delimited Records 

*Length prefixes* are another major concept in the wire format. The `LEN` wire type has a dynamic length, specified by a varint immediately after the tag, which is followed by the payload as usual.

​	长度前缀是数据线格式中的另一个主要概念。 `LEN` 数据线类型具有动态长度，由标签后紧跟的变量指定，然后照常是有效载荷。

Consider this message schema:

​	考虑此消息模式：

```proto
message Test2 {
  optional string b = 2;
}
```

A record for the field `b` is a string, and strings are `LEN`-encoded. If we set `b` to `"testing"`, we encoded as a `LEN` record with field number 2 containing the ASCII string `"testing"`. The result is ``120774657374696e67``. Breaking up the bytes,

​	字段 `b` 的记录是一个字符串，字符串是 `LEN` 编码的。如果我们将 `b` 设置为 `"testing"` ，我们编码为 `LEN` 记录，其中字段编号 2 包含 ASCII 字符串 `"testing"` 。结果是 ``120774657374696e67`` 。分解字节，

```proto
12 07 [74 65 73 74 69 6e 67]
```

we see that the tag, ``12``, is `00010 010`, or `2:LEN`. The byte that follows is the int32 varint `7`, and the next seven bytes are the UTF-8 encoding of `"testing"`. The int32 varint means that the max length of a string is 2GB.

​	我们看到标签 ``12`` 是 `00010 010` 或 `2:LEN` 。后面的字节是 int32 varint `7` ，接下来的七个字节是 `"testing"` 的 UTF-8 编码。int32 varint 表示字符串的最大长度为 2GB。

In Protoscope, this is written as `2:LEN 7 "testing"`. However, it can be incovenient to repeat the length of the string (which, in Protoscope text, is already quote-delimited). Wrapping Protoscope content in braces will generate a length prefix for it: `{"testing"}` is a shorthand for `7 "testing"`. `{}` is always inferred by fields to be a `LEN` record, so we can write this record simply as `2: {"testing"}`.

​	在 Protoscope 中，这写为 `2:LEN 7 "testing"` 。但是，重复字符串的长度（在 Protoscope 文本中，已经用引号分隔）可能不方便。用大括号包装 Protoscope 内容将为其生成长度前缀： `{"testing"}` 是 `7 "testing"` 的简写。 `{}` 始终由字段推断为 `LEN` 记录，因此我们可以简单地将此记录写为 `2: {"testing"}` 。

`bytes` fields are encoded in the same way.

​	`bytes` 字段以相同的方式编码。

### 子消息 Submessages 

Submessage fields also use the `LEN` wire type. Here’s a message definition with an embedded message of our original example message, `Test1`:

​	子消息字段也使用 `LEN` 线路类型。下面是带有我们原始示例消息的嵌入消息的消息定义， `Test1` ：

```proto
message Test3 {
  optional Test1 c = 3;
}
```

If `Test1`’s `a` field (i.e., `Test3`’s `c.a` field) is set to 150, we get ```1a03089601```. Breaking it up:

​	如果 `Test1` 的 `a` 字段（即 `Test3` 的 `c.a` 字段）设置为 150，我们得到 ```1a03089601``` 。分解如下：

```proto
 1a 03 [08 96 01]
```

The last three bytes (in `[]`) are exactly the same ones from our [very first example](https://protobuf.dev/programming-guides/encoding/#simple). These bytes are preceded by a `LEN`-typed tag, and a length of 3, exactly the same way as strings are encoded.

​	最后三个字节（在 `[]` 中）与我们第一个示例中的完全相同。这些字节之前是 `LEN` 类型标记，长度为 3，与字符串的编码方式完全相同。

In Protoscope, submessages are quite succinct. ```1a03089601``` can be written as `3: {1: 150}`.

​	在 Protoscope 中，子消息非常简洁。 ```1a03089601``` 可以写成 `3: {1: 150}` 。

## 可选和重复元素 Optional and Repeated Elements 

Missing `optional` fields are easy to encode: we just leave out the record if it’s not present. This means that “huge” protos with only a few fields set are quite sparse.

​	缺失的 `optional` 字段很容易编码：如果不存在，我们只需省略记录。这意味着仅设置了几个字段的“巨大”protos 非常稀疏。

`repeated` fields are a bit more complicated. Ordinary (not [packed](https://protobuf.dev/programming-guides/encoding/#packed)) repeated fields emit one record for every element of the field. Thus, if we have

​	`repeated` 字段稍微复杂一些。普通的（未打包的）重复字段为字段的每个元素发出一个记录。因此，如果我们有

```proto
message Test4 {
  optional string d = 4;
  repeated int32 e = 5;
}
```

and we construct a `Test4` message with `d` set to `"hello"`, and `e` set to `1`, `2`, and `3`, this *could* be encoded as ``220568656c6c6f280128022803``, or written out as Protoscope,

​	并且我们构造了一个 `Test4` 消息，其中 `d` 设置为 `"hello"` ， `e` 设置为 `1` 、 `2` 和 `3` ，这可以编码为 ``220568656c6c6f280128022803`` ，或写成 Protoscope，

```proto
4: {"hello"}
5: 1
5: 2
5: 3
```

However, records for `e` do not need to appear consecutively, and can be interleaved with other fields; only the order of records for the same field with respect to each other is preserved. Thus, this could also have been encoded as

​	但是， `e` 的记录不必连续出现，可以与其他字段交错；仅保留同一字段的记录彼此之间的顺序。因此，也可以将其编码为

```proto
5: 1
5: 2
4: {"hello"}
5: 3
```

### Oneofs

[`Oneof` fields](https://protobuf.dev/programming-guides/proto2#oneof) are encoded the same as if the fields were not in a `oneof`. The rules that apply to `oneofs` are independent of how they are represented on the wire.

​	`Oneof` 字段的编码方式与字段不在 `oneof` 中的编码方式相同。适用于 `oneofs` 的规则与它们在数据线上的表示方式无关。

### 后一个获胜 Last One Wins 

Normally, an encoded message would never have more than one instance of a non-`repeated` field. However, parsers are expected to handle the case in which they do. For numeric types and strings, if the same field appears multiple times, the parser accepts the *last* value it sees. For embedded message fields, the parser merges multiple instances of the same field, as if with the `Message::MergeFrom` method – that is, all singular scalar fields in the latter instance replace those in the former, singular embedded messages are merged, and `repeated` fields are concatenated. The effect of these rules is that parsing the concatenation of two encoded messages produces exactly the same result as if you had parsed the two messages separately and merged the resulting objects. That is, this:

​	通常，编码消息绝不会有多个非 `repeated` 字段的实例。但是，解析器应处理出现这种情况的情况。对于数字类型和字符串，如果同一个字段出现多次，解析器将接受它看到的最后一个值。对于嵌入式消息字段，解析器将合并同一字段的多个实例，就像使用 `Message::MergeFrom` 方法一样——即，后一个实例中的所有单数标量字段都将替换前一个实例中的字段，单数嵌入式消息将被合并， `repeated` 字段将被连接。这些规则的效果是，解析两个编码消息的连接产生的结果与您分别解析这两个消息并合并结果对象产生的结果完全相同。也就是说，这：

```cpp
MyMessage message;
message.ParseFromString(str1 + str2);
```

is equivalent to this:

​	等同于这：

```cpp
MyMessage message, message2;
message.ParseFromString(str1);
message2.ParseFromString(str2);
message.MergeFrom(message2);
```

This property is occasionally useful, as it allows you to merge two messages (by concatenation) even if you do not know their types.

​	此属性偶尔有用，因为它允许您合并两个消息（通过连接），即使您不知道它们的类型。

### 打包重复字段 Packed Repeated Fields 

Starting in v2.1.0, `repeated` fields of a primitive type (any [scalar type](https://protobuf.dev/programming-guides/proto2#scalar) that is not `string` or `bytes`) can be declared as “packed”. In proto2 this is done using the field option `[packed=true]`. In proto3 it is the default.

​	从 v2.1.0 开始， `repeated` 类型的字段（任何不是 `string` 或 `bytes` 的标量类型）可以声明为“packed”。在 proto2 中，这是使用字段选项 `[packed=true]` 来完成的。在 proto3 中，这是默认值。

Instead of being encoded as one record per entry, they are encoded as a single `LEN` record that contains each element concatenated. To decode, elements are decoded from the `LEN` record one by one until the payload is exhausted. The start of the next element is determined by the length of the previous, which itself depends on the type of the field.

​	它们不会被编码为每个条目一个记录，而是被编码为一个包含每个连接元素的单个 `LEN` 记录。要解码，元素会从 `LEN` 记录中逐个解码，直到有效负载用尽。下一个元素的开始由前一个元素的长度决定，而前一个元素的长度又取决于字段的类型。

For example, imagine you have the message type:

​	例如，假设您有消息类型：

```proto
message Test5 {
  repeated int32 f = 6 [packed=true];
}
```

Now let’s say you construct a `Test5`, providing the values 3, 270, and 86942 for the repeated field `f`. Encoded, this gives us ``3206038e029ea705``, or as Protoscope text,

​	现在假设您构造一个 `Test5` ，为重复字段 `f` 提供值 3、270 和 86942。编码后，这会给我们 ``3206038e029ea705`` ，或者作为 Protoscope 文本，

```proto
6: {3 270 86942}
```

Only repeated fields of primitive numeric types can be declared “packed”. These are types that would normally use the `VARINT`, `I32`, or `I64` wire types.

​	只有原始数字类型的重复字段才能声明为“packed”。这些类型通常会使用 `VARINT` 、 `I32` 或 `I64` 线路类型。

Note that although there’s usually no reason to encode more than one key-value pair for a packed repeated field, parsers must be prepared to accept multiple key-value pairs. In this case, the payloads should be concatenated. Each pair must contain a whole number of elements. The following is a valid encoding of the same message above that parsers must accept:

​	请注意，虽然通常没有理由为打包的重复字段编码多个键值对，但解析器必须准备好接受多个键值对。在这种情况下，应连接有效负载。每个对必须包含一定数量的元素。以下是解析器必须接受的上述相同消息的有效编码：

```proto
6: {3 270}
6: {86942}
```

Protocol buffer parsers must be able to parse repeated fields that were compiled as `packed` as if they were not packed, and vice versa. This permits adding `[packed=true]` to existing fields in a forward- and backward-compatible way.

​	协议缓冲区解析器必须能够解析已编译为 `packed` 的重复字段，就好像它们未打包一样，反之亦然。这允许以向前和向后兼容的方式将 `[packed=true]` 添加到现有字段。

### 映射 Maps 

Map fields are just a shorthand for a special kind of repeated field. If we have

​	映射字段只是特殊类型的重复字段的简写。如果我们有

```proto
message Test6 {
  map<string, int32> g = 7;
}
```

this is actually the same as

​	这实际上与

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

​	相同。因此，映射的编码方式与 `repeated` 消息字段完全相同：作为一系列 `LEN` -类型的记录，每个记录有两个字段。

## 组 Groups 

Groups are a deprecated feature that should not be used, but they remain in the wire format, and deserve a passing mention.

​	组是一个不推荐使用的已弃用功能，但不应在有线格式中使用，值得一提。

A group is a bit like a submessage, but it is delimited by special tags rather than by a `LEN` prefix. Each group in a message has a field number, which is used on these special tags.

​	组有点像子消息，但它由特殊标记而不是 `LEN` 前缀分隔。消息中的每个组都有一个字段编号，该编号用于这些特殊标记。

A group with field number `8` begins with an `8:SGROUP` tag. `SGROUP` records have empty payloads, so all this does is denote the start of the group. Once all the fields in the group are listed, a corresponding `8:EGROUP` tag denotes its end. `EGROUP` records also have no payload, so `8:EGROUP` is the entire record. Group field numbers need to match up. If we encounter `7:EGROUP` where we expect `8:EGROUP`, the message is mal-formed.

​	字段编号为 `8` 的组以 `8:SGROUP` 标记开始。 `SGROUP` 记录具有空有效负载，因此所有这些都表示组的开始。列出组中的所有字段后，相应的 `8:EGROUP` 标记表示其结束。 `EGROUP` 记录也没有有效负载，因此 `8:EGROUP` 是整个记录。组字段编号需要匹配。如果我们在预期 `8:EGROUP` 的位置遇到 `7:EGROUP` ，则消息格式错误。

Protoscope provides a convenient syntax for writing groups. Instead of writing

​	Protoscope 提供了一种方便的语法来编写组。Protoscope 允许

```proto
8:SGROUP
  1: 2
  3: {"foo"}
8:EGROUP
```

Protoscope allows
而不是编写

```proto
8: !{
  1: 2
  3: {"foo"}
}
```

This will generate the appropriate start and end group markers. The `!{}` syntax can only occur immediately after an un-typed tag expression, like `8:`.

​	这将生成适当的开始和结束组标记。 `!{}` 语法只能紧跟未键入的标记表达式之后，例如 `8:` 。

## 字段顺序 Field Order 

Field numbers may be declared in any order in a `.proto` file. The order chosen has no effect on how the messages are serialized.

​	字段编号可以在 `.proto` 文件中的任何顺序中声明。所选顺序对消息的序列化方式没有影响。

When a message is serialized, there is no guaranteed order for how its known or [unknown fields](https://protobuf.dev/programming-guides/proto2#updating) will be written. Serialization order is an implementation detail, and the details of any particular implementation may change in the future. Therefore, protocol buffer parsers must be able to parse fields in any order.

​	序列化消息时，没有保证其已知或未知字段的写入顺序。序列化顺序是实现细节，任何特定实现的细节都可能在未来发生变化。因此，协议缓冲区解析器必须能够按任何顺序解析字段。

### 含义 Implications 

- Do not assume the byte output of a serialized message is stable. This is especially true for messages with transitive bytes fields representing other serialized protocol buffer messages.
  
- 不要假设序列化消息的字节输出是稳定的。对于具有表示其他序列化协议缓冲区消息的传递字节字段的消息，尤其如此。
  
- By default, repeated invocations of serialization methods on the same protocol buffer message instance may not produce the same byte output. That is, the default serialization is not deterministic.

- 
  默认情况下，对同一协议缓冲区消息实例重复调用序列化方法可能不会产生相同的字节输出。也就是说，默认序列化不是确定性的。
  
  - Deterministic serialization only guarantees the same byte output for a particular binary. The byte output may change across different versions of the binary.
  - 确定性序列化仅保证特定二进制文件的字节输出相同。字节输出可能会因二进制文件的不同版本而改变。
  
- The following checks may fail for a protocol buffer message instance `foo`:

- 
以下检查可能会对协议缓冲区消息实例 `foo` 失败：
  
- `foo.SerializeAsString() == foo.SerializeAsString()`
  - `Hash(foo.SerializeAsString()) == Hash(foo.SerializeAsString())`
  - `CRC(foo.SerializeAsString()) == CRC(foo.SerializeAsString())`
  - `FingerPrint(foo.SerializeAsString()) == FingerPrint(foo.SerializeAsString())`

- Here are a few example scenarios where logically equivalent protocol buffer messages `foo` and  `bar` may serialize to different byte outputs:

- 
  以下是一些逻辑等效协议缓冲区消息 `foo` 和 `bar` 可能序列化为不同字节输出的示例场景：
  
  - `bar` is serialized by an old server that treats some fields as unknown.
  - `bar` 由将某些字段视为未知字段的旧服务器序列化。
  - `bar` is serialized by a server that is implemented in a different programming language and serializes fields in different order.
  - `bar` 由以不同编程语言实现并以不同顺序序列化字段的服务器序列化。
  - `bar` has a field that serializes in a non-deterministic manner.
  - `bar` 具有以非确定性方式序列化的字段。
  - `bar` has a field that stores a serialized byte output of a protocol buffer message which is serialized differently.
  - `bar` 具有存储协议缓冲区消息的序列化字节输出的字段，该消息以不同的方式序列化。
  - `bar` is serialized by a new server that serializes fields in a different order due to an implementation change.
  - `bar` 由一个新的服务器序列化，该服务器由于实现更改而以不同的顺序序列化字段。
  - `foo` and `bar` are concatenations of the same individual messages in a different order.
  - `foo` 和 `bar` 是相同单个消息以不同顺序连接而成。

## 编码的 Proto 大小限制 Encoded Proto Size Limitations 

Protos must be smaller than 2 GiB when serialized. Many proto implementations will refuse to serialize or parse messages that exceed this limit.

​	序列化时，Proto 必须小于 2 GiB。许多 proto 实现会拒绝序列化或解析超过此限制的消息。

## 简明参考卡 Condensed Reference Card 

The following provides the most prominent parts of the wire format in an easy-to-reference format.

​	以下内容以易于参考的格式提供了线格式中最突出的部分。

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

​	另请参阅 Protoscope 语言参考。

### Key

- `message := (tag value)*`

  A message is encoded as a sequence of zero or more pairs of tags and values. 

  消息编码为零个或多个标签和值对的序列。

- `tag := (field << 3) bit-or wire_type`

  A tag is a combination of a `wire_type`, stored in the least significant three bits, and the field number that is defined in the `.proto` file. 

  标签是 `wire_type` 的组合，存储在最低有效的三位中，以及在 `.proto` 文件中定义的字段编号。

- `value := varint for wire_type == VARINT, ...`

  A value is stored differently depending on the `wire_type` specified in the tag. 

  根据标签中指定的 `wire_type` 以不同的方式存储值。

- `varint := int32 | int64 | uint32 | uint64 | bool | enum | sint32 | sint64`

  You can use varint to store any of the listed data types. 

  您可以使用 varint 存储任何列出的数据类型。

- `i32 := sfixed32 | fixed32 | float`

  You can use fixed32 to store any of the listed data types. 

  您可以使用 fixed32 存储任何列出的数据类型。

- `i64 := sfixed64 | fixed64 | double`

  You can use fixed64 to store any of the listed data types. 

  您可以使用 fixed64 存储任何列出的数据类型。

- `len-prefix := size (message | string | bytes | packed)`

  A length-prefixed value is stored as a length (encoded as a varint), and then one of the listed data types. 

  长度前缀值存储为长度（编码为 varint），然后存储为列出的数据类型之一。

- `string := valid UTF-8 string (e.g. ASCII)`

  As described, a string must use UTF-8 character encoding. A string cannot exceed 2GB. 

  如所述，字符串必须使用 UTF-8 字符编码。字符串不能超过 2GB。

- `bytes := any sequence of 8-bit bytes`

  As described, bytes can store custom data types, up to 2GB in size. 

  如所述，字节可以存储自定义数据类型，大小最高为 2GB。

- `packed := varint* | i32* | i64*`

  Use the `packed` data type when you are storing consecutive values of the type described in the protocol definition. The tag is dropped for values after the first, which amortizes the costs of tags to one per field, rather than per element. 
  
  当您存储协议定义中描述的类型的连续值时，请使用 `packed` 数据类型。第一个值之后的标签将被删除，这会将标签的成本摊销到每个字段，而不是每个元素。
