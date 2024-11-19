+++
title = "语言指南（版本）"
date = 2024-11-17T09:35:36+08:00
weight = 1
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

> 原文：[https://protobuf.dev/programming-guides/editions/](https://protobuf.dev/programming-guides/editions/)
>
> 收录该文档的时间：`2024-11-17T09:35:36+08:00`

# Language Guide (editions) - 语言指南（版本）

Covers how to use the edition 2023 revision of the Protocol Buffers language in your project.

​	介绍如何在项目中使用 Protocol Buffers 语言的 **2023 版本**。

This guide describes how to use the protocol buffer language to structure your protocol buffer data, including `.proto` file syntax and how to generate data access classes from your `.proto` files. It covers **edition 2023** of the protocol buffers language. For information about how editions differ from proto2 and proto3 conceptually, see [Protobuf Editions Overview]({{< ref "/docs/ProtobufEditions/Overview" >}}).

​	本指南描述了如何使用 Protocol Buffers 语言来构建你的协议数据结构，包括 `.proto` 文件的语法以及如何从 `.proto` 文件生成数据访问类。内容基于 **2023 版本** 的 Protocol Buffers 语言规范。如果想了解版本与 proto2 和 proto3 在概念上的区别，请参阅 [Protobuf Editions概述]({{< ref "/docs/ProtobufEditions/Overview" >}})。

For information on the **proto2** syntax, see the [Proto2 Language Guide]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2" >}}).

​	**Proto2** 语法详见 [Proto2 语言指南]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2" >}})。

For information on **proto3** syntax, see the [Proto3 Language Guide]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto3" >}}).

​	**Proto3** 语法详见 [Proto3 语言指南]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto3" >}})。

This is a reference guide – for a step by step example that uses many of the features described in this document, see the [tutorial]({{< ref "/docs/Tutorials" >}}) for your chosen language.

​	这是一个参考指南。如果你需要一个详细的分步示例，请参阅针对所选语言的 [教程]({{< ref "/docs/Tutorials" >}})。

## 定义消息类型 Defining A Message Type

First let’s look at a very simple example. Let’s say you want to define a search request message format, where each search request has a query string, the particular page of results you are interested in, and a number of results per page. Here’s the `.proto` file you use to define the message type.

​	首先，让我们看一个简单的例子。假设你需要定义一个搜索请求的消息格式，包括查询字符串、结果页码以及每页返回的结果数量。下面是用来定义该消息类型的 `.proto` 文件：

```proto
edition = "2023";

message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 results_per_page = 3;
}
```

- The first line of the file specifies that you’re using edition 2023 of the protobuf language spec. 文件的第一行指定你使用的是 Protocol Buffers 语言规范的 **2023 版本**。
  - The `edition` (or `syntax` for proto2/proto3) must be the first non-empty, non-comment line of the file.
    - `edition`（或 proto2/proto3 中的 `syntax`）必须是文件中首个非空且非注释的行。
  - If no `edition` or `syntax` is specified, the protocol buffer compiler will assume you are using [proto2]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2" >}}).
    - 如果没有指定 `edition` 或 `syntax`，Protocol Buffers 编译器将假定你使用的是 [proto2]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2" >}})。
- The `SearchRequest` message definition specifies three fields (name/value pairs), one for each piece of data that you want to include in this type of message. Each field has a name and a type.
  - `SearchRequest` 消息定义了三个字段，每个字段对应一个数据部分。每个字段都有一个名称和类型。


### 指定字段类型 Specifying Field Types

In the earlier example, all the fields are [scalar types](https://protobuf.dev/programming-guides/editions/#scalar): two integers (`page_number` and `results_per_page`) and a string (`query`). You can also specify [enumerations](https://protobuf.dev/programming-guides/editions/#enum) and composite types like other message types for your field.

​	上述示例中的所有字段均为 [标量类型](https://protobuf.dev/programming-guides/editions/#scalar)，包括两个整数（`page_number` 和 `results_per_page`）以及一个字符串（`query`）。你还可以使用枚举类型或复合类型（如其他消息类型）定义字段。

### 分配字段编号 Assigning Field Numbers

You must give each field in your message definition a number between `1` and `536,870,911` with the following restrictions:

​	在消息定义中，必须为每个字段指定一个编号，范围在 `1` 到 `536,870,911` 之间，并遵循以下限制：

- The given number **must be unique** among all fields for that message.
  - 给定的编号在消息的所有字段中**必须唯一**。

- Field numbers `19,000` to `19,999` are reserved for the Protocol Buffers implementation. The protocol buffer compiler will complain if you use one of these reserved field numbers in your message.
  - 字段编号 `19,000` 到 `19,999` 是为 Protocol Buffers 实现保留的。如果在消息中使用这些保留编号，Protocol Buffers 编译器会报错。

- You cannot use any previously [reserved](https://protobuf.dev/programming-guides/editions/#fieldreserved) field numbers or any field numbers that have been allocated to [extensions](https://protobuf.dev/programming-guides/editions/#extensions).
  - 不能使用任何以前已被[保留](https://protobuf.dev/programming-guides/editions/#fieldreserved)的字段编号或已分配给[扩展](https://protobuf.dev/programming-guides/editions/#extensions)的字段编号。


This number **cannot be changed once your message type is in use** because it identifies the field in the [message wire format]({{< ref "/docs/ProgrammingGuides/Encoding" >}}). “Changing” a field number is equivalent to deleting that field and creating a new field with the same type but a new number. See [Deleting Fields](https://protobuf.dev/programming-guides/editions/#deleting) for how to do this properly.

​	一旦消息类型开始使用，此编号**不能更改**，因为它标识了[消息的线格式]({{< ref "/docs/ProgrammingGuides/Encoding" >}})。更改字段编号等同于删除该字段并创建一个具有相同类型但新编号的字段。有关正确执行此操作的方法，请参阅[删除字段](https://protobuf.dev/programming-guides/editions/#deleting)。

Field numbers **should never be reused**. Never take a field number out of the [reserved](https://protobuf.dev/programming-guides/editions/#fieldreserved) list for reuse with a new field definition. See [Consequences of Reusing Field Numbers](https://protobuf.dev/programming-guides/editions/#consequences).

​	字段编号**不应重复使用**。不要从[保留](https://protobuf.dev/programming-guides/editions/#fieldreserved)列表中取出字段编号来为新字段定义重新使用。详情见[字段编号重复使用的后果](https://protobuf.dev/programming-guides/editions/#consequences)。

You should use the field numbers 1 through 15 for the most-frequently-set fields. Lower field number values take less space in the wire format. For example, field numbers in the range 1 through 15 take one byte to encode. Field numbers in the range 16 through 2047 take two bytes. You can find out more about this in [Protocol Buffer Encoding]({{< ref "/docs/ProgrammingGuides/Encoding#structure" >}}).

​	建议将编号 `1` 到 `15` 分配给最常设置的字段。较低的字段编号在线格式中占用更少的空间。例如，编号在 `1` 到 `15` 范围内的字段只需一个字节进行编码，而编号在 `16` 到 `2047` 范围内的字段则需两个字节编码。有关更多信息，请参阅[Protocol Buffer 编码]({{< ref "/docs/ProgrammingGuides/Encoding#structure" >}})。

#### 字段编号重复使用的后果 Consequences of Reusing Field Numbers

Reusing a field number makes decoding wire-format messages ambiguous.

​	重复使用字段编号会导致解析线格式消息时产生歧义。

The protobuf wire format is lean and doesn’t provide a way to detect fields encoded using one definition and decoded using another.

​	Protocol Buffers 的线格式设计非常精简，无法检测到用一种定义编码字段后用另一种定义解码字段的情况。

Encoding a field using one definition and then decoding that same field with a different definition can lead to:

​	用一种定义编码字段，再用另一种定义解码字段可能导致以下后果：

- Developer time lost to debugging
  - 开发人员因调试问题浪费时间

- A parse/merge error (best case scenario)
  - 解析/合并错误（最好的情况）

- Leaked PII/SPII
  - 泄露 PII/SPII（个人可识别信息/敏感个人信息）

- Data corruption
  - 数据损坏


Common causes of field number reuse:

​	字段编号重复使用的常见原因：

- renumbering fields (sometimes done to achieve a more aesthetically pleasing number order for fields). Renumbering effectively deletes and re-adds all the fields involved in the renumbering, resulting in incompatible wire-format changes.
  - **重新编号字段**（有时为了使字段编号更整齐美观）。重新编号实际上会删除并重新添加涉及的所有字段，从而导致线格式不兼容。

- deleting a field and not [reserving](https://protobuf.dev/programming-guides/editions/#fieldreserved) the number to prevent future reuse. **删除字段但未[保留](https://protobuf.dev/programming-guides/editions/#fieldreserved)编号以防止将来重复使用**。
  - This has been a very easy mistake to make with [extension fields](https://protobuf.dev/programming-guides/editions/#extensions) for several reasons. [Extension Declarations]({{< ref "/docs/ProgrammingGuides/ExtensionDeclarations" >}}) provide a mechanism for reserving extension fields.
    - 由于各种原因，这在[扩展字段](https://protobuf.dev/programming-guides/editions/#extensions)中是一个常见错误。[扩展声明]({{< ref "/docs/ProgrammingGuides/ExtensionDeclarations" >}})提供了一种机制来保留扩展字段。

The field number is limited to 29 bits rather than 32 bits because three bits are used to specify the field’s wire format. For more on this, see the [Encoding topic]({{< ref "/docs/ProgrammingGuides/Encoding#structure" >}}).

​	字段编号限制为 29 位而非 32 位，因为其中三位用于指定字段的线格式。有关更多信息，请参阅[编码主题]({{< ref "/docs/ProgrammingGuides/Encoding#structure" >}})。

### 指定字段基数 Specifying Field Cardinality

Message fields can be one of the following:

​	消息字段可以是以下类型之一：

- *Singular*:

  A singular field has no explicit cardinality label. It has two possible states:

  ​	单一字段没有显式的基数标签，它可能有以下两种状态：

  - the field is set, and contains a value that was explicitly set or parsed from the wire. It will be serialized to the wire.
    - 字段已设置，包含明确设置或从线格式解析的值，并会序列化到线格式。

  - the field is unset, and will return the default value. It will not be serialized to the wire.
    - 字段未设置，将返回默认值，并不会序列化到线格式。


  You can check to see if the value was explicitly set.

  ​	您可以检查字段值是否已明确设置。

  Proto3 *implicit* fields that have been migrated to editions will use the `field_presence` feature set to the `IMPLICIT` value.

  ​	Proto3 **隐式**字段在迁移到版本后将使用 `field_presence` 特性，并设置为 `IMPLICIT` 值。

  Proto2 `required` fields that have been migrated to editions will also use the `field_presence` feature, but set to `LEGACY_REQUIRED`.

  ​	Proto2 的 `required` 字段在迁移到版本后也将使用 `field_presence` 特性，但设置为 `LEGACY_REQUIRED`。

- `repeated`: this field type can be repeated zero or more times in a well-formed message. The order of the repeated values will be preserved.

  - **`repeated`**: 这种字段类型可以在格式良好的消息中重复出现零次或多次。重复值的顺序会被保留。

- `map`: this is a paired key/value field type. See [Maps]({{< ref "/docs/ProgrammingGuides/Encoding#maps" >}}) for more on this field type.

  - **`map`**: 这是一个键值对类型字段。更多信息请参阅 [Maps]({{< ref "/docs/ProgrammingGuides/Encoding#maps" >}})。


#### 重复字段默认使用压缩编码 Repeated Fields are Packed by Default

In proto editions, `repeated` fields of scalar numeric types use `packed` encoding by default.

​	在 Protobuf Editions中，数值类型的 `repeated` 字段默认使用 `packed` 编码。

You can find out more about `packed` encoding in [Protocol Buffer Encoding]({{< ref "/docs/ProgrammingGuides/Encoding#packed" >}}).

​	有关 `packed` 编码的更多信息，请参阅 [Protocol Buffer Encoding]({{< ref "/docs/ProgrammingGuides/Encoding#packed" >}})。

#### 格式良好的消息 Well-formed Messages

The term “well-formed,” when applied to protobuf messages, refers to the bytes serialized/deserialized. The protoc parser validates that a given proto definition file is parseable.

​	“格式良好”是指序列化或反序列化的 Protobuf 消息字节。Protoc 解析器验证给定的 proto 定义文件是否可解析。

Singular fields can appear more than once in wire-format bytes. The parser will accept the input, but only the last instance of that field will be accessible through the generated bindings. See [Last One Wins]({{< ref "/docs/ProgrammingGuides/Encoding#last-one-wins" >}}) for more on this topic.

​	单一字段可以在线格式字节中出现多次。解析器会接受输入，但只有该字段的最后一个实例可以通过生成的绑定访问。更多信息请参阅 [最后一个有效]({{< ref "/docs/ProgrammingGuides/Encoding#last-one-wins" >}})。

### 添加更多消息类型 Adding More Message Types

Multiple message types can be defined in a single `.proto` file. This is useful if you are defining multiple related messages – so, for example, if you wanted to define the reply message format that corresponds to your `SearchResponse` message type, you could add it to the same `.proto`:

​	可以在单个 `.proto` 文件中定义多个消息类型。这在定义多个相关消息时非常有用。例如，如果您想定义与 `SearchRequest` 消息类型对应的回复消息格式，可以将其添加到同一 `.proto` 文件中：

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

​	**组合消息可能导致膨胀**：虽然可以在单个 `.proto` 文件中定义多个消息类型（例如 message、enum 和 service），但如果在一个文件中定义了大量依赖各异的消息类型，可能会导致依赖项膨胀。建议每个 `.proto` 文件尽可能少地包含消息类型。

### 添加注释 Adding Comments

To add comments to your `.proto` files:

​	在 `.proto` 文件中添加注释的方法：

- Prefer C/C++/Java line-end-style comments ‘//’ on the line before the .proto code element
  - 优先使用 C/C++/Java 风格的行尾注释 `//`，注释位于 `.proto` 代码元素的前一行。

- C-style inline/multi-line comments `/* ... */` are also accepted.
  - 也可以使用 C 风格的内联/多行注释 `/* ... */`。
  - When using multi-line comments, a margin line of ‘*’ is preferred.
    - 使用多行注释时，建议在每行前添加 `*` 边距线。


```proto
/**
 * SearchRequest represents a search query, with pagination options to
 * indicate which results to include in the response.
 */
 /**
 * SearchRequest 表示一个搜索查询，包含分页选项，
 * 用于指示响应中包含哪些结果。
 */
message SearchRequest {
  string query = 1;

  // Which page number do we want?
  // 想要获取的页码
  int32 page_number = 2;

  // Number of results to return per page.
  // 每页返回的结果数
  int32 results_per_page = 3;
}
```

### 删除字段 Deleting Fields

Deleting fields can cause serious problems if not done properly.

​	删除字段如果操作不当，可能会导致严重问题。

When you no longer need a field and all references have been deleted from client code, you may delete the field definition from the message. However, you **must** [reserve the deleted field number](https://protobuf.dev/programming-guides/editions/#fieldreserved). If you do not reserve the field number, it is possible for a developer to reuse that number in the future.

​	当您不再需要某字段，并且已从客户端代码中删除所有引用时，可以从消息中删除字段定义。但是，您**必须**[保留已删除的字段编号](https://protobuf.dev/programming-guides/editions/#fieldreserved)。如果不保留字段编号，将来开发者可能会重复使用该编号。

You should also reserve the field name to allow JSON and TextFormat encodings of your message to continue to parse.

​	同时，您应保留字段名称，以确保 JSON 和 TextFormat 编码的消息仍然可以解析。

#### 保留字段编号 Reserved Field Numbers

If you [update](https://protobuf.dev/programming-guides/editions/#updating) a message type by entirely deleting a field, or commenting it out, future developers can reuse the field number when making their own updates to the type. This can cause severe issues, as described in [Consequences of Reusing Field Numbers](https://protobuf.dev/programming-guides/editions/#consequences). To make sure this doesn’t happen, add your deleted field number to the `reserved` list.

​	如果通过完全删除或注释掉字段来[更新](https://protobuf.dev/programming-guides/editions/#updating)消息类型，未来开发者可能会在更新消息类型时重复使用字段编号。这可能会导致严重问题。为防止这种情况，请将已删除的字段编号添加到 `reserved` 列表中。

The protoc compiler will generate error messages if any future developers try to use these reserved field numbers.

​	Protoc 编译器会在未来开发者尝试使用这些保留字段编号时生成错误消息。

```proto
message Foo {
  reserved 2, 15, 9 to 11;
}
```

Reserved field number ranges are inclusive (`9 to 11` is the same as `9, 10, 11`).

​	字段编号范围是包含性的（例如 `9 to 11` 等同于 `9, 10, 11`）。

#### 保留字段名称 Reserved Field Names

Reusing an old field name later is generally safe, except when using TextProto or JSON encodings where the field name is serialized. To avoid this risk, you can add the deleted field name to the `reserved` list.

​	重新使用旧字段名称通常是安全的，但在使用 TextProto 或 JSON 编码时可能会出现风险，因为这些编码会序列化字段名称。为了避免风险，可以将已删除的字段名称添加到 `reserved` 列表中。

Reserved names affect only the protoc compiler behavior and not runtime behavior, with one exception: TextProto implementations may discard unknown fields (without raising an error like with other unknown fields) with reserved names at parse time (only the C++ and Go implementations do so today). Runtime JSON parsing is not affected by reserved names.

​	保留名称仅影响 Protoc 编译器行为，不影响运行时行为，但有一个例外：在解析时，TextProto 实现可能会丢弃具有保留名称的未知字段，而不会报告错误（目前仅 C++ 和 Go 实现有此行为）。运行时 JSON 解析不受保留名称的影响。

```proto
message Foo {
  reserved 2, 15, 9 to 11;
  reserved foo, bar;
}
```

Note that you can’t mix field names and field numbers in the same `reserved` statement.

​	请注意，不能在同一个 `reserved` 声明中混用字段名称和字段编号。

### 从 `.proto` 文件生成的内容是什么？ What’s Generated from Your `.proto`?

When you run the [protocol buffer compiler](https://protobuf.dev/programming-guides/editions/#generating) on a `.proto`, the compiler generates the code in your chosen language you’ll need to work with the message types you’ve described in the file, including getting and setting field values, serializing your messages to an output stream, and parsing your messages from an input stream.

​	当您运行 [protocol buffer 编译器](https://protobuf.dev/programming-guides/editions/#generating) 对 `.proto` 文件进行编译时，编译器会生成您所选编程语言的代码，这些代码包括使用 `.proto` 文件中定义的消息类型所需的功能，例如设置和获取字段值、将消息序列化到输出流以及从输入流解析消息。

- For **C++**, the compiler generates a `.h` and `.cc` file from each `.proto`, with a class for each message type described in your file.
  - 对于 **C++**，编译器为每个 `.proto` 文件生成一个 `.h` 和 `.cc` 文件，其中包含每种消息类型的类。

- For **Java**, the compiler generates a `.java` file with a class for each message type, as well as a special `Builder` class for creating message class instances.
  - 对于 **Java**，编译器为每种消息类型生成一个 `.java` 文件，并生成一个特殊的 `Builder` 类用于创建消息实例。

- For **Kotlin**, in addition to the Java generated code, the compiler generates a `.kt` file for each message type with an improved Kotlin API. This includes a DSL that simplifies creating message instances, a nullable field accessor, and a copy function.
  - 对于 **Kotlin**，除了生成 Java 代码外，编译器还会为每种消息类型生成一个 `.kt` 文件，其中包含改进的 Kotlin API，包括简化创建消息实例的 DSL、可为空的字段访问器以及复制功能。

- **Python** is a little different — the Python compiler generates a module with a static descriptor of each message type in your `.proto`, which is then used with a *metaclass* to create the necessary Python data access class at runtime.
  - **Python** 的处理方式略有不同——Python 编译器生成一个包含每种消息类型静态描述符的模块，并使用 *元类* 在运行时创建所需的 Python 数据访问类。

- For **Go**, the compiler generates a `.pb.go` file with a type for each message type in your file.
  - 对于 **Go**，编译器为每个 `.proto` 文件生成一个 `.pb.go` 文件，其中包含每种消息类型的类型定义。

- For **Ruby**, the compiler generates a `.rb` file with a Ruby module containing your message types.
  - 对于 **Ruby**，编译器生成一个 `.rb` 文件，其中包含一个包含消息类型的 Ruby 模块。

- For **Objective-C**, the compiler generates a `pbobjc.h` and `pbobjc.m` file from each `.proto`, with a class for each message type described in your file.
  - 对于 **Objective-C**，编译器为每个 `.proto` 文件生成一个 `pbobjc.h` 和 `pbobjc.m` 文件，其中包含每种消息类型的类。

- For **C#**, the compiler generates a `.cs` file from each `.proto`, with a class for each message type described in your file.
  - 对于 **C#**，编译器为每个 `.proto` 文件生成一个 `.cs` 文件，其中包含每种消息类型的类。

- For **PHP**, the compiler generates a `.php` message file for each message type described in your file, and a `.php` metadata file for each `.proto` file you compile. The metadata file is used to load the valid message types into the descriptor pool.
  - 对于 **PHP**，编译器为每种消息类型生成一个 `.php` 文件，并为每个 `.proto` 文件生成一个 `.php` 元数据文件。元数据文件用于将有效的消息类型加载到描述符池中。

- For **Dart**, the compiler generates a `.pb.dart` file with a class for each message type in your file.
  - 对于 **Dart**，编译器为每种消息类型生成一个 `.pb.dart` 文件，其中包含消息类型的类。


You can find out more about using the APIs for each language by following the tutorial for your chosen language. For even more API details, see the relevant [API reference]({{< ref "/docs/ReferenceGuides" >}}).

​	有关每种语言的 API 使用详情，请参阅对应语言的教程。更多 API 细节，请查看相关的 [API 参考]({{< ref "/docs/ReferenceGuides" >}})。

## 标量值类型 Scalar Value Types

A scalar message field can have one of the following types – the table shows the type specified in the `.proto` file, and the corresponding type in the automatically generated class:

​	标量消息字段可以具有以下类型之一——表格显示了 `.proto` 文件中指定的类型以及自动生成类中的对应类型：

| .proto Type | Notes                                                        | C++ Type | Java/Kotlin Type[1] | Python Type[3]                  | Go Type | Ruby Type                      | C# Type    | PHP Type          | Dart Type | Rust Type   |
| ----------- | ------------------------------------------------------------ | -------- | ------------------- | ------------------------------- | ------- | ------------------------------ | ---------- | ----------------- | --------- | ----------- |
| double      |                                                              | double   | double              | float                           | float64 | Float                          | double     | float             | double    | f64         |
| float       |                                                              | float    | float               | float                           | float32 | Float                          | float      | float             | double    | f32         |
| int32       | Uses variable-length encoding. Inefficient for encoding negative numbers – if your field is likely to have negative values, use sint32 instead. 使用可变长度编码。对于负数，效率低下。如果字段可能包含负数，建议使用 `sint32`。 | int32    | int                 | int                             | int32   | Fixnum or Bignum (as required) | int        | integer           | int       | i32         |
| int64       | Uses variable-length encoding. Inefficient for encoding negative numbers – if your field is likely to have negative values, use sint64 instead. 使用可变长度编码。对于负数，效率低下。如果字段可能包含负数，建议使用 `sint64`。 | int64    | long                | int/long[4]                     | int64   | Bignum                         | long       | integer/string[6] | Int64     | i64         |
| uint32      | Uses variable-length encoding. 使用可变长度编码。            | uint32   | int[2]              | int/long[4]                     | uint32  | Fixnum or Bignum (as required) | uint       | integer           | int       | u32         |
| uint64      | Uses variable-length encoding. 使用可变长度编码。            | uint64   | long[2]             | int/long[4]                     | uint64  | Bignum                         | ulong      | integer/string[6] | Int64     | u64         |
| sint32      | Uses variable-length encoding. Signed int value. These more efficiently encode negative numbers than regular int32s. 使用可变长度编码。有符号整数值。比常规 `int32` 更高效地编码负数。 | int32    | int                 | int                             | int32   | Fixnum or Bignum (as required) | int        | integer           | int       | i32         |
| sint64      | Uses variable-length encoding. Signed int value. These more efficiently encode negative numbers than regular int64s. 使用可变长度编码。有符号整数值。比常规 `int64` 更高效地编码负数。 | int64    | long                | int/long[4]                     | int64   | Bignum                         | long       | integer/string[6] | Int64     | i64         |
| fixed32     | Always four bytes. More efficient than uint32 if values are often greater than 228. 总是占用四个字节。如果值通常大于 `2^28`，比 `uint32` 更高效。 | uint32   | int[2]              | int/long[4]                     | uint32  | Fixnum or Bignum (as required) | uint       | integer           | int       | u32         |
| fixed64     | Always eight bytes. More efficient than uint64 if values are often greater than 256. 总是占用八个字节。如果值通常大于 `2^56`，比 `uint64` 更高效。 | uint64   | long[2]             | int/long[4]                     | uint64  | Bignum                         | ulong      | integer/string[6] | Int64     | u64         |
| sfixed32    | Always four bytes. 总是占用四个字节。                        | int32    | int                 | int                             | int32   | Fixnum or Bignum (as required) | int        | integer           | int       | i32         |
| sfixed64    | Always eight bytes. 总是占用八个字节。                       | int64    | long                | int/long[4]                     | int64   | Bignum                         | long       | integer/string[6] | Int64     | i64         |
| bool        |                                                              | bool     | boolean             | bool                            | bool    | TrueClass/FalseClass           | bool       | boolean           | bool      | bool        |
| string      | A string must always contain UTF-8 encoded or 7-bit ASCII text, and cannot be longer than 232. 必须始终包含 UTF-8 编码或 7 位 ASCII 文本，且不能超过 `2^32` 长度。 | string   | String              | str/unicode[5]                  | string  | String (UTF-8)                 | string     | string            | String    | ProtoString |
| bytes       | May contain any arbitrary sequence of bytes no longer than 232. 可包含任意字节序列，长度不超过 `2^32`。 | string   | ByteString          | str (Python 2) bytes (Python 3) | []byte  | String (ASCII-8BIT)            | ByteString | string            | List      | ProtoBytes  |

[1] Kotlin uses the corresponding types from Java, even for unsigned types, to ensure compatibility in mixed Java/Kotlin codebases.

​	[1] Kotlin 使用与 Java 对应的类型，即使对于无符号类型也是如此，以确保在混合 Java/Kotlin 代码库中的兼容性。

[2] In Java, unsigned 32-bit and 64-bit integers are represented using their signed counterparts, with the top bit simply being stored in the sign bit.

​	[2] 在 Java 中，无符号 32 位和 64 位整数使用其有符号对应类型表示，最高位简单地存储在符号位中。

[3] In all cases, setting values to a field will perform type checking to make sure it is valid.

​	[3] 在所有情况下，为字段设置值时都会执行类型检查以确保其有效。

[4] 64-bit or unsigned 32-bit integers are always represented as long when decoded, but can be an int if an int is given when setting the field. In all cases, the value must fit in the type represented when set. See [2].

​	[4] 64 位或无符号 32 位整数在解码时总是表示为 long 类型，但如果设置字段时提供了 int 类型，它也可以是 int 类型。在所有情况下，设置的值必须适合表示的类型。参见 [2]。

[5] Python strings are represented as unicode on decode but can be str if an ASCII string is given (this is subject to change).

​	[5] Python 字符串在解码时表示为 unicode，但如果给定 ASCII 字符串则可以是 str 类型（此行为可能会改变）。

[6] Integer is used on 64-bit machines and string is used on 32-bit machines.

​	[6] 在 64 位机器上使用整数类型，在 32 位机器上使用字符串类型。

You can find out more about how these types are encoded when you serialize your message in [Protocol Buffer Encoding]({{< ref "/docs/ProgrammingGuides/Encoding" >}}).

​	可以在 [Protocol Buffer Encoding]({{< ref "/docs/ProgrammingGuides/Encoding" >}}) 中了解更多关于这些类型在序列化消息时的编码方式。

## 默认字段值 Default Field Values

When a message is parsed, if the encoded message bytes do not contain a particular field, accessing that field in the parsed object returns the default value for that field. The default values are type-specific:

​	当解析消息时，如果编码的消息字节中不包含某个字段，则在解析对象中访问该字段将返回该字段的默认值。这些默认值是特定于类型的：

- For strings, the default value is the empty string.
  - 对于字符串，默认值是空字符串。

- For bytes, the default value is empty bytes.
  - 对于字节，默认值是空字节。

- For bools, the default value is false.
  - 对于布尔值，默认值是 `false`。

- For numeric types, the default value is zero.
  - 对于数值类型，默认值是 `0`。

- For message fields, the field is not set. Its exact value is language-dependent. See the [generated code guide]({{< ref "/docs/ReferenceGuides" >}}) for details.
  - 对于消息字段，字段未设置。其具体值依赖于语言。详情请参阅 [generated code guide]({{< ref "/docs/ReferenceGuides" >}})。

- For enums, the default value is the **first defined enum value**, which must be 0. See [Enum Default Value](https://protobuf.dev/programming-guides/editions/#enum-default).
  - 对于枚举，默认值是第一个定义的枚举值，其值必须为 `0`。参见 [Enum Default Value](https://protobuf.dev/programming-guides/editions/#enum-default)。


The default value for repeated fields is empty (generally an empty list in the appropriate language).

​	重复字段的默认值是空的（通常是对应语言中的空列表）。

The default value for map fields is empty (generally an empty map in the appropriate language).

​	映射字段的默认值是空的（通常是对应语言中的空映射）。

### 覆盖默认标量值 Overriding Default Scalar Values

In protobuf editions, you can specify explicit default values for singular non-message fields. For example, let’s say you want to provide a default value of 10 for the `SearchRequest.result_per_page` field:

​	在 Protobuf Editions中，可以为单一的非消息字段指定显式默认值。例如，假设要为 `SearchRequest.result_per_page` 字段提供默认值 `10`：

```proto
int32 result_per_page = 3 [default = 10];
```

If the sender does not specify `result_per_page`, the receiver will observe the following state:

​	如果发送方未指定 `result_per_page`，接收方将观察到以下状态：

- The result_per_page field is not present. That is, the `has_result_per_page()` (hazzer method) method would return `false`.
  - `result_per_page` 字段未设置。即，`has_result_per_page()` 方法将返回 `false`。

- The value of `result_per_page` (returned from the “getter”) is `10`.
  - 通过“getter”返回的 `result_per_page` 值是 `10`。


If the sender does send a value for `result_per_page` the default value of 10 is ignored and the sender’s value is returned from the “getter”.

​	如果发送方确实发送了 `result_per_page` 的值，则默认值 `10` 将被忽略，并返回发送方的值。

See the [generated code guide]({{< ref "/docs/ReferenceGuides" >}}) for your chosen language for more details about how defaults work in generated code.

​	有关默认值在生成代码中的工作方式的更多详细信息，请参阅您选择的语言的 [generated code guide]({{< ref "/docs/ReferenceGuides" >}})。

Explicit default values cannot be specified for fields that have the `field_presence` feature set to `IMPLICIT`.

​	对于 `field_presence` 功能设置为 `IMPLICIT` 的字段，无法指定显式默认值。

## 枚举类型 Enumerations

When you’re defining a message type, you might want one of its fields to only have one of a predefined list of values. For example, let’s say you want to add a `corpus` field for each `SearchRequest`, where the corpus can be `UNIVERSAL`, `WEB`, `IMAGES`, `LOCAL`, `NEWS`, `PRODUCTS` or `VIDEO`. You can do this very simply by adding an `enum` to your message definition with a constant for each possible value.

​	定义消息类型时，您可能希望其字段之一仅具有预定义值列表中的一个值。例如，假设您想为每个 `SearchRequest` 添加一个 `corpus` 字段，该字段的值可以是 `UNIVERSAL`、`WEB`、`IMAGES`、`LOCAL`、`NEWS`、`PRODUCTS` 或 `VIDEO`。可以通过为消息定义添加一个枚举（`enum`）来简单实现，每个可能的值对应一个常量。

In the following example we’ve added an `enum` called `Corpus` with all the possible values, and a field of type `Corpus`:

​	以下示例中，我们添加了一个名为 `Corpus` 的枚举，包含所有可能的值，以及一个 `Corpus` 类型的字段：

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

### 枚举默认值 Enum Default Value

The default value for the `SearchRequest.corpus` field is `CORPUS_UNSPECIFIED` because that is the first value defined in the enum.

​	`SearchRequest.corpus` 字段的默认值是 `CORPUS_UNSPECIFIED`，因为这是枚举中定义的第一个值。

In edition 2023, the first value defined in an enum definition **must** have the value zero and should have the name `ENUM_TYPE_NAME_UNSPECIFIED` or `ENUM_TYPE_NAME_UNKNOWN`. This is because:

​	在 2023 版中，枚举定义中的第一个值**必须**为 `0`，并且应命名为 `ENUM_TYPE_NAME_UNSPECIFIED` 或 `ENUM_TYPE_NAME_UNKNOWN`。这是因为：

- The zero value needs to be the first element for compatibility with [proto2]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2#enum-default" >}}) semantics, where the first enum value is the default unless a different value is explicitly specified.
  - 零值需要是第一个元素，以兼容 [proto2]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2#enum-default" >}}) 语义，其中第一个枚举值是默认值，除非明确指定了其他值。

- There must be a zero value for compatibility with [proto3]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto3#enum-default" >}}) semantics, where the zero value is used as the default value for all implicit-presence fields using this enum type.
  - 必须有一个零值，以兼容 [proto3]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto3#enum-default" >}}) 语义，其中零值用作所有隐式存在字段的默认值。


It is also recommended that this first, default value have no semantic meaning other than “this value was unspecified”.

​	也建议将枚举的第一个默认值设计为没有语义意义，仅表示“此值未指定”。

The default value for an enum field like `SearchRequest.corpus` field can be explicitly overridden like this:

​	对于类似 `SearchRequest.corpus` 字段的枚举字段，可以显式覆盖默认值，如下所示：

```fallback
  Corpus corpus = 4 [default = CORPUS_UNIVERSAL];
```

If an enum type has been migrated from proto2 using `option features.enum_type = CLOSED;` there is no restriction on the first value in the enum. It is not recommended to change the first value of these types of enums because it will change the default value for any fields using that enum type without an explicit field default.

​	如果某个枚举类型是通过 `option features.enum_type = CLOSED;` 从 proto2 迁移而来，那么枚举中第一个值不受限制。但不建议更改此类枚举的第一个值，因为这会改变使用该枚举类型的所有字段的默认值（如果未显式设置字段默认值）。

### 枚举值别名 Enum Value Aliases

You can define aliases by assigning the same value to different enum constants. To do this you need to set the `allow_alias` option to `true`. Otherwise, the protocol buffer compiler generates a warning message when aliases are found. Though all alias values are valid during deserialization, the first value is always used when serializing.

​	可以通过为不同的枚举常量赋相同的值来定义别名。为此，需要设置 `allow_alias` 选项为 `true`。否则，协议缓冲区编译器会在发现别名时生成警告消息。尽管在反序列化期间所有别名值都是有效的，但在序列化时始终使用第一个值。

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
  // ENAA_RUNNING = 1;  // Uncommenting this line will cause a warning message. 取消注释此行将导致警告消息。
  ENAA_FINISHED = 2;
}
```

Enumerator constants must be in the range of a 32-bit integer. Since `enum` values use [varint encoding]({{< ref "/docs/ProgrammingGuides/Encoding" >}}) on the wire, negative values are inefficient and thus not recommended. You can define `enum`s within a message definition, as in the earlier example, or outside – these `enum`s can be reused in any message definition in your `.proto` file. You can also use an `enum` type declared in one message as the type of a field in a different message, using the syntax `_MessageType_._EnumType_`.

​	枚举常量必须在 32 位整数范围内。由于枚举值在传输中使用 [varint 编码]({{< ref "/docs/ProgrammingGuides/Encoding" >}})，负值效率较低，因此不推荐使用负值。可以在消息定义内定义枚举（如前例），也可以在外部定义——这些枚举可以在 `.proto` 文件中的任何消息定义中重复使用。还可以将一个消息中声明的枚举类型用作另一个消息中字段的类型，使用语法 `_MessageType_._EnumType_`。

When you run the protocol buffer compiler on a `.proto` that uses an `enum`, the generated code will have a corresponding `enum` for Java, Kotlin, or C++, or a special `EnumDescriptor` class for Python that’s used to create a set of symbolic constants with integer values in the runtime-generated class.

​	在对使用枚举的 `.proto` 文件运行协议缓冲区编译器时，生成的代码将在 Java、Kotlin 或 C++ 中包含对应的 `enum`，或在 Python 中包含用于创建运行时生成类中的整数值符号常量集的特殊 `EnumDescriptor` 类。

#### Important

The generated code may be subject to language-specific limitations on the number of enumerators (low thousands for one language). Review the limitations for the languages you plan to use.

​	生成的代码可能受到语言特定的枚举常量数量限制（某些语言中限制为数千个）。请根据计划使用的语言检查其限制。

During deserialization, unrecognized enum values will be preserved in the message, though how this is represented when the message is deserialized is language-dependent. In languages that support open enum types with values outside the range of specified symbols, such as C++ and Go, the unknown enum value is simply stored as its underlying integer representation. In languages with closed enum types such as Java, a case in the enum is used to represent an unrecognized value, and the underlying integer can be accessed with special accessors. In either case, if the message is serialized the unrecognized value will still be serialized with the message.

​	在反序列化过程中，未识别的枚举值将保留在消息中，但消息反序列化时如何表示取决于语言。在支持开放枚举类型的语言（如 C++ 和 Go）中，未知的枚举值以其底层整数表示简单存储。在支持封闭枚举类型的语言（如 Java）中，将使用枚举中的一个特殊情况来表示未识别的值，可以通过特殊访问器访问其底层整数。在两种情况下，如果消息被序列化，未识别的值仍会随消息一起序列化。

> Important
>
> For information on how enums should work contrasted with how they currently work in different languages, see [Enum Behavior]({{< ref "/docs/ProgrammingGuides/EnumBehavior" >}}).
>
> ​	有关枚举在不同语言中的预期行为与当前行为的对比信息，请参阅 [Enum Behavior]({{< ref "/docs/ProgrammingGuides/EnumBehavior" >}})。

For more information about how to work with message `enum`s in your applications, see the [generated code guide]({{< ref "/docs/ReferenceGuides" >}}) for your chosen language.

​	有关如何在应用程序中处理消息 `enum` 的更多信息，请参阅您选择的语言的 [generated code guide]({{< ref "/docs/ReferenceGuides" >}})。

### 保留值 Reserved Values

If you [update](https://protobuf.dev/programming-guides/editions/#updating) an enum type by entirely removing an enum entry, or commenting it out, future users can reuse the numeric value when making their own updates to the type. This can cause severe issues if they later load old instances of the same `.proto`, including data corruption, privacy bugs, and so on. One way to make sure this doesn’t happen is to specify that the numeric values (and/or names, which can also cause issues for JSON serialization) of your deleted entries are `reserved`. The protocol buffer compiler will complain if any future users try to use these identifiers. You can specify that your reserved numeric value range goes up to the maximum possible value using the `max` keyword.

​	如果通过完全删除或注释掉某个枚举条目来 [更新](https://protobuf.dev/programming-guides/editions/#updating) 枚举类型，则未来用户可能会在更新类型时重用该数字值。这可能导致严重问题，例如在加载旧版本相同 `.proto` 的实例时出现数据损坏、隐私漏洞等。为确保不会发生这种情况，可以通过指定被删除条目的数字值（以及/或名称，对于 JSON 序列化可能造成问题的情况）为 `reserved` 来防止。协议缓冲区编译器会警告任何未来试图使用这些标识符的用户。可以使用 `max` 关键字指定保留的数字值范围延伸至最大可能值。

```proto
enum Foo {
  reserved 2, 15, 9 to 11, 40 to max;
  reserved FOO, BAR;
}
```

Note that you can’t mix field names and numeric values in the same `reserved` statement.

​	请注意，不能在同一条 `reserved` 声明中混合字段名称和数字值。

## 使用其他消息类型 Using Other Message Types

You can use other message types as field types. For example, let’s say you wanted to include `Result` messages in each `SearchResponse` message – to do this, you can define a `Result` message type in the same `.proto` and then specify a field of type `Result` in `SearchResponse`:

​	可以将其他消息类型用作字段类型。例如，假设希望在每个 `SearchResponse` 消息中包含 `Result` 消息，可以在同一个 `.proto` 文件中定义一个 `Result` 消息类型，然后在 `SearchResponse` 中指定一个 `Result` 类型的字段：

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

​	在上述示例中，`Result` 消息类型与 `SearchResponse` 定义在同一文件中——如果想要使用的字段类型的消息类型已经在另一个 `.proto` 文件中定义怎么办？

You can use definitions from other `.proto` files by *importing* them. To import another `.proto`’s definitions, you add an import statement to the top of your file:

​	可以通过 *导入* 它们来使用其他 `.proto` 文件中的定义。为导入另一个 `.proto` 的定义，在文件顶部添加一个 import 语句：

```proto
import "myproject/other_protos.proto";
```

By default, you can use definitions only from directly imported `.proto` files. However, sometimes you may need to move a `.proto` file to a new location. Instead of moving the `.proto` file directly and updating all the call sites in a single change, you can put a placeholder `.proto` file in the old location to forward all the imports to the new location using the `import public` notion.

​	默认情况下，您只能使用直接导入的 `.proto` 文件中的定义。但是，有时您可能需要将 `.proto` 文件移动到新位置。与其直接移动 `.proto` 文件并一次性更新所有调用位置，不如在旧位置放置一个占位 `.proto` 文件，通过 `import public` 概念将所有导入转发到新位置。

**Note that the public import functionality is not available in Java, Kotlin, TypeScript, JavaScript, GCL, as well as C++ targets that use protobuf static reflection.**

​	注意，公共导入功能在 Java、Kotlin、TypeScript、JavaScript、GCL，以及使用 protobuf 静态反射的 C++ 目标中不可用。

`import public` dependencies can be transitively relied upon by any code importing the proto containing the `import public` statement. For example:

​	`import public` 依赖项可以被任何导入包含 `import public` 声明的 proto 的代码传递性依赖。例如：

```proto
// new.proto
// All definitions are moved here
// 所有定义都移到了这里。
// old.proto
// This is the proto that all clients are importing.
// 这是所有客户端正在导入的 proto。
import public "new.proto";
import "other.proto";
// client.proto
import "old.proto";
// You use definitions from old.proto and new.proto, but not other.proto
// 您可以使用 old.proto 和 new.proto 的定义，但不能使用 other.proto。
```

The protocol compiler searches for imported files in a set of directories specified on the protocol compiler command line using the `-I`/`--proto_path` flag. If no flag was given, it looks in the directory in which the compiler was invoked. In general you should set the `--proto_path` flag to the root of your project and use fully qualified names for all imports.

​	协议编译器在通过 `-I`/`--proto_path` 标志指定的目录集中搜索导入的文件。如果未提供标志，它将在调用编译器的目录中查找。通常，您应该将 `--proto_path` 标志设置为项目的根目录，并对所有导入使用完全限定名称。

### 使用 proto2 和 proto3 消息类型 Using proto2 and proto3 Message Types

It’s possible to import [proto2]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2" >}}) and [proto3]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto3" >}}) message types and use them in your editions 2023 messages, and vice versa.

​	可以导入 [proto2]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2" >}}) 和 [proto3]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto3" >}}) 消息类型，并在 2023 年版本的消息中使用它们，反之亦然。

## 嵌套类型 Nested Types

You can define and use message types inside other message types, as in the following example – here the `Result` message is defined inside the `SearchResponse` message:

​	您可以在其他消息类型内部定义和使用消息类型，如以下示例所示——此处 `Result` 消息定义在 `SearchResponse` 消息中：

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

​	如果希望在父消息类型之外重用该消息类型，可以将其引用为 `_Parent_._Type_`：

```proto
message SomeOtherMessage {
  SearchResponse.Result result = 1;
}
```

You can nest messages as deeply as you like. In the example below, note that the two nested types named `Inner` are entirely independent, since they are defined within different messages:

​	您可以根据需要任意深度嵌套消息。在以下示例中，请注意两个名为 `Inner` 的嵌套类型是完全独立的，因为它们定义在不同的消息中：

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

​	如果现有的消息类型不再满足您的所有需求，例如，您希望消息格式包含额外的字段，但仍希望使用旧格式创建的代码，请放心！使用二进制线格式时，更新消息类型非常简单，不会破坏现有代码。

> Note
>
> If you use JSON or [proto text format]({{< ref "/docs/ReferenceGuides/ProtocolBuffers/TextFormatLanguageSpecification" >}}) to store your protocol buffer messages, the changes that you can make in your proto definition are different.
>
> ​	如果您使用 JSON 或 [proto 文本格式]({{< ref "/docs/ReferenceGuides/ProtocolBuffers/TextFormatLanguageSpecification" >}}) 存储协议缓冲区消息，则可以在 proto 定义中进行的更改会有所不同。

Check [Proto Best Practices]({{< ref "/docs/ProgrammingGuides/ProtoBestPractices" >}}) and the following rules:

​	请查阅 [Proto 最佳实践]({{< ref "/docs/ProgrammingGuides/ProtoBestPractices" >}}) 和以下规则：

- Don’t change the field numbers for any existing fields. “Changing” the field number is equivalent to deleting the field and adding a new field with the same type. If you want to renumber a field, see the instructions for [deleting a field](https://protobuf.dev/programming-guides/editions/#deleting).
  - 不要更改任何现有字段的字段编号。“更改”字段编号相当于删除该字段并添加具有相同类型的新字段。如果需要重新编号字段，请参阅有关[删除字段](https://protobuf.dev/programming-guides/editions/#deleting)的说明。

- If you add new fields, any messages serialized by code using your “old” message format can still be parsed by your new generated code. You should keep in mind the [default values](https://protobuf.dev/programming-guides/editions/#default) for these elements so that new code can properly interact with messages generated by old code. Similarly, messages created by your new code can be parsed by your old code: old binaries simply ignore the new field when parsing. See the [Unknown Fields](https://protobuf.dev/programming-guides/editions/#unknowns) section for details.
  - 如果添加了新字段，任何使用“旧”消息格式的代码序列化的消息仍然可以通过新生成的代码解析。请记住这些元素的[默认值](https://protobuf.dev/programming-guides/editions/#default)，以便新代码可以正确地与旧代码生成的消息交互。同样，新代码创建的消息可以通过旧代码解析：旧的二进制文件在解析时会忽略新字段。详细信息请参阅[未知字段](https://protobuf.dev/programming-guides/editions/#unknowns)部分。

- Fields can be removed, as long as the field number is not used again in your updated message type. You may want to rename the field instead, perhaps adding the prefix “OBSOLETE_”, or make the field number [reserved](https://protobuf.dev/programming-guides/editions/#fieldreserved), so that future users of your `.proto` can’t accidentally reuse the number.
  - 字段可以被移除，只要字段编号不再用于更新后的消息类型。您可能希望重命名该字段，例如添加前缀“OBSOLETE_”，或将字段编号设为[保留](https://protobuf.dev/programming-guides/editions/#fieldreserved)，以便未来的 `.proto` 用户无法意外重用该编号。

- `int32`, `uint32`, `int64`, `uint64`, and `bool` are all compatible – this means you can change a field from one of these types to another without breaking forwards- or backwards-compatibility. If a number is parsed from the wire which doesn’t fit in the corresponding type, you will get the same effect as if you had cast the number to that type in C++ (for example, if a 64-bit number is read as an int32, it will be truncated to 32 bits).
  - `int32`、`uint32`、`int64`、`uint64` 和 `bool` 彼此兼容——这意味着可以在这些类型之间切换字段，而不会破坏前向或后向兼容性。如果从线缆中解析的数字与相应类型不匹配，将得到与在 C++ 中将数字强制转换为该类型相同的效果（例如，如果 64 位数字被读取为 int32，则它将被截断为 32 位）。

- `sint32` and `sint64` are compatible with each other but are *not* compatible with the other integer types.
  - `sint32` 和 `sint64` 彼此兼容，但与其他整数类型*不兼容*。

- `string` and `bytes` are compatible as long as the bytes are valid UTF-8.
  - `string` 和 `bytes` 兼容，只要字节是有效的 UTF-8。

- Embedded messages are compatible with `bytes` if the bytes contain an encoded instance of the message.
  - 如果字节包含消息的编码实例，则嵌套消息与 `bytes` 兼容。

- `fixed32` is compatible with `sfixed32`, and `fixed64` with `sfixed64`.
  - `fixed32` 与 `sfixed32` 兼容，`fixed64` 与 `sfixed64` 兼容。

- For `string`, `bytes`, and message fields, singular is compatible with `repeated`. Given serialized data of a repeated field as input, clients that expect this field to be singular will take the last input value if it’s a primitive type field or merge all input elements if it’s a message type field. Note that this is **not** generally safe for numeric types, including bools and enums. Repeated fields of numeric types are serialized in the [packed]({{< ref "/docs/ProgrammingGuides/Encoding#packed" >}}) format by default, which will not be parsed correctly when a singular field is expected.
  - 对于 `string`、`bytes` 和消息字段，单个字段与 `repeated` 兼容。如果将重复字段的序列化数据作为输入，期望该字段是单个字段的客户端将取最后一个输入值（如果是原始类型字段），或者合并所有输入元素（如果是消息类型字段）。需要注意的是，这对包括布尔值和枚举在内的数字类型通常**不安全**。默认情况下，数字类型的重复字段以[打包格式]({{< ref "/docs/ProgrammingGuides/Encoding#packed" >}})序列化，如果期望的是单个字段，则无法正确解析。

- `enum` is compatible with `int32`, `uint32`, `int64`, and `uint64` in terms of wire format (note that values will be truncated if they don’t fit). However, be aware that client code may treat them differently when the message is deserialized: for example, unrecognized `enum` values will be preserved in the message, but how this is represented when the message is deserialized is language-dependent. Int fields always just preserve their value.
  - `enum` 与 `int32`、`uint32`、`int64` 和 `uint64` 在线缆格式上是兼容的（注意，如果值不适合，它们会被截断）。但是要注意，客户端代码在反序列化消息时可能会以不同方式处理它们：例如，未识别的 `enum` 值将在消息中保留，但消息反序列化时的表示方式取决于语言。整数字段始终仅保留其值。

- Changing a single `optional` field or extension into a member of a **new** `oneof` is binary compatible, however for some languages (notably, Go) the generated code’s API will change in incompatible ways. For this reason, Google does not make such changes in its public APIs, as documented in [AIP-180](https://google.aip.dev/180#moving-into-oneofs). With the same caveat about source-compatibility, moving multiple fields into a new `oneof` may be safe if you are sure that no code sets more than one at a time. Moving fields into an existing `oneof` is not safe. Likewise, changing a single field `oneof` to an `optional` field or extension is safe.
  - 将单个 `optional` 字段或扩展更改为 **新** `oneof` 的成员是二进制兼容的，但对于某些语言（尤其是 Go），生成的代码 API 会发生不兼容的变化。因此，Google 在其公共 API 中不进行此类更改，如 [AIP-180](https://google.aip.dev/180#moving-into-oneofs) 中所述。基于相同的源代码兼容性警告，如果您确信没有代码同时设置多个字段，将多个字段移入新的 `oneof` 可能是安全的。将字段移入现有 `oneof` 是不安全的。同样，将单个 `oneof` 字段更改为 `optional` 字段或扩展是安全的。

- Changing a field between a `map<K, V>` and the corresponding `repeated` message field is binary compatible (see [Maps](https://protobuf.dev/programming-guides/editions/#maps), below, for the message layout and other restrictions). However, the safety of the change is application-dependent: when deserializing and reserializing a message, clients using the `repeated` field definition will produce a semantically identical result; however, clients using the `map` field definition may reorder entries and drop entries with duplicate keys.
  - 在 `map<K, V>` 和相应的 `repeated` 消息字段之间更改字段是二进制兼容的（参见[Maps](https://protobuf.dev/programming-guides/editions/#maps)了解消息布局和其他限制）。但是，这种更改的安全性依赖于应用程序：在反序列化并重新序列化消息时，使用 `repeated` 字段定义的客户端将生成语义上相同的结果；然而，使用 `map` 字段定义的客户端可能会重新排序条目并删除重复键的条目。

## 未知字段 Unknown Fields

Unknown fields are well-formed protocol buffer serialized data representing fields that the parser does not recognize. For example, when an old binary parses data sent by a new binary with new fields, those new fields become unknown fields in the old binary.

​	未知字段是表示解析器无法识别的字段的良好协议缓冲区序列化数据。例如，当旧二进制文件解析新二进制文件发送的数据（包含新字段）时，这些新字段在旧二进制文件中成为未知字段。

Editions messages preserve unknown fields and includes them during parsing and in the serialized output, which matches proto2 and proto3 behavior.

​	编辑版本的消息保留未知字段，并在解析和序列化输出中包含它们，这与 proto2 和 proto3 的行为一致。

### 保留未知字段 Retaining Unknown Fields

Some actions can cause unknown fields to be lost. For example, if you do one of the following, unknown fields are lost:

​	某些操作可能会导致丢失未知字段。例如，如果执行以下操作之一，未知字段会丢失：

- Serialize a proto to JSON.
  - 将 proto 序列化为 JSON。

- Iterate over all of the fields in a message to populate a new message.
  - 遍历消息中的所有字段以填充新消息。


To avoid losing unknown fields, do the following:

​	为了避免丢失未知字段，请执行以下操作：

- Use binary; avoid using text formats for data exchange.
  - 使用二进制格式；避免在数据交换中使用文本格式。

- Use message-oriented APIs, such as `CopyFrom()` and `MergeFrom()`, to copy data rather than copying field-by-field
  - 使用面向消息的 API，例如 `CopyFrom()` 和 `MergeFrom()`，通过数据复制而不是逐字段复制。


TextFormat is a bit of a special case. Serializing to TextFormat prints unknown fields using their field numbers. But parsing TextFormat data back into a binary proto fails if there are entries that use field numbers.

​	TextFormat 是一个特殊的例子。将未知字段序列化为 TextFormat 会使用它们的字段号进行打印。但如果存在使用字段号的条目，TextFormat 数据重新解析为二进制 proto 会失败。

## 扩展 Extensions

An extension is a field defined outside of its container message; usually in a `.proto` file separate from the container message’s `.proto` file.

​	扩展是一种在容器消息之外定义字段的方式，通常是在与容器消息 `.proto` 文件分离的 `.proto` 文件中定义。

### 为什么要使用扩展？ Why Use Extensions?

There are two main reasons to use extensions:

​	使用扩展主要有两个原因：

- The container message’s `.proto` file will have fewer imports/dependencies. This can improve build times, break circular dependencies, and otherwise promote loose coupling. Extensions are very good for this.
  - 容器消息的 `.proto` 文件将具有更少的导入/依赖项。这可以提高构建速度，打破循环依赖，并促进松耦合。扩展在这方面表现很好。

- Allow systems to attach data to a container message with minimal dependency and coordination. Extensions are not a great solution for this because of the limited field number space and the [Consequences of Reusing Field Numbers](https://protobuf.dev/programming-guides/editions/#consequences). If your use case requires very low coordination for a large number of extensions, consider using the [`Any` message type]({{< ref "/docs/ReferenceGuides/ProtocolBuffers/Well-KnownTypes#any" >}}) instead.
  - 允许系统以最少的依赖性和协调性将数据附加到容器消息。扩展在这方面并不是一个理想的解决方案，因为字段号空间有限，并且会出现[字段号重用的后果](https://protobuf.dev/programming-guides/editions/#consequences)。如果您的用例需要非常低的协调性且有大量扩展，可以考虑使用 [`Any` 消息类型]({{< ref "/docs/ReferenceGuides/ProtocolBuffers/Well-KnownTypes#any" >}})。


### 扩展示例 Example Extension

Let’s look at an example extension:

​	以下是一个扩展的示例：

```proto
// file kittens/video_ext.proto

import "kittens/video.proto";
import "media/user_content.proto";

package kittens;

// This extension allows kitten videos in a media.UserContent message.
// 此扩展允许 media.UserContent 消息中包含 kitten 视频。
extend media.UserContent {
  // Video is a message imported from kittens/video.proto
  // Video 是从 kittens/video.proto 导入的消息
  repeated Video kitten_videos = 126;
}
```

Note that the file defining the extension (`kittens/video_ext.proto`) imports the container message’s file (`media/user_content.proto`).

​	请注意，定义扩展的文件（`kittens/video_ext.proto`）需要导入容器消息的文件（`media/user_content.proto`）。

The container message must reserve a subset of its field numbers for extensions.

​	容器消息必须为扩展预留一个字段号子集：

```proto
// file media/user_content.proto

package media;

// A container message to hold stuff that a user has created.
// 一个容器消息，用于保存用户创建的内容。
message UserContent {
  // Set verification to `DECLARATION` to enforce extension declarations for all
  // extensions in this range.
  // 将验证设置为 `DECLARATION`，以强制对该范围内的所有扩展进行声明。
  extensions 100 to 199 [verification = DECLARATION];
}
```

The container message’s file (`media/user_content.proto`) defines the message `UserContent`, which reserves field numbers [100 to 199] for extensions. It is recommended to set `verification = DECLARATION` for the range to require declarations for all its extensions.

​	容器消息的文件（`media/user_content.proto`）定义了消息 `UserContent`，为扩展预留了字段号范围 [100 至 199]。建议为此范围设置 `verification = DECLARATION`，以要求对所有扩展进行声明。

When the new extension (`kittens/video_ext.proto`) is added, a corresponding declaration should be added to `UserContent` and `verification` should be removed.

​	当添加新的扩展（如 `kittens/video_ext.proto`）时，应在 `UserContent` 中添加相应的声明，并移除 `verification`：

```fallback
// A container message to hold stuff that a user has created.
// 一个容器消息，用于保存用户创建的内容。
message UserContent {
  extensions 100 to 199 [
    declaration = {
      number: 126,
      full_name: ".kittens.kitten_videos",
      type: ".kittens.Video",
      repeated: true
    },
    // Ensures all field numbers in this extension range are declarations.
    // 确保该扩展范围内的所有字段号均为声明。
    verification = DECLARATION
  ];
}
```

`UserContent` declares that field number `126` will be used by a `repeated` extension field with the fully-qualified name `.kittens.kitten_videos` and the fully-qualified type `.kittens.Video`. To learn more about extension declarations see [Extension Declarations]({{< ref "/docs/ProgrammingGuides/ExtensionDeclarations" >}}).

​	`UserContent` 声明，字段号 `126` 将被一个 `repeated` 类型的扩展字段使用，其全限定名称为 `.kittens.kitten_videos`，全限定类型为 `.kittens.Video`。有关扩展声明的更多信息，请参阅[扩展声明]({{< ref "/docs/ProgrammingGuides/ExtensionDeclarations" >}})。

Note that the container message’s file (`media/user_content.proto`) **does not** import the kitten_video extension definition (`kittens/video_ext.proto`)

​	请注意，容器消息的文件（`media/user_content.proto`）**不会**导入 kitten_video 扩展的定义（`kittens/video_ext.proto`）。

There is no difference in the wire-format encoding of extension fields as compared to a standard field with the same field number, type, and cardinality. Therefore, it is safe to move a standard field out of its container to be an extension or to move an extension field into its container message as a standard field so long as the field number, type, and cardinality remain constant.

​	扩展字段的线缆格式编码与具有相同字段号、类型和基数的标准字段没有区别。因此，只要字段号、类型和基数保持不变，将标准字段移出其容器以成为扩展字段或将扩展字段移入其容器消息作为标准字段是安全的。

However, because extensions are defined outside of the container message, no specialized accessors are generated to get and set specific extension fields. For our example, the protobuf compiler **will not generate** `AddKittenVideos()` or `GetKittenVideos()` accessors. Instead, extensions are accessed through parameterized functions like: `HasExtension()`, `ClearExtension()`, `GetExtension()`, `MutableExtension()`, and `AddExtension()`.

​	然而，由于扩展是在容器消息之外定义的，因此不会生成用于获取和设置特定扩展字段的专用访问器。例如，protobuf 编译器**不会生成** `AddKittenVideos()` 或 `GetKittenVideos()` 访问器。相反，通过参数化函数访问扩展，例如：`HasExtension()`、`ClearExtension()`、`GetExtension()`、`MutableExtension()` 和 `AddExtension()`。

In C++, it would look something like:

​	在 C++ 中，这看起来如下所示：

```cpp
UserContent user_content;
user_content.AddExtension(kittens::kitten_videos, new kittens::Video());
assert(1 == user_content.GetExtensionCount(kittens::kitten_videos));
user_content.GetExtension(kittens::kitten_videos, 0);
```

### 定义扩展范围 Defining Extension Ranges

If you are the owner of a container message, you will need to define an extension range for the extensions to your message.

​	如果您是容器消息的所有者，则需要为您的消息定义扩展范围。

Field numbers allocated to extension fields cannot be reused for standard fields.

​	扩展字段分配的字段号不能重复用于标准字段。

It is safe to expand an extension range after it is defined. A good default is to allocate 1000 relatively small numbers, and densely populate that space using extension declarations:

​	扩展范围在定义后可以安全地扩展。一个良好的默认策略是分配 1000 个相对较小的字段号，并通过扩展声明密集地使用该范围：

```proto
message ModernExtendableMessage {
  // All extensions in this range should use extension declarations.
  // 该范围内的所有扩展应使用扩展声明。
  extensions 1000 to 2000 [verification = DECLARATION];
}
```

When adding a range for extension declarations before the actual extensions, you should add `verification = DECLARATION` to enforce that declarations are used for this new range. This placeholder can be removed once an actual declaration is added.

​	在实际扩展之前为扩展声明添加范围时，您应添加 `verification = DECLARATION` 来强制使用声明。一旦添加了实际的声明，可以移除此占位符。

It is safe to split an existing extension range into separate ranges that cover the same total range. This might be necessary for migrating a legacy message type to [Extension Declarations]({{< ref "/docs/ProgrammingGuides/ExtensionDeclarations" >}}). For example, before migration, the range might be defined as:

​	可以安全地将现有扩展范围拆分为覆盖相同总范围的多个范围。这可能在迁移遗留消息类型到[扩展声明]({{< ref "/docs/ProgrammingGuides/ExtensionDeclarations" >}})时必需。例如，迁移前的范围可能定义为：

```proto
message LegacyMessage {
  extensions 1000 to max;
}
```

And after migration (splitting the range) it can be:

​	迁移后（拆分范围）可以变为：

```proto
message LegacyMessage {
  // Legacy range that was using an unverified allocation scheme.
  // 使用未经验证分配方案的遗留范围。
  extensions 1000 to 524999999 [verification = UNVERIFIED];
  // Current range that uses extension declarations.
  // 当前范围使用扩展声明。
  extensions 525000000 to max  [verification = DECLARATION];
}
```

It is not safe to increase the start field number nor decrease the end field number to move or shrink an extension range. These changes can invalidate an existing extension.

​	增加范围的起始字段号或减少范围的结束字段号以移动或缩小扩展范围是不安全的。这些更改可能使现有扩展无效。

Prefer using field numbers 1 to 15 for standard fields that are populated in most instances of your proto. It is not recommended to use these numbers for extensions.

​	优先使用 1 到 15 的字段号分配给大多数实例中会填充的标准字段。不建议将这些字段号用于扩展。

If your numbering convention might involve extensions having very large field numbers, you can specify that your extension range goes up to the maximum possible field number using the `max` keyword:

​	如果您的编号方案可能涉及具有非常大字段号的扩展，可以使用 `max` 关键字指定扩展范围达到最大可能字段号：

```proto
message Foo {
  extensions 1000 to max;
}
```

`max` is 229 - 1, or 536,870,911.

​	`max` 是 2^29 - 1，即 536,870,911。

### 选择扩展字段号 Choosing Extension Numbers

Extensions are just fields that can be specified outside of their container messages. All the same rules for [Assigning Field Numbers](https://protobuf.dev/programming-guides/editions/#assigning) apply to extension field numbers. The same [Consequences of Reusing Field Numbers](https://protobuf.dev/programming-guides/editions/#consequences) also apply to reusing extension field numbers.

​	扩展字段只是可以在其容器消息之外指定的字段。所有[字段号分配规则](https://protobuf.dev/programming-guides/editions/#assigning)同样适用于扩展字段号。[重用字段号的后果](https://protobuf.dev/programming-guides/editions/#consequences)同样适用于扩展字段号的重用。

Choosing unique extension field numbers is simple if the container message uses [extension declarations]({{< ref "/docs/ProgrammingGuides/ExtensionDeclarations" >}}). When defining a new extension, choose the lowest field number above all other declarations from the highest extension range defined in the container message. For example, if a container message is defined like this:

​	如果容器消息使用[扩展声明]({{< ref "/docs/ProgrammingGuides/ExtensionDeclarations" >}})，选择唯一扩展字段号非常简单。在定义新扩展时，从容器消息定义的最高扩展范围中的所有其他声明之后，选择最低字段号。例如，如果一个容器消息定义如下：

```proto
message Container {
  // Legacy range that was using an unverified allocation scheme
  // 使用未经验证分配方案的遗留范围。
  extensions 1000 to 524999999;
  // Current range that uses extension declarations. (highest extension range)
  // 当前范围使用扩展声明（最高扩展范围）。
  extensions 525000000 to max  [
    declaration = {
      number: 525000001,
      full_name: ".bar.baz_ext",
      type: ".bar.Baz"
    }
    // 525,000,002 is the lowest field number above all other declarations
    // 525,000,002 是高于所有其他声明的最低字段号。
  ];
}
```

The next extension of `Container` should add a new declaration with the number `525000002`.

​	`Container` 的下一个扩展应添加字段号为 `525000002` 的新声明。

#### 未验证扩展字段号分配（不推荐）Unverified Extension Number Allocation (not recommended)

The owner of a container message may choose to forgo extension declarations in favor of their own unverified extension number allocation strategy.

​	容器消息的所有者可以选择放弃扩展声明，而采用自己未经验证的扩展字段号分配策略。

An unverified allocation scheme uses a mechanism external to the protobuf ecosystem to allocate extension field numbers within the selected extension range. One example could be using a monorepo’s commit number. This system is “unverified” from the protobuf compiler’s point of view since there is no way to check that an extension is using a properly acquired extension field number.

​	未经验证的分配方案使用 protobuf 生态系统外部的机制，在选定的扩展范围内分配扩展字段号。例如，可以使用 monorepo 的提交号。这种系统从 protobuf 编译器的角度来看是“未经验证的”，因为无法检查扩展是否使用了正确分配的字段号。

The benefit of an unverified system over a verified system like extension declarations is the ability to define an extension without coordinating with the container message owner.

​	未经验证系统相对于扩展声明等验证系统的优点在于，可以在无需与容器消息所有者协调的情况下定义扩展。

The downside of an unverified system is that the protobuf compiler cannot protect participants from reusing extension field numbers.

​	未经验证系统的缺点是 protobuf 编译器无法保护参与者免受字段号重用的影响。

**Unverified extension field number allocation strategies are not recommended** because the [Consequences of Reusing Field Numbers](https://protobuf.dev/programming-guides/editions/#consequences) fall on all extenders of a message (not just the developer that didn’t follow the recommendations). If your use case requires very low coordination, consider using the [`Any` message]({{< ref "/docs/ReferenceGuides/ProtocolBuffers/Well-KnownTypes#any" >}}) instead.

​	**不推荐使用未经验证的扩展字段号分配策略**，因为[重用字段号的后果](https://protobuf.dev/programming-guides/editions/#consequences)会影响消息的所有扩展者（不仅是未遵循建议的开发者）。如果您的用例需要极低的协调性，请考虑使用[`Any` 消息]({{< ref "/docs/ReferenceGuides/ProtocolBuffers/Well-KnownTypes#any" >}})代替。

Unverified extension field number allocation strategies are limited to the range 1 to 524,999,999. Field numbers 525,000,000 and above can only be used with extension declarations.

​	未经验证的扩展字段号分配策略的范围限制为 1 到 524,999,999。字段号 525,000,000 及以上只能用于扩展声明。

### 指定扩展类型 Specifying Extension Types

Extensions can be of any field type except `oneof`s and `map`s.

​	扩展字段可以是任何字段类型，但不能是 `oneof` 或 `map`。

### 嵌套扩展（不推荐） Nested Extensions (not recommended)

You can declare extensions in the scope of another message:

​	您可以在另一个消息的范围内声明扩展：

```proto
import "common/user_profile.proto";

package puppies;

message Photo {
  extend common.UserProfile {
    int32 likes_count = 111;
  }
  ...
}
```

In this case, the C++ code to access this extension is:

​	在这种情况下，访问此扩展的 C++ 代码为：

```cpp
UserProfile user_profile;
user_profile.SetExtension(puppies::Photo::likes_count, 42);
```

In other words, the only effect is that `likes_count` is defined within the scope of `puppies.Photo`.

​	这意味着 `likes_count` 被定义在 `puppies.Photo` 的范围内。

This is a common source of confusion: Declaring an `extend` block nested inside a message type *does not* imply any relationship between the outer type and the extended type. In particular, the earlier example *does not* mean that `Photo` is any sort of subclass of `UserProfile`. All it means is that the symbol `likes_count` is declared inside the scope of `Photo`; it’s simply a static member.

​	这种做法容易造成混淆：嵌套在消息类型中的 `extend` 块**并不**意味着外部类型与扩展类型之间有任何关系。例如，上述示例**并不**意味着 `Photo` 是 `UserProfile` 的任何子类。实际上，`likes_count` 只是一个静态成员符号，定义在 `Photo` 的范围内。

A common pattern is to define extensions inside the scope of the extension’s field type - for example, here’s an extension to `media.UserContent` of type `puppies.Photo`, where the extension is defined as part of `Photo`:

​	一个常见模式是将扩展定义在扩展字段类型的范围内。例如，以下是针对 `media.UserContent` 的扩展，其类型为 `puppies.Photo`，扩展被定义为 `Photo` 的一部分：

```proto
import "media/user_content.proto";

package puppies;

message Photo {
  extend media.UserContent {
    Photo puppy_photo = 127;
  }
  ...
}
```

However, there is no requirement that an extension with a message type be defined inside that type. You can also use the standard definition pattern:

​	然而，扩展的定义**并不要求**在类型内部。您也可以使用标准定义模式：

```proto
import "media/user_content.proto";

package puppies;

message Photo {
  ...
}

// This can even be in a different file.
// 这甚至可以在不同的文件中。
extend media.UserContent {
  Photo puppy_photo = 127;
}
```

This **standard (file-level) syntax is preferred** to avoid confusion. The nested syntax is often mistaken for subclassing by users who are not already familiar with extensions.

​	**推荐使用标准（文件级别）的语法**以避免混淆。嵌套语法通常会被不熟悉扩展机制的用户误认为是子类化。

## Any

The `Any` message type lets you use messages as embedded types without having their .proto definition. An `Any` contains an arbitrary serialized message as `bytes`, along with a URL that acts as a globally unique identifier for and resolves to that message’s type. To use the `Any` type, you need to [import](https://protobuf.dev/programming-guides/editions/#other) `google/protobuf/any.proto`.

​	`Any` 消息类型允许您将消息作为嵌入类型使用，而无需其 `.proto` 定义。`Any` 包含一个任意的序列化消息（以 `bytes` 存储），以及一个 URL 作为全局唯一标识符并解析为该消息的类型。要使用 `Any` 类型，您需要[导入](https://protobuf.dev/programming-guides/editions/#other)`google/protobuf/any.proto`：

```proto
import "google/protobuf/any.proto";

message ErrorStatus {
  string message = 1;
  repeated google.protobuf.Any details = 2;
}
```

The default type URL for a given message type is `type.googleapis.com/_packagename_._messagename_`.

​	给定消息类型的默认类型 URL 为 `type.googleapis.com/_packagename_._messagename_`。

Different language implementations will support runtime library helpers to pack and unpack `Any` values in a typesafe manner – for example, in Java, the `Any` type will have special `pack()` and `unpack()` accessors, while in C++ there are `PackFrom()` and `UnpackTo()` methods:

​	不同语言实现支持运行时库助手以类型安全的方式打包和解包 `Any` 值。例如，在 Java 中，`Any` 类型有特殊的 `pack()` 和 `unpack()` 方法；在 C++ 中有 `PackFrom()` 和 `UnpackTo()` 方法：

```cpp
// Storing an arbitrary message type in Any.
// 将任意消息类型存储到 Any 中。
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

If you want to limit contained messages to a small number of types and to require permission before adding new types to the list, consider using [extensions](https://protobuf.dev/programming-guides/editions/#extensions) with [extension declarations]({{< ref "/docs/ProgrammingGuides/ExtensionDeclarations" >}}) instead of `Any` message types.

​	如果您希望限制包含的消息类型数量，并要求在将新类型添加到列表前获得许可，可以考虑使用[扩展](https://protobuf.dev/programming-guides/editions/#extensions)和[扩展声明]({{< ref "/docs/ProgrammingGuides/ExtensionDeclarations" >}})替代 `Any` 消息类型。

## Oneof

If you have a message with many singular fields and where at most one field will be set at the same time, you can enforce this behavior and save memory by using the oneof feature.

​	如果您的消息有多个单一字段（singular field），并且一次最多只能设置一个字段，则可以通过使用 `oneof` 功能强制此行为并节省内存。

Oneof fields are like singular fields except all the fields in a oneof share memory, and at most one field can be set at the same time. Setting any member of the oneof automatically clears all the other members. You can check which value in a oneof is set (if any) using a special `case()` or `WhichOneof()` method, depending on your chosen language.

​	`Oneof` 字段与单一字段类似，但所有字段共享内存，并且一次最多只能设置一个字段。设置任何 `oneof` 成员会自动清除所有其他成员。您可以使用特殊的 `case()` 或 `WhichOneof()` 方法（取决于所选语言）检查 `oneof` 中哪个值已设置（如果有的话）。

Note that if *multiple values are set, the last set value as determined by the order in the proto will overwrite all previous ones*.

​	**注意**：如果**多个值被设置，最后设置的值（按 proto 中的顺序确定）将覆盖所有先前的值**。

Field numbers for oneof fields must be unique within the enclosing message.

​	`Oneof` 字段的字段号在包含的消息中必须是唯一的。

### Using Oneof

To define a oneof in your `.proto` you use the `oneof` keyword followed by your oneof name, in this case `test_oneof`:

​	要在 `.proto` 中定义一个 `oneof`，使用关键字 `oneof`，后跟您定义的名称，例如 `test_oneof`：

```proto
message SampleMessage {
  oneof test_oneof {
    string name = 4;
    SubMessage sub_message = 9;
  }
}
```

You then add your oneof fields to the oneof definition. You can add fields of any type, except `map` fields and `repeated` fields. If you need to add a repeated field to a oneof, you can use a message containing the repeated field.

​	然后，您将 `oneof` 字段添加到定义中。您可以添加任何类型的字段，但不能是 `map` 字段和 `repeated` 字段。如果需要向 `oneof` 添加一个重复字段（repeated field），可以使用包含该重复字段的消息。

In your generated code, oneof fields have the same getters and setters as regular fields. You also get a special method for checking which value (if any) in the oneof is set. You can find out more about the oneof API for your chosen language in the relevant [API reference]({{< ref "/docs/ReferenceGuides" >}}).

​	在生成的代码中，`oneof` 字段与常规字段具有相同的 getter 和 setter。此外，您还会获得一个特殊方法，用于检查 `oneof` 中设置了哪个值（如果有的话）。有关所选语言的 `oneof` API 的更多信息，请参阅相关的[API 参考]({{< ref "/docs/ReferenceGuides" >}})。

### Oneof Features

- Setting a oneof field will automatically clear all other members of the oneof. So if you set several oneof fields, only the *last* field you set will still have a value. 设置一个 `oneof` 字段会自动清除该 `oneof` 的所有其他成员。因此，如果设置了多个 `oneof` 字段，只有**最后**设置的字段会保留值。

  ```cpp
  SampleMessage message;
  message.set_name("name");
  CHECK(message.has_name());
  // Calling mutable_sub_message() will clear the name field and will set
  // sub_message to a new instance of SubMessage with none of its fields set.
  // 调用 mutable_sub_message() 会清除 name 字段，并将 sub_message 设置为一个新的 SubMessage 实例，
  // 其字段尚未设置。
  message.mutable_sub_message();
  CHECK(!message.has_name());
  ```

- If the parser encounters multiple members of the same oneof on the wire, only the last member seen is used in the parsed message.

  - 如果解析器在数据流中遇到同一 `oneof` 的多个成员，则仅使用最后一个成员。

- Extensions are not supported for oneof.

  - `oneof` 不支持扩展。

- A oneof cannot be `repeated`.

  - `oneof` 字段不能是 `repeated`。

- Reflection APIs work for oneof fields.

  - 反射 API 可用于 `oneof` 字段。

- If you set a oneof field to the default value (such as setting an int32 oneof field to 0), the “case” of that oneof field will be set, and the value will be serialized on the wire.

  - 如果将 `oneof` 字段设置为默认值（例如将一个 `int32` 的 `oneof` 字段设置为 `0`），该 `oneof` 字段的“状态”将被设置，并且值会在数据流中被序列化。

- If you’re using C++, make sure your code doesn’t cause memory crashes. The following sample code will crash because `sub_message` was already deleted by calling the `set_name()` method.

  - 如果您使用 C++，请确保代码不会引发内存崩溃。以下示例代码会崩溃，因为调用 `set_name()` 方法后，`sub_message` 已被删除。


  ```cpp
  SampleMessage message;
  SubMessage* sub_message = message.mutable_sub_message();
  message.set_name("name");      // Will delete sub_message 将删除 sub_message
  sub_message->set_...            // Crashes here 此处崩溃
  ```

- Again in C++, if you `Swap()` two messages with oneofs, each message will end up with the other’s oneof case: in the example below, `msg1` will have a `sub_message` and `msg2` will have a `name`.

  - 同样在 C++ 中，如果对包含 `oneof` 的两个消息调用 `Swap()`，每个消息将交换对方的 `oneof` 状态：在以下示例中，`msg1` 将包含 `sub_message`，而 `msg2` 将包含 `name`。

  ```cpp
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

​	在添加或删除 `oneof` 字段时需谨慎。如果检查 `oneof` 的值返回 `None` 或 `NOT_SET`，可能意味着 `oneof` 未设置，或者设置为 `oneof` 的不同版本中的字段。无法区分，因为无法判断数据流中的未知字段是否属于 `oneof`。

#### 标记重用问题 Tag Reuse Issues

- **Move singular fields into or out of a oneof**: You may lose some of your information (some fields will be cleared) after the message is serialized and parsed. However, you can safely move a single field into a **new** oneof and may be able to move multiple fields if it is known that only one is ever set. See [Updating A Message Type](https://protobuf.dev/programming-guides/editions/#updating) for further details.
  - **将单一字段移入或移出 `oneof`**：在消息序列化和解析后，可能会丢失一些信息（某些字段会被清除）。但您可以安全地将单个字段移入**新**的 `oneof`，如果已知仅有一个字段被设置，也可以移动多个字段。请参阅 [更新消息类型](https://protobuf.dev/programming-guides/editions/#updating) 了解更多细节。

- **Delete a oneof field and add it back**: This may clear your currently set oneof field after the message is serialized and parsed.
  - **删除并重新添加 `oneof` 字段**：在消息序列化和解析后，这可能会清除当前设置的 `oneof` 字段。

- **Split or merge oneof**: This has similar issues to moving singular fields.
  - **拆分或合并 `oneof`**：此操作的问题与移动单一字段类似。


## Maps

If you want to create an associative map as part of your data definition, protocol buffers provides a handy shortcut syntax:

​	如果希望在数据定义中创建关联映射，Protocol Buffers 提供了一种便捷的简写语法：

```proto
map<key_type, value_type> map_field = N;
```

…where the `key_type` can be any integral or string type (so, any [scalar](https://protobuf.dev/programming-guides/editions/#scalar) type except for floating point types and `bytes`). Note that neither enum nor proto messages are valid for `key_type`. The `value_type` can be any type except another map.

​	其中 `key_type` 可以是任何整型或字符串类型（即任何[标量](https://protobuf.dev/programming-guides/editions/#scalar)类型，但不包括浮点类型和 `bytes`）。注意，枚举或 Protobuf 消息不能用作 `key_type`。`value_type` 可以是除其他映射外的任何类型。

So, for example, if you wanted to create a map of projects where each `Project` message is associated with a string key, you could define it like this:

​	例如，如果想创建一个项目的映射，其中每个 `Project` 消息与一个字符串键相关联，可以这样定义：

```proto
map<string, Project> projects = 3;
```

### Maps Features

- Extensions are not supported for maps.
  - 映射不支持扩展。

- Map fields cannot be `repeated`.
  - 映射字段不能是 `repeated`。

- Wire format ordering and map iteration ordering of map values is undefined, so you cannot rely on your map items being in a particular order.
  - 数据流格式中的顺序和映射值的迭代顺序是未定义的，因此不能依赖映射条目的特定顺序。

- When generating text format for a `.proto`, maps are sorted by key. Numeric keys are sorted numerically.
  - 在生成 `.proto` 的文本格式时，映射会按键排序。数值键按数值顺序排序。

- When parsing from the wire or when merging, if there are duplicate map keys the last key seen is used. When parsing a map from text format, parsing may fail if there are duplicate keys.
  - 从数据流解析或合并映射时，如果存在重复键，则使用最后出现的键值。当从文本格式解析映射时，若有重复键，解析可能会失败。

- If you provide a key but no value for a map field, the behavior when the field is serialized is language-dependent. In C++, Java, Kotlin, and Python the default value for the type is serialized, while in other languages nothing is serialized.
  - 如果为映射字段提供键但未提供值，则序列化时的行为取决于语言。在 C++、Java、Kotlin 和 Python 中，会序列化该类型的默认值；而在其他语言中，则不会序列化任何内容。

- No symbol `FooEntry` can exist in the same scope as a map `foo`, because `FooEntry` is already used by the implementation of the map.
  - 在映射 `foo` 的范围内，不能存在符号 `FooEntry`，因为 `FooEntry` 已被映射的实现使用。


The generated map API is currently available for all supported languages. You can find out more about the map API for your chosen language in the relevant [API reference]({{< ref "/docs/ReferenceGuides" >}}).

​	当前，所有支持的语言均可使用生成的映射 API。有关所选语言的映射 API 的更多信息，请参阅相关[API 参考]({{< ref "/docs/ReferenceGuides" >}})。

### 向后兼容性 Backwards Compatibility

The map syntax is equivalent to the following on the wire, so protocol buffers implementations that do not support maps can still handle your data:

​	映射语法在数据流上等同于以下定义，因此不支持映射的 Protocol Buffers 实现仍然可以处理您的数据：

```proto
message MapFieldEntry {
  key_type key = 1;
  value_type value = 2;
}

repeated MapFieldEntry map_field = N;
```

Any protocol buffers implementation that supports maps must both produce and accept data that can be accepted by the earlier definition.

​	任何支持映射的 Protocol Buffers 实现都必须能够生成并接受与上述定义兼容的数据。

## Packages

You can add an optional `package` specifier to a `.proto` file to prevent name clashes between protocol message types.

​	您可以在 `.proto` 文件中添加可选的 `package` 指定符，以防止协议消息类型之间发生名称冲突。

```proto
package foo.bar;
message Open { ... }
```

You can then use the package specifier when defining fields of your message type:

​	然后，您可以在定义消息类型的字段时使用包指定符：

```proto
message Foo {
  ...
  foo.bar.Open open = 1;
  ...
}
```

The way a package specifier affects the generated code depends on your chosen language:

​	包指定符对生成代码的影响取决于您选择的语言：

- In **C++** the generated classes are wrapped inside a C++ namespace. For example, `Open` would be in the namespace `foo::bar`.
  - 在 **C++** 中，生成的类将被封装在一个 C++ 命名空间中。例如，`Open` 将位于命名空间 `foo::bar` 中。

- In **Java** and **Kotlin**, the package is used as the Java package, unless you explicitly provide an `option java_package` in your `.proto` file.
  - 在 **Java** 和 **Kotlin** 中，该包被用作 Java 包，除非您在 `.proto` 文件中显式提供 `option java_package`。

- In **Python**, the `package` directive is ignored, since Python modules are organized according to their location in the file system.
  - 在 **Python** 中，`package` 指令会被忽略，因为 Python 模块是根据它们在文件系统中的位置组织的。

- In **Go**, the `package` directive is ignored, and the generated `.pb.go` file is in the package named after the corresponding `go_proto_library` Bazel rule. For open source projects, you **must** provide either a `go_package` option or set the Bazel `-M` flag.
  - 在 **Go** 中，`package` 指令会被忽略，生成的 `.pb.go` 文件的包名称取决于相应的 `go_proto_library` Bazel 规则的命名。在开源项目中，您必须提供 `go_package` 选项或设置 Bazel 的 `-M` 标志。

- In **Ruby**, the generated classes are wrapped inside nested Ruby namespaces, converted to the required Ruby capitalization style (first letter capitalized; if the first character is not a letter, `PB_` is prepended). For example, `Open` would be in the namespace `Foo::Bar`.
  - 在 **Ruby** 中，生成的类会封装在嵌套的 Ruby 命名空间中，并转换为 Ruby 的大写风格（首字母大写；如果第一个字符不是字母，则会添加 `PB_` 前缀）。例如，`Open` 将位于命名空间 `Foo::Bar` 中。

- In **PHP** the package is used as the namespace after converting to PascalCase, unless you explicitly provide an `option php_namespace` in your `.proto` file. For example, `Open` would be in the namespace `Foo\Bar`.
  - 在 **PHP** 中，包被用作 PascalCase 风格的命名空间，除非您在 `.proto` 文件中显式提供 `option php_namespace`。例如，`Open` 将位于命名空间 `Foo\Bar` 中。

- In **C#** the package is used as the namespace after converting to PascalCase, unless you explicitly provide an `option csharp_namespace` in your `.proto` file. For example, `Open` would be in the namespace `Foo.Bar`.
  - 在 **C#** 中，包被用作 PascalCase 风格的命名空间，除非您在 `.proto` 文件中显式提供 `option csharp_namespace`。例如，`Open` 将位于命名空间 `Foo.Bar` 中。


Note that even when the `package` directive does not directly affect the generated code, for example in Python, it is still strongly recommended to specify the package for the `.proto` file, as otherwise it may lead to naming conflicts in descriptors and make the proto not portable for other languages.

​	请注意，即使 `package` 指令并未直接影响生成的代码，例如在 Python 中，仍然强烈建议为 `.proto` 文件指定 `package`，否则可能导致描述符中的命名冲突，并使该 proto 无法移植到其他语言中。

### Packages 和名称解析 Packages and Name Resolution

Type name resolution in the protocol buffer language works like C++: first the innermost scope is searched, then the next-innermost, and so on, with each package considered to be “inner” to its parent package. A leading ‘.’ (for example, `.foo.bar.Baz`) means to start from the outermost scope instead.

​	协议缓冲区语言中的类型名称解析类似于 C++：首先搜索最内部作用域，然后依次搜索更外层的作用域，每个包被视为其父包的“内部”。一个前导的 `.`（例如 `.foo.bar.Baz`）表示从最外层作用域开始。

The protocol buffer compiler resolves all type names by parsing the imported `.proto` files. The code generator for each language knows how to refer to each type in that language, even if it has different scoping rules.

​	协议缓冲区编译器通过解析导入的 `.proto` 文件来解析所有类型名称。每种语言的代码生成器知道如何在该语言中引用每种类型，即使它有不同的作用域规则。

## 定义服务 Defining Services

If you want to use your message types with an RPC (Remote Procedure Call) system, you can define an RPC service interface in a `.proto` file and the protocol buffer compiler will generate service interface code and stubs in your chosen language. So, for example, if you want to define an RPC service with a method that takes your `SearchRequest` and returns a `SearchResponse`, you can define it in your `.proto` file as follows:

​	如果希望将消息类型用于 RPC（远程过程调用）系统，可以在 `.proto` 文件中定义 RPC 服务接口，协议缓冲区编译器会为您选择的语言生成服务接口代码和存根代码。例如，如果想定义一个 RPC 服务，方法需要接受 `SearchRequest` 并返回 `SearchResponse`，可以在 `.proto` 文件中如下定义：

```proto
service SearchService {
  rpc Search(SearchRequest) returns (SearchResponse);
}
```

The most straightforward RPC system to use with protocol buffers is [gRPC](https://grpc.io/): a language- and platform-neutral open source RPC system developed at Google. gRPC works particularly well with protocol buffers and lets you generate the relevant RPC code directly from your `.proto` files using a special protocol buffer compiler plugin.

​	最简单的使用协议缓冲区的 RPC 系统是 [gRPC](https://grpc.io/)：这是 Google 开发的一种语言和平台中立的开源 RPC 系统。gRPC 尤其适合与协议缓冲区配合使用，您可以使用一个特殊的协议缓冲区编译器插件直接从 `.proto` 文件生成相关的 RPC 代码。

If you don’t want to use gRPC, it’s also possible to use protocol buffers with your own RPC implementation. You can find out more about this in the [Proto2 Language Guide]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2#services" >}}).

​	如果您不想使用 gRPC，也可以将协议缓冲区与您自己的 RPC 实现配合使用。有关详细信息，请参阅 [Proto2 语言指南]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2#services" >}})。

There are also a number of ongoing third-party projects to develop RPC implementations for Protocol Buffers. For a list of links to projects we know about, see the [third-party add-ons wiki page](https://github.com/protocolbuffers/protobuf/blob/master/docs/third_party.md).

​	此外，还存在一些正在进行的第三方项目，用于开发协议缓冲区的 RPC 实现。关于我们知道的项目的链接列表，请参阅 [第三方插件 Wiki 页面](https://github.com/protocolbuffers/protobuf/blob/master/docs/third_party.md)。

## JSON Mapping

The standard protobuf binary wire format is the preferred serialization format for communication between two systems that use protobufs. For communicating with systems that use JSON rather than protobuf wire format, Protobuf supports a canonical encoding in [ProtoJSON]({{< ref "/docs/ProgrammingGuides/ProtoJSONFormat" >}}).

​	标准的 Protobuf 二进制数据流格式是推荐的用于基于 Protobuf 通信的序列化格式。对于需要与使用 JSON 而非 Protobuf 数据流格式的系统通信的场景，Protobuf 支持 [ProtoJSON]({{< ref "/docs/ProgrammingGuides/ProtoJSONFormat" >}}) 的规范编码。

## Options

Individual declarations in a `.proto` file can be annotated with a number of *options*. Options do not change the overall meaning of a declaration, but may affect the way it is handled in a particular context. The complete list of available options is defined in [`/google/protobuf/descriptor.proto`](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/descriptor.proto).

​	在 `.proto` 文件中，个别声明可以通过多个 *选项* 进行注释。选项不会改变声明的整体含义，但可能会在特定上下文中影响其处理方式。完整的可用选项列表定义在 [`/google/protobuf/descriptor.proto`](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/descriptor.proto) 中。

Some options are file-level options, meaning they should be written at the top-level scope, not inside any message, enum, or service definition. Some options are message-level options, meaning they should be written inside message definitions. Some options are field-level options, meaning they should be written inside field definitions. Options can also be written on enum types, enum values, oneof fields, service types, and service methods; however, no useful options currently exist for any of these.

​	某些选项是文件级选项，意味着它们应该写在顶层作用域中，而不是消息、枚举或服务定义内部。某些选项是消息级选项，应该写在消息定义中。某些选项是字段级选项，应该写在字段定义中。选项也可以写在枚举类型、枚举值、`oneof` 字段、服务类型和服务方法上；不过，目前没有任何实用的选项适用于这些情况。

Here are a few of the most commonly used options:

​	以下是一些最常用的选项：

- `java_package` (file option): The package you want to use for your generated Java/Kotlin classes. If no explicit `java_package` option is given in the `.proto` file, then by default the proto package (specified using the “package” keyword in the `.proto` file) will be used. However, proto packages generally do not make good Java packages since proto packages are not expected to start with reverse domain names. If not generating Java or Kotlin code, this option has no effect. **`java_package`**（文件选项）：指定要用于生成的 Java/Kotlin 类的包。如果 `.proto` 文件中未显式给出 `java_package` 选项，则默认使用 proto 包（通过 `.proto` 文件中的 `package` 关键字指定）。然而，proto 包通常不是很好的 Java 包，因为它们通常不会以反向域名开头。如果不生成 Java 或 Kotlin 代码，此选项无效。

  ```proto
  option java_package = "com.example.foo";
  ```

- `java_outer_classname` (file option): The class name (and hence the file name) for the wrapper Java class you want to generate. If no explicit `java_outer_classname` is specified in the `.proto` file, the class name will be constructed by converting the `.proto` file name to camel-case (so `foo_bar.proto` becomes `FooBar.java`). If the `java_multiple_files` option is disabled, then all other classes/enums/etc. generated for the `.proto` file will be generated *within* this outer wrapper Java class as nested classes/enums/etc. If not generating Java code, this option has no effect. **`java_outer_classname`**（文件选项）：指定包装的 Java 类名（因此也是文件名）。如果 `.proto` 文件中未显式指定 `java_outer_classname`，则类名将通过将 `.proto` 文件名转换为 CamelCase 形式生成（例如，`foo_bar.proto` 生成 `FooBar.java`）。如果禁用了 `java_multiple_files` 选项，则为 `.proto` 文件生成的所有其他类、枚举等将作为嵌套类、枚举等包含在此外部 Java 类中。如果不生成 Java 代码，此选项无效。

  ```proto
  option java_outer_classname = "Ponycopter";
  ```

- `java_multiple_files` (file option): If false, only a single `.java` file will be generated for this `.proto` file, and all the Java classes/enums/etc. generated for the top-level messages, services, and enumerations will be nested inside of an outer class (see `java_outer_classname`). If true, separate `.java` files will be generated for each of the Java classes/enums/etc. generated for the top-level messages, services, and enumerations, and the wrapper Java class generated for this `.proto` file won’t contain any nested classes/enums/etc. This is a Boolean option which defaults to `false`. If not generating Java code, this option has no effect. **`java_multiple_files`**（文件选项）：如果为 `false`，则 `.proto` 文件仅生成一个 `.java` 文件，所有为顶级消息、服务和枚举生成的 Java 类/枚举等将嵌套在一个外部类中（请参阅 `java_outer_classname`）。如果为 `true`，则为顶级消息、服务和枚举生成的每个 Java 类/枚举等生成单独的 `.java` 文件，且为此 `.proto` 文件生成的包装 Java 类不会包含任何嵌套类/枚举等。这是一个布尔选项，默认为 `false`。如果不生成 Java 代码，此选项无效。

  ```proto
  option java_multiple_files = true;
  ```

- `optimize_for` (file option): Can be set to `SPEED`, `CODE_SIZE`, or `LITE_RUNTIME`. This affects the C++ and Java code generators (and possibly third-party generators) in the following ways: **`optimize_for`**（文件选项）：可以设置为 `SPEED`、`CODE_SIZE` 或 `LITE_RUNTIME`。它影响 C++ 和 Java 代码生成器（以及可能的第三方生成器），具体如下：

  - `SPEED` (default): The protocol buffer compiler will generate code for serializing, parsing, and performing other common operations on your message types. This code is highly optimized.
    - **`SPEED`（默认）**：协议缓冲区编译器将生成用于序列化、解析和执行其他常见操作的代码。这些代码高度优化。

  - `CODE_SIZE`: The protocol buffer compiler will generate minimal classes and will rely on shared, reflection-based code to implement serialization, parsing, and various other operations. The generated code will thus be much smaller than with `SPEED`, but operations will be slower. Classes will still implement exactly the same public API as they do in `SPEED` mode. This mode is most useful in apps that contain a very large number of `.proto` files and do not need all of them to be blindingly fast.
    - **`CODE_SIZE`**：协议缓冲区编译器将生成最小化的类，并依赖于基于共享反射的代码来实现序列化、解析和各种其他操作。生成的代码将比 `SPEED` 模式小得多，但操作速度较慢。类仍将实现与 `SPEED` 模式相同的公共 API。此模式最适合包含大量 `.proto` 文件且对速度要求不高的应用程序。

  - `LITE_RUNTIME`: The protocol buffer compiler will generate classes that depend only on the “lite” runtime library (`libprotobuf-lite` instead of `libprotobuf`). The lite runtime is much smaller than the full library (around an order of magnitude smaller) but omits certain features like descriptors and reflection. This is particularly useful for apps running on constrained platforms like mobile phones. The compiler will still generate fast implementations of all methods as it does in `SPEED` mode. Generated classes will only implement the `MessageLite` interface in each language, which provides only a subset of the methods of the full `Message` interface.
    - **`LITE_RUNTIME`**：协议缓冲区编译器将生成依赖于“轻量级”运行时库（`libprotobuf-lite` 而不是 `libprotobuf`）的类。轻量级运行时比完整库小得多（大约小一个数量级），但省略了某些功能，例如描述符和反射。这对运行在受限平台（如移动设备）上的应用程序特别有用。生成的类仍将实现与 `SPEED` 模式相同的快速方法，但只实现每种语言的 `MessageLite` 接口，这只提供了完整 `Message` 接口的一部分方法。


  ```proto
  option optimize_for = CODE_SIZE;
  ```

- `cc_generic_services`, `java_generic_services`, `py_generic_services` (file options): **Generic services are deprecated.** Whether or not the protocol buffer compiler should generate abstract service code based on [services definitions](https://protobuf.dev/programming-guides/editions/#services) in C++, Java, and Python, respectively. For legacy reasons, these default to `true`. However, as of version 2.3.0 (January 2010), it is considered preferable for RPC implementations to provide [code generator plugins](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.compiler.plugin.pb) to generate code more specific to each system, rather than rely on the “abstract” services. **`cc_generic_services`**、**`java_generic_services`**、**`py_generic_services`**（文件选项）：**通用服务已被弃用**。是否生成基于服务定义的抽象服务代码（参见 [服务定义](https://protobuf.dev/programming-guides/editions/#services)），分别适用于 C++、Java 和 Python。出于历史原因，这些选项默认为 `true`。然而，自 2.3.0 版本（2010 年 1 月）起，建议通过 RPC 实现提供的[代码生成插件](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.compiler.plugin.pb)生成更具体于每个系统的代码，而不是依赖“抽象”服务。

  ```proto
  // This file relies on plugins to generate service code.
  option cc_generic_services = false;
  option java_generic_services = false;
  option py_generic_services = false;
  ```

- `cc_enable_arenas` (file option): Enables [arena allocation]({{< ref "/docs/ReferenceGuides/CPlusPlus/ArenaAllocationGuide" >}}) for C++ generated code.

  - **`cc_enable_arenas`**（文件选项）：启用 [arena 分配]({{< ref "/docs/ReferenceGuides/CPlusPlus/ArenaAllocationGuide" >}}) 以用于生成的 C++ 代码。

- `objc_class_prefix` (file option): Sets the Objective-C class prefix which is prepended to all Objective-C generated classes and enums from this .proto. There is no default. You should use prefixes that are between 3-5 uppercase characters as [recommended by Apple](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Conventions/Conventions.html#//apple_ref/doc/uid/TP40011210-CH10-SW4). Note that all 2 letter prefixes are reserved by Apple.

  - **`objc_class_prefix`**（文件选项）：设置前缀，用于所有从该 `.proto` 文件生成的 Objective-C 类和枚举的名称。默认没有前缀。您应该使用 Apple 推荐的 3-5 个大写字符的前缀（[Apple 文档](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Conventions/Conventions.html#//apple_ref/doc/uid/TP40011210-CH10-SW4)）。请注意，所有 2 字母前缀都被 Apple 保留。

- `packed` (field option): In protobuf editions, this option is locked to `true`. To use unpacked wireformat, you can override this option using an editions feature. This provides compatibility with parsers prior to version 2.3.0 (rarely needed) as shown in the following example: **`packed`**（字段选项）：在 protobuf editions 中，此选项被锁定为 `true`。如果需要使用未打包的线格式，可以通过 editions 功能覆盖此选项。这为 2.3.0 之前的解析器提供了兼容性支持（很少需要）。示例如下：

  ```proto
  repeated int32 samples = 4 [features.repeated_field_encoding = EXPANDED];
  ```

- `deprecated` (field option): If set to `true`, indicates that the field is deprecated and should not be used by new code. In most languages this has no actual effect. In Java, this becomes a `@Deprecated` annotation. For C++, clang-tidy will generate warnings whenever deprecated fields are used. In the future, other language-specific code generators may generate deprecation annotations on the field’s accessors, which will in turn cause a warning to be emitted when compiling code which attempts to use the field. If the field is not used by anyone and you want to prevent new users from using it, consider replacing the field declaration with a [reserved](https://protobuf.dev/programming-guides/editions/#fieldreserved) statement. **`deprecated`**（字段选项）：如果设置为 `true`，表示该字段已弃用，不应被新代码使用。在大多数语言中，此选项不会产生实际效果。在 Java 中，这会变为 `@Deprecated` 注解。在 C++ 中，`clang-tidy` 在使用已弃用字段时会生成警告。未来，其他语言特定的代码生成器可能会在字段访问器上生成弃用注解，从而在编译尝试使用该字段的代码时发出警告。如果该字段未被任何人使用并且您希望防止新用户使用，可以考虑用 [reserved 声明](https://protobuf.dev/programming-guides/editions/#fieldreserved) 替代字段声明。

  ```proto
  int32 old_field = 6 [deprecated = true];
  ```

### 枚举值选项 Enum Value Options

Enum value options are supported. You can use the `deprecated` option to indicate that a value shouldn’t be used anymore. You can also create custom options using extensions.

​	支持为枚举值添加选项。可以使用 `deprecated` 选项指示某个值不应再被使用。您还可以使用扩展创建自定义选项。

The following example shows the syntax for adding these options:

​	以下示例展示了添加这些选项的语法：

```proto
import "google/protobuf/descriptor.proto";

extend google.protobuf.EnumValueOptions {
  string string_name = 123456789;
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

​	在 C++ 中读取 `string_name` 选项的代码可能如下：

```cpp
const absl::string_view foo = proto2::GetEnumDescriptor<Data>()
    ->FindValueByName("DATA_DISPLAY")->options().GetExtension(string_name);
```

See [Custom Options](https://protobuf.dev/programming-guides/editions/#customoptions) to see how to apply custom options to enum values and to fields.

​	有关如何将自定义选项应用于枚举值和字段的更多信息，请参见 [自定义选项](https://protobuf.dev/programming-guides/editions/#customoptions)。

### 自定义选项 Custom Options

Protocol Buffers also allows you to define and use your own options. Note that this is an **advanced feature** which most people don’t need. If you do think you need to create your own options, see the [Proto2 Language Guide]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2#customoptions" >}}) for details. Note that creating custom options uses [extensions]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2#extensions" >}}).

​	Protocol Buffers 允许您定义和使用自己的选项。请注意，这是一个 **高级功能**，大多数人不需要。如果您确实需要创建自己的选项，请参阅 [Proto2 语言指南]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2#customoptions" >}}) 了解详情。请注意，创建自定义选项需要使用 [扩展]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2#extensions" >}})。

### 选项保留 Option Retention

Options have a notion of *retention*, which controls whether an option is retained in the generated code. Options have *runtime retention* by default, meaning that they are retained in the generated code and are thus visible at runtime in the generated descriptor pool. However, you can set `retention = RETENTION_SOURCE` to specify that an option (or field within an option) must not be retained at runtime. This is called *source retention*.

​	选项具有 *保留模式*（retention），控制选项是否保留在生成的代码中。默认情况下，选项具有 *运行时保留*（runtime retention），这意味着它们会保留在生成的代码中，因此在生成的描述符池中运行时可见。然而，您可以设置 `retention = RETENTION_SOURCE`，以指定选项（或选项中的字段）在运行时不必保留。这称为 *源代码保留*（source retention）。

Option retention is an advanced feature that most users should not need to worry about, but it can be useful if you would like to use certain options without paying the code size cost of retaining them in your binaries. Options with source retention are still visible to `protoc` and `protoc` plugins, so code generators can use them to customize their behavior.

​	选项保留是一个高级功能，大多数用户无需担心，但如果您想在不增加二进制文件代码大小的情况下使用某些选项，这可能会很有用。具有源代码保留的选项仍然可以通过 `protoc` 和 `protoc` 插件看到，因此代码生成器可以使用它们自定义行为。

Retention can be set directly on an option, like this:

​	可以直接在选项上设置保留，如下所示：

```proto
extend google.protobuf.FileOptions {
  int32 source_retention_option = 1234
      [retention = RETENTION_SOURCE];
}
```

It can also be set on a plain field, in which case it takes effect only when that field appears inside an option:

​	也可以在普通字段上设置，这种情况下，仅当该字段出现在选项中时才生效：

```proto
message OptionsMessage {
  int32 source_retention_field = 1 [retention = RETENTION_SOURCE];
}
```

You can set `retention = RETENTION_RUNTIME` if you like, but this has no effect since it is the default behavior. When a message field is marked `RETENTION_SOURCE`, its entire contents are dropped; fields inside it cannot override that by trying to set `RETENTION_RUNTIME`.

​	您也可以设置 `retention = RETENTION_RUNTIME`，但这不会生效，因为它是默认行为。如果一个消息字段被标记为 `RETENTION_SOURCE`，其内容将完全被删除；其中的字段不能通过尝试设置 `RETENTION_RUNTIME` 来覆盖。

> Note
>
> As of Protocol Buffers 22.0, support for option retention is still in progress and only C++ and Java are supported. Go has support starting from 1.29.0. Python support is complete but has not made it into a release yet.
>
> ​	截至 Protocol Buffers 22.0，选项保留的支持仍在进行中，目前仅支持 C++ 和 Java。从 Go 1.29.0 开始支持 Go，而 Python 的支持已经完成，但尚未发布版本。

### 选项目标 Option Targets

Fields have a `targets` option which controls the types of entities that the field may apply to when used as an option. For example, if a field has `targets = TARGET_TYPE_MESSAGE` then that field cannot be set in a custom option on an enum (or any other non-message entity). Protoc enforces this and will raise an error if there is a violation of the target constraints.

​	字段具有 `targets` 选项，用于控制字段在用作选项时可以应用于的实体类型。例如，如果字段具有 `targets = TARGET_TYPE_MESSAGE`，那么该字段不能在枚举（或任何非消息实体）上的自定义选项中设置。`protoc` 强制执行此规则，如果违反目标约束，将会引发错误。

At first glance, this feature may seem unnecessary given that every custom option is an extension of the options message for a specific entity, which already constrains the option to that one entity. However, option targets are useful in the case where you have a shared options message applied to multiple entity types and you want to control the usage of individual fields in that message. For example:

​	乍看之下，这一功能似乎没有必要，因为每个自定义选项都是针对特定实体的选项消息的扩展，已经将选项约束到该实体。然而，当您将共享的选项消息应用于多种实体类型并希望控制该消息中各个字段的使用时，选项目标非常有用。例如：

```proto
message MyOptions {
  string file_only_option = 1 [targets = TARGET_TYPE_FILE];
  int32 message_and_enum_option = 2 [targets = TARGET_TYPE_MESSAGE,
                                     targets = TARGET_TYPE_ENUM];
}

extend google.protobuf.FileOptions {
  MyOptions file_options = 50000;
}

extend google.protobuf.MessageOptions {
  MyOptions message_options = 50000;
}

extend google.protobuf.EnumOptions {
  MyOptions enum_options = 50000;
}

// OK: this field is allowed on file options
// OK：此字段允许在文件选项上使用
option (file_options).file_only_option = "abc";

message MyMessage {
  // OK: this field is allowed on both message and enum options  
  // OK：此字段允许在消息和枚举选项上使用
  option (message_options).message_and_enum_option = 42;
}

enum MyEnum {
  MY_ENUM_UNSPECIFIED = 0;
  // Error: file_only_option cannot be set on an enum.
  // 错误：file_only_option 不能在枚举上设置。
  option (enum_options).file_only_option = "xyz";
}
```

## 生成类 Generating Your Classes

To generate the Java, Kotlin, Python, C++, Go, Ruby, Objective-C, or C# code that you need to work with the message types defined in a `.proto` file, you need to run the protocol buffer compiler `protoc` on the `.proto` file. If you haven’t installed the compiler, [download the package]({{< ref "/docs/Downloads" >}}) and follow the instructions in the README. For Go, you also need to install a special code generator plugin for the compiler; you can find this and installation instructions in the [golang/protobuf](https://github.com/golang/protobuf/) repository on GitHub.

​	要生成用于处理 `.proto` 文件中定义的消息类型的 Java、Kotlin、Python、C++、Go、Ruby、Objective-C 或 C# 代码，需要运行协议缓冲区编译器 `protoc`。如果尚未安装编译器，请[下载相关包]({{< ref "/docs/Downloads" >}})，并按照 README 中的说明进行操作。对于 Go，还需要为编译器安装一个特殊的代码生成器插件。可在 GitHub 的 [golang/protobuf](https://github.com/golang/protobuf/) 仓库中找到此插件及其安装说明。

The Protocol Compiler is invoked as follows:

​	协议缓冲区编译器的调用如下：

```sh
protoc --proto_path=IMPORT_PATH --cpp_out=DST_DIR --java_out=DST_DIR --python_out=DST_DIR --go_out=DST_DIR --ruby_out=DST_DIR --objc_out=DST_DIR --csharp_out=DST_DIR path/to/file.proto
```

- `IMPORT_PATH` specifies a directory in which to look for `.proto` files when resolving `import` directives. If omitted, the current directory is used. Multiple import directories can be specified by passing the `--proto_path` option multiple times; they will be searched in order. `-I=_IMPORT_PATH_` can be used as a short form of `--proto_path`.

  - `IMPORT_PATH` 指定了解析 `import` 指令时查找 `.proto` 文件的目录。如果未提供，则使用当前目录。可通过多次传递 `--proto_path` 选项指定多个导入目录，它们将按顺序搜索。`-I=_IMPORT_PATH_` 是 `--proto_path` 的简写。

- You can provide one or more *output directives*: 您可以提供一个或多个*输出指令*：

  - `--cpp_out` generates C++ code in `DST_DIR`. See the [C++ generated code reference]({{< ref "/docs/ReferenceGuides/CPlusPlus/GeneratedCodeGuide" >}}) for more.
    - `--cpp_out`：在 `DST_DIR` 中生成 C++ 代码。请参阅 [C++ 生成代码参考]({{< ref "/docs/ReferenceGuides/CPlusPlus/GeneratedCodeGuide" >}})。

  - `--java_out` generates Java code in `DST_DIR`. See the [Java generated code reference]({{< ref "/docs/ReferenceGuides/Java/GeneratedCodeGuide" >}}) for more.
    - `--java_out`：在 `DST_DIR` 中生成 Java 代码。请参阅 [Java 生成代码参考]({{< ref "/docs/ReferenceGuides/Java/GeneratedCodeGuide" >}})。

  - `--kotlin_out` generates additional Kotlin code in `DST_DIR`. See the [Kotlin generated code reference]({{< ref "/docs/ReferenceGuides/Kotlin/GeneratedCodeGuide" >}}) for more.
    - `--kotlin_out`：在 `DST_DIR` 中生成额外的 Kotlin 代码。请参阅 [Kotlin 生成代码参考]({{< ref "/docs/ReferenceGuides/Kotlin/GeneratedCodeGuide" >}})。

  - `--python_out` generates Python code in `DST_DIR`. See the [Python generated code reference]({{< ref "/docs/ReferenceGuides/Python/GeneratedCodeGuide" >}}) for more.
    - `--python_out`：在 `DST_DIR` 中生成 Python 代码。请参阅 [Python 生成代码参考]({{< ref "/docs/ReferenceGuides/Python/GeneratedCodeGuide" >}})。

  - `--go_out` generates Go code in `DST_DIR`. See the [Go generated code reference]({{< ref "/docs/ReferenceGuides/Go/GeneratedCodeGuide" >}}) for more.
    - `--go_out`：在 `DST_DIR` 中生成 Go 代码。请参阅 [Go 生成代码参考]({{< ref "/docs/ReferenceGuides/Go/GeneratedCodeGuide" >}})。

  - `--ruby_out` generates Ruby code in `DST_DIR`. See the [Ruby generated code reference]({{< ref "/docs/ReferenceGuides/Ruby/GeneratedCodeGuide" >}}) for more.
    - `--ruby_out`：在 `DST_DIR` 中生成 Ruby 代码。请参阅 [Ruby 生成代码参考]({{< ref "/docs/ReferenceGuides/Ruby/GeneratedCodeGuide" >}})。

  - `--objc_out` generates Objective-C code in `DST_DIR`. See the [Objective-C generated code reference]({{< ref "/docs/ReferenceGuides/Objective-C/GeneratedCodeGuide" >}}) for more.
    - `--objc_out`：在 `DST_DIR` 中生成 Objective-C 代码。请参阅 [Objective-C 生成代码参考]({{< ref "/docs/ReferenceGuides/Objective-C/GeneratedCodeGuide" >}})。

  - `--csharp_out` generates C# code in `DST_DIR`. See the [C# generated code reference]({{< ref "/docs/ReferenceGuides/CSharp/GeneratedCodeGuide" >}}) for more.
    - `--csharp_out`：在 `DST_DIR` 中生成 C# 代码。请参阅 [C# 生成代码参考]({{< ref "/docs/ReferenceGuides/CSharp/GeneratedCodeGuide" >}})。

  - `--php_out` generates PHP code in `DST_DIR`. See the [PHP generated code reference]({{< ref "/docs/ReferenceGuides/PHP/GeneratedCodeGuide" >}}) for more.
    - `--php_out`：在 `DST_DIR` 中生成 PHP 代码。请参阅 [PHP 生成代码参考]({{< ref "/docs/ReferenceGuides/PHP/GeneratedCodeGuide" >}})。


  As an extra convenience, if the `DST_DIR` ends in `.zip` or `.jar`, the compiler will write the output to a single ZIP-format archive file with the given name. `.jar` outputs will also be given a manifest file as required by the Java JAR specification. Note that if the output archive already exists, it will be overwritten.

  ​	为了方便，如果 `DST_DIR` 以 `.zip` 或 `.jar` 结尾，编译器会将输出写入一个具有给定名称的单一 ZIP 格式的归档文件中。`.jar` 输出还将按照 Java JAR 规范提供清单文件。请注意，如果输出归档文件已存在，将会被覆盖。

- You must provide one or more `.proto` files as input. Multiple `.proto` files can be specified at once. Although the files are named relative to the current directory, each file must reside in one of the `IMPORT_PATH`s so that the compiler can determine its canonical name.

  - 必须至少提供一个 `.proto` 文件作为输入。可以一次指定多个 `.proto` 文件。尽管这些文件相对于当前目录命名，但每个文件必须位于 `IMPORT_PATH` 之一中，以便编译器能够确定其规范名称。


## 文件位置 File location

Prefer not to put `.proto` files in the same directory as other language sources. Consider creating a subpackage `proto` for `.proto` files, under the root package for your project.

​	建议不要将 `.proto` 文件与其他语言的源文件放在同一目录中。考虑在项目的根包下为 `.proto` 文件创建一个名为 `proto` 的子包。

### 位置应与语言无关 Location Should be Language-agnostic

When working with Java code, it’s handy to put related `.proto` files in the same directory as the Java source. However, if any non-Java code ever uses the same protos, the path prefix will no longer make sense. So in general, put the protos in a related language-agnostic directory such as `//myteam/mypackage`.

​	在处理 Java 代码时，将相关的 `.proto` 文件与 Java 源文件放在同一目录中可能很方便。但是，如果任何非 Java 代码使用了相同的 protos，路径前缀将不再有意义。因此，一般情况下，请将 proto 文件放在类似 `//myteam/mypackage` 的相关语言无关目录中。

The exception to this rule is when it’s clear that the protos will be used only in a Java context, such as for testing.

​	如果明确 proto 文件仅在 Java 上下文中使用，例如用于测试，这是上述规则的例外。

## 支持的平台 Supported Platforms

For information about:

​	有关以下内容的信息：

- the operating systems, compilers, build systems, and C++ versions that are supported, see [Foundational C++ Support Policy](https://opensource.google/documentation/policies/cplusplus-support).
  - 支持的操作系统、编译器、构建系统和 C++ 版本，请参阅 [Foundational C++ Support Policy](https://opensource.google/documentation/policies/cplusplus-support)。

- the PHP versions that are supported, see [Supported PHP versions](https://cloud.google.com/php/getting-started/supported-php-versions).
  - 支持的 PHP 版本，请参阅 [Supported PHP versions](https://cloud.google.com/php/getting-started/supported-php-versions)。
