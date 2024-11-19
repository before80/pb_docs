+++
title = "语言指南（proto 2）"
date = 2024-11-17T09:35:36+08:00
weight = 10
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

> 原文：[https://protobuf.dev/programming-guides/proto2/](https://protobuf.dev/programming-guides/proto2/)
>
> 收录该文档的时间：`2024-11-17T09:35:36+08:00`

# Language Guide (proto 2) - 语言指南（proto 2）

Covers how to use the proto2 revision of Protocol Buffers language in your project.

​	介绍如何在项目中使用 Protocol Buffers 的 proto2 版本语言。

This guide describes how to use the protocol buffer language to structure your protocol buffer data, including `.proto` file syntax and how to generate data access classes from your `.proto` files. It covers the **proto2** revision of the protocol buffers language.

​	本指南描述了如何使用协议缓冲区语言来构建您的协议缓冲区数据，包括 `.proto` 文件语法以及如何从 `.proto` 文件生成数据访问类。本文档涵盖的是协议缓冲区语言的 **proto2** 版本。

For information on **editions** syntax, see the [Protobuf Editions Language Guide]({{< ref "/docs/ProgrammingGuides/LanguageGuideeditions" >}}).

​	有关 **editions** 语法的信息，请参阅 [Protobuf Editions Language Guide]({{< ref "/docs/ProgrammingGuides/LanguageGuideeditions" >}})。

For information on **proto3** syntax, see the [Proto3 Language Guide]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto3" >}}).

​	有关 **proto3** 语法的信息，请参阅 [Proto3 Language Guide]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto3" >}})。

This is a reference guide – for a step by step example that uses many of the features described in this document, see the [tutorial]({{< ref "/docs/Tutorials" >}}) for your chosen language.

​	这是一个参考指南——如果您需要使用本文档中描述的许多功能的分步示例，请参阅针对您选择语言的 [教程]({{< ref "/docs/Tutorials" >}})。

## 定义消息类型 Defining A Message Type

First let’s look at a very simple example. Let’s say you want to define a search request message format, where each search request has a query string, the particular page of results you are interested in, and a number of results per page. Here’s the `.proto` file you use to define the message type.

​	首先来看一个非常简单的示例。假设您想定义一个搜索请求消息格式，其中每个搜索请求都有一个查询字符串、一个感兴趣的结果页码以及每页的结果数量。以下是用于定义消息类型的 `.proto` 文件：

```proto
syntax = "proto2";

message SearchRequest {
  optional string query = 1;
  optional int32 page_number = 2;
  optional int32 results_per_page = 3;
}
```

- The first line of the file specifies that you’re using the proto2 revision of the protobuf language spec. 文件的第一行指定您正在使用协议缓冲区语言规范的 proto2 版本。
  - The `edition` (or `syntax` for proto2/proto3) must be the first non-empty, non-comment line of the file.
    - `edition`（或 proto2/proto3 的 `syntax`）必须是文件中第一行非空且非注释的内容。
  - If no `edition` or `syntax` is specified, the protocol buffer compiler will assume you are using [proto2]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2" >}}).
    - 如果未指定 `edition` 或 `syntax`，协议缓冲区编译器将假定您使用的是 [proto2]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2" >}})。
- The `SearchRequest` message definition specifies three fields (name/value pairs), one for each piece of data that you want to include in this type of message. Each field has a name and a type.
  - `SearchRequest` 消息定义了三个字段（名称/值对），每个字段表示此类消息中要包含的一项数据。每个字段都有名称和类型。


### 指定字段类型 Specifying Field Types

In the earlier example, all the fields are [scalar types](https://protobuf.dev/programming-guides/proto2/#scalar): two integers (`page_number` and `results_per_page`) and a string (`query`). You can also specify [enumerations](https://protobuf.dev/programming-guides/proto2/#enum) and composite types like other message types for your field.

​	在前面的示例中，所有字段都是[标量类型](https://protobuf.dev/programming-guides/proto2/#scalar)：两个整数（`page_number` 和 `results_per_page`）和一个字符串（`query`）。您还可以为字段指定[枚举类型](https://protobuf.dev/programming-guides/proto2/#enum)和复合类型（如其他消息类型）。

### 分配字段编号 Assigning Field Numbers

You must give each field in your message definition a number between `1` and `536,870,911` with the following restrictions:

​	在消息定义中，必须为每个字段分配一个编号，范围在 `1` 到 `536,870,911` 之间，但需满足以下限制：

- The given number **must be unique** among all fields for that message.
  - 编号在该消息中必须是**唯一的**。

- Field numbers `19,000` to `19,999` are reserved for the Protocol Buffers implementation. The protocol buffer compiler will complain if you use one of these reserved field numbers in your message.
  - 编号范围 `19,000` 到 `19,999` 保留给协议缓冲区实现。如果使用这些保留字段编号，协议缓冲区编译器将报错。

- You cannot use any previously [reserved](https://protobuf.dev/programming-guides/proto2/#fieldreserved) field numbers or any field numbers that have been allocated to [extensions](https://protobuf.dev/programming-guides/proto2/#extensions).
  - 不能使用已被[保留](https://protobuf.dev/programming-guides/proto2/#fieldreserved)的字段编号或已分配给[扩展](https://protobuf.dev/programming-guides/proto2/#extensions)的字段编号。


This number **cannot be changed once your message type is in use** because it identifies the field in the [message wire format]({{< ref "/docs/ProgrammingGuides/Encoding" >}}). “Changing” a field number is equivalent to deleting that field and creating a new field with the same type but a new number. See [Deleting Fields](https://protobuf.dev/programming-guides/proto2/#deleting) for how to do this properly.

​	字段编号在您的消息类型使用后**不能更改**，因为该编号用于标识[消息的线格式]({{< ref "/docs/ProgrammingGuides/Encoding" >}})。更改字段编号相当于删除该字段并创建一个具有相同类型但新编号的新字段。有关正确执行此操作的方法，请参阅 [删除字段](https://protobuf.dev/programming-guides/proto2/#deleting)。

Field numbers **should never be reused**. Never take a field number out of the [reserved](https://protobuf.dev/programming-guides/proto2/#fieldreserved) list for reuse with a new field definition. See [Consequences of Reusing Field Numbers](https://protobuf.dev/programming-guides/proto2/#consequences).

​	字段编号**绝不应重复使用**。不要将字段编号从[保留](https://protobuf.dev/programming-guides/proto2/#fieldreserved)列表中移出，以用于新的字段定义。有关更多信息，请参阅 [重用字段编号的后果](https://protobuf.dev/programming-guides/proto2/#consequences)。

You should use the field numbers 1 through 15 for the most-frequently-set fields. Lower field number values take less space in the wire format. For example, field numbers in the range 1 through 15 take one byte to encode. Field numbers in the range 16 through 2047 take two bytes. You can find out more about this in [Protocol Buffer Encoding]({{< ref "/docs/ProgrammingGuides/Encoding#structure" >}}).

​	建议将编号 1 到 15 用于最常设置的字段，因为较低的字段编号值在线格式中占用的空间更小。例如，编号范围 1 到 15 的字段用一个字节编码，而编号范围 16 到 2047 的字段用两个字节编码。更多信息请参阅 [协议缓冲区编码]({{< ref "/docs/ProgrammingGuides/Encoding#structure" >}})。

#### 重用字段编号的后果 Consequences of Reusing Field Numbers

Reusing a field number makes decoding wire-format messages ambiguous.

​	重用字段编号会导致解码线格式消息时产生歧义。

The protobuf wire format is lean and doesn’t provide a way to detect fields encoded using one definition and decoded using another.

​	协议缓冲区的线格式非常紧凑，无法检测到使用一种定义编码的字段在使用另一种定义解码时的情况。

Encoding a field using one definition and then decoding that same field with a different definition can lead to:

​	以一种定义编码字段，然后用不同的定义解码该字段可能导致以下问题：

- Developer time lost to debugging
  - 开发者耗费时间调试问题

- A parse/merge error (best case scenario)
  - 解析/合并错误（最好的情况）

- Leaked PII/SPII
  - 敏感信息泄露（PII/SPII）

- Data corruption
  - 数据损坏


Common causes of field number reuse:

​	重用字段编号的常见原因：

- renumbering fields (sometimes done to achieve a more aesthetically pleasing number order for fields). Renumbering effectively deletes and re-adds all the fields involved in the renumbering, resulting in incompatible wire-format changes.
  - 字段重编号（有时是为了实现更美观的字段编号顺序）。重编号实际上删除并重新添加了所有涉及的字段，从而导致不兼容的线格式更改。

- deleting a field and not [reserving](https://protobuf.dev/programming-guides/proto2/#fieldreserved) the number to prevent future reuse.
  - 删除字段后未将编号[保留](https://protobuf.dev/programming-guides/proto2/#fieldreserved)，防止将来重用。
  - This has been a very easy mistake to make with [extension fields](https://protobuf.dev/programming-guides/proto2/#extensions) for several reasons. [Extension Declarations]({{< ref "/docs/ProgrammingGuides/ExtensionDeclarations" >}}) provide a mechanism for reserving extension fields.
    - 对于[扩展字段](https://protobuf.dev/programming-guides/proto2/#extensions)，这种错误尤其容易发生。[扩展声明]({{< ref "/docs/ProgrammingGuides/ExtensionDeclarations" >}})提供了一种保留扩展字段的方法。


The field number is limited to 29 bits rather than 32 bits because three bits are used to specify the field’s wire format. For more on this, see the [Encoding topic]({{< ref "/docs/ProgrammingGuides/Encoding#structure" >}}).

​	字段编号限制为 29 位而非 32 位，因为有三位用于指定字段的线格式。有关更多信息，请参阅 [编码主题]({{< ref "/docs/ProgrammingGuides/Encoding#structure" >}})。



### 指定字段基数 Specifying Field Cardinality

Message fields can be one of the following:

​	消息字段可以是以下类型之一：

- *Singular*:

  In proto2, there are two types of singular fields:

  ​	在 proto2 中，有两种单值字段类型：

  - `optional`: (recommended) An `optional` field is in one of two possible states: `optional`：（推荐）一个 `optional` 字段有两种可能的状态：

    - the field is set, and contains a value that was explicitly set or parsed from the wire. It will be serialized to the wire.
      - 字段已设置，包含一个显式设置或从线上解析的值。它会被序列化到线上。

    - the field is unset, and will return the default value. It will not be serialized to the wire.
      - 字段未设置，将返回默认值。它不会被序列化到线上。


    You can check to see if the value was explicitly set.

    ​	您可以检查该值是否已显式设置。

  - `required`: **Do not use.** Required fields are so problematic they were removed from proto3 and editions. Semantics for required field should be implemented at the application layer. When it *is* used, a well-formed message must have exactly one of this field.

    - `required`：**不要使用。**必填字段的问题非常多，因此在 proto3 和后续版本中被移除。必填字段的语义应在应用层实现。如果使用，格式良好的消息必须正好包含该字段一次。

- `repeated`: this field type can be repeated zero or more times in a well-formed message. The order of the repeated values will be preserved.

  - **`repeated`**：此字段类型可以在格式良好的消息中重复零次或多次。重复值的顺序将被保留。

- `map`: this is a paired key/value field type. See [Maps]({{< ref "/docs/ProgrammingGuides/Encoding#maps" >}}) for more on this field type.

  - **`map`**：这是一个键/值对字段类型。有关更多信息，请参阅 [映射]({{< ref "/docs/ProgrammingGuides/Encoding#maps" >}})。


#### 为新的 `repeated` 字段使用打包编码 Use Packed Encoding for New Repeated Fields

For historical reasons, `repeated` fields of scalar numeric types (for example, `int32`, `int64`, `enum`) aren’t encoded as efficiently as they could be. New code should use the special option `[packed = true]` to get a more efficient encoding. For example:

​	由于历史原因，标量数字类型的 `repeated` 字段（例如 `int32`、`int64`、`enum`）的编码效率较低。新的代码应使用特殊选项 `[packed = true]`，以获得更高效的编码。例如：

```proto
repeated int32 samples = 4 [packed = true];
repeated ProtoEnum results = 5 [packed = true];
```

You can find out more about `packed` encoding in [Protocol Buffer Encoding]({{< ref "/docs/ProgrammingGuides/Encoding#packed" >}}).

​	您可以在 [协议缓冲区编码]({{< ref "/docs/ProgrammingGuides/Encoding#packed" >}}) 中了解更多关于 `packed` 编码的信息。

### 必须字段已被强烈废弃 Required is Strongly Deprecated

> Important
>
> **Required Is Forever** As mentioned earlier **`required` must not be used for new fields**. Semantics for required fields should be implemented at the application layer instead. Existing `required` fields should be treated as permanent, immutable elements of the message definition. It is nearly impossible to safely change a field from `required` to `optional`. If there is any chance that a stale reader exists, it will consider messages without this field to be incomplete and may reject or drop them.
>
> ​	**必须字段是永久性的** 正如前面提到的，**不应为新字段使用 `required`**。必须字段的语义应该在应用层实现。现有的 `required` 字段应被视为消息定义的永久性、不可变元素。从 `required` 更改为 `optional` 几乎是不可能安全完成的。如果存在任何可能的旧版本读取器，它将认为没有此字段的消息是不完整的，可能会拒绝或丢弃这些消息。

A second issue with required fields appears when someone adds a value to an enum. In this case, the unrecognized enum value is treated as if it were missing, which also causes the required value check to fail.

​	另一个与必须字段相关的问题是在有人向枚举中添加值时出现的。在这种情况下，无法识别的枚举值会被视为缺失，这也会导致必须值检查失败。

### 格式良好的消息 Well-formed Messages

The term “well-formed,” when applied to protobuf messages, refers to the bytes serialized/deserialized. The protoc parser validates that a given proto definition file is parseable.

​	“格式良好”一词在应用于 protobuf 消息时，指的是已序列化/反序列化的字节流。`protoc` 解析器验证给定的 proto 定义文件是否可解析。

Singular fields can appear more than once in wire-format bytes. The parser will accept the input, but only the last instance of that field will be accessible through the generated bindings. See [Last One Wins]({{< ref "/docs/ProgrammingGuides/Encoding#last-one-wins" >}}) for more on this topic.

​	单值字段可以在线格式字节中出现多次。解析器会接受这些输入，但只有该字段的最后一个实例可以通过生成的绑定访问。有关更多信息，请参阅 [最后一个值生效]({{< ref "/docs/ProgrammingGuides/Encoding#last-one-wins" >}})。

### 添加更多消息类型 Adding More Message Types

Multiple message types can be defined in a single `.proto` file. This is useful if you are defining multiple related messages – so, for example, if you wanted to define the reply message format that corresponds to your `SearchResponse` message type, you could add it to the same `.proto`:

​	可以在单个 `.proto` 文件中定义多个消息类型。这在定义多个相关消息时非常有用。例如，如果您想定义与 `SearchRequest` 消息类型对应的回复消息格式，您可以将其添加到同一个 `.proto` 文件中：

```proto
message SearchRequest {
  optional string query = 1;
  optional int32 page_number = 2;
  optional int32 results_per_page = 3;
}

message SearchResponse {
 ...
}
```

**Combining Messages leads to bloat** While multiple message types (such as message, enum, and service) can be defined in a single `.proto` file, it can also lead to dependency bloat when large numbers of messages with varying dependencies are defined in a single file. It’s recommended to include as few message types per `.proto` file as possible.

​	**组合消息可能导致依赖性膨胀** 虽然可以在一个 `.proto` 文件中定义多个消息类型（例如消息、枚举和服务），但如果在一个文件中定义了大量消息类型且它们具有不同的依赖项，则可能会导致依赖性膨胀。建议每个 `.proto` 文件中尽可能少地包含消息类型。

### 添加注释 Adding Comments

To add comments to your `.proto` files:

​	可以向 `.proto` 文件中添加注释：

- Prefer C/C++/Java line-end-style comments ‘//’ on the line before the .proto code element
  - 首选使用 C/C++/Java 的行尾注释风格 `//`，并将注释放在 `.proto` 代码元素的前一行。

- C-style inline/multi-line comments `/* ... */` are also accepted.
  - 也接受 C 风格的内联/多行注释 `/* ... */`。
  - When using multi-line comments, a margin line of ‘*’ is preferred.
    - 使用多行注释时，建议使用带有边界行的 `*`。


```proto
/**
 * SearchRequest represents a search query, with pagination options to
 * indicate which results to include in the response.
 */
 /**
 * SearchRequest 表示一个搜索查询，带有分页选项以指示响应中要包含的结果。
 */
message SearchRequest {
  optional string query = 1;

  // Which page number do we want?
  // 我们想要哪一页的结果？
  optional int32 page_number = 2;

  // Number of results to return per page.
  // 每页返回的结果数量。
  optional int32 results_per_page = 3;
}
```

### 删除字段 Deleting Fields

Deleting fields can cause serious problems if not done properly.

​	删除字段如果操作不当可能会导致严重问题。

**Do not delete** `required` fields. This is almost impossible to do safely.

​	**不要删除** `required` 字段。这几乎无法安全完成。

When you no longer need a field and all references have been deleted from client code, you may delete the field definition from the message. However, you **must** [reserve the deleted field number](https://protobuf.dev/programming-guides/proto2/#fieldreserved). If you do not reserve the field number, it is possible for a developer to reuse that number in the future.

​	当您不再需要某个字段，并且客户端代码中的所有引用都已删除时，可以从消息中删除字段定义。但您**必须**[保留被删除的字段编号](https://protobuf.dev/programming-guides/proto2/#fieldreserved)。如果不保留字段编号，开发人员将来可能会重用该编号。

You should also reserve the field name to allow JSON and TextFormat encodings of your message to continue to parse.

​	您还应该保留字段名称，以便 JSON 和 TextFormat 编码的消息能够继续解析。



#### 保留字段编号 Reserved Field Numbers

If you [update](https://protobuf.dev/programming-guides/proto2/#updating) a message type by entirely deleting a field, or commenting it out, future developers can reuse the field number when making their own updates to the type. This can cause severe issues, as described in [Consequences of Reusing Field Numbers](https://protobuf.dev/programming-guides/proto2/#consequences). To make sure this doesn’t happen, add your deleted field number to the `reserved` list.

​	如果通过完全删除字段或注释掉字段来[更新](https://protobuf.dev/programming-guides/proto2/#updating)消息类型，未来的开发者可能会在更新类型时重用该字段编号。这可能会导致严重问题，如 [重用字段编号的后果](https://protobuf.dev/programming-guides/proto2/#consequences)中所述。为防止这种情况，应该将删除的字段编号添加到 `reserved` 列表中。

The protoc compiler will generate error messages if any future developers try to use these reserved field numbers.

​	如果未来的开发者尝试使用这些保留的字段编号，`protoc` 编译器将生成错误消息。

```proto
message Foo {
  reserved 2, 15, 9 to 11;
}
```

Reserved field number ranges are inclusive (`9 to 11` is the same as `9, 10, 11`).

​	保留的字段编号范围是包含的（`9 to 11` 等同于 `9, 10, 11`）。

#### 保留字段名称 Reserved Field Names

Reusing an old field name later is generally safe, except when using TextProto or JSON encodings where the field name is serialized. To avoid this risk, you can add the deleted field name to the `reserved` list.

​	在大多数情况下，稍后重用旧的字段名称是安全的，但在使用 TextProto 或 JSON 编码时会存在风险，因为这些编码会序列化字段名称。为避免此风险，可以将已删除的字段名称添加到 `reserved` 列表中。

Reserved names affect only the protoc compiler behavior and not runtime behavior, with one exception: TextProto implementations may discard unknown fields (without raising an error like with other unknown fields) with reserved names at parse time (only the C++ and Go implementations do so today). Runtime JSON parsing is not affected by reserved names.

​	保留名称只影响 `protoc` 编译器行为，而不会影响运行时行为，但有一个例外：TextProto 实现可能在解析时丢弃保留名称的未知字段（与其他未知字段不同，此时不会报错，当前仅 C++ 和 Go 实现具有此行为）。运行时 JSON 解析不受保留名称影响。

```proto
message Foo {
  reserved 2, 15, 9 to 11;
  reserved "foo", "bar";
}
```

Note that you can’t mix field names and field numbers in the same `reserved` statement.

​	请注意，不能在同一个 `reserved` 声明中混用字段名称和字段编号。

### 从 `.proto` 文件中生成什么？ What’s Generated from Your `.proto`?

When you run the [protocol buffer compiler](https://protobuf.dev/programming-guides/proto2/#generating) on a `.proto`, the compiler generates the code in your chosen language you’ll need to work with the message types you’ve described in the file, including getting and setting field values, serializing your messages to an output stream, and parsing your messages from an input stream.

​	当您使用 [Protocol Buffers 编译器](https://protobuf.dev/programming-guides/proto2/#generating) 编译 `.proto` 文件时，编译器会生成您所选语言的代码，这些代码包括您在文件中描述的消息类型所需的功能，如获取和设置字段值、将消息序列化到输出流、以及从输入流解析消息。

- For **C++**, the compiler generates a `.h` and `.cc` file from each `.proto`, with a class for each message type described in your file.
  - 对于 **C++**，编译器为每个 `.proto` 文件生成 `.h` 和 `.cc` 文件，其中包含该文件中描述的每个消息类型的类。

- For **Java**, the compiler generates a `.java` file with a class for each message type, as well as a special `Builder` class for creating message class instances.
  - 对于 **Java**，编译器为每个消息类型生成一个 `.java` 文件，还会生成一个特殊的 `Builder` 类，用于创建消息类实例。

- For **Kotlin**, in addition to the Java generated code, the compiler generates a `.kt` file for each message type with an improved Kotlin API. This includes a DSL that simplifies creating message instances, a nullable field accessor, and a copy function.
  - 对于 **Kotlin**，除了生成的 Java 代码外，编译器还为每个消息类型生成一个 `.kt` 文件，提供改进的 Kotlin API，包括一个简化创建消息实例的 DSL、可空字段访问器和复制功能。

- **Python** is a little different — the Python compiler generates a module with a static descriptor of each message type in your `.proto`, which is then used with a *metaclass* to create the necessary Python data access class at runtime.
  - **Python** 的处理方式略有不同——Python 编译器生成一个模块，其中包含每个消息类型的静态描述符。这些描述符通过 *元类* 在运行时创建必要的 Python 数据访问类。

- For **Go**, the compiler generates a `.pb.go` file with a type for each message type in your file.
  - 对于 **Go**，编译器为文件中的每个消息类型生成一个 `.pb.go` 文件。

- For **Ruby**, the compiler generates a `.rb` file with a Ruby module containing your message types.
  - 对于 **Ruby**，编译器为文件中的消息类型生成一个 `.rb` 文件，其中包含一个包含这些消息类型的 Ruby 模块。

- For **Objective-C**, the compiler generates a `pbobjc.h` and `pbobjc.m` file from each `.proto`, with a class for each message type described in your file.
  - 对于 **Objective-C**，编译器为每个 `.proto` 文件生成 `pbobjc.h` 和 `pbobjc.m` 文件，其中包含该文件中描述的每个消息类型的类。

- For **C#**, the compiler generates a `.cs` file from each `.proto`, with a class for each message type described in your file.
  - 对于 **C#**，编译器为每个 `.proto` 文件生成一个 `.cs` 文件，其中包含该文件中描述的每个消息类型的类。

- For **PHP**, the compiler generates a `.php` message file for each message type described in your file, and a `.php` metadata file for each `.proto` file you compile. The metadata file is used to load the valid message types into the descriptor pool.
  - 对于 **PHP**，编译器为每个消息类型生成一个 `.php` 消息文件，并为每个 `.proto` 文件生成一个 `.php` 元数据文件。元数据文件用于将有效的消息类型加载到描述符池中。

- For **Dart**, the compiler generates a `.pb.dart` file with a class for each message type in your file.
  - 对于 **Dart**，编译器为文件中的每个消息类型生成一个 `.pb.dart` 文件，其中包含一个类。


You can find out more about using the APIs for each language by following the tutorial for your chosen language. For even more API details, see the relevant [API reference]({{< ref "/docs/ReferenceGuides" >}}).

​	您可以通过您所选语言的教程了解更多有关使用这些 API 的信息。如需更详细的 API 信息，请参阅相关的 [API 参考]({{< ref "/docs/ReferenceGuides" >}})。

## 标量值类型 Scalar Value Types

A scalar message field can have one of the following types – the table shows the type specified in the `.proto` file, and the corresponding type in the automatically generated class:

​	标量消息字段可以具有以下类型之一——表格显示了 `.proto` 文件中指定的类型以及自动生成的类中的对应类型：

| .proto Type | Notes                                                        | C++ Type | Java/Kotlin Type[1] | Python Type[3]                       | Go Type  | Ruby Type                      | C# Type    | PHP Type          | Dart Type | Rust Type   |
| ----------- | ------------------------------------------------------------ | -------- | ------------------- | ------------------------------------ | -------- | ------------------------------ | ---------- | ----------------- | --------- | ----------- |
| double      |                                                              | double   | double              | float                                | *float64 | Float                          | double     | float             | double    | f64         |
| float       |                                                              | float    | float               | float                                | *float32 | Float                          | float      | float             | double    | f32         |
| int32       | Uses variable-length encoding. Inefficient for encoding negative numbers – if your field is likely to have negative values, use sint32 instead.使用可变长度编码。不适合编码负数——如果字段可能包含负值，请使用 `sint32`。 | int32    | int                 | int                                  | int32    | Fixnum or Bignum (as required) | int        | integer           | *int32    | i32         |
| int64       | Uses variable-length encoding. Inefficient for encoding negative numbers – if your field is likely to have negative values, use sint64 instead.使用可变长度编码。不适合编码负数——如果字段可能包含负值，请使用 `sint64`。 | int64    | long                | int/long[4]                          | *int64   | Bignum                         | long       | integer/string[6] | Int64     | i64         |
| uint32      | Uses variable-length encoding.使用可变长度编码。             | uint32   | int[2]              | int/long[4]                          | *uint32  | Fixnum or Bignum (as required) | uint       | integer           | int       | u32         |
| uint64      | Uses variable-length encoding.使用可变长度编码。             | uint64   | long[2]             | int/long[4]                          | *uint64  | Bignum                         | ulong      | integer/string[6] | Int64     | u64         |
| sint32      | Uses variable-length encoding. Signed int value. These more efficiently encode negative numbers than regular int32s.使用可变长度编码。支持有符号整数值，更高效地编码负数。 | int32    | int                 | int                                  | int32    | Fixnum or Bignum (as required) | int        | integer           | *int32    | i32         |
| sint64      | Uses variable-length encoding. Signed int value. These more efficiently encode negative numbers than regular int64s.使用可变长度编码。支持有符号整数值，更高效地编码负数。 | int64    | long                | int/long[4]                          | *int64   | Bignum                         | long       | integer/string[6] | Int64     | i64         |
| fixed32     | Always four bytes. More efficient than uint32 if values are often greater than 228.始终占用 4 字节。如果值通常大于 `2^28`，则比 `uint32` 更高效。 | uint32   | int[2]              | int/long[4]                          | *uint32  | Fixnum or Bignum (as required) | uint       | integer           | int       | u32         |
| fixed64     | Always eight bytes. More efficient than uint64 if values are often greater than 256.始终占用 8 字节。如果值通常大于 `2^56`，则比 `uint64` 更高效。 | uint64   | long[2]             | int/long[4]                          | *uint64  | Bignum                         | ulong      | integer/string[6] | Int64     | u64         |
| sfixed32    | Always four bytes.始终占用 4 字节。                          | int32    | int                 | int                                  | *int32   | Fixnum or Bignum (as required) | int        | integer           | int       | i32         |
| sfixed64    | Always eight bytes.始终占用 8 字节。                         | int64    | long                | int/long[4]                          | *int64   | Bignum                         | long       | integer/string[6] | Int64     | i64         |
| bool        |                                                              | bool     | boolean             | bool                                 | *bool    | TrueClass/FalseClass           | bool       | boolean           | bool      | bool        |
| string      | A string must always contain UTF-8 encoded or 7-bit ASCII text, and cannot be longer than 232.字符串必须始终包含 UTF-8 或 7 位 ASCII 编码文本，且不能超过 `2^32` 长度。 | string   | String              | unicode (Python 2) or str (Python 3) | *string  | String (UTF-8)                 | string     | string            | String    | ProtoString |
| bytes       | May contain any arbitrary sequence of bytes no longer than 232.可以包含任意长度不超过 `2^32` 的字节序列。 | string   | ByteString          | bytes                                | []byte   | String (ASCII-8BIT)            | ByteString | string            | List      | ProtoBytes  |

[1] Kotlin uses the corresponding types from Java, even for unsigned types, to ensure compatibility in mixed Java/Kotlin codebases.

​	[1] Kotlin 即使对无符号类型也使用 Java 的对应类型，以确保在 Java/Kotlin 混合代码库中的兼容性。

[2] In Java, unsigned 32-bit and 64-bit integers are represented using their signed counterparts, with the top bit simply being stored in the sign bit.

​	[2] 在 Java 中，无符号的 32 位和 64 位整数使用其有符号的对应类型表示，最高位简单地存储在符号位中。

[3] In all cases, setting values to a field will perform type checking to make sure it is valid.

​	[3] 在所有情况下，为字段设置值时都会执行类型检查以确保其有效。

[4] 64-bit or unsigned 32-bit integers are always represented as long when decoded, but can be an int if an int is given when setting the field. In all cases, the value must fit in the type represented when set. See [2].

​	[4] 解码时，64 位或无符号 32 位整数始终表示为 long，但如果设置字段时提供了 int，它可以是 int。在所有情况下，设置时值必须符合表示的类型。详见 [2]。

[5] Proto2 typically doesn’t ever check the UTF-8 validity of string fields. Behavior varies between languages though, and invalid UTF-8 data should not be stored in string fields.

​	[5] Proto2 通常不会检查字符串字段的 UTF-8 有效性。行为因语言而异，不应在字符串字段中存储无效的 UTF-8 数据。

[6] Integer is used on 64-bit machines and string is used on 32-bit machines.

​	[6] 在 64 位机器上使用整数表示，而在 32 位机器上使用字符串表示。

You can find out more about how these types are encoded when you serialize your message in [Protocol Buffer Encoding]({{< ref "/docs/ProgrammingGuides/Encoding" >}}).

​	您可以在 [Protocol Buffer 编码]({{< ref "/docs/ProgrammingGuides/Encoding" >}}) 中了解有关序列化消息时这些类型的编码方式的更多信息。

## 默认字段值 Default Field Values

When a message is parsed, if the encoded message bytes do not contain a particular field, accessing that field in the parsed object returns the default value for that field. The default values are type-specific:

​	解析消息时，如果编码的消息字节不包含某个特定字段，则在解析对象中访问该字段会返回该字段的默认值。默认值是特定于类型的：

- For strings, the default value is the empty string.
  - 对于字符串，默认值为空字符串。

- For bytes, the default value is empty bytes.
  - 对于字节，默认值为空字节。

- For bools, the default value is false.
  - 对于布尔值，默认值为 `false`。

- For numeric types, the default value is zero.
  - 对于数值类型，默认值为 `0`。

- For message fields, the field is not set. Its exact value is language-dependent. See the [generated code guide]({{< ref "/docs/ReferenceGuides" >}}) for your language for details.
  - 对于消息字段，该字段未设置，其确切值依赖于语言。请参阅所选语言的 [生成代码指南]({{< ref "/docs/ReferenceGuides" >}}) 以获取详细信息。

- For enums, the default value is the **first defined enum value**, which should be 0 (recommended for compatibility with proto3). See [Enum Default Value](https://protobuf.dev/programming-guides/proto2/#enum-default).
  - 对于枚举，默认值是**第一个定义的枚举值**，建议将其定义为 `0` 以与 proto3 兼容。详见 [枚举默认值](https://protobuf.dev/programming-guides/proto2/#enum-default)。


The default value for repeated fields is empty (generally an empty list in the appropriate language).

​	对于重复字段，默认值为空（通常是该语言中适当的空列表）。

The default value for map fields is empty (generally an empty map in the appropriate language).

​	对于映射字段，默认值为空（通常是该语言中适当的空映射）。

### 覆盖标量默认值 Overriding Default Scalar Values

In proto2, you can specify explicit default values for singular non-message fields. For example, let’s say you want to provide a default value of 10 for the `SearchRequest.result_per_page` field:

​	在 proto2 中，您可以为单一非消息字段指定显式默认值。例如，假设您希望为 `SearchRequest.result_per_page` 字段提供默认值 `10`：

```proto
optional int32 result_per_page = 3 [default = 10];
```

If the sender does not specify `result_per_page`, the receiver will observe the following state:

​	如果发送方未指定 `result_per_page`，接收方将观察到以下状态：

- The result_per_page field is not present. That is, the `has_result_per_page()` (hazzer method) method would return `false`.
  - `result_per_page` 字段不存在。即 `has_result_per_page()`（hazzer 方法）返回 `false`。

- The value of `result_per_page` (returned from the “getter”) is `10`.
  - `result_per_page` 的值（通过 “getter” 返回）为 `10`。


If the sender does send a value for `result_per_page` the default value of 10 is ignored and the sender’s value is returned from the “getter”.

​	如果发送方确实发送了 `result_per_page` 的值，则忽略默认值 `10`，接收方的 “getter” 返回发送方的值。

See the [generated code guide]({{< ref "/docs/ReferenceGuides" >}}) for your chosen language for more details about how defaults work in generated code.

​	详见所选语言的 [生成代码指南]({{< ref "/docs/ReferenceGuides" >}}) 了解有关生成代码中默认值的更多细节。

Because the default value for enums is the first defined enum value, take care when adding a value to the beginning of an enum value list. See the [Updating a Message Type](https://protobuf.dev/programming-guides/proto2/#updating) section for guidelines on how to safely change definitions.

​	由于枚举的默认值是第一个定义的枚举值，在枚举值列表的开头添加值时需谨慎。参阅 [更新消息类型](https://protobuf.dev/programming-guides/proto2/#updating) 部分，了解安全更改定义的指南。

## 枚举 Enumerations

When you’re defining a message type, you might want one of its fields to only have one of a predefined list of values. For example, let’s say you want to add a `corpus` field for each `SearchRequest`, where the corpus can be `UNIVERSAL`, `WEB`, `IMAGES`, `LOCAL`, `NEWS`, `PRODUCTS` or `VIDEO`. You can do this very simply by adding an `enum` to your message definition with a constant for each possible value.

​	在定义消息类型时，您可能希望其字段之一仅具有预定义值列表中的一个值。例如，您希望为每个 `SearchRequest` 添加一个 `corpus` 字段，其中 corpus 可以是 `UNIVERSAL`、`WEB`、`IMAGES`、`LOCAL`、`NEWS`、`PRODUCTS` 或 `VIDEO`。可以通过在消息定义中添加一个枚举来实现，每个可能的值都作为一个常量。

In the following example we’ve added an `enum` called `Corpus` with all the possible values, and a field of type `Corpus`:

​	以下示例中，我们添加了一个名为 `Corpus` 的枚举以及一个类型为 `Corpus` 的字段：

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
  optional string query = 1;
  optional int32 page_number = 2;
  optional int32 results_per_page = 3;
  optional Corpus corpus = 4;
}
```

### 枚举默认值 Enum Default Value

The default value for the `SearchRequest.corpus` field is `CORPUS_UNSPECIFIED` because that is the first value defined in the enum.

​	`SearchRequest.corpus` 字段的默认值为 `CORPUS_UNSPECIFIED`，因为它是枚举中定义的第一个值。

It is strongly recommended to define the first value of every enum as `ENUM_TYPE_NAME_UNSPECIFIED = 0;` or `ENUM_TYPE_NAME_UNKNOWN = 0;`. This is because of the way proto2 handles unknown values for enum fields.

​	强烈建议将每个枚举的第一个值定义为 `ENUM_TYPE_NAME_UNSPECIFIED = 0;` 或 `ENUM_TYPE_NAME_UNKNOWN = 0;`。这是因为 proto2 处理枚举字段的未知值方式。

It is also recommended that this first, default value have no semantic meaning other than “this value was unspecified”.

​	还建议将此第一个默认值定义为“此值未指定”，而不具有语义意义。

The default value for an enum field like `SearchRequest.corpus` field can be explicitly overridden like this:

​	您还可以显式覆盖枚举字段（如 `SearchRequest.corpus` 字段）的默认值：

```fallback
  optional Corpus corpus = 4 [default = CORPUS_UNIVERSAL];
```

### 枚举值别名 Enum Value Aliases

You can define aliases by assigning the same value to different enum constants. To do this you need to set the `allow_alias` option to `true`. Otherwise, the protocol buffer compiler generates a warning message when aliases are found. Though all alias values are valid during deserialization, the first value is always used when serializing.

​	您可以通过为不同的枚举常量分配相同的值来定义别名。要实现这一点，您需要将 `allow_alias` 选项设置为 `true`。否则，Protocol Buffers 编译器会在发现别名时生成警告消息。尽管在反序列化时所有别名值都是有效的，但序列化时始终使用第一个值。

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
  // ENAA_RUNNING = 1;  // Uncommenting this line will cause a warning message. 取消注释此行会导致警告消息。
  ENAA_FINISHED = 2;
}
```

Enumerator constants must be in the range of a 32-bit integer. Since `enum` values use [varint encoding]({{< ref "/docs/ProgrammingGuides/Encoding" >}}) on the wire, negative values are inefficient and thus not recommended. You can define `enum`s within a message definition, as in the earlier example, or outside – these `enum`s can be reused in any message definition in your `.proto` file. You can also use an `enum` type declared in one message as the type of a field in a different message, using the syntax `_MessageType_._EnumType_`.

​	由于 `enum` 值在传输中使用 [varint 编码]({{< ref "/docs/ProgrammingGuides/Encoding" >}})，负值效率较低，因此不推荐使用。您可以在消息定义中定义 `enum`（如前例），也可以在消息外部定义——这些枚举可以在 `.proto` 文件中的任何消息定义中重用。您还可以使用一个消息中声明的 `enum` 类型作为另一消息中字段的类型，语法为 `_MessageType_._EnumType_`。

When you run the protocol buffer compiler on a `.proto` that uses an `enum`, the generated code will have a corresponding `enum` for Java, Kotlin, or C++, or a special `EnumDescriptor` class for Python that’s used to create a set of symbolic constants with integer values in the runtime-generated class.

​	当您对包含 `enum` 的 `.proto` 文件运行 Protocol Buffer 编译器时，生成的代码将在 Java、Kotlin 或 C++ 中具有对应的 `enum`，而在 Python 中会生成一个特殊的 `EnumDescriptor` 类，用于在运行时生成的类中创建一组具有整数值的符号常量。

> Important
>
> The generated code may be subject to language-specific limitations on the number of enumerators (low thousands for one language). Review the limitations for the languages you plan to use.
>
> ​	生成的代码可能会受到特定语言的枚举器数量限制（某些语言限制为几千个）。请检查您计划使用的语言的限制。

> Important
>
> For information on how enums should work contrasted with how they currently work in different languages, see [Enum Behavior]({{< ref "/docs/ProgrammingGuides/EnumBehavior" >}}).
>
> ​	有关枚举应如何工作的详细信息，以及它们在不同语言中的当前行为，请参阅 [枚举行为]({{< ref "/docs/ProgrammingGuides/EnumBehavior" >}})。

Removing enum values is a breaking change for persisted protos. Instead of removing a value, mark the value with the `reserved` keyword to prevent the enum value from being code-generated, or keep the value but indicate that it will be removed later by using the `deprecated` field option:

​	移除枚举值会对持久化的 Proto 数据造成破坏性更改。与其移除值，不如使用 `reserved` 关键字标记值，以防止枚举值生成代码，或者保留该值并使用 `deprecated` 字段选项标明其将来会被移除：

```proto
enum PhoneType {
  PHONE_TYPE_UNSPECIFIED = 0;
  PHONE_TYPE_MOBILE = 1;
  PHONE_TYPE_HOME = 2;
  PHONE_TYPE_WORK = 3 [deprecated=true];
  reserved 4,5;
}
```

For more information about how to work with message `enum`s in your applications, see the [generated code guide]({{< ref "/docs/ReferenceGuides" >}}) for your chosen language.

​	有关在应用程序中如何处理消息 `enum` 的更多信息，请参阅所选语言的 [生成代码指南]({{< ref "/docs/ReferenceGuides" >}})。

### 保留值 Reserved Values

If you [update](https://protobuf.dev/programming-guides/proto2/#updating) an enum type by entirely removing an enum entry, or commenting it out, future users can reuse the numeric value when making their own updates to the type. This can cause severe issues if they later load old instances of the same `.proto`, including data corruption, privacy bugs, and so on. One way to make sure this doesn’t happen is to specify that the numeric values (and/or names, which can also cause issues for JSON serialization) of your deleted entries are `reserved`. The protocol buffer compiler will complain if any future users try to use these identifiers. You can specify that your reserved numeric value range goes up to the maximum possible value using the `max` keyword.

​	如果您通过完全删除或注释掉枚举条目来 [更新](https://protobuf.dev/programming-guides/proto2/#updating) 枚举类型，将来用户可能会在更新类型时重用这些数值。这可能导致严重问题，例如加载相同 `.proto` 的旧实例时出现数据损坏或隐私问题等。为确保不会发生这种情况，可以通过 `reserved` 指定被删除条目的数值（和/或名称），从而防止未来用户重用这些标识符。Protocol Buffer 编译器会在未来用户尝试使用这些标识符时发出警告。您可以使用 `max` 关键字指定保留数值范围直到最大可能值：

```proto
enum Foo {
  reserved 2, 15, 9 to 11, 40 to max;
  reserved "FOO", "BAR";
}
```

Note that you can’t mix field names and numeric values in the same `reserved` statement.

​	注意，您不能在同一个 `reserved` 语句中混合字段名称和数值。

## 使用其他消息类型 Using Other Message Types

You can use other message types as field types. For example, let’s say you wanted to include `Result` messages in each `SearchResponse` message – to do this, you can define a `Result` message type in the same `.proto` and then specify a field of type `Result` in `SearchResponse`:

​	您可以将其他消息类型用作字段类型。例如，假设您希望在每个 `SearchResponse` 消息中包含 `Result` 消息，可以在同一个 `.proto` 中定义一个 `Result` 消息类型，然后在 `SearchResponse` 中指定一个类型为 `Result` 的字段：

```proto
message SearchResponse {
  repeated Result results = 1;
}

message Result {
  optional string url = 1;
  optional string title = 2;
  repeated string snippets = 3;
}
```

### 导入定义 Importing Definitions

In the earlier example, the `Result` message type is defined in the same file as `SearchResponse` – what if the message type you want to use as a field type is already defined in another `.proto` file?

​	在前例中，`Result` 消息类型与 `SearchResponse` 定义在同一个文件中——如果您希望使用的消息类型已经定义在另一个 `.proto` 文件中怎么办？

You can use definitions from other `.proto` files by *importing* them. To import another `.proto`’s definitions, you add an import statement to the top of your file:

​	您可以通过 *import*（导入）使用其他 `.proto` 文件中的定义。为导入另一个 `.proto` 文件的定义，需要在文件顶部添加 `import` 声明：

```proto
import "myproject/other_protos.proto";
```

By default, you can use definitions only from directly imported `.proto` files. However, sometimes you may need to move a `.proto` file to a new location. Instead of moving the `.proto` file directly and updating all the call sites in a single change, you can put a placeholder `.proto` file in the old location to forward all the imports to the new location using the `import public` notion.

​	默认情况下，您只能使用直接导入的 `.proto` 文件中的定义。然而，有时您可能需要将一个 `.proto` 文件移动到新位置。与直接移动文件并更新所有调用点不同，您可以在旧位置放置一个占位 `.proto` 文件，通过 `import public` 将所有导入转发到新位置。

**Note that the public import functionality is not available in Java, Kotlin, TypeScript, JavaScript, GCL, as well as C++ targets that use protobuf static reflection.**

​	**请注意，`import public` 功能在 Java、Kotlin、TypeScript、JavaScript、GCL 以及使用 Protobuf 静态反射的 C++ 目标中不可用。**

`import public` dependencies can be transitively relied upon by any code importing the proto containing the `import public` statement. For example:

​	`import public` 依赖项可以被导入包含 `import public` 语句的 Proto 文件的任何代码传递性地依赖。例如：

```proto
// new.proto
// All definitions are moved here
// 所有定义都移到这里
// old.proto
// This is the proto that all clients are importing.
// 客户端都在导入这个 Proto 文件。
import public "new.proto";
import "other.proto";
// client.proto
import "old.proto";
// You use definitions from old.proto and new.proto, but not other.proto
// 您可以使用 old.proto 和 new.proto 的定义，但不能使用 other.proto 的定义。
```

The protocol compiler searches for imported files in a set of directories specified on the protocol compiler command line using the `-I`/`--proto_path` flag. If no flag was given, it looks in the directory in which the compiler was invoked. In general you should set the `--proto_path` flag to the root of your project and use fully qualified names for all imports.

​	Protocol Buffer 编译器会在通过 `-I`/`--proto_path` 标志指定的目录集中搜索导入的文件。如果没有指定标志，它将在调用编译器的目录中查找。通常，应将 `--proto_path` 标志设置为项目的根目录，并对所有导入使用全限定名。

### 使用 proto3 消息类型 Using proto3 Message Types

It’s possible to import [proto3]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto3" >}}) message types and use them in your proto2 messages, and vice versa. However, proto2 enums cannot be used directly in proto3 syntax (it’s okay if an imported proto2 message uses them).

​	可以导入 [proto3]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto3" >}}) 消息类型并在 proto2 消息中使用，反之亦然。然而，proto2 枚举不能直接在 proto3 语法中使用（如果是导入的 proto2 消息中使用它们，则没有问题）。

## 嵌套类型 Nested Types

You can define and use message types inside other message types, as in the following example – here the `Result` message is defined inside the `SearchResponse` message:

​	可以在其他消息类型中定义和使用消息类型。例如，下面的例子中 `Result` 消息被定义在 `SearchResponse` 消息内部：

```proto
message SearchResponse {
  message Result {
    optional string url = 1;
    optional string title = 2;
    repeated string snippets = 3;
  }
  repeated Result results = 1;
}
```

If you want to reuse this message type outside its parent message type, you refer to it as `_Parent_._Type_`:

​	如果想在父消息类型之外重用此消息类型，可以使用 `_Parent_._Type_` 的方式引用它：

```proto
message SomeOtherMessage {
  optional SearchResponse.Result result = 1;
}
```

You can nest messages as deeply as you like. In the example below, note that the two nested types named `Inner` are entirely independent, since they are defined within different messages:

​	可以无限深度嵌套消息类型。例如，下面的例子中注意两个名为 `Inner` 的嵌套类型是完全独立的，因为它们分别定义在不同的消息内：

```proto
message Outer {       // Level 0
  message MiddleAA {  // Level 1
    message Inner {   // Level 2
      optional int64 ival = 1;
      optional bool  booly = 2;
    }
  }
  message MiddleBB {  // Level 1
    message Inner {   // Level 2
      optional int32  ival = 1;
      optional bool   booly = 2;
    }
  }
}
```

### Groups

**Note that the groups feature is deprecated and should not be used when creating new message types. Use nested message types instead.**

​	注意：groups 特性已被弃用，创建新消息类型时不应使用。请改用嵌套消息类型。

Groups are another way to nest information in your message definitions. For example, another way to specify a `SearchResponse` containing a number of `Result`s is as follows:

​	Groups 是在消息定义中嵌套信息的另一种方式。例如，指定一个包含若干 `Result` 的 `SearchResponse` 的另一种方法如下：

```proto
message SearchResponse {
  repeated group Result = 1 {
    optional string url = 1;
    optional string title = 2;
    repeated string snippets = 3;
  }
}
```

A group simply combines a nested message type and a field into a single declaration. In your code, you can treat this message just as if it had a `Result` type field called `result` (the latter name is converted to lower-case so that it does not conflict with the former). Therefore, this example is exactly equivalent to the `SearchResponse` earlier, except that the message has a different [wire format]({{< ref "/docs/ProgrammingGuides/Encoding" >}}).

​	group 简单地将嵌套消息类型和字段合并为一个声明。在代码中，可以像对待具有 `Result` 类型字段 `result` 的消息一样对待此消息（后者名称会被转换为小写以避免冲突）。因此，此例子与之前的 `SearchResponse` 完全等价，除了该消息具有不同的 [传输格式]({{< ref "/docs/ProgrammingGuides/Encoding" >}})。

## 更新消息类型 Updating a Message Type

If an existing message type no longer meets all your needs – for example, you’d like the message format to have an extra field – but you’d still like to use code created with the old format, don’t worry! It’s very simple to update message types without breaking any of your existing code when you use the binary wire format.

​	如果现有消息类型无法满足需求，例如需要为消息格式添加额外字段，但仍希望使用旧格式生成的代码，不用担心！使用二进制传输格式时，更新消息类型非常简单，不会破坏现有代码。

> Note
>
> If you use JSON or [proto text format]({{< ref "/docs/ReferenceGuides/ProtocolBuffers/TextFormatLanguageSpecification" >}}) to store your protocol buffer messages, the changes that you can make in your proto definition are different.
>
> ​	如果使用 JSON 或 [proto 文本格式]({{< ref "/docs/ReferenceGuides/ProtocolBuffers/TextFormatLanguageSpecification" >}}) 存储协议缓冲区消息，则可以对 proto 定义进行的更改有所不同。

Check [Proto Best Practices]({{< ref "/docs/ProgrammingGuides/ProtoBestPractices" >}}) and the following rules:

​	请参考 [Proto 最佳实践]({{< ref "/docs/ProgrammingGuides/ProtoBestPractices" >}}) 和以下规则：

- Don’t change the field numbers for any existing fields. “Changing” the field number is equivalent to deleting the field and adding a new field with the same type. If you want to renumber a field, see the instructions for [deleting a field](https://protobuf.dev/programming-guides/proto2/#deleting).
  - **不要更改任何现有字段的字段编号。** “更改”字段编号等同于删除该字段并添加具有相同类型的新字段。如果想重新编号字段，请参阅 [删除字段](https://protobuf.dev/programming-guides/proto2/#deleting) 的说明。

- Any new fields that you add should be `optional` or `repeated`. This means that any messages serialized by code using your “old” message format can still be parsed by your new generated code, as they won’t be missing any `required` elements. You should keep in mind the [default values](https://protobuf.dev/programming-guides/proto2/#optional) for these elements so that new code can properly interact with messages generated by old code. Similarly, messages created by your new code can be parsed by your old code: old binaries simply ignore the new field when parsing. However, the unknown fields are not discarded, and if the message is later serialized, the unknown fields are serialized along with it – so if the message is passed on to new code, the new fields are still available. See the [Unknown Fields](https://protobuf.dev/programming-guides/proto2/#unknowns) section for details.
  - 新添加的字段应为 `optional` 或 `repeated`。这意味着，使用“旧”消息格式生成的代码序列化的消息仍然可以由新生成的代码解析，因为它们不会缺少任何 `required` 元素。同时，旧代码生成的消息可以被新代码解析。旧二进制文件在解析时会忽略新字段，但未知字段不会被丢弃。如果消息稍后被序列化，未知字段会随消息一起序列化，因此当消息传递给新代码时，新字段仍然可用。详见 [未知字段](https://protobuf.dev/programming-guides/proto2/#unknowns) 部分。

- Non-required fields can be removed, as long as the field number is not used again in your updated message type. You may want to rename the field instead, perhaps adding the prefix “OBSOLETE_”, or make the field number [reserved](https://protobuf.dev/programming-guides/proto2/#fieldreserved), so that future users of your `.proto` can’t accidentally reuse the number.
  - 可以移除非必需字段，只要字段编号在更新的消息类型中不再使用。建议重命名该字段（例如加上前缀 “OBSOLETE_”），或将字段编号标记为 [reserved](https://protobuf.dev/programming-guides/proto2/#fieldreserved)，以防止未来的 `.proto` 用户意外重用编号。

- A non-required field can be converted to an [extension](https://protobuf.dev/programming-guides/proto2/#extensions) and vice versa, as long as the type and number stay the same.
  - 非必需字段可以转换为 [扩展](https://protobuf.dev/programming-guides/proto2/#extensions)，反之亦然，只要类型和编号保持不变。

- `int32`, `uint32`, `int64`, `uint64`, and `bool` are all compatible – this means you can change a field from one of these types to another without breaking forwards- or backwards-compatibility. If a number is parsed from the wire which doesn’t fit in the corresponding type, you will get the same effect as if you had cast the number to that type in C++ (for example, if a 64-bit number is read as an int32, it will be truncated to 32 bits).
  - **`int32`、`uint32`、`int64`、`uint64` 和 `bool` 之间兼容**，可以在这些类型之间自由切换而不破坏前向或后向兼容性。如果从数据流中解析的数字不适合相应类型，将得到类似于在 C++ 中类型转换的效果（例如，如果将 64 位数字读取为 `int32`，它将被截断为 32 位）。

- `sint32` and `sint64` are compatible with each other but are *not* compatible with the other integer types.
  - `sint32` 和 `sint64` 彼此兼容，但与其他整数类型不兼容。

- `string` and `bytes` are compatible as long as the bytes are valid UTF-8.
  - **`string` 和 `bytes` 是兼容的**，前提是字节是有效的 UTF-8。

- Embedded messages are compatible with `bytes` if the bytes contain an encoded instance of the message.
  - 如果 `bytes` 包含编码的消息实例，则嵌套消息与 `bytes` 兼容。

- `fixed32` is compatible with `sfixed32`, and `fixed64` with `sfixed64`.
  - `fixed32` 与 `sfixed32` 兼容，`fixed64` 与 `sfixed64` 兼容。

- For `string`, `bytes`, and message fields, singular is compatible with `repeated`. Given serialized data of a repeated field as input, clients that expect this field to be singular will take the last input value if it’s a primitive type field or merge all input elements if it’s a message type field. Note that this is **not** generally safe for numeric types, including bools and enums. Repeated fields of numeric types may be serialized in the [packed]({{< ref "/docs/ProgrammingGuides/Encoding#packed" >}}) format, which will not be parsed correctly when a singular field is expected.
  - 对于 `string`、`bytes` 和消息字段，单个字段与 `repeated` 兼容。如果将重复字段的序列化数据作为输入，期望字段为单个的客户端会选择最后一个输入值（如果是基础类型字段）或合并所有输入元素（如果是消息类型字段）。**注意：这对于包括布尔值和枚举在内的数值类型通常并不安全**。数值类型的重复字段可能会以 [packed]({{< ref "/docs/ProgrammingGuides/Encoding#packed" >}}) 格式序列化，当期望单个字段时可能无法正确解析。

- Changing a default value is generally OK, as long as you remember that default values are never sent over the wire. Thus, if a program receives a message in which a particular field isn’t set, the program will see the default value as it was defined in that program’s version of the protocol. It will NOT see the default value that was defined in the sender’s code.
  - 改变默认值通常是可以的，但需要记住默认值不会通过数据流发送。因此，如果程序接收到一条特定字段未设置的消息，程序将看到该字段在其版本协议中定义的默认值，而不是发送方代码中定义的默认值。

- `enum` is compatible with `int32`, `uint32`, `int64`, and `uint64` in terms of wire format (note that values will be truncated if they don’t fit). However, be aware that client code may treat them differently when the message is deserialized. Notably, unrecognized `enum` values are discarded when the message is deserialized, which makes the field’s `has..` accessor return false and its getter return the first value listed in the `enum` definition, or the default value if one is specified. In the case of repeated enum fields, any unrecognized values are stripped out of the list. However, an integer field will always preserve its value. Because of this, you need to be very careful when upgrading an integer to an `enum` in terms of receiving out of bounds enum values on the wire.
  - 在传输格式上，`enum` 与 `int32`、`uint32`、`int64` 和 `uint64` 兼容（注意值超出范围会被截断）。但请注意，客户端代码在反序列化消息时可能会有不同的处理方式。例如，未识别的 `enum` 值在消息反序列化时会被丢弃，这会导致字段的 `has..` 访问器返回 `false`，其 getter 返回枚举定义中的第一个值（如果指定了默认值，则返回默认值）。在重复枚举字段的情况下，所有未识别的值会从列表中移除。然而，整数字段会始终保留其值。因此，当从整数升级到 `enum` 时，需要特别小心以处理数据流中超出范围的枚举值。

- In the current Java and C++ implementations, when unrecognized `enum` values are stripped out, they are stored along with other unknown fields. Note that this can result in strange behavior if this data is serialized and then reparsed by a client that recognizes these values. In the case of optional fields, even if a new value was written after the original message was deserialized, the old value will be still read by clients that recognize it. In the case of repeated fields, the old values will appear after any recognized and newly-added values, which means that order will not be preserved.
  - **在当前 Java 和 C++ 实现中，未识别的 `enum` 值被移除后，会与其他未知字段一起存储。** 这种行为在序列化数据并重新解析为识别这些值的客户端时可能导致奇怪的行为。对于可选字段，即使在原始消息反序列化后写入了新值，识别它们的客户端仍会读取旧值。对于重复字段，旧值会出现在所有识别值和新添加值之后，这意味着顺序可能不被保留。

- Changing a single `optional` field or extension into a member of a **new** `oneof` is binary compatible, however for some languages (notably, Go) the generated code’s API will change in incompatible ways. For this reason, Google does not make such changes in its public APIs, as documented in [AIP-180](https://google.aip.dev/180#moving-into-oneofs). With the same caveat about source-compatibility, moving multiple fields into a new `oneof` may be safe if you are sure that no code sets more than one at a time. Moving fields into an existing `oneof` is not safe. Likewise, changing a single field `oneof` to an `optional` field or extension is safe.
  - **将单个 `optional` 字段或扩展字段更改为** **新** 的 `oneof` 成员是二进制兼容的，但对于某些语言（特别是 Go），生成代码的 API 会以不兼容的方式改变。为此，Google 不在其公共 API 中进行此类更改，详情见 [AIP-180](https://google.aip.dev/180#moving-into-oneofs)。在某些情况下，如果确保代码不会同时设置多个字段，将多个字段移动到新 `oneof` 中可能是安全的。将字段移动到现有 `oneof` 中不安全。同样，将单个字段从 `oneof` 更改为 `optional` 或扩展字段是安全的。

- Changing a field between a `map<K, V>` and the corresponding `repeated` message field is binary compatible (see [Maps](https://protobuf.dev/programming-guides/proto2/#maps), below, for the message layout and other restrictions). However, the safety of the change is application-dependent: when deserializing and reserializing a message, clients using the `repeated` field definition will produce a semantically identical result; however, clients using the `map` field definition may reorder entries and drop entries with duplicate keys.
  - **在 `map<K, V>` 和对应的 `repeated` 消息字段之间进行更改是二进制兼容的**（参见 [Maps](https://protobuf.dev/programming-guides/proto2/#maps) 了解消息布局及其他限制）。然而，此更改的安全性依赖于应用场景：当反序列化和重新序列化消息时，使用 `repeated` 字段定义的客户端将生成语义上相同的结果；然而，使用 `map` 字段定义的客户端可能会重新排序条目或丢弃具有重复键的条目。


## 未知字段 Unknown Fields

Unknown fields are well-formed protocol buffer serialized data representing fields that the parser does not recognize. For example, when an old binary parses data sent by a new binary with new fields, those new fields become unknown fields in the old binary.

​	未知字段是表示解析器无法识别的字段的合法协议缓冲区序列化数据。例如，当旧二进制文件解析由具有新字段的新二进制文件发送的数据时，这些新字段在旧二进制文件中变为未知字段。

Originally, proto3 messages always discarded unknown fields during parsing, but in version 3.5 we reintroduced the preservation of unknown fields to match the proto2 behavior. In versions 3.5 and later, unknown fields are retained during parsing and included in the serialized output.

​	最初，proto3 消息在解析时总是丢弃未知字段，但在版本 3.5 中，为了匹配 proto2 行为，重新引入了保留未知字段的功能。从 3.5 版本及更高版本开始，未知字段会在解析时保留，并包含在序列化输出中。

### 保留未知字段 Retaining Unknown Fields

Some actions can cause unknown fields to be lost. For example, if you do one of the following, unknown fields are lost:

​	某些操作可能会导致未知字段丢失。例如，如果执行以下操作之一，未知字段将丢失：

- Serialize a proto to JSON.
  - 将 proto 序列化为 JSON。

- Iterate over all of the fields in a message to populate a new message.
  - 遍历消息中的所有字段以填充新消息。


To avoid losing unknown fields, do the following:

​	为避免丢失未知字段，请执行以下操作：

- Use binary; avoid using text formats for data exchange.
  - 使用二进制格式；避免在数据交换中使用文本格式。

- Use message-oriented APIs, such as `CopyFrom()` and `MergeFrom()`, to copy data rather than copying field-by-field
  - 使用面向消息的 API，例如 `CopyFrom()` 和 `MergeFrom()`，以复制数据而不是逐字段复制。


TextFormat is a bit of a special case. Serializing to TextFormat prints unknown fields using their field numbers. But parsing TextFormat data back into a binary proto fails if there are entries that use field numbers.

​	**TextFormat 是一个特殊情况。** 序列化为 TextFormat 时，使用字段编号打印未知字段。但如果 TextFormat 数据中存在字段编号条目，则将其解析回二进制 proto 会失败。

## 扩展 Extensions

An extension is a field defined outside of its container message; usually in a `.proto` file separate from the container message’s `.proto` file.

​	扩展是定义在其容器消息外部的字段；通常位于与容器消息的 `.proto` 文件不同的 `.proto` 文件中。

### 为什么使用扩展？ Why Use Extensions?

There are two main reasons to use extensions:

​	使用扩展有两个主要原因：

- The container message’s `.proto` file will have fewer imports/dependencies. This can improve build times, break circular dependencies, and otherwise promote loose coupling. Extensions are very good for this. **减少容器消息 `.proto` 文件的导入/依赖。** 这可以提高构建速度，打破循环依赖，并促进松耦合。扩展在这方面表现良好。
- Allow systems to attach data to a container message with minimal dependency and coordination. Extensions are not a great solution for this because of the limited field number space and the [Consequences of Reusing Field Numbers](https://protobuf.dev/programming-guides/proto2/#consequences). If your use case requires very low coordination for a large number of extensions, consider using the [`Any` message type]({{< ref "/docs/ReferenceGuides/ProtocolBuffers/Well-KnownTypes#any" >}}) instead. **允许系统以最小依赖和协调的方式向容器消息添加数据。** 然而，由于有限的字段编号空间和 [字段编号重用的后果](https://protobuf.dev/programming-guides/proto2/#consequences)，扩展并不是最佳解决方案。如果您的使用场景需要非常低的协调性来支持大量扩展，请考虑使用 [`Any` 消息类型]({{< ref "/docs/ReferenceGuides/ProtocolBuffers/Well-KnownTypes#any" >}})。

### 示例扩展 Example Extension

Let’s look at an example extension:

​	以下是一个扩展的示例：

```proto
// file kittens/video_ext.proto

import "kittens/video.proto";
import "media/user_content.proto";

package kittens;

// This extension allows kitten videos in a media.UserContent message.
// 此扩展允许在 media.UserContent 消息中包含 kitten 视频。
extend media.UserContent {
  // Video is a message imported from kittens/video.proto
  // Video 是从 kittens/video.proto 导入的消息
  repeated Video kitten_videos = 126;
}
```

Note that the file defining the extension (`kittens/video_ext.proto`) imports the container message’s file (`media/user_content.proto`).

​	请注意，定义扩展的文件 (`kittens/video_ext.proto`) 会导入容器消息的文件 (`media/user_content.proto`)。

The container message must reserve a subset of its field numbers for extensions.

​	容器消息必须为扩展保留一部分字段编号范围。

```proto
// file media/user_content.proto

package media;

// A container message to hold stuff that a user has created.
// 一个容器消息，用于存储用户创建的内容。
message UserContent {
  // Set verification to `DECLARATION` to enforce extension declarations for all
  // extensions in this range.
  // 将验证设置为 `DECLARATION`，以强制对此范围内的所有扩展进行声明。
  extensions 100 to 199 [verification = DECLARATION];
}
```

The container message’s file (`media/user_content.proto`) defines the message `UserContent`, which reserves field numbers [100 to 199] for extensions. It is recommended to set `verification = DECLARATION` for the range to require declarations for all its extensions.

​	容器消息的文件 (`media/user_content.proto`) 定义了消息 `UserContent`，并为扩展保留了字段编号范围 [100 到 199]。建议为此范围设置 `verification = DECLARATION`，以要求对其所有扩展进行声明。

When the new extension (`kittens/video_ext.proto`) is added, a corresponding declaration should be added to `UserContent` and `verification` should be removed.

​	当添加新的扩展 (`kittens/video_ext.proto`) 时，应在 `UserContent` 中添加相应的声明并移除 `verification`。

```fallback
// A container message to hold stuff that a user has created.
// 一个容器消息，用于存储用户创建的内容。
message UserContent {
  extensions 100 to 199 [
    declaration = {
      number: 126,
      full_name: ".kittens.kitten_videos",
      type: ".kittens.Video",
      repeated: true
    },
    // Ensures all field numbers in this extension range are declarations.
    // 确保此扩展范围内的所有字段编号都是声明。
    verification = DECLARATION
  ];
}
```

`UserContent` declares that field number `126` will be used by a `repeated` extension field with the fully-qualified name `.kittens.kitten_videos` and the fully-qualified type `.kittens.Video`. To learn more about extension declarations see [Extension Declarations]({{< ref "/docs/ProgrammingGuides/ExtensionDeclarations" >}}).

​	`UserContent` 声明字段编号 `126` 将由一个具有完全限定名称 `.kittens.kitten_videos` 和完全限定类型 `.kittens.Video` 的 `repeated` 扩展字段使用。有关扩展声明的更多信息，请参阅 [扩展声明]({{< ref "/docs/ProgrammingGuides/ExtensionDeclarations" >}})。

Note that the container message’s file (`media/user_content.proto`) **does not** import the kitten_video extension definition (`kittens/video_ext.proto`)

​	请注意，容器消息的文件 (`media/user_content.proto`) **不会** 导入 kitten 视频扩展定义 (`kittens/video_ext.proto`)。

There is no difference in the wire-format encoding of extension fields as compared to a standard field with the same field number, type, and cardinality. Therefore, it is safe to move a standard field out of its container to be an extension or to move an extension field into its container message as a standard field so long as the field number, type, and cardinality remain constant.

​	扩展字段的线格式编码与具有相同字段编号、类型和基数的标准字段没有区别。因此，只要字段编号、类型和基数保持不变，将标准字段移出其容器作为扩展，或将扩展字段移入其容器作为标准字段都是安全的。

However, because extensions are defined outside of the container message, no specialized accessors are generated to get and set specific extension fields. For our example, the protobuf compiler **will not generate** `AddKittenVideos()` or `GetKittenVideos()` accessors. Instead, extensions are accessed through parameterized functions like: `HasExtension()`, `ClearExtension()`, `GetExtension()`, `MutableExtension()`, and `AddExtension()`.

​	然而，由于扩展是在容器消息外部定义的，因此不会生成专门的访问器来获取和设置特定的扩展字段。在本示例中，protobuf 编译器 **不会生成** `AddKittenVideos()` 或 `GetKittenVideos()` 访问器。相反，扩展字段是通过参数化函数访问的，例如：`HasExtension()`、`ClearExtension()`、`GetExtension()`、`MutableExtension()` 和 `AddExtension()`。

In C++, it would look something like:

​	在 C++ 中，访问扩展字段的方式如下：

```cpp
UserContent user_content;
user_content.AddExtension(kittens::kitten_videos, new kittens::Video());
assert(1 == user_content.GetExtensionCount(kittens::kitten_videos));
user_content.GetExtension(kittens::kitten_videos, 0);
```

### 定义扩展范围 Defining Extension Ranges

If you are the owner of a container message, you will need to define an extension range for the extensions to your message.

​	如果您是容器消息的所有者，需要为消息的扩展定义扩展范围。

Field numbers allocated to extension fields cannot be reused for standard fields.

​	分配给扩展字段的字段编号不能用于标准字段。

It is safe to expand an extension range after it is defined. A good default is to allocate 1000 relatively small numbers, and densely populate that space using extension declarations:

​	扩展范围在定义后可以安全扩展。一个好的默认值是分配 1000 个相对较小的编号，并使用扩展声明密集填充该范围：

```proto
message ModernExtendableMessage {
  // All extensions in this range should use extension declarations.
  // 此范围内的所有扩展应使用扩展声明。
  extensions 1000 to 2000 [verification = DECLARATION];
}
```

When adding a range for extension declarations before the actual extensions, you should add `verification = DECLARATION` to enforce that declarations are used for this new range. This placeholder can be removed once an actual declaration is added.

​	在实际定义扩展之前添加扩展声明范围时，应通过 `verification = DECLARATION` 来强制要求为此新范围使用声明。一旦添加了实际声明，可以移除此占位符。

It is safe to split an existing extension range into separate ranges that cover the same total range. This might be necessary for migrating a legacy message type to [Extension Declarations]({{< ref "/docs/ProgrammingGuides/ExtensionDeclarations" >}}). For example, before migration, the range might be defined as:

​	将现有扩展范围拆分为多个覆盖相同总范围的独立范围是安全的。这种做法在将遗留消息类型迁移到 [扩展声明]({{< ref "/docs/ProgrammingGuides/ExtensionDeclarations" >}}) 时可能是必要的。例如，迁移前范围可能如下所示：

```proto
message LegacyMessage {
  extensions 1000 to max;
}
```

And after migration (splitting the range) it can be:

​	迁移后范围拆分如下：

```proto
message LegacyMessage {
  // Legacy range that was using an unverified allocation scheme.
  // 使用未经验证分配方案的遗留范围。
  extensions 1000 to 524999999 [verification = UNVERIFIED];
  // Current range that uses extension declarations.
  // 当前使用扩展声明的范围。
  extensions 525000000 to max  [verification = DECLARATION];
}
```

It is not safe to increase the start field number nor decrease the end field number to move or shrink an extension range. These changes can invalidate an existing extension.

​	**不安全操作**：增加起始字段编号或减少结束字段编号以移动或缩小扩展范围。这些更改可能会使现有扩展失效。

Prefer using field numbers 1 to 15 for standard fields that are populated in most instances of your proto. It is not recommended to use these numbers for extensions.

​	建议为大多数实例中填充的标准字段优先使用字段编号 1 到 15。不建议将这些编号用于扩展。

If your numbering convention might involve extensions having very large field numbers, you can specify that your extension range goes up to the maximum possible field number using the `max` keyword:

​	如果您的编号方案可能涉及非常大的扩展字段编号，可以使用 `max` 关键字指定扩展范围直到最大可能字段编号：

```proto
message Foo {
  extensions 1000 to max;
}
```

`max` is 229 - 1, or 536,870,911.

​	`max` 表示 2<sup>29</sup> - 1，即 536,870,911。

### 选择扩展编号 Choosing Extension Numbers

Extensions are just fields that can be specified outside of their container messages. All the same rules for [Assigning Field Numbers](https://protobuf.dev/programming-guides/proto2/#assigning) apply to extension field numbers. The same [Consequences of Reusing Field Numbers](https://protobuf.dev/programming-guides/proto2/#consequences) also apply to reusing extension field numbers.

​	扩展只是可以在其容器消息之外指定的字段。所有用于 [分配字段编号](https://protobuf.dev/programming-guides/proto2/#assigning) 的规则也适用于扩展字段编号。重复使用扩展字段编号的 [后果](https://protobuf.dev/programming-guides/proto2/#consequences) 也同样适用。

Choosing unique extension field numbers is simple if the container message uses [extension declarations]({{< ref "/docs/ProgrammingGuides/ExtensionDeclarations" >}}). When defining a new extension, choose the lowest field number above all other declarations from the highest extension range defined in the container message. For example, if a container message is defined like this:

​	如果容器消息使用 [扩展声明]({{< ref "/docs/ProgrammingGuides/ExtensionDeclarations" >}})，选择唯一扩展字段编号非常简单。定义新扩展时，从容器消息定义的最高扩展范围中的所有其他声明之后选择最低字段编号。例如，容器消息定义如下：

```proto
message Container {
  // Legacy range that was using an unverified allocation scheme
  // 使用未经验证分配方案的遗留范围。
  extensions 1000 to 524999999;
  // Current range that uses extension declarations. (highest extension range)
  // 当前使用扩展声明的范围。(最高扩展范围)
  extensions 525000000 to max  [
    declaration = {
      number: 525000001,
      full_name: ".bar.baz_ext",
      type: ".bar.Baz"
    }
    // 525,000,002 is the lowest field number above all other declarations
    // 525,000,002 是所有其他声明之后的最低字段编号。
  ];
}
```

The next extension of `Container` should add a new declaration with the number `525000002`.

​	容器的下一个扩展应添加字段编号为 `525000002` 的新声明。

#### 未验证的扩展编号分配（不推荐） Unverified Extension Number Allocation (not recommended)

The owner of a container message may choose to forgo extension declarations in favor of their own unverified extension number allocation strategy.

​	容器消息的所有者可以选择放弃扩展声明，而采用未验证的扩展编号分配策略。

An unverified allocation scheme uses a mechanism external to the protobuf ecosystem to allocate extension field numbers within the selected extension range. One example could be using a monorepo’s commit number. This system is “unverified” from the protobuf compiler’s point of view since there is no way to check that an extension is using a properly acquired extension field number.

​	未验证的分配方案使用 protobuf 生态系统外部的机制在选定的扩展范围内分配扩展字段编号。例如，可以使用 monorepo 的提交编号。这种系统从 protobuf 编译器的角度来看是“未验证的”，因为无法检查扩展是否正确获取了扩展字段编号。

The benefit of an unverified system over a verified system like extension declarations is the ability to define an extension without coordinating with the container message owner.

​	未验证系统相比于扩展声明等验证系统的优势在于能够在无需与容器消息所有者协调的情况下定义扩展。

The downside of an unverified system is that the protobuf compiler cannot protect participants from reusing extension field numbers.

​	未验证系统的劣势是 protobuf 编译器无法保护参与者避免重复使用扩展字段编号。

**Unverified extension field number allocation strategies are not recommended** because the [Consequences of Reusing Field Numbers](https://protobuf.dev/programming-guides/proto2/#consequences) fall on all extenders of a message (not just the developer that didn’t follow the recommendations). If your use case requires very low coordination, consider using the [`Any` message]({{< ref "/docs/ReferenceGuides/ProtocolBuffers/Well-KnownTypes#any" >}}) instead.

​	**不推荐使用未验证扩展字段编号分配策略**，因为[重复使用字段编号的后果](https://protobuf.dev/programming-guides/proto2/#consequences)会影响消息的所有扩展者，而不仅仅是未遵循建议的开发者。如果您的用例需要极低的协调性，建议使用 [`Any` 消息]({{< ref "/docs/ReferenceGuides/ProtocolBuffers/Well-KnownTypes#any" >}}) 代替。

Unverified extension field number allocation strategies are limited to the range 1 to 524,999,999. Field numbers 525,000,000 and above can only be used with extension declarations.

​	未验证的扩展字段编号分配范围仅限于 1 到 524,999,999。编号 525,000,000 及以上仅能用于扩展声明。

### 指定扩展类型 Specifying Extension Types

Extensions can be of any field type except `oneof`s and `map`s.

​	扩展可以是任意字段类型，但不能是 `oneof` 或 `map` 类型。

### 嵌套扩展（不推荐）Nested Extensions (not recommended)

You can declare extensions in the scope of another message:

​	可以在另一个消息的作用域内声明扩展：

```proto
import "common/user_profile.proto";

package puppies;

message Photo {
  extend common.UserProfile {
    optional int32 likes_count = 111;
  }
  ...
}
```

In this case, the C++ code to access this extension is:

​	在这种情况下，访问此扩展的 C++ 代码如下：

```cpp
UserProfile user_profile;
user_profile.SetExtension(puppies::Photo::likes_count, 42);
```

In other words, the only effect is that `likes_count` is defined within the scope of `puppies.Photo`.

​	换句话说，`likes_count` 仅在 `puppies.Photo` 的作用域内定义。

This is a common source of confusion: Declaring an `extend` block nested inside a message type *does not* imply any relationship between the outer type and the extended type. In particular, the earlier example *does not* mean that `Photo` is any sort of subclass of `UserProfile`. All it means is that the symbol `likes_count` is declared inside the scope of `Photo`; it’s simply a static member.

​	这是一种常见的混淆来源：在消息类型中嵌套声明 `extend` 块*并不*意味着外部类型与扩展类型之间有任何关系。特别是，前面的示例*并不*意味着 `Photo` 是 `UserProfile` 的任何形式的子类。这仅仅表示符号 `likes_count` 在 `Photo` 的作用域内声明；它只是一个静态成员。

A common pattern is to define extensions inside the scope of the extension’s field type - for example, here’s an extension to `media.UserContent` of type `puppies.Photo`, where the extension is defined as part of `Photo`:

​	一种常见模式是将扩展定义在扩展字段类型的作用域内。例如，以下是一个针对 `media.UserContent` 的扩展，其类型为 `puppies.Photo`，扩展定义为 `Photo` 的一部分：

```proto
import "media/user_content.proto";

package puppies;

message Photo {
  extend media.UserContent {
    optional Photo puppy_photo = 127;
  }
  ...
}
```

However, there is no requirement that an extension with a message type be defined inside that type. You can also use the standard definition pattern:

​	然而，并不要求一个消息类型的扩展必须在该类型内部定义。您也可以使用标准定义模式：

```proto
import "media/user_content.proto";

package puppies;

message Photo {
  ...
}

// This can even be in a different file.
extend media.UserContent {
  optional Photo puppy_photo = 127;
}
```

This **standard (file-level) syntax is preferred** to avoid confusion. The nested syntax is often mistaken for subclassing by users who are not already familiar with extensions.

​	**标准（文件级别）语法**是首选，以避免混淆。嵌套语法经常被不熟悉扩展的用户误认为是子类化。

## Any

The `Any` message type lets you use messages as embedded types without having their .proto definition. An `Any` contains an arbitrary serialized message as `bytes`, along with a URL that acts as a globally unique identifier for and resolves to that message’s type. To use the `Any` type, you need to [import](https://protobuf.dev/programming-guides/proto2/#other) `google/protobuf/any.proto`.

​	`Any` 消息类型允许您将消息用作嵌入类型，而无需其 `.proto` 定义。`Any` 包含一个任意的序列化消息（以 `bytes` 表示），以及一个 URL，该 URL 充当消息类型的全局唯一标识符并解析到该类型。要使用 `Any` 类型，您需要 [导入](https://protobuf.dev/programming-guides/proto2/#other) `google/protobuf/any.proto`。

```proto
import "google/protobuf/any.proto";

message ErrorStatus {
  string message = 1;
  repeated google.protobuf.Any details = 2;
}
```

The default type URL for a given message type is `type.googleapis.com/_packagename_._messagename_`.

​	对于给定消息类型的默认类型 URL 为 `type.googleapis.com/_packagename_._messagename_`。

Different language implementations will support runtime library helpers to pack and unpack `Any` values in a typesafe manner – for example, in Java, the `Any` type will have special `pack()` and `unpack()` accessors, while in C++ there are `PackFrom()` and `UnpackTo()` methods:

​	不同语言的实现将支持运行时库助手，以类型安全的方式打包和解包 `Any` 值。例如，在 Java 中，`Any` 类型将具有特殊的 `pack()` 和 `unpack()` 访问器，而在 C++ 中有 `PackFrom()` 和 `UnpackTo()` 方法：

```c++
// Storing an arbitrary message type in Any.
// 将任意消息类型存储在 Any 中。
NetworkErrorDetails details = ...;
ErrorStatus status;
status.add_details()->PackFrom(details);

// Reading an arbitrary message from Any.
// 从 Any 中读取任意消息。
ErrorStatus status = ...;
for (const google::protobuf::Any& detail : status.details()) {
  if (detail.Is<NetworkErrorDetails>()) {
    NetworkErrorDetails network_error;
    detail.UnpackTo(&network_error);
    ... processing network_error ...
  }
}
```

If you want to limit contained messages to a small number of types and to require permission before adding new types to the list, consider using [extensions](https://protobuf.dev/programming-guides/proto2/#extensions) with [extension declarations]({{< ref "/docs/ProgrammingGuides/ExtensionDeclarations" >}}) instead of `Any` message types.

​	如果您希望将包含的消息限制为少量类型，并要求在将新类型添加到列表之前进行许可，请考虑使用 [扩展](https://protobuf.dev/programming-guides/proto2/#extensions)和 [扩展声明]({{< ref "/docs/ProgrammingGuides/ExtensionDeclarations" >}})，而不是 `Any` 消息类型。

## Oneof

If you have a message with many optional fields and where at most one field will be set at the same time, you can enforce this behavior and save memory by using the oneof feature.

​	如果您的消息包含许多可选字段且一次最多只能设置一个字段，您可以使用 `oneof` 特性来强制执行此行为并节省内存。

Oneof fields are like optional fields except all the fields in a oneof share memory, and at most one field can be set at the same time. Setting any member of the oneof automatically clears all the other members. You can check which value in a oneof is set (if any) using a special `case()` or `WhichOneof()` method, depending on your chosen language.

​	`Oneof` 字段类似于可选字段，但 `oneof` 中的所有字段共享内存，并且一次最多只能设置一个字段。设置 `oneof` 的任何成员会自动清除所有其他成员。您可以使用特殊的 `case()` 或 `WhichOneof()` 方法（取决于您选择的语言）检查 `oneof` 中设置了哪个值（如果有的话）。

Note that if *multiple values are set, the last set value as determined by the order in the proto will overwrite all previous ones*.

​	请注意，如果*设置了多个值，根据 `proto` 中的顺序，最后设置的值将覆盖所有先前设置的值*。

Field numbers for oneof fields must be unique within the enclosing message.

​	`Oneof` 字段的字段编号在包含的消息中必须唯一。

### Using Oneof

To define a oneof in your `.proto` you use the `oneof` keyword followed by your oneof name, in this case `test_oneof`:

​	要在 `.proto` 文件中定义 `oneof`，您可以使用 `oneof` 关键字，后跟您的 `oneof` 名称，例如 `test_oneof`：

```proto
message SampleMessage {
  oneof test_oneof {
     string name = 4;
     SubMessage sub_message = 9;
  }
}
```

You then add your oneof fields to the oneof definition. You can add fields of any type except `map` fields, but you cannot use the `required`, `optional`, or `repeated` keywords. If you need to add a repeated field to a oneof, you can use a message containing the repeated field.

​	然后将您的 `oneof` 字段添加到 `oneof` 定义中。您可以添加任何类型的字段，除了 `map` 字段，但不能使用 `required`、`optional` 或 `repeated` 关键字。如果需要将重复字段添加到 `oneof` 中，可以使用包含重复字段的消息。

In your generated code, oneof fields have the same getters and setters as regular `optional` fields. You also get a special method for checking which value (if any) in the oneof is set. You can find out more about the oneof API for your chosen language in the relevant [API reference]({{< ref "/docs/ReferenceGuides" >}}).

​	在生成的代码中，`oneof` 字段具有与常规 `optional` 字段相同的 getter 和 setter。您还可以获得一个特殊方法，用于检查 `oneof` 中的哪个值（如果有）已被设置。有关您选择的语言的 `oneof` API 的更多信息，请参阅相关的 [API 参考]({{< ref "/docs/ReferenceGuides" >}})。

### Oneof 特性 Oneof Features

- Setting a oneof field will automatically clear all other members of the oneof. So if you set several oneof fields, only the *last* field you set will still have a value. 设置一个 `oneof` 字段会自动清除 `oneof` 中的所有其他成员。因此，如果设置了多个 `oneof` 字段，只有最后一个设置的字段会保留值。

  ```c++
  SampleMessage message;
  message.set_name("name");
  CHECK(message.has_name());
  // Calling mutable_sub_message() will clear the name field and will set
  // sub_message to a new instance of SubMessage with none of its fields set.
  // 调用 mutable_sub_message() 会清除 name 字段，并将 sub_message 设置为一个新的 SubMessage 实例，且其字段未设置。
  message.mutable_sub_message();
  CHECK(!message.has_name());
  ```

- If the parser encounters multiple members of the same oneof on the wire, only the last member seen is used in the parsed message.

  - 如果解析器在同一 `oneof` 中遇到多个成员，则仅使用最后一个成员的值来解析消息。

- Extensions are not supported for oneof.

  - `Oneof` 不支持扩展。

- A oneof cannot be `repeated`.

  - 一个 `oneof` 字段不能是 `repeated` 类型。

- Reflection APIs work for oneof fields.

  - 反射 API 对 `oneof` 字段有效。

- If you set a oneof field to the default value (such as setting an int32 oneof field to 0), the “case” of that oneof field will be set, and the value will be serialized on the wire.

  - 如果将 `oneof` 字段设置为默认值（例如将 int32 的 `oneof` 字段设置为 0），该 `oneof` 字段的“case”状态将被设置，并且该值会被序列化到线上。

- If you’re using C++, make sure your code doesn’t cause memory crashes. The following sample code will crash because `sub_message` was already deleted by calling the `set_name()` method. 如果您使用 C++，确保代码不会导致内存崩溃。以下示例代码会崩溃，因为调用 `set_name()` 方法后，`sub_message` 已被删除：

  ```c++
  SampleMessage message;
  SubMessage* sub_message = message.mutable_sub_message();
  message.set_name("name");      // Will delete sub_message 将删除 sub_message
  sub_message->set_...            // Crashes here 此处会崩溃
  ```

- Again in C++, if you `Swap()` two messages with oneofs, each message will end up with the other’s oneof case: in the example below, `msg1` will have a `sub_message` and `msg2` will have a `name`. 同样在 C++ 中，如果对包含 `oneof` 的两个消息调用 `Swap()`，每个消息都会交换彼此的 `oneof` 状态：在下面的示例中，`msg1` 将具有 `sub_message`，而 `msg2` 将具有 `name`。

  ```c++
  SampleMessage msg1;
  msg1.set_name("name");
  SampleMessage msg2;
  msg2.mutable_sub_message();
  msg1.swap(&msg2);
  CHECK(msg1.has_sub_message());
  CHECK(msg2.has_name());
  ```

### 向后兼容性问题 Backwards-compatibility issues

Be careful when adding or removing oneof fields. If checking the value of a oneof returns `None`/`NOT_SET`, it could mean that the oneof has not been set or it has been set to a field in a different version of the oneof. There is no way to tell the difference, since there’s no way to know if an unknown field on the wire is a member of the oneof.

​	在添加或移除 `oneof` 字段时需要小心。如果检查 `oneof` 的值返回 `None`/`NOT_SET`，可能表示 `oneof` 未被设置，或者被设置为不同版本 `oneof` 中的字段。没有办法区分这种情况，因为无法知道在线上的未知字段是否属于该 `oneof`。

#### 标签重用问题 Tag Reuse Issues

- **Move optional fields into or out of a oneof**: You may lose some of your information (some fields will be cleared) after the message is serialized and parsed. However, you can safely move a single field into a **new** oneof and may be able to move multiple fields if it is known that only one is ever set. See [Updating A Message Type](https://protobuf.dev/programming-guides/proto2/#updating) for further details.
  - **将可选字段移动到或移出 `oneof`**：在消息序列化和解析后，您可能会丢失一些信息（某些字段会被清除）。不过，您可以安全地将单个字段移动到**新的** `oneof` 中；如果确定一次仅设置一个字段，也可以移动多个字段。有关详细信息，请参阅 [更新消息类型](https://protobuf.dev/programming-guides/proto2/#updating)。

- **Delete a oneof field and add it back**: This may clear your currently set oneof field after the message is serialized and parsed.
  - **删除一个 `oneof` 字段并重新添加**：这可能会清除当前已设置的 `oneof` 字段，在消息序列化和解析后表现为未设置。

- **Split or merge oneof**: This has similar issues to moving `optional` fields.
  - **拆分或合并 `oneof`**：这与移动 `optional` 字段存在类似问题。


## Maps

If you want to create an associative map as part of your data definition, protocol buffers provides a handy shortcut syntax:

​	如果您希望在数据定义中创建关联映射，Protocol Buffers 提供了一种便捷的语法：

```proto
map<key_type, value_type> map_field = N;
```

…where the `key_type` can be any integral or string type (so, any [scalar](https://protobuf.dev/programming-guides/proto2/#scalar) type except for floating point types and `bytes`). Note that neither enum nor proto messages are valid for `key_type`. The `value_type` can be any type except another map.

​	...其中 `key_type` 可以是任何整数或字符串类型（即任何 [标量](https://protobuf.dev/programming-guides/proto2/#scalar) 类型，但不包括浮点类型和 `bytes`）。请注意，枚举和消息类型不能用作 `key_type`。`value_type` 可以是除映射外的任何类型。

So, for example, if you wanted to create a map of projects where each `Project` message is associated with a string key, you could define it like this:

​	例如，如果想创建一个项目的映射，其中每个 `Project` 消息与一个字符串键关联，可以这样定义：

```proto
map<string, Project> projects = 3;
```

### 映射特性 Maps Features

- Extensions are not supported for maps.
  - 映射不支持扩展。

- Maps cannot be `repeated`, `optional`, or `required`.
  - 映射字段不能是 `repeated`、`optional` 或 `required`。

- Wire format ordering and map iteration ordering of map values is undefined, so you cannot rely on your map items being in a particular order.
  - 映射值的线格式排序和映射迭代顺序是未定义的，因此无法依赖映射项的特定顺序。

- When generating text format for a `.proto`, maps are sorted by key. Numeric keys are sorted numerically.
  - 当为 `.proto` 生成文本格式时，映射按键排序。数值键按数值排序。

- When parsing from the wire or when merging, if there are duplicate map keys the last key seen is used. When parsing a map from text format, parsing may fail if there are duplicate keys.
  - 当从线格式解析或合并映射时，如果存在重复的映射键，最后看到的键值对会被保留。从文本格式解析映射时，如果有重复键，可能会解析失败。

- If you provide a key but no value for a map field, the behavior when the field is serialized is language-dependent. In C++, Java, Kotlin, and Python the default value for the type is serialized, while in other languages nothing is serialized.
  - 如果提供了一个键但没有提供值，则该字段序列化的行为依赖于语言。在 C++、Java、Kotlin 和 Python 中，会序列化该类型的默认值，而在其他语言中不会序列化任何内容。

- No symbol `FooEntry` can exist in the same scope as a map `foo`, because `FooEntry` is already used by the implementation of the map.
  - 在映射 `foo` 的作用域内，不能存在符号 `FooEntry`，因为 `FooEntry` 已被映射的实现占用。


The generated map API is currently available for all supported languages. You can find out more about the map API for your chosen language in the relevant [API reference]({{< ref "/docs/ReferenceGuides" >}}).

​	生成的映射 API 目前适用于所有支持的语言。有关您选择的语言的映射 API 的更多信息，请参阅相关 [API 参考]({{< ref "/docs/ReferenceGuides" >}})。

### 向后兼容性 Backwards Compatibility

The map syntax is equivalent to the following on the wire, so protocol buffers implementations that do not support maps can still handle your data:

​	映射语法在在线格式中等同于以下内容，因此不支持映射的 Protocol Buffers 实现仍然可以处理您的数据：

```proto
message MapFieldEntry {
  optional key_type key = 1;
  optional value_type value = 2;
}

repeated MapFieldEntry map_field = N;
```

Any protocol buffers implementation that supports maps must both produce and accept data that can be accepted by the earlier definition.

​	任何支持映射的 Protocol Buffers 实现都必须能够生成和接受与上述定义兼容的数据。

## Packages

You can add an optional `package` specifier to a `.proto` file to prevent name clashes between protocol message types.

​	您可以在 `.proto` 文件中添加一个可选的 `package` 指定符，以防止协议消息类型之间发生名称冲突。

```proto
package foo.bar;
message Open { ... }
```

You can then use the package specifier when defining fields of your message type:

​	然后，您可以在定义消息类型的字段时使用 `package` 指定符：

```proto
message Foo {
  ...
  optional foo.bar.Open open = 1;
  ...
}
```

The way a package specifier affects the generated code depends on your chosen language:

​	`package` 指定符对生成代码的影响取决于您选择的语言：

- In **C++** the generated classes are wrapped inside a C++ namespace. For example, `Open` would be in the namespace `foo::bar`.
  - **C++** 中，生成的类会被包装在 C++ 命名空间中。例如，`Open` 会在命名空间 `foo::bar` 中。

- In **Java** and **Kotlin**, the package is used as the Java package, unless you explicitly provide an `option java_package` in your `.proto` file.
  - **Java** 和 **Kotlin** 中，`package` 会作为 Java 包，除非您在 `.proto` 文件中显式提供了 `option java_package`。

- In **Python**, the `package` directive is ignored, since Python modules are organized according to their location in the file system.
  - **Python** 中，`package` 指令被忽略，因为 Python 模块是根据其在文件系统中的位置组织的。

- In **Go**, the `package` directive is ignored, and the generated `.pb.go` file is in the package named after the corresponding `go_proto_library` Bazel rule. For open source projects, you **must** provide either a `go_package` option or set the Bazel `-M` flag.
  - **Go** 中，`package` 指令被忽略，生成的 `.pb.go` 文件会位于对应 Bazel 规则 `go_proto_library` 命名的包中。对于开源项目，您**必须**提供一个 `go_package` 选项或设置 Bazel 的 `-M` 标志。

- In **Ruby**, the generated classes are wrapped inside nested Ruby namespaces, converted to the required Ruby capitalization style (first letter capitalized; if the first character is not a letter, `PB_` is prepended). For example, `Open` would be in the namespace `Foo::Bar`.
  - **Ruby** 中，生成的类会被包装在嵌套的 Ruby 命名空间中，转换为所需的 Ruby 大写风格（首字母大写；如果首字符不是字母，则会加上 `PB_`）。例如，`Open` 会在命名空间 `Foo::Bar` 中。

- In **PHP** the package is used as the namespace after converting to PascalCase, unless you explicitly provide an `option php_namespace` in your `.proto` file. For example, `Open` would be in the namespace `Foo\Bar`.
  - **PHP** 中，`package` 会转换为 PascalCase 作为命名空间，除非您在 `.proto` 文件中显式提供了 `option php_namespace`。例如，`Open` 会在命名空间 `Foo\Bar` 中。

- In **C#** the package is used as the namespace after converting to PascalCase, unless you explicitly provide an `option csharp_namespace` in your `.proto` file. For example, `Open` would be in the namespace `Foo.Bar`.
  - **C#** 中，`package` 会转换为 PascalCase 作为命名空间，除非您在 `.proto` 文件中显式提供了 `option csharp_namespace`。例如，`Open` 会在命名空间 `Foo.Bar` 中。


Note that even when the `package` directive does not directly affect the generated code, for example in Python, it is still strongly recommended to specify the package for the `.proto` file, as otherwise it may lead to naming conflicts in descriptors and make the proto not portable for other languages.

​	请注意，即使 `package` 指令不会直接影响生成的代码（例如在 Python 中），仍然强烈建议为 `.proto` 文件指定 `package`，否则可能会导致描述符中发生名称冲突，并使该 proto 文件对其他语言不具有可移植性。

### 包与名称解析 Packages and Name Resolution

Type name resolution in the protocol buffer language works like C++: first the innermost scope is searched, then the next-innermost, and so on, with each package considered to be “inner” to its parent package. A leading ‘.’ (for example, `.foo.bar.Baz`) means to start from the outermost scope instead.

​	协议缓冲区语言中的类型名称解析方式类似于 C++：首先搜索最内层的作用域，然后逐步向外扩展，每个包被视为其父包的“内层”。一个以 `.` 开头的名称（例如 `.foo.bar.Baz`）表示从最外层作用域开始搜索。

The protocol buffer compiler resolves all type names by parsing the imported `.proto` files. The code generator for each language knows how to refer to each type in that language, even if it has different scoping rules.

​	协议缓冲区编译器通过解析导入的 `.proto` 文件来解析所有类型名称。每种语言的代码生成器都知道如何在该语言中引用每种类型，即使它具有不同的作用域规则。

## 定义服务 Defining Services

If you want to use your message types with an RPC (Remote Procedure Call) system, you can define an RPC service interface in a `.proto` file and the protocol buffer compiler will generate service interface code and stubs in your chosen language. So, for example, if you want to define an RPC service with a method that takes your `SearchRequest` and returns a `SearchResponse`, you can define it in your `.proto` file as follows:

​	如果您希望将消息类型与 RPC（远程过程调用）系统一起使用，可以在 `.proto` 文件中定义一个 RPC 服务接口，并由协议缓冲区编译器生成服务接口代码和存根代码。例如，如果您想定义一个 RPC 服务，其中包含一个方法接收 `SearchRequest` 并返回 `SearchResponse`，可以在 `.proto` 文件中这样定义：

```proto
service SearchService {
  rpc Search(SearchRequest) returns (SearchResponse);
}
```

By default, the protocol compiler will then generate an abstract interface called `SearchService` and a corresponding “stub” implementation. The stub forwards all calls to an `RpcChannel`, which in turn is an abstract interface that you must define yourself in terms of your own RPC system. For example, you might implement an `RpcChannel` which serializes the message and sends it to a server via HTTP. In other words, the generated stub provides a type-safe interface for making protocol-buffer-based RPC calls, without locking you into any particular RPC implementation. So, in C++, you might end up with code like this:

​	默认情况下，协议缓冲区编译器会生成一个名为 `SearchService` 的抽象接口及一个对应的“存根”实现。存根将所有调用转发到一个 `RpcChannel`，而 `RpcChannel` 是一个抽象接口，您需要根据自己的 RPC 系统自行定义。例如，您可以实现一个 `RpcChannel` 来序列化消息并通过 HTTP 将其发送到服务器。换句话说，生成的存根提供了一种基于协议缓冲区的 RPC 调用的类型安全接口，而不会绑定到任何特定的 RPC 实现中。因此，在 C++ 中，您可能会看到如下代码：

```c++
using google::protobuf;

protobuf::RpcChannel* channel;
protobuf::RpcController* controller;
SearchService* service;
SearchRequest request;
SearchResponse response;

void DoSearch() {
  // You provide classes MyRpcChannel and MyRpcController, which implement
  // the abstract interfaces protobuf::RpcChannel and protobuf::RpcController.
  // 提供 MyRpcChannel 和 MyRpcController 类，这些类实现了抽象接口
  // protobuf::RpcChannel 和 protobuf::RpcController。
  channel = new MyRpcChannel("somehost.example.com:1234");
  controller = new MyRpcController;

  // The protocol compiler generates the SearchService class based on the
  // definition given earlier.
  // 协议缓冲区编译器根据之前的定义生成 SearchService 类。
  service = new SearchService::Stub(channel);

  // Set up the request.
  // 设置请求。
  request.set_query("protocol buffers");

  // Execute the RPC.
  // 执行 RPC。
  service->Search(controller, &request, &response,
                  protobuf::NewCallback(&Done));
}

void Done() {
  delete service;
  delete channel;
  delete controller;
}
```

All service classes also implement the `Service` interface, which provides a way to call specific methods without knowing the method name or its input and output types at compile time. On the server side, this can be used to implement an RPC server with which you could register services.

​	所有服务类也实现了 `Service` 接口，该接口提供了一种方法，可以在不知道方法名称或其输入和输出类型的情况下调用特定方法。在服务器端，这可用于实现一个 RPC 服务器，您可以在其中注册服务。

```c++
using google::protobuf;

class ExampleSearchService : public SearchService {
 public:
  void Search(protobuf::RpcController* controller,
              const SearchRequest* request,
              SearchResponse* response,
              protobuf::Closure* done) {
    if (request->query() == "google") {
      response->add_result()->set_url("http://www.google.com");
    } else if (request->query() == "protocol buffers") {
      response->add_result()->set_url("http://protobuf.googlecode.com");
    }
    done->Run();
  }
};

int main() {
  // You provide class MyRpcServer.  It does not have to implement any
  // particular interface; this is just an example.
  // 提供类 MyRpcServer。它不需要实现任何特定接口；这是一个示例。
  MyRpcServer server;

  protobuf::Service* service = new ExampleSearchService;
  server.ExportOnPort(1234, service);
  server.Run();

  delete service;
  return 0;
}
```

If you don’t want to plug in your own existing RPC system, you can use [gRPC](https://github.com/grpc/grpc-common): a language- and platform-neutral open source RPC system developed at Google. gRPC works particularly well with protocol buffers and lets you generate the relevant RPC code directly from your `.proto` files using a special protocol buffer compiler plugin. However, as there are potential compatibility issues between clients and servers generated with proto2 and proto3, we recommend that you use proto3 for defining gRPC services. You can find out more about proto3 syntax in the [Proto3 Language Guide]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto3" >}}). If you do want to use proto2 with gRPC, you need to use version 3.0.0 or higher of the protocol buffers compiler and libraries.

​	如果您不想集成已有的 RPC 系统，可以使用 [gRPC](https://github.com/grpc/grpc-common)：由 Google 开发的一个语言和平台中立的开源 RPC 系统。gRPC 与协议缓冲区结合得非常好，并允许您使用协议缓冲区编译器的特殊插件直接从 `.proto` 文件生成相关的 RPC 代码。不过，由于使用 proto2 和 proto3 定义的客户端和服务器之间可能存在兼容性问题，我们建议使用 proto3 定义 gRPC 服务。有关 proto3 语法的更多信息，请参阅 [Proto3 语言指南]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto3" >}})。如果您确实希望在 gRPC 中使用 proto2，您需要使用 3.0.0 或更高版本的协议缓冲区编译器和库。

In addition to gRPC, there are also a number of ongoing third-party projects to develop RPC implementations for Protocol Buffers. For a list of links to projects we know about, see the [third-party add-ons wiki page](https://github.com/protocolbuffers/protobuf/blob/master/docs/third_party.md).

​	除了 gRPC，还有许多第三方正在开发基于协议缓冲区的 RPC 实现的项目。有关我们所知项目的链接列表，请参阅 [第三方附加组件 wiki 页面](https://github.com/protocolbuffers/protobuf/blob/master/docs/third_party.md)。

## JSON 映射 JSON Mapping

The standard protobuf binary wire format is the preferred serialization format for communication between two systems that use protobufs. For communicating with systems that use JSON rather than protobuf wire format, Protobuf supports a canonical encoding in [JSON]({{< ref "/docs/ProgrammingGuides/ProtoJSONFormat" >}}).

​	标准的 Protobuf 二进制线格式是两个使用 Protobuf 的系统之间通信的首选序列化格式。如果需要与使用 JSON 而不是 Protobuf 线格式的系统通信，Protobuf 支持一种标准的 [JSON]({{< ref "/docs/ProgrammingGuides/ProtoJSONFormat" >}}) 编码。

## 选项 Options

Individual declarations in a `.proto` file can be annotated with a number of *options*. Options do not change the overall meaning of a declaration, but may affect the way it is handled in a particular context. The complete list of available options is defined in [`/google/protobuf/descriptor.proto`](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/descriptor.proto).

​	`.proto` 文件中的各个声明可以用许多*选项*注解。选项不会改变声明的整体含义，但可能会影响它在特定上下文中的处理方式。完整的可用选项列表定义在 [`/google/protobuf/descriptor.proto`](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/descriptor.proto) 中。

Some options are file-level options, meaning they should be written at the top-level scope, not inside any message, enum, or service definition. Some options are message-level options, meaning they should be written inside message definitions. Some options are field-level options, meaning they should be written inside field definitions. Options can also be written on enum types, enum values, oneof fields, service types, and service methods; however, no useful options currently exist for any of these.

​	某些选项是文件级选项，意味着它们应该写在顶级作用域中，而不是任何消息、枚举或服务定义中。某些选项是消息级选项，意味着它们应该写在消息定义中。某些选项是字段级选项，意味着它们应该写在字段定义中。选项也可以应用于枚举类型、枚举值、`oneof` 字段、服务类型和服务方法，但目前这些方面没有有用的选项。

Here are a few of the most commonly used options:

​	以下是一些最常用的选项：

- `java_package` (file option): The package you want to use for your generated Java/Kotlin classes. If no explicit `java_package` option is given in the `.proto` file, then by default the proto package (specified using the “package” keyword in the `.proto` file) will be used. However, proto packages generally do not make good Java packages since proto packages are not expected to start with reverse domain names. If not generating Java or Kotlin code, this option has no effect. `java_package`（文件选项）：您希望生成的 Java/Kotlin 类使用的包。如果 `.proto` 文件中没有明确给出 `java_package` 选项，则默认使用 `.proto` 文件中通过“package”关键字指定的包。不过，proto 包通常不是好的 Java 包，因为它们不以反向域名开头。如果未生成 Java 或 Kotlin 代码，此选项无效。

  ```proto
  option java_package = "com.example.foo";
  ```

- `java_outer_classname` (file option): The class name (and hence the file name) for the wrapper Java class you want to generate. If no explicit `java_outer_classname` is specified in the `.proto` file, the class name will be constructed by converting the `.proto` file name to camel-case (so `foo_bar.proto` becomes `FooBar.java`). If the `java_multiple_files` option is disabled, then all other classes/enums/etc. generated for the `.proto` file will be generated *within* this outer wrapper Java class as nested classes/enums/etc. If not generating Java code, this option has no effect. `java_outer_classname`（文件选项）：您希望生成的 Java 包装类的类名（因此也是文件名）。如果 `.proto` 文件中没有明确指定 `java_outer_classname`，则类名会通过将 `.proto` 文件名转换为驼峰命名法来构建（例如，`foo_bar.proto` 变为 `FooBar.java`）。如果 `java_multiple_files` 选项被禁用，则为此 `.proto` 文件生成的所有其他类/枚举等将作为嵌套类/枚举生成在此外部包装类中。如果未生成 Java 代码，此选项无效。

  ```proto
  option java_outer_classname = "Ponycopter";
  ```

- `java_multiple_files` (file option): If false, only a single `.java` file will be generated for this `.proto` file, and all the Java classes/enums/etc. generated for the top-level messages, services, and enumerations will be nested inside of an outer class (see `java_outer_classname`). If true, separate `.java` files will be generated for each of the Java classes/enums/etc. generated for the top-level messages, services, and enumerations, and the wrapper Java class generated for this `.proto` file won’t contain any nested classes/enums/etc. This is a Boolean option which defaults to `false`. If not generating Java code, this option has no effect. `java_multiple_files`（文件选项）：如果为 `false`，只会为此 `.proto` 文件生成一个 `.java` 文件，顶级消息、服务和枚举生成的所有 Java 类/枚举等都将嵌套在外部类中（参见 `java_outer_classname`）。如果为 `true`，将为顶级消息、服务和枚举生成的每个 Java 类/枚举等生成单独的 `.java` 文件，此 `.proto` 文件生成的包装 Java 类将不包含任何嵌套类/枚举等。这是一个布尔选项，默认值为 `false`。如果未生成 Java 代码，此选项无效。

  ```proto
  option java_multiple_files = true;
  ```

- `optimize_for` (file option): Can be set to `SPEED`, `CODE_SIZE`, or `LITE_RUNTIME`. This affects the C++ and Java code generators (and possibly third-party generators) in the following ways: `optimize_for`（文件选项）：可以设置为 `SPEED`、`CODE_SIZE` 或 `LITE_RUNTIME`。这会以以下方式影响 C++ 和 Java 代码生成器（以及可能的第三方生成器）：

  - `SPEED` (default): The protocol buffer compiler will generate code for serializing, parsing, and performing other common operations on your message types. This code is highly optimized.
    - `SPEED`（默认）：协议缓冲区编译器会生成高度优化的代码，用于序列化、解析以及对消息类型执行其他常见操作。

  - `CODE_SIZE`: The protocol buffer compiler will generate minimal classes and will rely on shared, reflection-based code to implement serialization, parsing, and various other operations. The generated code will thus be much smaller than with `SPEED`, but operations will be slower. Classes will still implement exactly the same public API as they do in `SPEED` mode. This mode is most useful in apps that contain a very large number of `.proto` files and do not need all of them to be blindingly fast.
    - `CODE_SIZE`：协议缓冲区编译器会生成最小化的类，并依赖于共享的基于反射的代码来实现序列化、解析以及其他各种操作。因此，生成的代码会比使用 `SPEED` 时小得多，但操作会更慢。类仍将实现与 `SPEED` 模式下完全相同的公共 API。此模式最适用于包含大量 `.proto` 文件且并不需要所有文件都非常快的应用程序。

  - `LITE_RUNTIME`: The protocol buffer compiler will generate classes that depend only on the “lite” runtime library (`libprotobuf-lite` instead of `libprotobuf`). The lite runtime is much smaller than the full library (around an order of magnitude smaller) but omits certain features like descriptors and reflection. This is particularly useful for apps running on constrained platforms like mobile phones. The compiler will still generate fast implementations of all methods as it does in `SPEED` mode. Generated classes will only implement the `MessageLite` interface in each language, which provides only a subset of the methods of the full `Message` interface. `LITE_RUNTIME`：协议缓冲区编译器会生成仅依赖“轻量级”运行时库（`libprotobuf-lite` 而非 `libprotobuf`）的类。轻量级运行时比完整库小得多（小一个数量级），但省略了某些功能，如描述符和反射。这对于运行在受限平台（如移动设备）上的应用程序特别有用。编译器仍会生成所有方法的快速实现，与 `SPEED` 模式相同。生成的类仅实现每种语言中的 `MessageLite` 接口，该接口只提供完整 `Message` 接口的一部分方法。

  ```proto
  option optimize_for = CODE_SIZE;
  ```

- `cc_generic_services`, `java_generic_services`, `py_generic_services` (file options): **Generic services are deprecated.** Whether or not the protocol buffer compiler should generate abstract service code based on [services definitions](https://protobuf.dev/programming-guides/proto2/#services) in C++, Java, and Python, respectively. For legacy reasons, these default to `true`. However, as of version 2.3.0 (January 2010), it is considered preferable for RPC implementations to provide [code generator plugins](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.compiler.plugin.pb) to generate code more specific to each system, rather than rely on the “abstract” services. **`cc_generic_services`、`java_generic_services`、`py_generic_services`（文件选项）**：**通用服务已被弃用**。是否让协议缓冲区编译器基于 [服务定义](https://protobuf.dev/programming-guides/proto2/#services) 生成抽象服务代码，分别适用于 C++、Java 和 Python。出于遗留原因，这些选项默认值为 `true`。然而，自 2.3.0 版本（2010 年 1 月）起，推荐为 RPC 实现提供 [代码生成器插件](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.compiler.plugin.pb)，以生成更适合每个系统的代码，而不是依赖于“抽象”服务。

  ```proto
  // This file relies on plugins to generate service code.
  option cc_generic_services = false;
  option java_generic_services = false;
  option py_generic_services = false;
  ```

- `cc_enable_arenas` (file option): Enables [arena allocation]({{< ref "/docs/ReferenceGuides/CPlusPlus/ArenaAllocationGuide" >}}) for C++ generated code.

  - **`cc_enable_arenas`（文件选项）**：启用 C++ 生成代码的 [arena 分配]({{< ref "/docs/ReferenceGuides/CPlusPlus/ArenaAllocationGuide" >}})。

- `objc_class_prefix` (file option): Sets the Objective-C class prefix which is prepended to all Objective-C generated classes and enums from this .proto. There is no default. You should use prefixes that are between 3-5 uppercase characters as [recommended by Apple](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Conventions/Conventions.html#//apple_ref/doc/uid/TP40011210-CH10-SW4). Note that all 2 letter prefixes are reserved by Apple.

  - **`objc_class_prefix`（文件选项）**：设置 Objective-C 类前缀，该前缀会加在此 `.proto` 文件生成的所有 Objective-C 类和枚举之前。默认值为空。您应使用 3-5 个大写字符的前缀，遵循 [Apple 的推荐](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Conventions/Conventions.html#//apple_ref/doc/uid/TP40011210-CH10-SW4)。注意，所有 2 个字母的前缀已被 Apple 保留。

- `message_set_wire_format` (message option): If set to `true`, the message uses a different binary format intended to be compatible with an old format used inside Google called `MessageSet`. Users outside Google will probably never need to use this option. The message must be declared exactly as follows: **`message_set_wire_format`（消息选项）**：如果设置为 `true`，该消息将使用一种与 Google 内部旧格式 `MessageSet` 兼容的不同二进制格式。Google 以外的用户可能永远不需要使用此选项。消息必须声明为以下形式：

  ```proto
  message Foo {
    option message_set_wire_format = true;
    extensions 4 to max;
  }
  ```

- `packed` (field option): If set to `true` on a repeated field of a basic numeric type, it causes a more compact [encoding]({{< ref "/docs/ProgrammingGuides/Encoding#packed" >}}) to be used. The only reason to not use this option is if you need compatibility with parsers prior to version 2.3.0. When these older parsers would ignore packed data when it was not expected. Therefore, it was not possible to change an existing field to packed format without breaking wire compatibility. In 2.3.0 and later, this change is safe, as parsers for packable fields will always accept both formats, but be careful if you have to deal with old programs using old protobuf versions. **`packed`（字段选项）**：如果对基本数值类型的 `repeated` 字段设置为 `true`，则会使用更紧凑的[编码格式]({{< ref "/docs/ProgrammingGuides/Encoding#packed" >}})。唯一不使用此选项的原因是需要与 2.3.0 之前版本的解析器兼容。这些较早的解析器在未预期打包数据时会忽略它。因此，在不破坏线格式兼容性的情况下，无法将现有字段更改为打包格式。从 2.3.0 开始，对于可打包字段，这种更改是安全的，因为解析器将始终接受两种格式，但如果需要与旧版 Protobuf 程序交互，仍需小心。

  ```proto
  repeated int32 samples = 4 [packed = true];
  ```

- `deprecated` (field option): If set to `true`, indicates that the field is deprecated and should not be used by new code. In most languages this has no actual effect. In Java, this becomes a `@Deprecated` annotation. For C++, clang-tidy will generate warnings whenever deprecated fields are used. In the future, other language-specific code generators may generate deprecation annotations on the field’s accessors, which will in turn cause a warning to be emitted when compiling code which attempts to use the field. If the field is not used by anyone and you want to prevent new users from using it, consider replacing the field declaration with a [reserved](https://protobuf.dev/programming-guides/proto2/#fieldreserved) statement. **`deprecated`（字段选项）**：如果设置为 `true`，表示该字段已被弃用，新代码不应再使用它。在大多数语言中，这没有实际影响。在 Java 中，这将成为 `@Deprecated` 注解。在 C++ 中，`clang-tidy` 在使用已弃用字段时会生成警告。未来，其他语言的代码生成器可能会在字段的访问器上生成弃用注解，这将导致在编译尝试使用该字段的代码时发出警告。如果该字段未被任何人使用并且您希望防止新用户使用它，可以考虑将字段声明替换为 [reserved](https://protobuf.dev/programming-guides/proto2/#fieldreserved) 声明。

  ```proto
  optional int32 old_field = 6 [deprecated=true];
  ```

### 枚举值选项 Enum Value Options

Enum value options are supported. You can use the `deprecated` option to indicate that a value shouldn’t be used anymore. You can also create custom options using extensions.

​	支持枚举值选项。可以使用 `deprecated` 选项来指示某个值不应再被使用。您还可以使用扩展来创建自定义选项。

The following example shows the syntax for adding these options:

​	以下示例展示了添加这些选项的语法：

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

The C++ code to read the `string_name` option might look something like this:

​	C++ 中读取 `string_name` 选项的代码示例如下：

```cpp
const absl::string_view foo = proto2::GetEnumDescriptor<Data>()
    ->FindValueByName("DATA_DISPLAY")->options().GetExtension(string_name);
```

See [Custom Options](https://protobuf.dev/programming-guides/proto2/#customoptions) to see how to apply custom options to enum values and to fields.

​	有关如何将自定义选项应用于枚举值和字段，请参阅 [Custom Options](https://protobuf.dev/programming-guides/proto2/#customoptions)。

### 自定义选项 Custom Options

Protocol Buffers also allows you to define and use your own options. Note that this is an **advanced feature** which most people don’t need. Since options are defined by the messages defined in `google/protobuf/descriptor.proto` (like `FileOptions` or `FieldOptions`), defining your own options is simply a matter of [extending](https://protobuf.dev/programming-guides/proto2/#extensions) those messages. For example:

​	协议缓冲区还允许您定义和使用自己的选项。请注意，这是一个**高级功能**，大多数用户可能不需要使用。由于选项是由 `google/protobuf/descriptor.proto` 中定义的消息（例如 `FileOptions` 或 `FieldOptions`）所定义的，因此定义自定义选项只是 [扩展](https://protobuf.dev/programming-guides/proto2/#extensions) 这些消息的问题。例如：

```proto
import "google/protobuf/descriptor.proto";

extend google.protobuf.MessageOptions {
  optional string my_option = 51234;
}

message MyMessage {
  option (my_option) = "Hello world!";
}
```

Here we have defined a new message-level option by extending `MessageOptions`. When we then use the option, the option name must be enclosed in parentheses to indicate that it is an extension. We can now read the value of `my_option` in C++ like so:

​	在这里，我们通过扩展 `MessageOptions` 定义了一个新的消息级选项。当我们使用该选项时，选项名称必须用括号括起来，以表示它是一个扩展。我们现在可以在 C++ 中读取 `my_option` 的值，如下所示：

```proto
string value = MyMessage::descriptor()->options().GetExtension(my_option);
```

Here, `MyMessage::descriptor()->options()` returns the `MessageOptions` protocol message for `MyMessage`. Reading custom options from it is just like reading any other [extension](https://protobuf.dev/programming-guides/proto2/#extensions).

​	这里，`MyMessage::descriptor()->options()` 返回 `MyMessage` 的 `MessageOptions` 协议消息。从中读取自定义选项与读取其他 [扩展](https://protobuf.dev/programming-guides/proto2/#extensions) 类似。

Similarly, in Java we would write:

​	同样，在 Java 中，可以这样写：

```java
String value = MyProtoFile.MyMessage.getDescriptor().getOptions()
  .getExtension(MyProtoFile.myOption);
```

In Python it would be:

​	在 Python 中可以这样写：

```python
value = my_proto_file_pb2.MyMessage.DESCRIPTOR.GetOptions()
  .Extensions[my_proto_file_pb2.my_option]
```

Custom options can be defined for every kind of construct in the Protocol Buffers language. Here is an example that uses every kind of option:

​	自定义选项可以定义在协议缓冲区语言中的每种构造上。以下是一个使用所有选项类型的示例：

```proto
import "google/protobuf/descriptor.proto";

extend google.protobuf.FileOptions {
  optional string my_file_option = 50000;
}
extend google.protobuf.MessageOptions {
  optional int32 my_message_option = 50001;
}
extend google.protobuf.FieldOptions {
  optional float my_field_option = 50002;
}
extend google.protobuf.OneofOptions {
  optional int64 my_oneof_option = 50003;
}
extend google.protobuf.EnumOptions {
  optional bool my_enum_option = 50004;
}
extend google.protobuf.EnumValueOptions {
  optional uint32 my_enum_value_option = 50005;
}
extend google.protobuf.ServiceOptions {
  optional MyEnum my_service_option = 50006;
}
extend google.protobuf.MethodOptions {
  optional MyMessage my_method_option = 50007;
}

option (my_file_option) = "Hello world!";

message MyMessage {
  option (my_message_option) = 1234;

  optional int32 foo = 1 [(my_field_option) = 4.5];
  optional string bar = 2;
  oneof qux {
    option (my_oneof_option) = 42;

    string quux = 3;
  }
}

enum MyEnum {
  option (my_enum_option) = true;

  FOO = 1 [(my_enum_value_option) = 321];
  BAR = 2;
}

message RequestType {}
message ResponseType {}

service MyService {
  option (my_service_option) = FOO;

  rpc MyMethod(RequestType) returns(ResponseType) {
    // Note:  my_method_option has type MyMessage.  We can set each field
    //   within it using a separate "option" line.
    // 注意：my_method_option 的类型是 MyMessage。我们可以通过单独的 "option" 行设置其每个字段。
    option (my_method_option).foo = 567;
    option (my_method_option).bar = "Some string";
  }
}
```

Note that if you want to use a custom option in a package other than the one in which it was defined, you must prefix the option name with the package name, just as you would for type names. For example:

​	请注意，如果要在定义它的包之外使用自定义选项，必须在选项名称前加上包名，就像类型名称一样。例如：

```proto
// foo.proto
import "google/protobuf/descriptor.proto";
package foo;
extend google.protobuf.MessageOptions {
  optional string my_option = 51234;
}
// bar.proto
import "foo.proto";
package bar;
message MyMessage {
  option (foo.my_option) = "Hello world!";
}
```

One last thing: Since custom options are extensions, they must be assigned field numbers like any other field or extension. In the examples earlier, we have used field numbers in the range 50000-99999. This range is reserved for internal use within individual organizations, so you can use numbers in this range freely for in-house applications. If you intend to use custom options in public applications, however, then it is important that you make sure that your field numbers are globally unique. To obtain globally unique field numbers, send a request to add an entry to [protobuf global extension registry](https://github.com/protocolbuffers/protobuf/blob/master/docs/options.md). Usually you only need one extension number. You can declare multiple options with only one extension number by putting them in a sub-message:

​	最后，由于自定义选项是扩展，因此必须像其他字段或扩展一样分配字段编号。在前面的示例中，我们使用了 50000-99999 范围内的字段编号。此范围保留供各组织内部使用，因此您可以自由地为内部应用程序使用这些编号。但是，如果您打算在公共应用程序中使用自定义选项，则必须确保字段编号是全局唯一的。要获取全局唯一的字段编号，请请求在 [protobuf 全球扩展注册表](https://github.com/protocolbuffers/protobuf/blob/master/docs/options.md) 中添加一个条目。通常，您只需要一个扩展编号。您可以通过将多个选项放在子消息中，只声明一个扩展编号：

```proto
message FooOptions {
  optional int32 opt1 = 1;
  optional string opt2 = 2;
}

extend google.protobuf.FieldOptions {
  optional FooOptions foo_options = 1234;
}

// usage:
// 使用方法：
message Bar {
  optional int32 a = 1 [(foo_options).opt1 = 123, (foo_options).opt2 = "baz"];
  // alternative aggregate syntax (uses TextFormat):
  optional int32 b = 2 [(foo_options) = { opt1: 123 opt2: "baz" }];
}
```

Also, note that each option type (file-level, message-level, field-level, etc.) has its own number space, so, for example, you could declare extensions of FieldOptions and MessageOptions with the same number.

​	此外，请注意，每种选项类型（文件级、消息级、字段级等）都有自己的编号空间，因此，例如，可以使用相同的编号声明 `FieldOptions` 和 `MessageOptions` 的扩展。

### 选项保留 Option Retention

Options have a notion of *retention*, which controls whether an option is retained in the generated code. Options have *runtime retention* by default, meaning that they are retained in the generated code and are thus visible at runtime in the generated descriptor pool. However, you can set `retention = RETENTION_SOURCE` to specify that an option (or field within an option) must not be retained at runtime. This is called *source retention*.

​	选项具有*保留*的概念，这决定了选项是否会保留在生成的代码中。默认情况下，选项具有*运行时保留*（runtime retention），这意味着它们会保留在生成的代码中，并且在生成的描述符池中可以在运行时查看。然而，您可以设置 `retention = RETENTION_SOURCE` 来指定选项（或选项中的字段）不必在运行时保留。这被称为*源保留*（source retention）。

Option retention is an advanced feature that most users should not need to worry about, but it can be useful if you would like to use certain options without paying the code size cost of retaining them in your binaries. Options with source retention are still visible to `protoc` and `protoc` plugins, so code generators can use them to customize their behavior.

​	选项保留是一个高级功能，大多数用户不需要担心这一点，但它在以下场景中可能有用：当您希望使用某些选项但又不想在二进制文件中保留它们而导致代码体积增大时。具有源保留的选项在 `protoc` 和 `protoc` 插件中仍然可见，因此代码生成器可以使用它们来自定义行为。

Retention can be set directly on an option, like this:

​	可以直接在选项上设置保留，例如：

```proto
extend google.protobuf.FileOptions {
  optional int32 source_retention_option = 1234
      [retention = RETENTION_SOURCE];
}
```

It can also be set on a plain field, in which case it takes effect only when that field appears inside an option:

​	也可以设置在普通字段上，此时保留效果仅在该字段出现在选项中时生效：

```proto
message OptionsMessage {
  optional int32 source_retention_field = 1 [retention = RETENTION_SOURCE];
}
```

You can set `retention = RETENTION_RUNTIME` if you like, but this has no effect since it is the default behavior. When a message field is marked `RETENTION_SOURCE`, its entire contents are dropped; fields inside it cannot override that by trying to set `RETENTION_RUNTIME`.

​	如果需要，也可以将 `retention` 设置为 `RETENTION_RUNTIME`，但这没有实际影响，因为这是默认行为。如果消息字段被标记为 `RETENTION_SOURCE`，其所有内容都会被删除；即使其内部字段尝试设置为 `RETENTION_RUNTIME` 也无效。

> Note
>
> As of Protocol Buffers 22.0, support for option retention is still in progress and only C++ and Java are supported. Go has support starting from 1.29.0. Python support is complete but has not made it into a release yet.
>
> ​	截至 Protocol Buffers 22.0，选项保留功能仍在完善中，目前仅支持 C++ 和 Java。Go 的支持从 1.29.0 开始。Python 支持已完成，但尚未发布。

### 选项目标 Option Targets

Fields have a `targets` option which controls the types of entities that the field may apply to when used as an option. For example, if a field has `targets = TARGET_TYPE_MESSAGE` then that field cannot be set in a custom option on an enum (or any other non-message entity). Protoc enforces this and will raise an error if there is a violation of the target constraints.

​	字段具有一个 `targets` 选项，用于控制字段在用作选项时可以应用的实体类型。例如，如果一个字段具有 `targets = TARGET_TYPE_MESSAGE`，则该字段不能在枚举（或任何其他非消息实体）上的自定义选项中设置。`protoc` 将对此进行强制检查，如果违反目标约束，将引发错误。

At first glance, this feature may seem unnecessary given that every custom option is an extension of the options message for a specific entity, which already constrains the option to that one entity. However, option targets are useful in the case where you have a shared options message applied to multiple entity types and you want to control the usage of individual fields in that message. For example:

​	乍一看，这个功能似乎没有必要，因为每个自定义选项都是特定实体的选项消息的扩展，已经限制了选项只能应用于该实体。然而，当您希望将共享的选项消息应用于多个实体类型，并希望控制消息中各字段的使用时，选项目标就非常有用。例如：

```proto
message MyOptions {
  optional string file_only_option = 1 [targets = TARGET_TYPE_FILE];
  optional int32 message_and_enum_option = 2 [targets = TARGET_TYPE_MESSAGE,
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
// OK: 该字段允许用于文件选项
option (file_options).file_only_option = "abc";

message MyMessage {
  // OK: this field is allowed on both message and enum options
  // OK: 该字段允许用于消息和枚举选项
  option (message_options).message_and_enum_option = 42;
}

enum MyEnum {
  MY_ENUM_UNSPECIFIED = 0;
  // Error: file_only_option cannot be set on an enum.
  // 错误: file_only_option 不能设置在枚举上
  option (enum_options).file_only_option = "xyz";
}
```

### 生成类 Generating Your Classes

To generate the Java, Kotlin, Python, C++, Go, Ruby, Objective-C, or C# code that you need to work with the message types defined in a `.proto` file, you need to run the protocol buffer compiler `protoc` on the `.proto` file. If you haven’t installed the compiler, [download the package]({{< ref "/docs/Downloads" >}}) and follow the instructions in the README. For Go, you also need to install a special code generator plugin for the compiler; you can find this and installation instructions in the [golang/protobuf](https://github.com/golang/protobuf/) repository on GitHub.

​	要生成 Java、Kotlin、Python、C++、Go、Ruby、Objective-C 或 C# 代码，以便处理 `.proto` 文件中定义的消息类型，您需要在 `.proto` 文件上运行协议缓冲编译器 `protoc`。如果尚未安装编译器，请[下载软件包]({{< ref "/docs/Downloads" >}})并按照 README 中的说明操作。对于 Go，还需要为编译器安装一个特殊的代码生成器插件；您可以在 GitHub 的 [golang/protobuf](https://github.com/golang/protobuf/) 仓库中找到该插件及安装说明。

The Protocol Compiler is invoked as follows:

​	协议缓冲编译器的调用方式如下：

```sh
protoc --proto_path=IMPORT_PATH --cpp_out=DST_DIR --java_out=DST_DIR --python_out=DST_DIR --go_out=DST_DIR --ruby_out=DST_DIR --objc_out=DST_DIR --csharp_out=DST_DIR path/to/file.proto
```

- `IMPORT_PATH` specifies a directory in which to look for `.proto` files when resolving `import` directives. If omitted, the current directory is used. Multiple import directories can be specified by passing the `--proto_path` option multiple times; they will be searched in order. `-I=_IMPORT_PATH_` can be used as a short form of `--proto_path`.

  - `IMPORT_PATH` 指定了解析 `import` 指令时查找 `.proto` 文件的目录。如果省略，将使用当前目录。可以通过多次传递 `--proto_path` 选项来指定多个导入目录；这些目录将按顺序搜索。`-I=_IMPORT_PATH_` 是 `--proto_path` 的简写形式。

- You can provide one or more *output directives*: 您可以提供一个或多个*输出指令*：

  - `--cpp_out` generates C++ code in `DST_DIR`. See the [C++ generated code reference]({{< ref "/docs/ReferenceGuides/CPlusPlus/GeneratedCodeGuide" >}}) for more.
    - `--cpp_out` 在 `DST_DIR` 中生成 C++ 代码。参见 [C++ 生成代码参考]({{< ref "/docs/ReferenceGuides/CPlusPlus/GeneratedCodeGuide" >}})。

  - `--java_out` generates Java code in `DST_DIR`. See the [Java generated code reference]({{< ref "/docs/ReferenceGuides/Java/GeneratedCodeGuide" >}}) for more.
    - `--java_out` 在 `DST_DIR` 中生成 Java 代码。参见 [Java 生成代码参考]({{< ref "/docs/ReferenceGuides/Java/GeneratedCodeGuide" >}})。

  - `--kotlin_out` generates additional Kotlin code in `DST_DIR`. See the [Kotlin generated code reference]({{< ref "/docs/ReferenceGuides/Kotlin/GeneratedCodeGuide" >}}) for more.
    - `--kotlin_out` 在 `DST_DIR` 中生成额外的 Kotlin 代码。参见 [Kotlin 生成代码参考]({{< ref "/docs/ReferenceGuides/Kotlin/GeneratedCodeGuide" >}})。

  - `--python_out` generates Python code in `DST_DIR`. See the [Python generated code reference]({{< ref "/docs/ReferenceGuides/Python/GeneratedCodeGuide" >}}) for more.
    - `--python_out` 在 `DST_DIR` 中生成 Python 代码。参见 [Python 生成代码参考]({{< ref "/docs/ReferenceGuides/Python/GeneratedCodeGuide" >}})。

  - `--go_out` generates Go code in `DST_DIR`. See the [Go generated code reference]({{< ref "/docs/ReferenceGuides/Go/GeneratedCodeGuide" >}}) for more.
    - `--go_out` 在 `DST_DIR` 中生成 Go 代码。参见 [Go 生成代码参考]({{< ref "/docs/ReferenceGuides/Go/GeneratedCodeGuide" >}})。

  - `--ruby_out` generates Ruby code in `DST_DIR`. See the [Ruby generated code reference]({{< ref "/docs/ReferenceGuides/Ruby/GeneratedCodeGuide" >}}) for more.
    - `--ruby_out` 在 `DST_DIR` 中生成 Ruby 代码。参见 [Ruby 生成代码参考]({{< ref "/docs/ReferenceGuides/Ruby/GeneratedCodeGuide" >}})。

  - `--objc_out` generates Objective-C code in `DST_DIR`. See the [Objective-C generated code reference]({{< ref "/docs/ReferenceGuides/Objective-C/GeneratedCodeGuide" >}}) for more.
    - `--objc_out` 在 `DST_DIR` 中生成 Objective-C 代码。参见 [Objective-C 生成代码参考]({{< ref "/docs/ReferenceGuides/Objective-C/GeneratedCodeGuide" >}})。

  - `--csharp_out` generates C# code in `DST_DIR`. See the [C# generated code reference]({{< ref "/docs/ReferenceGuides/CSharp/GeneratedCodeGuide" >}}) for more.
    - `--csharp_out` 在 `DST_DIR` 中生成 C# 代码。参见 [C# 生成代码参考]({{< ref "/docs/ReferenceGuides/CSharp/GeneratedCodeGuide" >}})。

  - `--php_out` generates PHP code in `DST_DIR`. See the [PHP generated code reference]({{< ref "/docs/ReferenceGuides/PHP/GeneratedCodeGuide" >}}) for more.
    - `--php_out` 在 `DST_DIR` 中生成 PHP 代码。参见 [PHP 生成代码参考]({{< ref "/docs/ReferenceGuides/PHP/GeneratedCodeGuide" >}})。


  As an extra convenience, if the `DST_DIR` ends in `.zip` or `.jar`, the compiler will write the output to a single ZIP-format archive file with the given name. `.jar` outputs will also be given a manifest file as required by the Java JAR specification. Note that if the output archive already exists, it will be overwritten.

  ​	为了方便，如果 `DST_DIR` 以 `.zip` 或 `.jar` 结尾，编译器会将输出写入具有给定名称的单个 ZIP 格式存档文件中。`.jar` 输出还将包含 Java JAR 规范所需的清单文件。请注意，如果输出存档已存在，它将被覆盖。

- You must provide one or more `.proto` files as input. Multiple `.proto` files can be specified at once. Although the files are named relative to the current directory, each file must reside in one of the `IMPORT_PATH`s so that the compiler can determine its canonical name.

  - 您必须提供一个或多个 `.proto` 文件作为输入。可以一次指定多个 `.proto` 文件。尽管文件是相对于当前目录命名的，但每个文件必须位于某个 `IMPORT_PATH` 中，以便编译器确定其规范名称。


### 文件位置 File location

Prefer not to put `.proto` files in the same directory as other language sources. Consider creating a subpackage `proto` for `.proto` files, under the root package for your project.

​	尽量不要将 `.proto` 文件与其他语言源代码放在同一目录下。建议在项目的根包下为 `.proto` 文件创建一个子包 `proto`。

#### 文件位置应与语言无关 Location Should be Language-agnostic

When working with Java code, it’s handy to put related `.proto` files in the same directory as the Java source. However, if any non-Java code ever uses the same protos, the path prefix will no longer make sense. So in general, put the protos in a related language-agnostic directory such as `//myteam/mypackage`.

​	对于 Java 代码，将相关的 `.proto` 文件与 Java 源代码放在同一目录下是很方便的。然而，如果有任何非 Java 代码使用相同的 `.proto` 文件，这样的路径前缀将不再适用。因此，一般情况下，应将 `.proto` 文件放在与语言无关的相关目录中，例如 `//myteam/mypackage`。

The exception to this rule is when it’s clear that the protos will be used only in a Java context, such as for testing.

​	但如果明确 `.proto` 文件仅在 Java 环境中使用，例如测试场景，则可以例外。

### 支持的平台 Supported Platforms

For information about:

- the operating systems, compilers, build systems, and C++ versions that are supported, see [Foundational C++ Support Policy](https://opensource.google/documentation/policies/cplusplus-support).
  - 支持的操作系统、编译器、构建系统和 C++ 版本，请参见 [C++ 基础支持政策](https://opensource.google/documentation/policies/cplusplus-support)。
- the PHP versions that are supported, see [Supported PHP versions](https://cloud.google.com/php/getting-started/supported-php-versions).
  - 支持的 PHP 版本，请参见 [支持的 PHP 版本](https://cloud.google.com/php/getting-started/supported-php-versions)。
