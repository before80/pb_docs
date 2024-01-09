+++
title = "语言指南 (proto 3)"
weight = 40
description = "介绍如何在项目中使用 Protocol Buffers 版本 3。"
type = "docs"

+++

> 原文网址： https://protobuf.dev/programming-guides/proto3/

## Language Guide (proto 3) 语言指南 (proto 3)

Covers how to use the version 3 of Protocol Buffers in your project.

​	介绍如何在项目中使用 Protocol Buffers 版本 3。



This guide describes how to use the protocol buffer language to structure your protocol buffer data, including `.proto` file syntax and how to generate data access classes from your `.proto` files. It covers the **proto3** version of the protocol buffers language: for information on the **proto2** syntax, see the [Proto2 Language Guide](https://protobuf.dev/programming-guides/proto2).

​	本指南介绍如何使用 Protocol Buffer 语言来构建 Protocol Buffer 数据，包括 `.proto` 文件语法以及如何从 `.proto` 文件生成数据访问类。它介绍了 Protocol Buffers 语言的 proto3 版本：有关 proto2 语法的更多信息，请参阅 Proto2 语言指南。

This is a reference guide – for a step by step example that uses many of the features described in this document, see the [tutorial](https://protobuf.dev/getting-started) for your chosen language.

​	这是一份参考指南 - 有关使用本文档中所述的许多功能的分步示例，请参阅您所选语言的教程。

## 定义消息类型 Defining A Message Type 

First let’s look at a very simple example. Let’s say you want to define a search request message format, where each search request has a query string, the particular page of results you are interested in, and a number of results per page. Here’s the `.proto` file you use to define the message type.

​	首先，我们来看一个非常简单的示例。假设您想定义一个搜索请求消息格式，其中每个搜索请求都有一个查询字符串、您感兴趣的特定结果页面以及每页的结果数。以下是用于定义消息类型的 `.proto` 文件。

```proto
syntax = "proto3";

message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 results_per_page = 3;
}
```

- The first line of the file specifies that you’re using `proto3` syntax: if you don’t do this the protocol buffer compiler will assume you are using [proto2](https://protobuf.dev/programming-guides/proto2). This must be the first non-empty, non-comment line of the file.
- 文件的第一行指定您正在使用 `proto3` 语法：如果您不这样做，Protocol Buffer 编译器将假定您正在使用 proto2。这必须是文件中的第一行非空、非注释行。
- The `SearchRequest` message definition specifies three fields (name/value pairs), one for each piece of data that you want to include in this type of message. Each field has a name and a type.
- 消息定义指定三个字段（名称/值对），每个字段对应要包含在此类消息中的每条数据。每个字段都有一个名称和一个类型。

### 指定字段类型 Specifying Field Types 

In the earlier example, all the fields are [scalar types](https://protobuf.dev/programming-guides/proto3/#scalar): two integers (`page_number` and `results_per_page`) and a string (`query`). You can also specify [enumerations](https://protobuf.dev/programming-guides/proto3/#enum) and composite types like other message types for your field.

​	在前面的示例中，所有字段都是标量类型：两个整数（ `page_number` 和 `results_per_page` ）和一个字符串（ `query` ）。您还可以为字段指定枚举和复合类型，如其他消息类型。

### 分配字段编号 Assigning Field Numbers 

You must give each field in your message definition a number between `1` and `536,870,911` with the following restrictions:

​	您必须为消息定义中的每个字段指定一个介于 `1` 和 `536,870,911` 之间的编号，并遵守以下限制：

- The given number **must be unique** among all fields for that message.
- 给定的编号对于该消息的所有字段必须是唯一的。
- Field numbers `19,000` to `19,999` are reserved for the Protocol Buffers implementation. The protocol buffer compiler will complain if you use one of these reserved field numbers in your message.
- 字段编号 `19,000` 到 `19,999` 为 Protocol Buffers 实现保留。如果您在消息中使用这些保留的字段编号之一，协议缓冲区编译器会发出警告。
- You cannot use any previously [reserved](https://protobuf.dev/programming-guides/proto3/#fieldreserved) field numbers or any field numbers that have been allocated to [extensions](https://protobuf.dev/programming-guides/proto3/#extensions).
- 您不能使用任何以前保留的字段编号或已分配给扩展的任何字段编号。

This number **cannot be changed once your message type is in use** because it identifies the field in the [message wire format](https://protobuf.dev/programming-guides/encoding). “Changing” a field number is equivalent to deleting that field and creating a new field with the same type but a new number. See [Deleting Fields](https://protobuf.dev/programming-guides/proto3/#deleting) for how to do this properly.

​	一旦您的消息类型正在使用，此数字就无法更改，因为它标识了消息线格式中的字段。“更改”字段号等同于删除该字段并创建一个具有相同类型但新编号的新字段。有关如何正确执行此操作，请参阅删除字段。

Field numbers **should never be reused**. Never take a field number out of the [reserved](https://protobuf.dev/programming-guides/proto3/#fieldreserved) list for reuse with a new field definition. See [Consequences of Reusing Field Numbers](https://protobuf.dev/programming-guides/proto3/#consequences).

​	字段号绝不应重复使用。切勿从保留列表中取出字段号以与新字段定义重复使用。请参阅重复使用字段号的后果。

You should use the field numbers 1 through 15 for the most-frequently-set fields. Lower field number values take less space in the wire format. For example, field numbers in the range 1 through 15 take one byte to encode. Field numbers in the range 16 through 2047 take two bytes. You can find out more about this in [Protocol Buffer Encoding](https://protobuf.dev/programming-guides/encoding#structure).

​	您应将字段号 1 到 15 用于设置最频繁的字段。较低的字段号值在有线格式中占用的空间更少。例如，范围 1 到 15 中的字段号需要一个字节进行编码。范围 16 到 2047 中的字段号需要两个字节。您可以在协议缓冲区编码中找到更多相关信息。

#### 重复使用字段号的后果 Consequences of Reusing Field Numbers 

Reusing a field number makes decoding wire-format messages ambiguous.

​	重复使用字段号会使解码有线格式消息变得模棱两可。

The protobuf wire format is lean and doesn’t provide a way to detect fields encoded using one definition and decoded using another.

​	protobuf 有线格式精简，无法检测使用一种定义编码并使用另一种定义解码的字段。

Encoding a field using one definition and then decoding that same field with a different definition can lead to:

​	使用一种定义对字段进行编码，然后使用不同的定义对同一字段进行解码，可能会导致：

- Developer time lost to debugging
- 开发人员浪费时间进行调试
- A parse/merge error (best case scenario)
- 解析/合并错误（最佳情况假设）
- Leaked PII/SPII
- 泄露的 PII/SPII
- Data corruption
- 数据损坏

Common causes of field number reuse:

​	字段编号重复使用的一些常见原因：

- renumbering fields (sometimes done to achieve a more aesthetically pleasing number order for fields). Renumbering effectively deletes and re-adds all the fields involved in the renumbering, resulting in incompatible wire-format changes.
- 重新编号字段（有时这样做是为了实现字段更美观的编号顺序）。重新编号实际上会删除并重新添加重新编号中涉及的所有字段，从而导致不兼容的线格式更改。
- deleting a field and not [reserving](https://protobuf.dev/programming-guides/proto3/#fieldreserved) the number to prevent future reuse.
- 删除字段且不保留编号以防止将来重复使用。

The max field is 29 bits instead of the more-typical 32 bits because three lower bits are used for the wire format. For more on this, see the [Encoding topic](https://protobuf.dev/programming-guides/encoding#structure).

​	最大字段为 29 位，而不是更典型的 32 位，因为三个低位用于线格式。有关此内容的更多信息，请参阅编码主题。



###  指定字段标签 Specifying Field Labels

Message fields can be one of the following:

​	消息字段可以是以下之一：

- `optional`: An `optional` field is in one of two possible states:
  
- `optional` ： `optional` 字段处于两种可能状态之一：
  
  - the field is set, and contains a value that was explicitly set or parsed from the wire. It will be serialized to the wire.
  - 字段已设置，并且包含显式设置或从线路中解析的值。它将被序列化到线路中。
  - the field is unset, and will return the default value. It will not be serialized to the wire.
  - 字段未设置，并将返回默认值。它不会被序列化到线路中。
  
  You can check to see if the value was explicitly set.
  
  ​	您可以检查该值是否已显式设置。

- `repeated`: this field type can be repeated zero or more times in a well-formed message. The order of the repeated values will be preserved.
  
- `repeated` ：这种字段类型可以在格式良好的消息中重复零次或多次。将保留重复值顺序。
  
- `map`: this is a paired key/value field type. See [Maps](https://protobuf.dev/programming-guides/encoding#maps) for more on this field type.
  
- `map` ：这是一个配对的键/值字段类型。有关此字段类型的更多信息，请参阅映射。
  
- If no explicit field label is applied, the default field label, called “implicit field presence,” is assumed. (You cannot explicitly set a field to this state.) A well-formed message can have zero or one of this field (but not more than one). You also cannot determine whether a field of this type was parsed from the wire. An implicit presence field will be serialized to the wire unless it is the default value. For more on this subject, see [Field Presence](https://protobuf.dev/programming-guides/field_presence).
  
- 如果未应用显式字段标签，则假定默认字段标签，称为“隐式字段显示”。（您无法将字段显式设置为此状态。）格式良好的消息可以具有零个或一个此字段（但不能多于一个）。您也无法确定此类型的字段是否已从线路中解析。隐式显示字段将被序列化到线路中，除非它是默认值。有关此主题的更多信息，请参阅字段显示。

In proto3, `repeated` fields of scalar numeric types use `packed` encoding by default. You can find out more about `packed` encoding in [Protocol Buffer Encoding](https://protobuf.dev/programming-guides/encoding#packed).

​	在 proto3 中，标量数字类型的 `repeated` 字段默认使用 `packed` 编码。您可以在协议缓冲区编码中找到有关 `packed` 编码的更多信息。

#### 格式良好的消息 Well-formed Messages 

The term “well-formed,” when applied to protobuf messages, refers to the bytes serialized/deserialized. The protoc parser validates that a given proto definition file is parseable.

​	当应用于 protobuf 消息时，“格式良好”一词是指序列化/反序列化的字节。protoc 解析器验证给定的 proto 定义文件是否可解析。

In the case of `optional` fields that have more than one value, the protoc parser will accept the input, but only uses the last field. So, the “bytes” may not be “well-formed” but the resulting message would have only one and would be “well-formed” (but would not roundtrip the same).

​	对于具有多个值 `optional` 字段的情况，protoc 解析器将接受输入，但仅使用最后一个字段。因此，“字节”可能不是“格式良好”，但结果消息只有一个，并且是“格式良好”（但不会循环往复相同）。

### 添加更多消息类型 Adding More Message Types 

Multiple message types can be defined in a single `.proto` file. This is useful if you are defining multiple related messages – so, for example, if you wanted to define the reply message format that corresponds to your `SearchResponse` message type, you could add it to the same `.proto`:

​	可以在单个 `.proto` 文件中定义多个消息类型。如果您要定义多个相关消息，这很有用——因此，例如，如果您想定义与 `SearchResponse` 消息类型相对应的回复消息格式，您可以将其添加到相同的 `.proto` 中：

```proto
message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 results_per_page = 3;
}

message SearchResponse {
 ...
}
```

**Combining Messages leads to bloat** While multiple message types (such as message, enum, and service) can be defined in a single `.proto` file, it can also lead to dependency bloat when large numbers of messages with varying dependencies are defined in a single file. It’s recommended to include as few message types per `.proto` file as possible.

​	组合消息会导致膨胀虽然可以在单个 `.proto` 文件中定义多个消息类型（例如消息、枚举和服务），但当在单个文件中定义大量具有不同依赖关系的消息时，它也可能导致依赖关系膨胀。建议每个 `.proto` 文件中包含尽可能少的消息类型。

### 添加注释 Adding Comments 

To add comments to your `.proto` files, use C/C++-style `//` and `/* ... */` syntax.

​	要在 `.proto` 文件中添加注释，请使用 C/C++ 风格的 `//` 和 `/* ... */` 语法。

```proto
/* SearchRequest represents a search query, with pagination options to
 * indicate which results to include in the response. */

message SearchRequest {
  string query = 1;
  int32 page_number = 2;  // Which page number do we want?
  int32 results_per_page = 3;  // Number of results to return per page.
}
```

### 删除字段 Deleting Fields 

Deleting fields can cause serious problems if not done properly.

​	如果不正确地删除字段，可能会导致严重问题。

When you no longer need a field and all references have been deleted from client code, you may delete the field definition from the message. However, you **must** [reserve the deleted field number](https://protobuf.dev/programming-guides/proto3/#fieldreserved). If you do not reserve the field number, it is possible for a developer to reuse that number in the future.

​	当不再需要某个字段且已从客户端代码中删除所有引用时，可以从消息中删除该字段定义。但是，必须保留已删除的字段编号。如果不保留字段编号，开发人员将来可能会重复使用该编号。

You should also reserve the field name to allow JSON and TextFormat encodings of your message to continue to parse.

​	还应保留字段名称，以允许继续解析消息的 JSON 和 TextFormat 编码。

### 保留的字段 Reserved Fields 

If you [update](https://protobuf.dev/programming-guides/proto3/#updating) a message type by entirely deleting a field, or commenting it out, future developers can reuse the field number when making their own updates to the type. This can cause severe issues, as described in [Consequences of Reusing Field Numbers](https://protobuf.dev/programming-guides/proto3/#consequences).

​	如果通过完全删除字段或将其注释掉来更新消息类型，则未来的开发人员在对该类型进行自己的更新时可以重复使用该字段编号。这可能会导致严重问题，如重复使用字段编号的后果中所述。

To make sure this doesn’t happen, add your deleted field number to the `reserved` list. To make sure JSON and TextFormat instances of your message can still be parsed, also add the deleted field name to a `reserved` list.

​	为确保不会发生这种情况，请将已删除的字段编号添加到 `reserved` 列表中。为确保仍可以解析消息的 JSON 和 TextFormat 实例，还应将已删除的字段名称添加到 `reserved` 列表中。

The protocol buffer compiler will complain if any future developers try to use these reserved field numbers or names.

​	如果任何未来的开发人员尝试使用这些保留的字段编号或名称，则协议缓冲区编译器会发出警告。

```proto
message Foo {
  reserved 2, 15, 9 to 11;
  reserved "foo", "bar";
}
```

Reserved field number ranges are inclusive (`9 to 11` is the same as `9, 10, 11`). Note that you can’t mix field names and field numbers in the same `reserved` statement.

​	保留的字段编号范围是包含的（ `9 to 11` 与 `9, 10, 11` 相同）。请注意，您不能在同一个 `reserved` 语句中混合字段名称和字段编号。

### 从您的 `.proto` 中生成什么？ What’s Generated from Your `.proto`? 

When you run the [protocol buffer compiler](https://protobuf.dev/programming-guides/proto3/#generating) on a `.proto`, the compiler generates the code in your chosen language you’ll need to work with the message types you’ve described in the file, including getting and setting field values, serializing your messages to an output stream, and parsing your messages from an input stream.

​	当您对 `.proto` 运行协议缓冲区编译器时，编译器会以您选择的语言生成代码，您需要使用这些代码来处理您在文件中描述的消息类型，包括获取和设置字段值、将您的消息序列化到输出流以及从输入流解析您的消息。

- For **C++**, the compiler generates a `.h` and `.cc` file from each `.proto`, with a class for each message type described in your file.
- 对于 C++，编译器会从每个 `.proto` 生成一个 `.h` 和 `.cc` 文件，其中包含一个类，用于描述您文件中的每种消息类型。
- For **Java**, the compiler generates a `.java` file with a class for each message type, as well as a special `Builder` class for creating message class instances.
- 对于 Java，编译器会生成一个 `.java` 文件，其中包含一个类，用于描述每种消息类型，以及一个用于创建消息类实例的特殊 `Builder` 类。
- For **Kotlin**, in addition to the Java generated code, the compiler generates a `.kt` file for each message type with an improved Kotlin API. This includes a DSL that simplifies creating message instances, a nullable field accessor, and a copy function.
- 对于 Kotlin，除了生成的 Java 代码外，编译器还会为每种消息类型生成一个 `.kt` 文件，其中包含改进的 Kotlin API。这包括一个简化创建消息实例的 DSL、一个可为空的字段访问器和一个复制函数。
- **Python** is a little different — the Python compiler generates a module with a static descriptor of each message type in your `.proto`, which is then used with a *metaclass* to create the necessary Python data access class at runtime.
- Python 有点不同——Python 编译器会生成一个模块，其中包含 `.proto` 中每个消息类型的静态描述符，然后使用元类在运行时创建必要的 Python 数据访问类。
- For **Go**, the compiler generates a `.pb.go` file with a type for each message type in your file.
- 对于 Go，编译器会生成一个 `.pb.go` 文件，其中包含文件中每个消息类型的类型。
- For **Ruby**, the compiler generates a `.rb` file with a Ruby module containing your message types.
- 对于 Ruby，编译器会生成一个 `.rb` 文件，其中包含一个包含消息类型的 Ruby 模块。
- For **Objective-C**, the compiler generates a `pbobjc.h` and `pbobjc.m` file from each `.proto`, with a class for each message type described in your file.
- 对于 Objective-C，编译器会从每个 `.proto` 生成一个 `pbobjc.h` 和 `pbobjc.m` 文件，其中包含文件中描述的每个消息类型的类。
- For **C#**, the compiler generates a `.cs` file from each `.proto`, with a class for each message type described in your file.
- 对于 C#，编译器会从每个 `.proto` 生成一个 `.cs` 文件，其中包含文件中描述的每个消息类型的类。
- For **PHP**, the compiler generates a `.php` message file for each message type described in your file, and a `.php` metadata file for each `.proto` file you compile. The metadata file is used to load the valid message types into the descriptor pool.
- 对于 PHP，编译器会为文件中描述的每个消息类型生成一个 `.php` 消息文件，并为编译的每个 `.proto` 文件生成一个 `.php` 元数据文件。元数据文件用于将有效的消息类型加载到描述符池中。
- For **Dart**, the compiler generates a `.pb.dart` file with a class for each message type in your file.
- 对于 Dart，编译器会生成一个 `.pb.dart` 文件，其中包含文件中每个消息类型的类。

You can find out more about using the APIs for each language by following the tutorial for your chosen language. For even more API details, see the relevant [API reference](https://protobuf.dev/reference/).

​	您可以通过按照您所选语言的教程来了解有关使用每种语言的 API 的更多信息。有关更多 API 详细信息，请参阅相关的 API 参考。

## 标量值类型 Scalar Value Types 

A scalar message field can have one of the following types – the table shows the type specified in the `.proto` file, and the corresponding type in the automatically generated class:

​	标量消息字段可以具有以下类型之一 - 表格显示了 `.proto` 文件中指定了类型，以及自动生成的类中的相应类型：

| .proto Type .proto 类型 | Notes 备注                                                   | C++ Type C++ 类型 | Java/Kotlin Type[1] Java/Kotlin 类型 [1] | Python Type[3] Python 类型 [3]  | Go Type Go 类型 | Ruby Type Ruby 类型                                          | C# Type C# 类型 | PHP Type PHP 类型                 | Dart Type Dart 类型 |
| ----------------------- | ------------------------------------------------------------ | ----------------- | ---------------------------------------- | ------------------------------- | --------------- | ------------------------------------------------------------ | --------------- | --------------------------------- | ------------------- |
| double                  |                                                              | double            | double                                   | float                           | float64         | Float                                                        | double          | float 浮点                        | double 双精度       |
| float 浮点              |                                                              | float 浮点        | float 浮点                               | float 浮点                      | float32         | Float 浮点                                                   | float 浮点      | float                             | double              |
| int32                   | Uses variable-length encoding. Inefficient for encoding negative numbers – if your field is likely to have negative values, use sint32 instead. 使用可变长度编码。对编码负数效率低下——如果您的字段可能具有负值，请改用 sint32。 | int32             | int                                      | int                             | int32           | Fixnum or Bignum (as required) Fixnum 或 Bignum（视需要而定） | int             | integer                           | int                 |
| int64                   | Uses variable-length encoding. Inefficient for encoding negative numbers – if your field is likely to have negative values, use sint64 instead. 使用可变长度编码。对编码负数效率低下——如果您的字段可能具有负值，请改用 sint64。 | int64             | long                                     | int/long[4]                     | int64           | Bignum                                                       | long            | integer/string[6]                 | Int64               |
| uint32                  | Uses variable-length encoding. 使用可变长度编码。            | uint32            | int[2]                                   | int/long[4]                     | uint32          | Fixnum or Bignum (as required) 定点数或大数（视需要而定）    | uint            | integer                           | int                 |
| uint64                  | Uses variable-length encoding. 使用可变长度编码。            | uint64            | long[2]                                  | int/long[4]                     | uint64          | Bignum                                                       | ulong           | integer/string[6] 整数/字符串 [6] | Int64               |
| sint32                  | Uses variable-length encoding. Signed int value. These more efficiently encode negative numbers than regular int32s. 使用可变长度编码。有符号整数值。这些比常规 int32 更有效地编码负数。 | int32             | int                                      | int                             | int32           | Fixnum or Bignum (as required) Fixnum 或 Bignum（视需要而定） | int             | integer                           | int                 |
| sint64                  | Uses variable-length encoding. Signed int value. These more efficiently encode negative numbers than regular int64s. 使用可变长度编码。有符号整数值。这些比常规 int64 更有效地编码负数。 | int64             | long                                     | int/long[4]                     | int64           | Bignum                                                       | long            | integer/string[6]                 | Int64               |
| fixed32                 | Always four bytes. More efficient than uint32 if values are often greater than 228. 始终为四个字节。如果值通常大于 2 28 ，则比 uint32 更有效率。 | uint32            | int[2]                                   | int/long[4]                     | uint32          | Fixnum or Bignum (as required) Fixnum 或 Bignum（视需要而定） | uint            | integer                           | int                 |
| fixed64                 | Always eight bytes. More efficient than uint64 if values are often greater than 256. 始终为八个字节。如果值通常大于 2 56 ，则比 uint64 更有效率。 | uint64            | long[2]                                  | int/long[4]                     | uint64          | Bignum                                                       | ulong           | integer/string[6]                 | Int64               |
| sfixed32                | Always four bytes. 始终为四个字节。                          | int32             | int                                      | int                             | int32           | Fixnum or Bignum (as required) Fixnum 或 Bignum（视需要而定） | int             | integer                           | int                 |
| sfixed64                | Always eight bytes. 始终为八个字节。                         | int64             | long                                     | int/long[4]                     | int64           | Bignum                                                       | long            | integer/string[6] 整数/字符串 [6] | Int64               |
| bool 布尔               |                                                              | bool 布尔         | boolean 布尔                             | bool 布尔                       | bool 布尔       | TrueClass/FalseClass 布尔                                    | bool 真/假      | boolean 布尔                      | bool                |
| string 字符串           | A string must always contain UTF-8 encoded or 7-bit ASCII text, and cannot be longer than 232. 字符串必须始终包含 UTF-8 编码或 7 位 ASCII 文本，且长度不能超过 2 32 . | string 字符串     | String                                   | str/unicode[5]                  | string 字符串   | String (UTF-8) 字符串 (UTF-8)                                | string 字符串   | string 字符串                     | String              |
| bytes 字节              | May contain any arbitrary sequence of bytes no longer than 232. 可以包含任意长度不超过 2 32 的任意字节序列。 | string 字符串     | ByteString                               | str (Python 2) bytes (Python 3) | []byte          | String (ASCII-8BIT) 字符串 (ASCII-8BIT)                      | ByteString      | string 字符串                     | List 列表           |

[1] Kotlin uses the corresponding types from Java, even for unsigned types, to ensure compatibility in mixed Java/Kotlin codebases.

[1] Kotlin 使用来自 Java 的相应类型，即使是无符号类型，以确保在混合 Java/Kotlin 代码库中兼容。

[2] In Java, unsigned 32-bit and 64-bit integers are represented using their signed counterparts, with the top bit simply being stored in the sign bit.

[2] 在 Java 中，无符号 32 位和 64 位整数使用其有符号对应项表示，最高位仅存储在符号位中。

[3] In all cases, setting values to a field will perform type checking to make sure it is valid.

[3] 在所有情况下，将值设置为字段都会执行类型检查以确保其有效。

[4] 64-bit or unsigned 32-bit integers are always represented as long when decoded, but can be an int if an int is given when setting the field. In all cases, the value must fit in the type represented when set. See [2].

[4] 在解码时，64 位或无符号 32 位整数始终表示为 long，但如果在设置字段时给出了 int，则可以是 int。在所有情况下，该值都必须适合在设置时表示的类型。请参阅 [2]。

[5] Python strings are represented as unicode on decode but can be str if an ASCII string is given (this is subject to change).

[5] Python 字符串在解码时表示为 unicode，但如果给出了 ASCII 字符串，则可以是 str（这可能会更改）。

[6] Integer is used on 64-bit machines and string is used on 32-bit machines.

[6] 在 64 位机器上使用 Integer，在 32 位机器上使用 string。

You can find out more about how these types are encoded when you serialize your message in [Protocol Buffer Encoding](https://protobuf.dev/programming-guides/encoding).

您可以在 Protocol Buffer 编码中序列化消息时找到有关如何对这些类型进行编码的更多信息。

## 默认值 Default Values 

When a message is parsed, if the encoded message does not contain a particular implicit presence element, accessing the corresponding field in the parsed object returns the default value for that field. These defaults are type-specific:

​	解析消息时，如果编码的消息不包含某个隐式存在元素，则访问解析对象中的相应字段会返回该字段的默认值。这些默认值是特定于类型的：

- For strings, the default value is the empty string.
- 对于字符串，默认值是空字符串。
- For bytes, the default value is empty bytes.
- 对于字节，默认值是空字节。
- For bools, the default value is false.
- 对于布尔值，默认值是 false。
- For numeric types, the default value is zero. For float and double types, -0.0 and 0.0 are treated as equivalent, and will round-trip.
- 对于数字类型，默认值是零。对于 float 和 double 类型，-0.0 和 0.0 被视为相等，并且会进行双向转换。
- For enums, the default value is the **first defined enum value**, which must be 0.
- 对于枚举，默认值是第一个已定义的枚举值，该值必须为 0。
- For message fields, the field is not set. Its exact value is language-dependent. See the [generated code guide](https://protobuf.dev/reference/) for details.
- 对于消息字段，该字段未设置。其确切值取决于语言。有关详细信息，请参阅生成的代码指南。

The default value for repeated fields is empty (generally an empty list in the appropriate language).

​	重复字段的默认值为空（通常是相应语言中的空列表）。

Note that for scalar message fields, once a message is parsed there’s no way of telling whether a field was explicitly set to the default value (for example whether a boolean was set to `false`) or just not set at all: you should bear this in mind when defining your message types. For example, don’t have a boolean that switches on some behavior when set to `false` if you don’t want that behavior to also happen by default. Also note that if a scalar message field **is** set to its default, the value will not be serialized on the wire. If a float or double value is set to -0 or +0, it will not be serialized.

​	请注意，对于标量消息字段，一旦解析消息，就无法判断字段是否显式设置为默认值（例如，布尔值是否设置为 `false` ）或根本未设置：在定义消息类型时应牢记这一点。例如，如果不想默认发生某种行为，请勿将布尔值设置为 `false` 以启用某种行为。另请注意，如果标量消息字段设置为其默认值，则该值不会在网络上序列化。如果浮点值或双精度值设置为 -0 或 +0，则不会序列化。

See the [generated code guide](https://protobuf.dev/reference/) for your chosen language for more details about how defaults work in generated code.

​	有关默认值在生成代码中如何工作的更多详细信息，请参阅您所选语言的生成代码指南。

## 枚举 Enumerations 

When you’re defining a message type, you might want one of its fields to only have one of a predefined list of values. For example, let’s say you want to add a `corpus` field for each `SearchRequest`, where the corpus can be `UNIVERSAL`, `WEB`, `IMAGES`, `LOCAL`, `NEWS`, `PRODUCTS` or `VIDEO`. You can do this very simply by adding an `enum` to your message definition with a constant for each possible value.

​	在定义消息类型时，您可能希望其字段之一仅具有预定义值列表中的一个值。例如，假设您想为每个 `SearchRequest` 添加一个 `corpus` 字段，其中语料库可以是 `UNIVERSAL` 、 `WEB` 、 `IMAGES` 、 `LOCAL` 、 `NEWS` 、 `PRODUCTS` 或 `VIDEO` 。您可以通过向消息定义中添加 `enum` 来非常简单地做到这一点，其中包含每个可能值的常量。

In the following example we’ve added an `enum` called `Corpus` with all the possible values, and a field of type `Corpus`:

​	在以下示例中，我们添加了一个名为 `enum` 的 `Corpus` ，其中包含所有可能的值，以及一个类型为 `Corpus` 的字段：

```proto
enum Corpus {
  CORPUS_UNSPECIFIED = 0;
  CORPUS_UNIVERSAL = 1;
  CORPUS_WEB = 2;
  CORPUS_IMAGES = 3;
  CORPUS_LOCAL = 4;
  CORPUS_NEWS = 5;
  CORPUS_PRODUCTS = 6;
  CORPUS_VIDEO = 7;
}

message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 results_per_page = 3;
  Corpus corpus = 4;
}
```

As you can see, the `Corpus` enum’s first constant maps to zero: every enum definition **must** contain a constant that maps to zero as its first element. This is because:

​	如您所见， `Corpus` 枚举的第一个常量映射到零：每个枚举定义都必须包含一个常量，该常量映射到零作为其第一个元素。这是因为：

- There must be a zero value, so that we can use 0 as a numeric [default value](https://protobuf.dev/programming-guides/proto3/#default).
- 必须有一个零值，以便我们可以使用 0 作为数字默认值。
- The zero value needs to be the first element, for compatibility with the [proto2](https://protobuf.dev/programming-guides/proto2) semantics where the first enum value is the default unless a different value is explicitly specified.
- 零值需要是第一个元素，以便与 proto2 语义兼容，其中第一个枚举值是默认值，除非明确指定了其他值。

You can define aliases by assigning the same value to different enum constants. To do this you need to set the `allow_alias` option to `true`. Otherwise, the protocol buffer compiler generates a warning message when aliases are found. Though all alias values are valid during deserialization, the first value is always used when serializing.

​	您可以通过将相同的值分配给不同的枚举常量来定义别名。为此，您需要将 `allow_alias` 选项设置为 `true` 。否则，当找到别名时，协议缓冲区编译器会生成一条警告消息。尽管在反序列化期间所有别名值都是有效的，但在序列化时始终使用第一个值。

```proto
enum EnumAllowingAlias {
  option allow_alias = true;
  EAA_UNSPECIFIED = 0;
  EAA_STARTED = 1;
  EAA_RUNNING = 1;
  EAA_FINISHED = 2;
}

enum EnumNotAllowingAlias {
  ENAA_UNSPECIFIED = 0;
  ENAA_STARTED = 1;
  // ENAA_RUNNING = 1;  // Uncommenting this line will cause a warning message.
  ENAA_FINISHED = 2;
}
```

Enumerator constants must be in the range of a 32-bit integer. Since `enum` values use [varint encoding](https://protobuf.dev/programming-guides/encoding) on the wire, negative values are inefficient and thus not recommended. You can define `enum`s within a message definition, as in the earlier example, or outside – these `enum`s can be reused in any message definition in your `.proto` file. You can also use an `enum` type declared in one message as the type of a field in a different message, using the syntax `_MessageType_._EnumType_`.

​	枚举常量必须在 32 位整数的范围内。由于 `enum` 值在传输中使用可变长度整数编码，因此负值效率低下，因此不建议使用。您可以像在前面的示例中一样在消息定义中定义 `enum` ，也可以在外部定义 - 这些 `enum` 可以重复用于 `.proto` 文件中的任何消息定义。您还可以使用在一条消息中声明的 `enum` 类型作为另一条消息中字段的类型，使用语法 `_MessageType_._EnumType_` 。

When you run the protocol buffer compiler on a `.proto` that uses an `enum`, the generated code will have a corresponding `enum` for Java, Kotlin, or C++, or a special `EnumDescriptor` class for Python that’s used to create a set of symbolic constants with integer values in the runtime-generated class.

​	当您对使用 `enum` 的 `.proto` 运行协议缓冲区编译器时，生成的代码将为 Java、Kotlin 或 C++ 具有相应的 `enum` ，或者为 Python 具有一个特殊的 `EnumDescriptor` 类，该类用于在运行时生成的类中创建一组具有整数值的符号常量。

#### 重要 Important 

The generated code may be subject to language-specific limitations on the number of enumerators (low thousands for one language). Review the limitations for the languages you plan to use.

​	生成的代码可能会受到语言对枚举器数量的限制（一种语言的限制为低数千）。查看您计划使用的语言的限制。

During deserialization, unrecognized enum values will be preserved in the message, though how this is represented when the message is deserialized is language-dependent. In languages that support open enum types with values outside the range of specified symbols, such as C++ and Go, the unknown enum value is simply stored as its underlying integer representation. In languages with closed enum types such as Java, a case in the enum is used to represent an unrecognized value, and the underlying integer can be accessed with special accessors. In either case, if the message is serialized the unrecognized value will still be serialized with the message.

​	在反序列化过程中，无法识别的枚举值将保留在消息中，尽管消息反序列化时的表示方式取决于语言。在支持具有超出指定符号范围的值的开放枚举类型的语言（例如 C++ 和 Go）中，未知枚举值仅存储为其底层整数表示形式。在具有封闭枚举类型的语言（例如 Java）中，枚举中的一个情况用于表示无法识别的值，并且可以使用特殊访问器访问底层整数。在任何一种情况下，如果对消息进行序列化，则无法识别的值仍将随消息一起序列化。

#### 重要 Important 

For information on how enums should work contrasted with how they currently work in different languages, see [Enum Behavior](https://protobuf.dev/programming-guides/enum).

​	有关枚举应如何工作与它们当前如何在不同语言中工作形成对比的信息，请参阅枚举行为。

For more information about how to work with message `enum`s in your applications, see the [generated code guide](https://protobuf.dev/reference/) for your chosen language.

​	有关如何在应用程序中使用消息 `enum` 的更多信息，请参阅您所选语言的生成代码指南。

### 保留值 Reserved Values 

If you [update](https://protobuf.dev/programming-guides/proto3/#updating) an enum type by entirely removing an enum entry, or commenting it out, future users can reuse the numeric value when making their own updates to the type. This can cause severe issues if they later load old versions of the same `.proto`, including data corruption, privacy bugs, and so on. One way to make sure this doesn’t happen is to specify that the numeric values (and/or names, which can also cause issues for JSON serialization) of your deleted entries are `reserved`. The protocol buffer compiler will complain if any future users try to use these identifiers. You can specify that your reserved numeric value range goes up to the maximum possible value using the `max` keyword.

​	如果您通过完全删除枚举项或将其注释掉来更新枚举类型，则以后的用户在对类型进行自己的更新时可以重复使用该数字值。如果他们以后加载同一 `.proto` 的旧版本，这可能会导致严重的问题，包括数据损坏、隐私漏洞等。确保这种情况不会发生的一种方法是指定已删除项的数字值（和/或名称，这也可能导致 JSON 序列化出现问题）为 `reserved` 。如果任何以后的用户尝试使用这些标识符，协议缓冲区编译器会发出警告。您可以使用 `max` 关键字指定您的保留数字值范围达到最大可能值。

```proto
enum Foo {
  reserved 2, 15, 9 to 11, 40 to max;
  reserved "FOO", "BAR";
}
```

Note that you can’t mix field names and numeric values in the same `reserved` statement.

​	请注意，您不能在同一 `reserved` 语句中混合字段名称和数字值。

## 使用其他消息类型 Using Other Message Types 

You can use other message types as field types. For example, let’s say you wanted to include `Result` messages in each `SearchResponse` message – to do this, you can define a `Result` message type in the same `.proto` and then specify a field of type `Result` in `SearchResponse`:

​	您可以将其他消息类型用作字段类型。例如，假设您想在每个 `SearchResponse` 消息中包含 `Result` 消息 - 为此，您可以在同一 `.proto` 中定义 `Result` 消息类型，然后在 `SearchResponse` 中指定类型为 `Result` 的字段：

```proto
message SearchResponse {
  repeated Result results = 1;
}

message Result {
  string url = 1;
  string title = 2;
  repeated string snippets = 3;
}
```

### 导入定义 Importing Definitions 

In the earlier example, the `Result` message type is defined in the same file as `SearchResponse` – what if the message type you want to use as a field type is already defined in another `.proto` file?

​	在前面的示例中， `Result` 消息类型与 `SearchResponse` 定义在同一个文件中——如果要将作为字段类型使用的消息类型已经定义在另一个 `.proto` 文件中，该怎么办？

You can use definitions from other `.proto` files by *importing* them. To import another `.proto`’s definitions, you add an import statement to the top of your file:

​	您可以通过导入来使用其他 `.proto` 文件中的定义。要导入另一个 `.proto` 的定义，请将导入语句添加到文件的顶部：

```proto
import "myproject/other_protos.proto";
```

By default, you can use definitions only from directly imported `.proto` files. However, sometimes you may need to move a `.proto` file to a new location. Instead of moving the `.proto` file directly and updating all the call sites in a single change, you can put a placeholder `.proto` file in the old location to forward all the imports to the new location using the `import public` notion.

​	默认情况下，您只能使用直接导入的 `.proto` 文件中的定义。但是，有时您可能需要将 `.proto` 文件移动到新位置。您可以将占位符 `.proto` 文件放在旧位置，以使用 `import public` 概念将所有导入转发到新位置，而无需直接移动 `.proto` 文件并更新单个更改中的所有调用站点。

**Note that the public import functionality is not available in Java.
请注意，Java 中不提供公共导入功能。**

`import public` dependencies can be transitively relied upon by any code importing the proto containing the `import public` statement. For example:

​	任何导入包含 `import public` 语句的原型的代码都可以间接依赖 `import public` 依赖项。例如：

```proto
// new.proto
// All definitions are moved here
// old.proto
// This is the proto that all clients are importing.
import public "new.proto";
import "other.proto";
// client.proto
import "old.proto";
// You use definitions from old.proto and new.proto, but not other.proto
```

The protocol compiler searches for imported files in a set of directories specified on the protocol compiler command line using the `-I`/`--proto_path` flag. If no flag was given, it looks in the directory in which the compiler was invoked. In general you should set the `--proto_path` flag to the root of your project and use fully qualified names for all imports.

​	协议编译器在使用 `-I` / `--proto_path` 标志在协议编译器命令行上指定的一组目录中搜索导入的文件。如果未给出标志，它将在编译器被调用的目录中查找。通常，您应该将 `--proto_path` 标志设置为项目的根目录，并对所有导入使用完全限定名称。

### 使用 proto2 消息类型 Using proto2 Message Types 

It’s possible to import [proto2](https://protobuf.dev/programming-guides/proto2) message types and use them in your proto3 messages, and vice versa. However, proto2 enums cannot be used directly in proto3 syntax (it’s okay if an imported proto2 message uses them).

​	可以导入 proto2 消息类型并在 proto3 消息中使用它们，反之亦然。但是，proto2 枚举不能直接在 proto3 语法中使用（如果导入的 proto2 消息使用它们，则可以）。

## 嵌套类型 Nested Types 

You can define and use message types inside other message types, as in the following example – here the `Result` message is defined inside the `SearchResponse` message:

​	您可以在其他消息类型中定义和使用消息类型，如下例所示 - 此处 `Result` 消息在 `SearchResponse` 消息中定义：

```proto
message SearchResponse {
  message Result {
    string url = 1;
    string title = 2;
    repeated string snippets = 3;
  }
  repeated Result results = 1;
}
```

If you want to reuse this message type outside its parent message type, you refer to it as `_Parent_._Type_`:

​	如果您想在其父消息类型之外重用此消息类型，则可以将其称为 `_Parent_._Type_` ：

```proto
message SomeOtherMessage {
  SearchResponse.Result result = 1;
}
```

You can nest messages as deeply as you like. In the example below, note that the two nested types named `Inner` are entirely independent, since they are defined within different messages:

​	您可以根据需要深度嵌套消息。在下面的示例中，请注意，名为 `Inner` 的两个嵌套类型完全独立，因为它们是在不同的消息中定义的：

```proto
message Outer {       // Level 0
  message MiddleAA {  // Level 1
    message Inner {   // Level 2
      int64 ival = 1;
      bool  booly = 2;
    }
  }
  message MiddleBB {  // Level 1
    message Inner {   // Level 2
      int32 ival = 1;
      bool  booly = 2;
    }
  }
}
```

## 更新消息类型 Updating A Message Type 

If an existing message type no longer meets all your needs – for example, you’d like the message format to have an extra field – but you’d still like to use code created with the old format, don’t worry! It’s very simple to update message types without breaking any of your existing code when you use the binary wire format.

​	如果现有消息类型不再满足您的所有需求——例如，您希望消息格式具有一个额外的字段——但您仍希望使用旧格式创建的代码，请不要担心！使用二进制线格式时，更新消息类型非常简单，而不会破坏任何现有代码。

#### 注意 Note 

If you use JSON or [proto text format](https://protobuf.dev/reference/protobuf/textformat-spec) to store your protocol buffer messages, the changes that you can make in your proto definition are different.

​	如果您使用 JSON 或 proto 文本格式来存储协议缓冲区消息，则您可以在 proto 定义中进行的更改有所不同。

Check [Proto Best Practices](https://protobuf.dev/programming-guides/dos-donts) and the following rules:

​	查看 Proto 最佳做法和以下规则：

- Don’t change the field numbers for any existing fields. “Changing” the field number is equivalent to deleting the field and adding a new field with the same type. If you want to renumber a field, see the instructions for [deleting a field](https://protobuf.dev/programming-guides/proto3/#deleting).
- 不要更改任何现有字段的字段编号。“更改”字段编号等同于删除该字段并添加一个具有相同类型的新字段。如果您想重新编号一个字段，请参阅删除字段的说明。
- If you add new fields, any messages serialized by code using your “old” message format can still be parsed by your new generated code. You should keep in mind the [default values](https://protobuf.dev/programming-guides/proto3/#default) for these elements so that new code can properly interact with messages generated by old code. Similarly, messages created by your new code can be parsed by your old code: old binaries simply ignore the new field when parsing. See the [Unknown Fields](https://protobuf.dev/programming-guides/proto3/#unknowns) section for details.
- 如果您添加新字段，则使用“旧”消息格式的代码序列化的任何消息仍可由新生成的代码解析。您应该记住这些元素的默认值，以便新代码可以与旧代码生成的消息正确交互。同样，旧代码可以解析由新代码创建的消息：旧二进制文件在解析时会简单地忽略新字段。有关详细信息，请参阅未知字段部分。
- Fields can be removed, as long as the field number is not used again in your updated message type. You may want to rename the field instead, perhaps adding the prefix “OBSOLETE_”, or make the field number [reserved](https://protobuf.dev/programming-guides/proto3/#fieldreserved), so that future users of your `.proto` can’t accidentally reuse the number._
- 只要在更新的消息类型中不再使用字段编号，就可以删除字段。您可能希望重命名字段，也许添加前缀“OBSOLETE_”，或使字段编号保留，以便您 `.proto` 的未来用户无法意外地重复使用该编号。
- `int32`, `uint32`, `int64`, `uint64`, and `bool` are all compatible – this means you can change a field from one of these types to another without breaking forwards- or backwards-compatibility. If a number is parsed from the wire which doesn’t fit in the corresponding type, you will get the same effect as if you had cast the number to that type in C++ (for example, if a 64-bit number is read as an int32, it will be truncated to 32 bits).
- `int32` 、 `uint32` 、 `int64` 、 `uint64` 和 `bool` 都兼容——这意味着您可以将字段从其中一种类型更改为另一种类型，而不会破坏向前或向后兼容性。如果从数据线解析的数字不适合相应的类型，您将获得与在 C++ 中将数字强制转换为该类型时相同的效果（例如，如果将 64 位数字读作 int32，它将被截断为 32 位）。
- `sint32` and `sint64` are compatible with each other but are *not* compatible with the other integer types.
- `sint32` 和 `sint64` 相互兼容，但与其他整数类型不兼容。
- `string` and `bytes` are compatible as long as the bytes are valid UTF-8.
- `string` 和 `bytes` 兼容，只要字节是有效的 UTF-8。
- Embedded messages are compatible with `bytes` if the bytes contain an encoded version of the message.
- 嵌入式消息与 `bytes` 兼容，如果字节包含消息的编码版本。
- `fixed32` is compatible with `sfixed32`, and `fixed64` with `sfixed64`.
- `fixed32` 与 `sfixed32` 兼容， `fixed64` 与 `sfixed64` 兼容。
- For `string`, `bytes`, and message fields, `optional` is compatible with `repeated`. Given serialized data of a repeated field as input, clients that expect this field to be `optional` will take the last input value if it’s a primitive type field or merge all input elements if it’s a message type field. Note that this is **not** generally safe for numeric types, including bools and enums. Repeated fields of numeric types can be serialized in the [packed](https://protobuf.dev/programming-guides/encoding#packed) format, which will not be parsed correctly when an `optional` field is expected.
- 对于 `string` 、 `bytes` 和消息字段， `optional` 与 `repeated` 兼容。给定重复字段的序列化数据作为输入，如果该字段是原始类型字段，则期望该字段为 `optional` 的客户端将采用最后一个输入值；如果该字段是消息类型字段，则合并所有输入元素。请注意，对于数字类型（包括布尔值和枚举）来说，这通常是不安全的。数字类型的重复字段可以采用压缩格式序列化，当期望 `optional` 字段时，这种格式不会被正确解析。
- `enum` is compatible with `int32`, `uint32`, `int64`, and `uint64` in terms of wire format (note that values will be truncated if they don’t fit). However, be aware that client code may treat them differently when the message is deserialized: for example, unrecognized proto3 `enum` types will be preserved in the message, but how this is represented when the message is deserialized is language-dependent. Int fields always just preserve their value.
- `enum` 在线格式方面与 `int32` 、 `uint32` 、 `int64` 和 `uint64` 兼容（请注意，如果值不合适，它们将被截断）。但是，请注意，当消息被反序列化时，客户端代码可能会以不同的方式处理它们：例如，无法识别的 proto3 `enum` 类型将保留在消息中，但当消息被反序列化时如何表示它取决于语言。Int 字段始终只保留其值。
- Changing a single `optional` field or extension into a member of a **new** `oneof` is binary compatible, however for some languages (notably, Go) the generated code’s API will change in incompatible ways. For this reason, Google does not make such changes in its public APIs, as documented in [AIP-180](https://google.aip.dev/180#moving-into-oneofs). With the same caveat about source-compatibility, moving multiple fields into a new `oneof` may be safe if you are sure that no code sets more than one at a time. Moving fields into an existing `oneof` is not safe. Likewise, changing a single field `oneof` to an `optional` field or extension is safe.
- 将单个 `optional` 字段或扩展更改为新 `oneof` 的成员在二进制上是兼容的，但是对于某些语言（尤其是 Go），生成的代码的 API 将以不兼容的方式更改。因此，Google 不会在其公开 API 中进行此类更改，如 AIP-180 中所述。对于源兼容性有同样的警告，如果您确定没有代码一次设置多个字段，则可以将多个字段移动到新的 `oneof` 中。将字段移动到现有的 `oneof` 中是不安全的。同样，将单个字段 `oneof` 更改为 `optional` 字段或扩展是安全的。
- Changing a field between a `map<K, V>` and the corresponding `repeated` message field is binary compatible (see [Maps](https://protobuf.dev/programming-guides/proto3/#maps), below, for the message layout and other restrictions). However, the safety of the change is application-dependent: when deserializing and reserializing a message, clients using the `repeated` field definition will produce a semantically identical result; however, clients using the `map` field definition may reorder entries and drop entries with duplicate keys.
- 在 `map<K, V>` 和相应的 `repeated` 消息字段之间更改字段是二进制兼容的（有关消息布局和其他限制，请参阅下方的映射）。但是，更改的安全性取决于应用程序：在对消息进行反序列化和重新序列化时，使用 `repeated` 字段定义的客户端将生成语义上相同的结果；但是，使用 `map` 字段定义的客户端可能会重新排序条目并删除具有重复键的条目。

## 未知字段 Unknown Fields 

Unknown fields are well-formed protocol buffer serialized data representing fields that the parser does not recognize. For example, when an old binary parses data sent by a new binary with new fields, those new fields become unknown fields in the old binary.

​	未知字段是格式良好的协议缓冲区序列化数据，表示解析器无法识别的字段。例如，当旧二进制文件解析由具有新字段的新二进制文件发送的数据时，这些新字段在旧二进制文件中变为未知字段。

Proto3 messages preserve unknown fields and includes them during parsing and in the serialized output, which matches proto2 behavior.

​	Proto3 消息保留未知字段，并在解析期间和序列化输出中包含它们，这与 proto2 行为相匹配。

## Any

The `Any` message type lets you use messages as embedded types without having their .proto definition. An `Any` contains an arbitrary serialized message as `bytes`, along with a URL that acts as a globally unique identifier for and resolves to that message’s type. To use the `Any` type, you need to [import](https://protobuf.dev/programming-guides/proto3/#other) `google/protobuf/any.proto`.

​	使用 `Any` 消息类型，您可以将消息用作嵌入式类型，而无需其 .proto 定义。 `Any` 包含一个任意序列化的消息作为 `bytes` ，以及一个充当该消息类型的全局唯一标识符并解析为该消息类型的 URL。要使用 `Any` 类型，您需要导入 `google/protobuf/any.proto` 。

```proto
import "google/protobuf/any.proto";

message ErrorStatus {
  string message = 1;
  repeated google.protobuf.Any details = 2;
}
```

The default type URL for a given message type is `type.googleapis.com/_packagename_._messagename_`.

​	给定消息类型的默认类型 URL 为 `type.googleapis.com/_packagename_._messagename_` 。

Different language implementations will support runtime library helpers to pack and unpack `Any` values in a typesafe manner – for example, in Java, the `Any` type will have special `pack()` and `unpack()` accessors, while in C++ there are `PackFrom()` and `UnpackTo()` methods:

​	不同的语言实现将支持运行时库帮助程序，以类型安全的方式打包和解包 `Any` 值 - 例如，在 Java 中， `Any` 类型将具有特殊的 `pack()` 和 `unpack()` 访问器，而在 C++ 中有 `PackFrom()` 和 `UnpackTo()` 方法：

```c++
// Storing an arbitrary message type in Any.
NetworkErrorDetails details = ...;
ErrorStatus status;
status.add_details()->PackFrom(details);

// Reading an arbitrary message from Any.
ErrorStatus status = ...;
for (const google::protobuf::Any& detail : status.details()) {
  if (detail.Is<NetworkErrorDetails>()) {
    NetworkErrorDetails network_error;
    detail.UnpackTo(&network_error);
    ... processing network_error ...
  }
}
```

**Currently the runtime libraries for working with `Any` types are under development**.

​	目前，用于处理 `Any` 类型的运行时库正在开发中。

The `Any` message types can hold arbitrary proto3 messages, similar to proto2 messages which can allow [extensions](https://protobuf.dev/programming-guides/proto2#extensions).

​	`Any` 消息类型可以保存任意 proto3 消息，类似于可以允许扩展的 proto2 消息。

## Oneof

If you have a message with many fields and where at most one field will be set at the same time, you can enforce this behavior and save memory by using the oneof feature.

​	如果您有一个包含许多字段的消息，并且最多只能同时设置一个字段，则可以使用 oneof 功能来强制执行此行为并节省内存。

Oneof fields are like regular fields except all the fields in a oneof share memory, and at most one field can be set at the same time. Setting any member of the oneof automatically clears all the other members. You can check which value in a oneof is set (if any) using a special `case()` or `WhichOneof()` method, depending on your chosen language.

​	其中一个字段类似于常规字段，但 oneof 中的所有字段共享内存，并且最多只能同时设置一个字段。设置 oneof 的任何成员都会自动清除所有其他成员。您可以使用特殊的 `case()` 或 `WhichOneof()` 方法（取决于您选择的语言）来检查 oneof 中设置了哪个值（如果有）。

Note that if *multiple values are set, the last set value as determined by the order in the proto will overwrite all previous ones*.

​	请注意，如果设置了多个值，则按 proto 中的顺序确定的最后一个设置值将覆盖所有前一个值。

Field numbers for oneof fields must be unique within the enclosing message.

​	oneof 字段的字段号在封闭消息中必须是唯一的。

### 使用 Oneof Using Oneof 

To define a oneof in your `.proto` you use the `oneof` keyword followed by your oneof name, in this case `test_oneof`:

​	要在 `.proto` 中定义 oneof，请使用 `oneof` 关键字，后跟您的 oneof 名称，在本例中为 `test_oneof` ：

```proto
message SampleMessage {
  oneof test_oneof {
    string name = 4;
    SubMessage sub_message = 9;
  }
}
```

You then add your oneof fields to the oneof definition. You can add fields of any type, except `map` fields and `repeated` fields. If you need to add a repeated field to a oneof, you can use a message containing the repeated field.

​	然后将您的 oneof 字段添加到 oneof 定义中。您可以添加任何类型的字段，但 `map` 字段和 `repeated` 字段除外。如果您需要将重复字段添加到 oneof，则可以使用包含重复字段的消息。

In your generated code, oneof fields have the same getters and setters as regular fields. You also get a special method for checking which value (if any) in the oneof is set. You can find out more about the oneof API for your chosen language in the relevant [API reference](https://protobuf.dev/reference/).

​	在生成的代码中，oneof 字段具有与常规字段相同的 getter 和 setter。您还可以获得一种特殊的方法来检查 oneof 中设置了哪个值（如果有）。您可以在相关 API 参考中了解有关您所选语言的 oneof API 的更多信息。

### Oneof 功能 Oneof Features 

- Setting a oneof field will automatically clear all other members of the oneof. So if you set several oneof fields, only the *last* field you set will still have a value.
  
- 设置 oneof 字段会自动清除 oneof 的所有其他成员。因此，如果您设置了多个 oneof 字段，则只有您设置的最后一个字段仍具有值。
  
  ```c++
  SampleMessage message;
  message.set_name("name");
  CHECK_EQ(message.name(), "name");
  // Calling mutable_sub_message() will clear the name field and will set
  // sub_message to a new instance of SubMessage with none of its fields set.
  message.mutable_sub_message();
  CHECK(message.name().empty());
  ```
  
- If the parser encounters multiple members of the same oneof on the wire, only the last member seen is used in the parsed message.
  
- 如果解析器在数据线路上遇到同一 oneof 的多个成员，则仅在已解析的消息中使用最后看到的成员。
  
- A oneof cannot be `repeated`.
  
- oneof 不能是 `repeated` 。
  
- Reflection APIs work for oneof fields.
  
- 反射 API 适用于 oneof 字段。
  
- If you set a oneof field to the default value (such as setting an int32 oneof field to 0), the “case” of that oneof field will be set, and the value will be serialized on the wire.
  
- 如果您将 oneof 字段设置为默认值（例如将 int32 oneof 字段设置为 0），则会设置该 oneof 字段的“case”，并且该值将在数据线路上序列化。
  
- If you’re using C++, make sure your code doesn’t cause memory crashes. The following sample code will crash because `sub_message` was already deleted by calling the `set_name()` method.
  
- 如果您使用的是 C++，请确保您的代码不会导致内存崩溃。以下示例代码会崩溃，因为 `sub_message` 已通过调用 `set_name()` 方法删除。
  
  ```c++
  SampleMessage message;
  SubMessage* sub_message = message.mutable_sub_message();
  message.set_name("name");      // Will delete sub_message
  sub_message->set_...            // Crashes here
  ```
  
- Again in C++, if you `Swap()` two messages with oneofs, each message will end up with the other’s oneof case: in the example below, `msg1` will have a `sub_message` and `msg2` will have a `name`.
  
- 同样在 C++ 中，如果您 `Swap()` 两个带有 oneof 的消息，则每个消息最终都会具有另一个消息的 oneof case：在以下示例中， `msg1` 将具有 `sub_message` ， `msg2` 将具有 `name` 。
  
  ```c++
  SampleMessage msg1;
  msg1.set_name("name");
  SampleMessage msg2;
  msg2.mutable_sub_message();
  msg1.swap(&msg2);
  CHECK(msg1.has_sub_message());
  CHECK_EQ(msg2.name(), "name");
  ```

### 向后兼容性问题 Backwards-compatibility issues 

Be careful when adding or removing oneof fields. If checking the value of a oneof returns `None`/`NOT_SET`, it could mean that the oneof has not been set or it has been set to a field in a different version of the oneof. There is no way to tell the difference, since there’s no way to know if an unknown field on the wire is a member of the oneof.

​	添加或移除 oneof 字段时要小心。如果检查 oneof 的值返回 `None` / `NOT_SET` ，则可能意味着 oneof 尚未设置或已设置为 oneof 不同版本中的字段。无法区分两者，因为无法知道线路上未知的字段是否是 oneof 的成员。

#### 标签重用问题 Tag Reuse Issues 

- **Move fields into or out of a oneof**: You may lose some of your information (some fields will be cleared) after the message is serialized and parsed. However, you can safely move a single field into a **new** oneof and may be able to move multiple fields if it is known that only one is ever set. See [Updating A Message Type](https://protobuf.dev/programming-guides/proto3/#updating) for further details.
- 将字段移入或移出 oneof：在消息序列化和解析后，您可能会丢失部分信息（某些字段将被清除）。但是，您可以安全地将单个字段移入新的 oneof，并且如果已知仅设置了一个字段，则可以移动多个字段。有关更多详细信息，请参阅更新消息类型。
- **Delete a oneof field and add it back**: This may clear your currently set oneof field after the message is serialized and parsed.
- 删除 oneof 字段并将其添加回来：在消息序列化和解析后，这可能会清除您当前设置的 oneof 字段。
- **Split or merge oneof**: This has similar issues to moving regular fields.
- 拆分或合并 oneof：这与移动常规字段的问题类似。

## 映射 Maps 

If you want to create an associative map as part of your data definition, protocol buffers provides a handy shortcut syntax:

​	如果您想在数据定义中创建关联映射，则协议缓冲区提供了一个方便的快捷语法：

```proto
map<key_type, value_type> map_field = N;
```

…where the `key_type` can be any integral or string type (so, any [scalar](https://protobuf.dev/programming-guides/proto3/#scalar) type except for floating point types and `bytes`). Note that enum is not a valid `key_type`. The `value_type` can be any type except another map.

​	…其中 `key_type` 可以是任何整数或字符串类型（因此，除了浮点类型和 `bytes` 之外的任何标量类型）。请注意，枚举不是有效的 `key_type` 。 `value_type` 可以是任何类型，除了另一个映射。

So, for example, if you wanted to create a map of projects where each `Project` message is associated with a string key, you could define it like this:

​	因此，例如，如果您想创建一个项目映射，其中每个 `Project` 消息都与字符串键相关联，则可以像这样定义它：

```proto
map<string, Project> projects = 3;
```

### 映射功能 Maps Features 

- Map fields cannot be `repeated`.
- 映射字段不能是 `repeated` 。
- Wire format ordering and map iteration ordering of map values are undefined, so you cannot rely on your map items being in a particular order.
- 映射值的线格式顺序和映射迭代顺序是未定义的，因此您无法依赖于映射项按特定顺序排列。
- When generating text format for a `.proto`, maps are sorted by key. Numeric keys are sorted numerically.
- 为 `.proto` 生成文本格式时，会按键对映射进行排序。数字键按数字顺序排序。
- When parsing from the wire or when merging, if there are duplicate map keys the last key seen is used. When parsing a map from text format, parsing may fail if there are duplicate keys.
- 从线路解析或合并时，如果存在重复的映射键，则使用看到的最后一个键。从文本格式解析映射时，如果存在重复的键，则解析可能会失败。
- If you provide a key but no value for a map field, the behavior when the field is serialized is language-dependent. In C++, Java, Kotlin, and Python the default value for the type is serialized, while in other languages nothing is serialized.
- 如果您为映射字段提供键但未提供值，则字段序列化时的行为取决于语言。在 C++、Java、Kotlin 和 Python 中，会序列化该类型的默认值，而在其他语言中则不会序列化任何内容。

The generated map API is currently available for all supported languages. You can find out more about the map API for your chosen language in the relevant [API reference](https://protobuf.dev/reference/).

​	生成的映射 API 目前可用于所有受支持的语言。您可以在相关 API 参考中找到有关您所选语言的映射 API 的更多信息。

### 向后兼容性 Backwards compatibility 

The map syntax is equivalent to the following on the wire, so protocol buffers implementations that do not support maps can still handle your data:

​	映射语法等同于线上的以下内容，因此不支持映射的协议缓冲区实现仍然可以处理您的数据：

```proto
message MapFieldEntry {
  key_type key = 1;
  value_type value = 2;
}

repeated MapFieldEntry map_field = N;
```

Any protocol buffers implementation that supports maps must both produce and accept data that can be accepted by the earlier definition.

​	任何支持映射的协议缓冲区实现都必须生成和接受可被早期定义接受的数据。

## 包 Packages 

You can add an optional `package` specifier to a `.proto` file to prevent name clashes between protocol message types.

​	您可以在 `.proto` 文件中添加一个可选的 `package` 说明符，以防止协议消息类型之间的名称冲突。

```proto
package foo.bar;
message Open { ... }
```

You can then use the package specifier when defining fields of your message type:

​	然后，您可以在定义消息类型的字段时使用包说明符：

```proto
message Foo {
  ...
  foo.bar.Open open = 1;
  ...
}
```

The way a package specifier affects the generated code depends on your chosen language:

​	包说明符影响生成代码的方式取决于您选择的语言：

- In **C++** the generated classes are wrapped inside a C++ namespace. For example, `Open` would be in the namespace `foo::bar`.
- 在 C++ 中，生成的类被包装在 C++ 命名空间内。例如， `Open` 将位于命名空间 `foo::bar` 中。
- In **Java** and **Kotlin**, the package is used as the Java package, unless you explicitly provide an `option java_package` in your `.proto` file.
- 在 Java 和 Kotlin 中，该包用作 Java 包，除非您在 `.proto` 文件中明确提供 `option java_package` 。
- In **Python**, the `package` directive is ignored, since Python modules are organized according to their location in the file system.
- 在 Python 中， `package` 指令被忽略，因为 Python 模块是根据它们在文件系统中的位置组织的。
- In **Go**, the `package` directive is ignored, and the generated `.pb.go` file is in the package named after the corresponding `go_proto_library` Bazel rule. For open source projects, you **must** provide either a `go_package` option or set the Bazel `-M` flag.
- 在 Go 中， `package` 指令被忽略，生成的 `.pb.go` 文件位于以相应的 `go_proto_library` Bazel 规则命名的包中。对于开源项目，您必须提供 `go_package` 选项或设置 Bazel `-M` 标志。
- In **Ruby**, the generated classes are wrapped inside nested Ruby namespaces, converted to the required Ruby capitalization style (first letter capitalized; if the first character is not a letter, `PB_` is prepended). For example, `Open` would be in the namespace `Foo::Bar`.
- 在 Ruby 中，生成的类被包装在嵌套的 Ruby 命名空间中，转换为所需的 Ruby 大写样式（第一个字母大写；如果第一个字符不是字母，则添加 `PB_` 作为前缀）。例如， `Open` 将位于命名空间 `Foo::Bar` 中。
- In **PHP** the package is used as the namespace after converting to PascalCase, unless you explicitly provide an `option php_namespace` in your `.proto` file. For example, `Open` would be in the namespace `Foo\Bar`.
- 在 PHP 中，包在转换为 PascalCase 后用作命名空间，除非您在 `.proto` 文件中明确提供 `option php_namespace` 。例如， `Open` 将位于命名空间 `Foo\Bar` 中。
- In **C#** the package is used as the namespace after converting to PascalCase, unless you explicitly provide an `option csharp_namespace` in your `.proto` file. For example, `Open` would be in the namespace `Foo.Bar`.
- 在 C# 中，包在转换为 PascalCase 后用作命名空间，除非您在 `.proto` 文件中明确提供 `option csharp_namespace` 。例如， `Open` 将位于命名空间 `Foo.Bar` 中。

Note that even when the `package` directive does not directly affect the generated code, for example in Python, it is still strongly recommended to specify the package for the `.proto` file, as otherwise it may lead to naming conflicts in descriptors and make the proto not portable for other languages.

​	请注意，即使 `package` 指令不会直接影响生成的代码（例如在 Python 中），强烈建议您仍然为 `.proto` 文件指定包，否则可能会导致描述符中的命名冲突，并使 proto 无法移植到其他语言。

### 包和名称解析 Packages and Name Resolution 

Type name resolution in the protocol buffer language works like C++: first the innermost scope is searched, then the next-innermost, and so on, with each package considered to be “inner” to its parent package. A leading ‘.’ (for example, `.foo.bar.Baz`) means to start from the outermost scope instead.

​	协议缓冲区语言中的类型名称解析的工作方式类似于 C++：首先搜索最内部的作用域，然后搜索次内部的作用域，依此类推，每个包都被视为其父包的“内部”。前导“.”（例如， `.foo.bar.Baz` ）表示从最外部的作用域开始。

The protocol buffer compiler resolves all type names by parsing the imported `.proto` files. The code generator for each language knows how to refer to each type in that language, even if it has different scoping rules.

​	协议缓冲区编译器通过解析导入的 `.proto` 文件来解析所有类型名称。每种语言的代码生成器都知道如何引用该语言中的每种类型，即使它具有不同的范围规则。

## 定义服务 Defining Services 

If you want to use your message types with an RPC (Remote Procedure Call) system, you can define an RPC service interface in a `.proto` file and the protocol buffer compiler will generate service interface code and stubs in your chosen language. So, for example, if you want to define an RPC service with a method that takes your `SearchRequest` and returns a `SearchResponse`, you can define it in your `.proto` file as follows:

​	如果您想将消息类型与 RPC（远程过程调用）系统配合使用，可以在 `.proto` 文件中定义一个 RPC 服务接口，协议缓冲区编译器将在您选择的语言中生成服务接口代码和存根。因此，例如，如果您想定义一个具有采用 `SearchRequest` 并返回 `SearchResponse` 的方法的 RPC 服务，则可以在 `.proto` 文件中按如下方式定义它：

```proto
service SearchService {
  rpc Search(SearchRequest) returns (SearchResponse);
}
```

The most straightforward RPC system to use with protocol buffers is [gRPC](https://grpc.io/): a language- and platform-neutral open source RPC system developed at Google. gRPC works particularly well with protocol buffers and lets you generate the relevant RPC code directly from your `.proto` files using a special protocol buffer compiler plugin.

​	与协议缓冲区配合使用最直接的 RPC 系统是 gRPC：一种在 Google 开发的语言和平台无关的开源 RPC 系统。gRPC 与协议缓冲区配合得特别好，并允许您使用特殊的协议缓冲区编译器插件直接从 `.proto` 文件生成相关的 RPC 代码。

If you don’t want to use gRPC, it’s also possible to use protocol buffers with your own RPC implementation. You can find out more about this in the [Proto2 Language Guide](https://protobuf.dev/programming-guides/proto2#services).

​	如果您不想使用 gRPC，也可以将协议缓冲区与您自己的 RPC 实现配合使用。您可以在 Proto2 语言指南中了解更多相关信息。

There are also a number of ongoing third-party projects to develop RPC implementations for Protocol Buffers. For a list of links to projects we know about, see the [third-party add-ons wiki page](https://github.com/protocolbuffers/protobuf/blob/master/docs/third_party.md).

​	还有一些正在进行的第三方项目来为 Protocol Buffers 开发 RPC 实现。有关我们了解的项目的链接列表，请参阅第三方加载项 wiki 页面。

## JSON 映射 JSON Mapping 

Proto3 supports a canonical encoding in JSON, making it easier to share data between systems. The encoding is described on a type-by-type basis in the table below.

​	Proto3 支持 JSON 中的规范编码，从而更轻松地在系统之间共享数据。编码在下面的表格中按类型逐一描述。

When parsing JSON-encoded data into a protocol buffer, if a value is missing or if its value is `null`, it will be interpreted as the corresponding [default value](https://protobuf.dev/programming-guides/proto3/#default).

​	在将 JSON 编码的数据解析为协议缓冲区时，如果某个值缺失或其值为 `null` ，它将被解释为相应的默认值。

When generating JSON-encoded output from a protocol buffer, if a protobuf field has the default value and if the field doesn’t support field presence, it will be omitted from the output by default. An implementation may provide options to include fields with default values in the output.

​	在从协议缓冲区生成 JSON 编码的输出时，如果某个 protobuf 字段具有默认值，并且该字段不支持字段显示，则默认情况下它将从输出中省略。实现可以提供选项，以在输出中包含具有默认值字段。

A proto3 field that is defined with the `optional` keyword supports field presence. Fields that have a value set and that support field presence always include the field value in the JSON-encoded output, even if it is the default value.

​	使用 `optional` 关键字定义的 proto3 字段支持字段显示。具有值设置且支持字段显示的字段始终在 JSON 编码的输出中包含字段值，即使它是默认值。

| proto3                                        | JSON                        | JSON example JSON 示例                      | Notes 备注                                                   |
| --------------------------------------------- | --------------------------- | ------------------------------------------- | ------------------------------------------------------------ |
| message                                       | object 对象                 | `{"fooBar": v, "g": null, ...}`             | Generates JSON objects. Message field names are mapped to lowerCamelCase and become JSON object keys. If the `json_name` field option is specified, the specified value will be used as the key instead. Parsers accept both the lowerCamelCase name (or the one specified by the `json_name` option) and the original proto field name. `null` is an accepted value for all field types and treated as the default value of the corresponding field type. However, `null` cannot be used for the `json_name` value. For more on why, see [Stricter validation for json_name](https://protobuf.dev/news/2023-04-28#json-name). 生成 JSON 对象。消息字段名映射到 lowerCamelCase 并成为 JSON 对象键。如果指定了 `json_name` 字段选项，则会使用指定的值作为键。解析器接受 lowerCamelCase 名称（或由 `json_name` 选项指定的那个名称）和原始的 proto 字段名。对于所有字段类型， `null` 都是一个可接受的值，并且被视为相应字段类型的默认值。但是， `null` 不能用于 `json_name` 值。有关原因的更多信息，请参阅 json_name 的更严格验证。 |
| enum                                          | string 字符串               | `"FOO_BAR"`                                 | The name of the enum value as specified in proto is used. Parsers accept both enum names and integer values. 使用 proto 中指定的枚举值名称。解析器接受枚举名称和整数值。 |
| map<K,V>                                      | object 对象                 | `{"k": v, ...}`                             | All keys are converted to strings. 所有键都转换为字符串。    |
| repeated V                                    | array 数组                  | `[v, ...]`                                  | `null` is accepted as the empty list `[]`. `null` 被接受为空列表 `[]` 。 |
| bool                                          | true, false true、false     | `true, false`                               |                                                              |
| string 字符串                                 | string 字符串               | `"Hello World!"`                            |                                                              |
| bytes 字节                                    | base64 string base64 字符串 | `"YWJjMTIzIT8kKiYoKSctPUB+"`                | JSON value will be the data encoded as a string using standard base64 encoding with paddings. Either standard or URL-safe base64 encoding with/without paddings are accepted. JSON 值将是使用标准 base64 编码（带填充）对数据进行编码的字符串。接受带/不带填充的标准或 URL 安全 base64 编码。 |
| int32, fixed32, uint32 int32、fixed32、uint32 | number 数字                 | `1, -10, 0`                                 | JSON value will be a decimal number. Either numbers or strings are accepted. JSON 值将是十进制数。接受数字或字符串。 |
| int64, fixed64, uint64 int64、fixed64、uint64 | string 字符串               | `"1", "-10"`                                | JSON value will be a decimal string. Either numbers or strings are accepted. JSON 值将是十进制字符串。接受数字或字符串。 |
| float, double float、double                   | number 数字                 | `1.1, -10.0, 0, "NaN", "Infinity"`          | JSON value will be a number or one of the special string values "NaN", "Infinity", and "-Infinity". Either numbers or strings are accepted. Exponent notation is also accepted. -0 is considered equivalent to 0. JSON 值将是数字或特殊字符串值“NaN”、“Infinity”和“-Infinity”之一。接受数字或字符串。也接受指数表示法。-0 被视为等于 0。 |
| Any                                           | `object`                    | `{"@type": "url", "f": v, ... }`            | If the `Any` contains a value that has a special JSON mapping, it will be converted as follows: `{"@type": xxx, "value": yyy}`. Otherwise, the value will be converted into a JSON object, and the `"@type"` field will be inserted to indicate the actual data type. 如果 `Any` 包含具有特殊 JSON 映射的值，则会按如下方式进行转换： `{"@type": xxx, "value": yyy}` 。否则，该值将被转换为 JSON 对象，并将插入 `"@type"` 字段以指示实际数据类型。 |
| Timestamp 时间戳                              | string 字符串               | `"1972-01-01T10:00:20.021Z"`                | Uses RFC 3339, where generated output will always be Z-normalized and uses 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. 使用 RFC 3339，其中生成的输出将始终是 Z 归一化的，并使用 0、3、6 或 9 个小数位。也接受除“Z”之外的偏移量。 |
| Duration 持续时间                             | string 字符串               | `"1.000340012s", "1s"`                      | Generated output always contains 0, 3, 6, or 9 fractional digits, depending on required precision, followed by the suffix "s". Accepted are any fractional digits (also none) as long as they fit into nano-seconds precision and the suffix "s" is required. 生成的输出始终包含 0、3、6 或 9 个小数位，具体取决于所需精度，后跟后缀“s”。只要符合纳秒精度且需要后缀“s”，则接受任何小数位（也包括无小数位）。 |
| Struct 结构                                   | `object`                    | `{ ... }`                                   | Any JSON object. See `struct.proto`. 任何 JSON 对象。请参阅 `struct.proto` 。 |
| Wrapper types 包装器类型                      | various types 各种类型      | `2, "2", "foo", true, "true", null, 0, ...` | Wrappers use the same representation in JSON as the wrapped primitive type, except that `null` is allowed and preserved during data conversion and transfer. 包装器在 JSON 中使用与包装的原始类型相同的表示形式，但 `null` 在数据转换和传输过程中允许并保留。 |
| FieldMask 字段掩码                            | string 字符串               | `"f.fooBar,h"`                              | See `field_mask.proto`. 请参阅 `field_mask.proto` 。         |
| ListValue 列表值                              | array 数组                  | `[foo, bar, ...]`                           |                                                              |
| Value 值                                      | value                       |                                             | Any JSON value. Check [google.protobuf.Value](https://protobuf.dev/reference/protobuf/google.protobuf#value) for details. 任何 JSON 值。有关详细信息，请查看 google.protobuf.Value。 |
| NullValue                                     | null                        |                                             | JSON null                                                    |
| Empty                                         | object 对象                 | `{}`                                        | An empty JSON object 一个空的 JSON 对象                      |

### JSON 选项 JSON Options 

A proto3 JSON implementation may provide the following options:

​	proto3 JSON 实现可以提供以下选项：

- **Emit fields with default values**: Fields with default values are omitted by default in proto3 JSON output. An implementation may provide an option to override this behavior and output fields with their default values.
- 输出具有默认值的字段：默认情况下，proto3 JSON 输出中会忽略具有默认值的字段。实现可以提供一个选项来覆盖此行为并输出具有其默认值的字段。
- **Ignore unknown fields**: Proto3 JSON parser should reject unknown fields by default but may provide an option to ignore unknown fields in parsing.
- 忽略未知字段：默认情况下，Proto3 JSON 解析器应拒绝未知字段，但可以提供一个选项来忽略解析中的未知字段。
- **Use proto field name instead of lowerCamelCase name**: By default proto3 JSON printer should convert the field name to lowerCamelCase and use that as the JSON name. An implementation may provide an option to use proto field name as the JSON name instead. Proto3 JSON parsers are required to accept both the converted lowerCamelCase name and the proto field name.
- 使用 proto 字段名称而不是 lowerCamelCase 名称：默认情况下，proto3 JSON 打印机应将字段名称转换为 lowerCamelCase 并将其用作 JSON 名称。实现可以提供一个选项来使用 proto 字段名称作为 JSON 名称。Proto3 JSON 解析器必须同时接受转换后的 lowerCamelCase 名称和 proto 字段名称。
- **Emit enum values as integers instead of strings**: The name of an enum value is used by default in JSON output. An option may be provided to use the numeric value of the enum value instead.
- 将枚举值作为整数而不是字符串输出：默认情况下，JSON 输出中使用枚举值名称。可以提供一个选项来使用枚举值的数字值。

## 选项 Options 

Individual declarations in a `.proto` file can be annotated with a number of *options*. Options do not change the overall meaning of a declaration, but may affect the way it is handled in a particular context. The complete list of available options is defined in [`/google/protobuf/descriptor.proto`](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/descriptor.proto).

​	`.proto` 文件中的各个声明都可以用多个选项进行注释。选项不会改变声明的整体含义，但可能会影响其在特定上下文中的处理方式。可用选项的完整列表在 `/google/protobuf/descriptor.proto` 中定义。

Some options are file-level options, meaning they should be written at the top-level scope, not inside any message, enum, or service definition. Some options are message-level options, meaning they should be written inside message definitions. Some options are field-level options, meaning they should be written inside field definitions. Options can also be written on enum types, enum values, oneof fields, service types, and service methods; however, no useful options currently exist for any of these.

​	某些选项是文件级选项，这意味着它们应该写在顶级作用域，而不是任何消息、枚举或服务定义中。某些选项是消息级选项，这意味着它们应该写在消息定义中。某些选项是字段级选项，这意味着它们应该写在字段定义中。选项还可以写在枚举类型、枚举值、oneof 字段、服务类型和服务方法上；但是，目前对于所有这些选项都没有任何有用的选项。

Here are a few of the most commonly used options:

​	以下是一些最常用的选项：

- `java_package` (file option): The package you want to use for your generated Java/Kotlin classes. If no explicit `java_package` option is given in the `.proto` file, then by default the proto package (specified using the “package” keyword in the `.proto` file) will be used. However, proto packages generally do not make good Java packages since proto packages are not expected to start with reverse domain names. If not generating Java or Kotlin code, this option has no effect.
  
- `java_package` （文件选项）：您想要用于生成的 Java/Kotlin 类的包。如果在 `.proto` 文件中没有给出明确的 `java_package` 选项，那么默认情况下将使用 proto 包（在 `.proto` 文件中使用“package”关键字指定）。但是，proto 包通常不会成为好的 Java 包，因为 proto 包预计不会以反向域名开头。如果不生成 Java 或 Kotlin 代码，此选项无效。
  
  ```proto
  option java_package = "com.example.foo";
  ```
  
- `java_outer_classname` (file option): The class name (and hence the file name) for the wrapper Java class you want to generate. If no explicit `java_outer_classname` is specified in the `.proto` file, the class name will be constructed by converting the `.proto` file name to camel-case (so `foo_bar.proto` becomes `FooBar.java`). If the `java_multiple_files` option is disabled, then all other classes/enums/etc. generated for the `.proto` file will be generated *within* this outer wrapper Java class as nested classes/enums/etc. If not generating Java code, this option has no effect.
  
- `java_outer_classname` （文件选项）：要生成的包装器 Java 类的类名（因此也是文件名）。如果在 `.proto` 文件中未指定显式 `java_outer_classname` ，则类名将通过将 `.proto` 文件名转换为驼峰式来构造（因此 `foo_bar.proto` 变为 `FooBar.java` ）。如果禁用 `java_multiple_files` 选项，则为 `.proto` 文件生成的所有其他类/枚举等将在该外部包装器 Java 类中作为嵌套类/枚举等生成。如果不生成 Java 代码，则此选项无效。
  
  ```proto
  option java_outer_classname = "Ponycopter";
  ```
  
- `java_multiple_files` (file option): If false, only a single `.java` file will be generated for this `.proto` file, and all the Java classes/enums/etc. generated for the top-level messages, services, and enumerations will be nested inside of an outer class (see `java_outer_classname`). If true, separate `.java` files will be generated for each of the Java classes/enums/etc. generated for the top-level messages, services, and enumerations, and the wrapper Java class generated for this `.proto` file won’t contain any nested classes/enums/etc. This is a Boolean option which defaults to `false`. If not generating Java code, this option has no effect.
  
- `java_multiple_files` （文件选项）：如果为 false，则仅为此 `.proto` 文件生成一个 `.java` 文件，并且为顶级消息、服务和枚举生成的 Java 类/枚举等都将嵌套在外部类中（请参阅 `java_outer_classname` ）。如果为 true，则将为为顶级消息、服务和枚举生成的每个 Java 类/枚举等生成单独的 `.java` 文件，并且为此 `.proto` 文件生成的包装器 Java 类不会包含任何嵌套类/枚举等。这是一个布尔选项，默认为 `false` 。如果不生成 Java 代码，则此选项无效。
  
  ```proto
  option java_multiple_files = true;
  ```
  
- `optimize_for` (file option): Can be set to `SPEED`, `CODE_SIZE`, or `LITE_RUNTIME`. This affects the C++ and Java code generators (and possibly third-party generators) in the following ways:
  
- `optimize_for` （文件选项）：可以设置为 `SPEED` 、 `CODE_SIZE` 或 `LITE_RUNTIME` 。这会以以下方式影响 C++ 和 Java 代码生成器（以及可能的第三方生成器）：
  
  - `SPEED` (default): The protocol buffer compiler will generate code for serializing, parsing, and performing other common operations on your message types. This code is highly optimized.
  - `SPEED` （默认）：协议缓冲区编译器将生成代码，用于对您的消息类型执行序列化、解析和其他常见操作。此代码经过高度优化。
  - `CODE_SIZE`: The protocol buffer compiler will generate minimal classes and will rely on shared, reflection-based code to implement serialialization, parsing, and various other operations. The generated code will thus be much smaller than with `SPEED`, but operations will be slower. Classes will still implement exactly the same public API as they do in `SPEED` mode. This mode is most useful in apps that contain a very large number of `.proto` files and do not need all of them to be blindingly fast.
  - `CODE_SIZE` ：协议缓冲区编译器将生成最小的类，并将依赖基于反射的共享代码来实现序列化、解析和各种其他操作。因此，生成的代码将比 `SPEED` 小得多，但操作会更慢。类仍将实现与 `SPEED` 模式中完全相同的公共 API。此模式在包含大量 `.proto` 文件且不需要所有文件都非常快的应用中最为有用。
  - `LITE_RUNTIME`: The protocol buffer compiler will generate classes that depend only on the “lite” runtime library (`libprotobuf-lite` instead of `libprotobuf`). The lite runtime is much smaller than the full library (around an order of magnitude smaller) but omits certain features like descriptors and reflection. This is particularly useful for apps running on constrained platforms like mobile phones. The compiler will still generate fast implementations of all methods as it does in `SPEED` mode. Generated classes will only implement the `MessageLite` interface in each language, which provides only a subset of the methods of the full `Message` interface.
  - `LITE_RUNTIME` ：协议缓冲区编译器将生成仅依赖于“精简”运行时库的类（ `libprotobuf-lite` 而不是 `libprotobuf` ）。精简运行时比完整库小得多（大约小一个数量级），但省略了描述符和反射等某些功能。这对于在受限平台（如移动电话）上运行的应用特别有用。编译器仍将像在 `SPEED` 模式中一样生成所有方法的快速实现。生成的类将仅在每种语言中实现 `MessageLite` 接口，该接口仅提供完整 `Message` 接口方法的一个子集。
  
  ```proto
  option optimize_for = CODE_SIZE;
  ```
  
- `cc_generic_services`, `java_generic_services`, `py_generic_services` (file options): **Generic services are deprecated.** Whether or not the protocol buffer compiler should generate abstract service code based on [services definitions](https://protobuf.dev/programming-guides/proto3/#services) in C++, Java, and Python, respectively. For legacy reasons, these default to `true`. However, as of version 2.3.0 (January 2010), it is considered preferable for RPC implementations to provide [code generator plugins](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.compiler.plugin.pb) to generate code more specific to each system, rather than rely on the “abstract” services.
  
- `cc_generic_services` 、 `java_generic_services` 、 `py_generic_services` （文件选项）：通用服务已弃用。协议缓冲区编译器是否应根据 C++、Java 和 Python 中的服务定义生成抽象服务代码。出于传统原因，这些默认为 `true` 。但是，从 2.3.0 版（2010 年 1 月）开始，对于 RPC 实现来说，最好提供代码生成器插件来生成更特定于每个系统的代码，而不是依赖于“抽象”服务。
  
  ```proto
  // This file relies on plugins to generate service code.
  option cc_generic_services = false;
  option java_generic_services = false;
  option py_generic_services = false;
  ```
  
- `cc_enable_arenas` (file option): Enables [arena allocation](https://protobuf.dev/reference/cpp/arenas) for C++ generated code.
  
- `cc_enable_arenas` （文件选项）：为 C++ 生成的代码启用 arena 分配。
  
- `objc_class_prefix` (file option): Sets the Objective-C class prefix which is prepended to all Objective-C generated classes and enums from this .proto. There is no default. You should use prefixes that are between 3-5 uppercase characters as [recommended by Apple](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Conventions/Conventions.html#//apple_ref/doc/uid/TP40011210-CH10-SW4). Note that all 2 letter prefixes are reserved by Apple.
  
- `objc_class_prefix` （文件选项）：设置 Objective-C 类前缀，该前缀会添加到此 .proto 生成的所有 Objective-C 类和枚举之前。没有默认值。您应使用 Apple 建议的 3-5 个大写字符作为前缀。请注意，所有 2 个字母的前缀均由 Apple 保留。
  
- `packed` (field option): Defaults to `true` on a repeated field of a basic numeric type, causing a more compact [encoding](https://protobuf.dev/programming-guides/encoding#packed) to be used. There is no downside to using this option, but it can be set to `false`. Note that prior to version 2.3.0, parsers that received packed data when not expected would ignore it. Therefore, it was not possible to change an existing field to packed format without breaking wire compatibility. In 2.3.0 and later, this change is safe, as parsers for packable fields will always accept both formats, but be careful if you have to deal with old programs using old protobuf versions.
  
- `packed` （字段选项）：对于基本数字类型的重复字段，默认为 `true` ，从而使用更紧凑的编码。使用此选项没有缺点，但可以将其设置为 `false` 。请注意，在 2.3.0 版本之前，当解析器在未预期时收到打包数据，它会忽略该数据。因此，不可能将现有字段更改为打包格式，而不会破坏线路兼容性。在 2.3.0 及更高版本中，此更改是安全的，因为可打包字段的解析器将始终接受这两种格式，但如果您必须处理使用旧版 protobuf 版本的旧程序，请务必小心。
  
  ```proto
  repeated int32 samples = 4 [packed = false];
  ```
  
- `deprecated` (field option): If set to `true`, indicates that the field is deprecated and should not be used by new code. In most languages this has no actual effect. In Java, this becomes a `@Deprecated` annotation. For C++, clang-tidy will generate warnings whenever deprecated fields are used. In the future, other language-specific code generators may generate deprecation annotations on the field’s accessors, which will in turn cause a warning to be emitted when compiling code which attempts to use the field. If the field is not used by anyone and you want to prevent new users from using it, consider replacing the field declaration with a [reserved](https://protobuf.dev/programming-guides/proto3/#fieldreserved) statement.
  
- `deprecated` （字段选项）：如果设置为 `true` ，则表示该字段已弃用，新代码不应使用该字段。在大多数语言中，这没有任何实际效果。在 Java 中，这将成为 `@Deprecated` 注释。对于 C++，clang-tidy 将在使用弃用字段时生成警告。将来，其他特定于语言的代码生成器可能会在字段的访问器上生成弃用注释，这反过来会导致在编译尝试使用该字段的代码时发出警告。如果没有人使用该字段，并且您想阻止新用户使用它，请考虑用保留语句替换字段声明。
  
  ```proto
  int32 old_field = 6 [deprecated = true];
  ```

### 枚举值选项 Enum Value Options 

Enum value options are supported. You can use the `deprecated` option to indicate that a value shouldn’t be used anymore. You can also create custom options using extensions.

​	支持枚举值选项。您可以使用 `deprecated` 选项来指示不再使用某个值。您还可以使用扩展名创建自定义选项。

The following example shows the syntax for adding these options:

​	以下示例显示了添加这些选项的语法：

```proto
import "google/protobuf/descriptor.proto";

extend google.protobuf.EnumValueOptions {
  optional string string_name = 123456789;
}

enum Data {
  DATA_UNSPECIFIED = 0;
  DATA_SEARCH = 1 [deprecated = true];
  DATA_DISPLAY = 2 [
    (string_name) = "display_value"
  ];
}
```

See [Custom Options](https://protobuf.dev/programming-guides/proto3/#customoptions) to see how to apply custom options to enum values and to fields.

​	请参阅自定义选项，了解如何将自定义选项应用于枚举值和字段。

### 自定义选项 Custom Options 

Protocol Buffers also allows you to define and use your own options. Note that this is an **advanced feature** which most people don’t need. If you do think you need to create your own options, see the [Proto2 Language Guide](https://protobuf.dev/programming-guides/proto2#customoptions) for details. Note that creating custom options uses [extensions](https://protobuf.dev/programming-guides/proto2#extensions), which are permitted only for custom options in proto3.

​	Protocol Buffers 还允许您定义和使用自己的选项。请注意，这是一个高级功能，大多数人不需要。如果您确实认为需要创建自己的选项，请参阅 Proto2 语言指南了解详情。请注意，创建自定义选项使用扩展，而扩展仅允许在 proto3 中用于自定义选项。

### 选项保留 Option Retention 

Options have a notion of *retention*, which controls whether an option is retained in the generated code. Options have *runtime retention* by default, meaning that they are retained in the generated code and are thus visible at runtime in the generated descriptor pool. However, you can set `retention = RETENTION_SOURCE` to specify that an option (or field within an option) must not be retained at runtime. This is called *source retention*.

​	选项具有保留的概念，它控制选项是否保留在生成的代码中。默认情况下，选项具有运行时保留，这意味着它们保留在生成的代码中，因此在生成的描述符池中可以在运行时看到它们。但是，您可以设置 `retention = RETENTION_SOURCE` 来指定选项（或选项中的字段）不得在运行时保留。这称为源保留。

Option retention is an advanced feature that most users should not need to worry about, but it can be useful if you would like to use certain options without paying the code size cost of retaining them in your binaries. Options with source retention are still visible to `protoc` and `protoc` plugins, so code generators can use them to customize their behavior.

​	选项保留是一项高级功能，大多数用户不必担心，但如果您想使用某些选项而无需支付将它们保留在二进制文件中的代码大小成本，它可能很有用。具有源保留的选项仍然对 `protoc` 和 `protoc` 插件可见，因此代码生成器可以使用它们来自定义其行为。

Retention can be set directly on an option, like this:

​	保留可以直接在选项上设置，如下所示：

```proto
extend google.protobuf.FileOptions {
  optional int32 source_retention_option = 1234
      [retention = RETENTION_SOURCE];
}
```

It can also be set on a plain field, in which case it takes effect only when that field appears inside an option:

​	它也可以设置在一个普通字段上，在这种情况下，它仅在该字段出现在选项中时才生效：

```proto
message OptionsMessage {
  int32 source_retention_field = 1 [retention = RETENTION_SOURCE];
}
```

You can set `retention = RETENTION_RUNTIME` if you like, but this has no effect since it is the default behavior. When a message field is marked `RETENTION_SOURCE`, its entire contents are dropped; fields inside it cannot override that by trying to set `RETENTION_RUNTIME`.

​	您可以根据需要设置 `retention = RETENTION_RUNTIME` ，但这没有任何效果，因为这是默认行为。当一个消息字段被标记为 `RETENTION_SOURCE` 时，它的整个内容都会被丢弃；其中的字段不能通过尝试设置 `RETENTION_RUNTIME` 来覆盖它。

#### 注意 Note 

As of Protocol Buffers 22.0, support for option retention is still in progress and only C++ and Java are supported. Go has support starting from 1.29.0. Python support is complete but has not made it into a release yet.

​	从 Protocol Buffers 22.0 开始，对选项保留的支持仍在进行中，并且仅支持 C++ 和 Java。Go 从 1.29.0 开始支持。Python 支持已完成，但尚未发布。

### 选项目标 Option Targets 

Fields have a `targets` option which controls the types of entities that the field may apply to when used as an option. For example, if a field has `targets = TARGET_TYPE_MESSAGE` then that field cannot be set in a custom option on an enum (or any other non-message entity). Protoc enforces this and will raise an error if there is a violation of the target constraints.

​	字段有一个 `targets` 选项，它控制字段用作选项时可以应用到的实体类型。例如，如果一个字段具有 `targets = TARGET_TYPE_MESSAGE` ，那么该字段不能在枚举（或任何其他非消息实体）的自定义选项中设置。Protoc 会强制执行此操作，如果违反目标约束，它会引发错误。

At first glance, this feature may seem unnecessary given that every custom option is an extension of the options message for a specific entity, which already constrains the option to that one entity. However, option targets are useful in the case where you have a shared options message applied to multiple entity types and you want to control the usage of individual fields in that message. For example:

​	乍一看，此功能似乎不必要，因为每个自定义选项都是特定实体的选项消息的扩展，而该扩展已经将选项限制为该一个实体。但是，在将共享选项消息应用于多种实体类型并且您想要控制该消息中各个字段的使用时，选项目标很有用。例如：

```proto
message MyOptions {
  string file_only_option = 1 [targets = TARGET_TYPE_FILE];
  int32 message_and_enum_option = 2 [targets = TARGET_TYPE_MESSAGE,
                                     targets = TARGET_TYPE_ENUM];
}

extend google.protobuf.FileOptions {
  optional MyOptions file_options = 50000;
}

extend google.protobuf.MessageOptions {
  optional MyOptions message_options = 50000;
}

extend google.protobuf.EnumOptions {
  optional MyOptions enum_options = 50000;
}

// OK: this field is allowed on file options
option (file_options).file_only_option = "abc";

message MyMessage {
  // OK: this field is allowed on both message and enum options
  option (message_options).message_and_enum_option = 42;
}

enum MyEnum {
  MY_ENUM_UNSPECIFIED = 0;
  // Error: file_only_option cannot be set on an enum.
  option (enum_options).file_only_option = "xyz";
}
```

## 生成您的类 Generating Your Classes 

To generate the Java, Kotlin, Python, C++, Go, Ruby, Objective-C, or C# code that you need to work with the message types defined in a `.proto` file, you need to run the protocol buffer compiler `protoc` on the `.proto` file. If you haven’t installed the compiler, [download the package](https://protobuf.dev/downloads) and follow the instructions in the README. For Go, you also need to install a special code generator plugin for the compiler; you can find this and installation instructions in the [golang/protobuf](https://github.com/golang/protobuf/) repository on GitHub.

​	要生成使用 `.proto` 文件中定义的消息类型所需的 Java、Kotlin、Python、C++、Go、Ruby、Objective-C 或 C# 代码，您需要在 `.proto` 文件上运行协议缓冲区编译器 `protoc` 。如果您尚未安装编译器，请下载该软件包并按照自述文件中的说明进行操作。对于 Go，您还需要为编译器安装一个特殊的代码生成器插件；您可以在 GitHub 上的 golang/protobuf 代码库中找到该插件和安装说明。

The Protocol Compiler is invoked as follows:

​	协议编译器调用如下：

```sh
protoc --proto_path=IMPORT_PATH --cpp_out=DST_DIR --java_out=DST_DIR --python_out=DST_DIR --go_out=DST_DIR --ruby_out=DST_DIR --objc_out=DST_DIR --csharp_out=DST_DIR path/to/file.proto
```

- `IMPORT_PATH` specifies a directory in which to look for `.proto` files when resolving `import` directives. If omitted, the current directory is used. Multiple import directories can be specified by passing the `--proto_path` option multiple times; they will be searched in order. `-I=_IMPORT_PATH_` can be used as a short form of `--proto_path`.
  
- `IMPORT_PATH` 指定在解析 `import` 指令时查找 `.proto` 文件的目录。如果省略，则使用当前目录。可以通过多次传递 `--proto_path` 选项来指定多个导入目录；它们将按顺序进行搜索。 `-I=_IMPORT_PATH_` 可用作 `--proto_path` 的简写形式。
  
- You can provide one or more *output directives*:
  
- 您可以提供一个或多个输出指令：
  
  - `--cpp_out` generates C++ code in `DST_DIR`. See the [C++ generated code reference](https://protobuf.dev/reference/cpp/cpp-generated) for more.
  - `--cpp_out` 在 `DST_DIR` 中生成 C++ 代码。有关更多信息，请参阅 C++ 生成的代码参考。
  - `--java_out` generates Java code in `DST_DIR`. See the [Java generated code reference](https://protobuf.dev/reference/java/java-generated) for more.
  - `--java_out` 在 `DST_DIR` 中生成 Java 代码。有关更多信息，请参阅 Java 生成的代码参考。
  - `--kotlin_out` generates additional Kotlin code in `DST_DIR`. See the [Kotlin generated code reference](https://protobuf.dev/reference/kotlin/kotlin-generated) for more.
  - `--kotlin_out` 在 `DST_DIR` 中生成其他 Kotlin 代码。有关更多信息，请参阅 Kotlin 生成的代码参考。
  - `--python_out` generates Python code in `DST_DIR`. See the [Python generated code reference](https://protobuf.dev/reference/python/python-generated) for more.
  - `--python_out` 在 `DST_DIR` 中生成 Python 代码。有关更多信息，请参阅 Python 生成的代码参考。
  - `--go_out` generates Go code in `DST_DIR`. See the [Go generated code reference](https://protobuf.dev/reference/go/go-generated) for more.
  - `--go_out` 在 `DST_DIR` 中生成 Go 代码。有关更多信息，请参阅 Go 生成的代码参考。
  - `--ruby_out` generates Ruby code in `DST_DIR`. See the [Ruby generated code reference](https://protobuf.dev/reference/ruby/ruby-generated) for more.
  - `--ruby_out` 在 `DST_DIR` 中生成 Ruby 代码。有关更多信息，请参阅 Ruby 生成的代码参考。
  - `--objc_out` generates Objective-C code in `DST_DIR`. See the [Objective-C generated code reference](https://protobuf.dev/reference/objective-c/objective-c-generated) for more.
  - `--objc_out` 在 `DST_DIR` 中生成 Objective-C 代码。有关更多信息，请参阅 Objective-C 生成的代码参考。
  - `--csharp_out` generates C# code in `DST_DIR`. See the [C# generated code reference](https://protobuf.dev/reference/csharp/csharp-generated) for more.
  - `--csharp_out` 在 `DST_DIR` 中生成 C# 代码。有关更多信息，请参阅 C# 生成的代码参考。
  - `--php_out` generates PHP code in `DST_DIR`. See the [PHP generated code reference](https://protobuf.dev/reference/php/php-generated) for more.
  - `--php_out` 在 `DST_DIR` 中生成 PHP 代码。有关更多信息，请参阅生成的 PHP 代码参考。
  
  As an extra convenience, if the `DST_DIR` ends in `.zip` or `.jar`, the compiler will write the output to a single ZIP-format archive file with the given name. `.jar` outputs will also be given a manifest file as required by the Java JAR specification. Note that if the output archive already exists, it will be overwritten.
  
  ​	为了更加方便，如果 `DST_DIR` 以 `.zip` 或 `.jar` 结尾，编译器会将输出写入具有给定名称的单个 ZIP 格式存档文件。 `.jar` 输出还将提供清单文件，这是 Java JAR 规范所要求的。请注意，如果输出存档已存在，它将被覆盖。

- You must provide one or more `.proto` files as input. Multiple `.proto` files can be specified at once. Although the files are named relative to the current directory, each file must reside in one of the `IMPORT_PATH`s so that the compiler can determine its canonical name.
  
- 您必须提供一个或多个 `.proto` 文件作为输入。可以一次指定多个 `.proto` 文件。尽管这些文件相对于当前目录命名，但每个文件都必须驻留在某个 `IMPORT_PATH` 中，以便编译器能够确定其规范名称。

## 文件位置 File location 

Prefer not to put `.proto` files in the same directory as other language sources. Consider creating a subpackage `proto` for `.proto` files, under the root package for your project.

​	最好不要将 `.proto` 文件放在与其他语言源相同的目录中。考虑为 `.proto` 文件创建子包 `proto` ，位于项目的根包下。

### 位置应与语言无关 Location Should be Language-agnostic 

When working with Java code, it’s handy to put related `.proto` files in the same directory as the Java source. However, if any non-Java code ever uses the same protos, the path prefix will no longer make sense. So in general, put the protos in a related language-agnostic directory such as `//myteam/mypackage`.

​	在使用 Java 代码时，将相关的 `.proto` 文件放在与 Java 源代码相同的目录中很方便。但是，如果任何非 Java 代码使用相同的 protos，则路径前缀将不再有意义。因此，通常将 protos 放在与语言无关的相关目录中，例如 `//myteam/mypackage` 。

The exception to this rule is when it’s clear that the protos will be used only in a Java context, such as for testing.

​	此规则的例外情况是，当明确指出 protos 仅在 Java 上下文中使用时，例如用于测试。

## 支持的平台 Supported Platforms 

For information about:

​	有关以下信息：

- the operating systems, compilers, build systems, and C++ versions that are supported, see [Foundational C++ Support Policy](https://opensource.google/documentation/policies/cplusplus-support).
- 受支持的操作系统、编译器、构建系统和 C++ 版本，请参阅基础 C++ 支持政策。
- the PHP versions that are supported, see [Supported PHP versions](https://cloud.google.com/php/getting-started/supported-php-versions).
- 受支持的 PHP 版本，请参阅受支持的 PHP 版本。
