+++
title = "语言指南（proto 3）"
date = 2024-11-17T09:35:36+08:00
weight = 20
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

> 原文：[https://protobuf.dev/programming-guides/proto3/](https://protobuf.dev/programming-guides/proto3/)
>
> 收录该文档的时间：`2024-11-17T09:35:36+08:00`

# Language Guide (proto 3) - 语言指南（proto 3）

Covers how to use the proto3 revision of the Protocol Buffers language in your project.

​	涵盖如何在项目中使用 Protocol Buffers 语言的 proto3 修订版。

This guide describes how to use the protocol buffer language to structure your protocol buffer data, including `.proto` file syntax and how to generate data access classes from your `.proto` files. It covers the **proto3** revision of the protocol buffers language.

​	本指南描述了如何使用 Protocol Buffers 语言来构建 Protocol Buffers 数据，包括 `.proto` 文件的语法以及如何从 `.proto` 文件生成数据访问类。它涵盖了 Protocol Buffers 语言的 **proto3** 修订版。

For information on **editions** syntax, see the [Protobuf Editions Language Guide](https://protobuf.dev/programming-guides/editions).

​	有关 **editions** 语法的信息，请参阅 [Protobuf Editions Language Guide](https://protobuf.dev/programming-guides/editions)。

For information on the **proto2** syntax, see the [Proto2 Language Guide](https://protobuf.dev/programming-guides/proto2).

​	有关 **proto2** 语法的信息，请参阅 [Proto2 Language Guide](https://protobuf.dev/programming-guides/proto2)。

This is a reference guide – for a step by step example that uses many of the features described in this document, see the [tutorial](https://protobuf.dev/getting-started) for your chosen language.

​	这是一个参考指南——有关使用本文档中描述的许多特性的一步步示例，请参阅您所选语言的 [教程](https://protobuf.dev/getting-started)。

## 定义消息类型 Defining A Message Type

First let’s look at a very simple example. Let’s say you want to define a search request message format, where each search request has a query string, the particular page of results you are interested in, and a number of results per page. Here’s the `.proto` file you use to define the message type.

​	首先让我们来看一个非常简单的示例。假设您希望定义一个搜索请求的消息格式，其中每个搜索请求包含一个查询字符串、感兴趣的结果页以及每页结果的数量。以下是用于定义消息类型的 `.proto` 文件。

```proto
syntax = "proto3";

message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 results_per_page = 3;
}
```

- The first line of the file specifies that you’re using the proto3 revision of the protobuf language spec. 文件的第一行指定您正在使用 protobuf 语言规范的 proto3 修订版。
  - The `edition` (or `syntax` for proto2/proto3) must be the first non-empty, non-comment line of the file.
    - `edition`（或对于 proto2/proto3 的 `syntax`）必须是文件中的第一行非空、非注释内容。
  - If no `edition` or `syntax` is specified, the protocol buffer compiler will assume you are using [proto2](https://protobuf.dev/programming-guides/proto2).
    - 如果未指定 `edition` 或 `syntax`，Protocol Buffers 编译器将假定您使用的是 [proto2](https://protobuf.dev/programming-guides/proto2)。
- The `SearchRequest` message definition specifies three fields (name/value pairs), one for each piece of data that you want to include in this type of message. Each field has a name and a type.
  - `SearchRequest` 消息定义指定了三个字段（名称/值对），分别对应此类消息中要包含的每个数据片段。每个字段都有一个名称和一个类型。


### 指定字段类型 Specifying Field Types

In the earlier example, all the fields are [scalar types](https://protobuf.dev/programming-guides/proto3/#scalar): two integers (`page_number` and `results_per_page`) and a string (`query`). You can also specify [enumerations](https://protobuf.dev/programming-guides/proto3/#enum) and composite types like other message types for your field.

​	在上述示例中，所有字段均为 [标量类型](https://protobuf.dev/programming-guides/proto3/#scalar)：两个整数（`page_number` 和 `results_per_page`）以及一个字符串（`query`）。您还可以为字段指定 [枚举类型](https://protobuf.dev/programming-guides/proto3/#enum) 和复合类型（如其他消息类型）。

### 分配字段编号 Assigning Field Numbers

You must give each field in your message definition a number between `1` and `536,870,911` with the following restrictions:

​	您必须为消息定义中的每个字段分配一个编号，范围在 `1` 到 `536,870,911` 之间，并受以下限制：

- The given number **must be unique** among all fields for that message.
  - 给定编号在该消息的所有字段中必须是唯一的。

- Field numbers `19,000` to `19,999` are reserved for the Protocol Buffers implementation. The protocol buffer compiler will complain if you use one of these reserved field numbers in your message.
  - 编号 `19,000` 到 `19,999` 是为 Protocol Buffers 实现保留的。如果您在消息中使用这些保留字段编号，Protocol Buffers 编译器会报错。

- You cannot use any previously [reserved](https://protobuf.dev/programming-guides/proto3/#fieldreserved) field numbers or any field numbers that have been allocated to [extensions](https://protobuf.dev/programming-guides/proto2#extensions).
  - 您不能使用任何已被 [保留](https://protobuf.dev/programming-guides/proto3/#fieldreserved) 的字段编号或分配给 [扩展](https://protobuf.dev/programming-guides/proto2#extensions) 的字段编号。


This number **cannot be changed once your message type is in use** because it identifies the field in the [message wire format](https://protobuf.dev/programming-guides/encoding). “Changing” a field number is equivalent to deleting that field and creating a new field with the same type but a new number. See [Deleting Fields](https://protobuf.dev/programming-guides/proto3/#deleting) for how to do this properly.

​	字段编号一旦用于定义消息类型后 **不能更改**，因为它标识了 [消息的线格式](https://protobuf.dev/programming-guides/encoding)。更改字段编号相当于删除该字段并使用新编号创建一个具有相同类型的新字段。有关正确执行此操作的方法，请参阅 [删除字段](https://protobuf.dev/programming-guides/proto3/#deleting)。

Field numbers **should never be reused**. Never take a field number out of the [reserved](https://protobuf.dev/programming-guides/proto3/#fieldreserved) list for reuse with a new field definition. See [Consequences of Reusing Field Numbers](https://protobuf.dev/programming-guides/proto3/#consequences).

​	字段编号 **不应被重复使用**。不要从 [保留字段](https://protobuf.dev/programming-guides/proto3/#fieldreserved) 列表中取出字段编号用于定义新的字段。有关详细信息，请参阅 [重复使用字段编号的后果](https://protobuf.dev/programming-guides/proto3/#consequences)。

You should use the field numbers 1 through 15 for the most-frequently-set fields. Lower field number values take less space in the wire format. For example, field numbers in the range 1 through 15 take one byte to encode. Field numbers in the range 16 through 2047 take two bytes. You can find out more about this in [Protocol Buffer Encoding](https://protobuf.dev/programming-guides/encoding#structure).

​	建议将编号 `1` 到 `15` 分配给使用频率最高的字段。较低的字段编号值在线格式中占用的空间更少。例如，字段编号在 `1` 到 `15` 范围内需要 1 字节编码，而 `16` 到 `2047` 范围内需要 2 字节编码。有关更多信息，请参阅 [Protocol Buffer 编码](https://protobuf.dev/programming-guides/encoding#structure)。

#### 重复使用字段编号的后果 Consequences of Reusing Field Numbers

Reusing a field number makes decoding wire-format messages ambiguous.

​	重复使用字段编号会导致解析线格式消息时出现歧义。

The protobuf wire format is lean and doesn’t provide a way to detect fields encoded using one definition and decoded using another.

​	Protocol Buffers 的线格式精简，不提供方法来检测使用一个定义编码并使用另一个定义解码的字段。

Encoding a field using one definition and then decoding that same field with a different definition can lead to:

​	以一个定义编码字段并使用不同定义解码相同字段可能导致：

- Developer time lost to debugging
  - 开发者因调试问题浪费时间

- A parse/merge error (best case scenario)
  - 解析/合并错误（最好的情况）

- Leaked PII/SPII
  - 泄露 PII/SPII

- Data corruption
  - 数据损坏


Common causes of field number reuse:

​	字段编号重复使用的常见原因：

- renumbering fields (sometimes done to achieve a more aesthetically pleasing number order for fields). Renumbering effectively deletes and re-adds all the fields involved in the renumbering, resulting in incompatible wire-format changes.
  - 重新编号字段（有时为实现更美观的字段编号顺序）。重新编号会删除并重新添加所有涉及的字段，从而导致不兼容的线格式更改。

- deleting a field and not [reserving](https://protobuf.dev/programming-guides/proto3/#fieldreserved) the number to prevent future reuse.
  - 删除字段时未 [保留](https://protobuf.dev/programming-guides/proto3/#fieldreserved) 字段编号以防止未来的重复使用。


The field number is limited to 29 bits rather than 32 bits because three bits are used to specify the field’s wire format. For more on this, see the [Encoding topic](https://protobuf.dev/programming-guides/encoding#structure).

​	字段编号被限制为 29 位，而不是 32 位，因为有三位用于指定字段的线格式。有关更多信息，请参阅 [编码主题](https://protobuf.dev/programming-guides/encoding#structure)。



### 指定字段基数 Specifying Field Cardinality

Message fields can be one of the following:

​	消息字段可以是以下之一：

- *Singular*:

  In proto3, there are two types of singular fields: 在 proto3 中，单一字段有两种类型：

  - `optional`: (recommended) An `optional` field is in one of two possible states: **可选（optional）**（推荐）：`optional` 字段可以有以下两种状态：

    - the field is set, and contains a value that was explicitly set or parsed from the wire. It will be serialized to the wire.
      - 字段被设置，并包含一个显式设置或从线格式解析的值。它将被序列化到线格式中。

    - the field is unset, and will return the default value. It will not be serialized to the wire.
      - 字段未设置，并返回默认值。它不会被序列化到线格式中。


    You can check to see if the value was explicitly set.

    ​	您可以检查值是否被显式设置。

    `optional` is recommended over *implicit* fields for maximum compatibility with protobuf editions and proto2.

    ​	`optional` 相较于 *隐式* 字段，推荐用于最大限度地兼容 Protocol Buffers 版本和 proto2。

  - *implicit*: (not recommended) An implicit field has no explicit cardinality label and behaves as follows: **隐式（implicit）**（不推荐）：隐式字段没有显式的基数标签，其行为如下：

    - if the field is a message type, it behaves just like an `optional` field.
      - 如果字段是消息类型，其行为与 `optional` 字段完全相同。
    - if the field is not a message, it has two states:
      - 如果字段不是消息类型，其有以下两种状态：
      - the field is set to a non-default (non-zero) value that was explicitly set or parsed from the wire. It will be serialized to the wire.
        - 字段被设置为非默认值（非零值），且此值显式设置或从线格式中解析。它将被序列化到线格式中。
      - the field is set to the default (zero) value. It will not be serialized to the wire. In fact, you cannot determine whether the default (zero) value was set or parsed from the wire or not provided at all. For more on this subject, see [Field Presence](https://protobuf.dev/programming-guides/field_presence).
        - 字段被设置为默认值（零值）。它不会被序列化到线格式中。事实上，您无法确定默认值是被设置的、从线格式中解析的，还是根本未提供。有关更多信息，请参阅 [字段存在性](https://protobuf.dev/programming-guides/field_presence)。
    

- `repeated`: this field type can be repeated zero or more times in a well-formed message. The order of the repeated values will be preserved.

  - **重复（repeated）**：此字段类型可以在一个格式良好的消息中重复出现零次或多次。重复值的顺序将被保留。

- `map`: this is a paired key/value field type. See [Maps](https://protobuf.dev/programming-guides/encoding#maps) for more on this field type.

  - **映射（map）**：这是一个键/值配对字段类型。有关此字段类型的更多信息，请参阅 [映射](https://protobuf.dev/programming-guides/encoding#maps)。


#### 重复字段默认打包 Repeated Fields are Packed by Default

In proto3, `repeated` fields of scalar numeric types use `packed` encoding by default.

​	在 proto3 中，标量数值类型的 `repeated` 字段默认使用 `packed` 编码。

You can find out more about `packed` encoding in [Protocol Buffer Encoding](https://protobuf.dev/programming-guides/encoding#packed).

​	有关 `packed` 编码的更多信息，请参阅 [Protocol Buffer 编码](https://protobuf.dev/programming-guides/encoding#packed)。

#### 消息类型字段始终具有字段存在性 Message Type Fields Always Have Field Presence

In proto3, message-type fields already have field presence. Because of this, adding the `optional` modifier doesn’t change the field presence for the field.

​	在 proto3 中，消息类型字段已经具有字段存在性。因此，为字段添加 `optional` 修饰符不会改变字段的存在性。

The definitions for `Message2` and `Message3` in the following code sample generate the same code for all languages, and there is no difference in representation in binary, JSON, and TextFormat:

​	以下代码示例中 `Message2` 和 `Message3` 的定义在所有语言中生成的代码相同，在二进制、JSON 和文本格式中的表示没有区别：

```proto
syntax="proto3";

package foo.bar;

message Message1 {}

message Message2 {
  Message1 foo = 1;
}

message Message3 {
  optional Message1 bar = 1;
}
```

#### 格式良好的消息 Well-formed Messages

The term “well-formed,” when applied to protobuf messages, refers to the bytes serialized/deserialized. The protoc parser validates that a given proto definition file is parseable.

​	“格式良好”一词应用于 Protocol Buffers 消息时，是指序列化/反序列化的字节。`protoc` 解析器验证给定的 `.proto` 定义文件是否可解析。

Singular fields can appear more than once in wire-format bytes. The parser will accept the input, but only the last instance of that field will be accessible through the generated bindings. See [Last One Wins](https://protobuf.dev/programming-guides/encoding#last-one-wins) for more on this topic.

​	单一字段可以在线格式字节中多次出现。解析器会接受输入，但仅最后一个字段实例通过生成的绑定访问。有关更多信息，请参阅 [最后一个获胜](https://protobuf.dev/programming-guides/encoding#last-one-wins)。

### 添加更多消息类型 Adding More Message Types

Multiple message types can be defined in a single `.proto` file. This is useful if you are defining multiple related messages – so, for example, if you wanted to define the reply message format that corresponds to your `SearchResponse` message type, you could add it to the same `.proto`:

​	可以在一个 `.proto` 文件中定义多个消息类型。这在定义多个相关消息时很有用。例如，如果您想定义一个与 `SearchResponse` 消息类型对应的回复消息格式，可以将其添加到同一个 `.proto` 文件中：

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

​	**合并消息会导致膨胀**，虽然可以在一个 `.proto` 文件中定义多个消息类型（如消息、枚举和服务），但当单个文件中定义了大量具有不同依赖项的消息时，这可能导致依赖项膨胀。建议每个 `.proto` 文件包含尽可能少的消息类型。

### 添加注释 Adding Comments

To add comments to your `.proto` files:

​	要为 `.proto` 文件添加注释：

- Prefer C/C++/Java line-end-style comments ‘//’ on the line before the .proto code element
  - 优先使用 C/C++/Java 样式的行尾注释 `//`，将其放在 .proto 代码元素的上一行。

- C-style inline/multi-line comments `/* ... */` are also accepted.
  - 也可以使用 C 风格的内联/多行注释 `/* ... */`。
  - When using multi-line comments, a margin line of ‘*’ is preferred.
    - 使用多行注释时，建议添加以 `*` 为边界的格式。


```proto
/**
 * SearchRequest represents a search query, with pagination options to
 * indicate which results to include in the response.
 */
 /**
 * SearchRequest 表示一个搜索查询，
 * 包含分页选项以指明响应中要包含的结果。
 */
message SearchRequest {
  string query = 1;

  // Which page number do we want?
  // 请求的页码。
  int32 page_number = 2;

  // Number of results to return per page.
  // 每页返回的结果数量。
  int32 results_per_page = 3;
}
```

### 删除字段 Deleting Fields

Deleting fields can cause serious problems if not done properly.

​	如果未正确操作，删除字段可能导致严重问题。

When you no longer need a field and all references have been deleted from client code, you may delete the field definition from the message. However, you **must** [reserve the deleted field number](https://protobuf.dev/programming-guides/proto3/#fieldreserved). If you do not reserve the field number, it is possible for a developer to reuse that number in the future.

​	当您不再需要某个字段，并且所有引用都已从客户端代码中删除时，您可以从消息中删除该字段定义。但是，您 **必须** [保留被删除的字段编号](https://protobuf.dev/programming-guides/proto3/#fieldreserved)。如果不保留字段编号，则未来的开发者可能会重复使用该编号。

You should also reserve the field name to allow JSON and TextFormat encodings of your message to continue to parse.

​	您还应保留字段名称，以允许 JSON 和 TextFormat 编码的消息继续解析。



#### 保留字段编号 Reserved Field Numbers

If you [update](https://protobuf.dev/programming-guides/proto3/#updating) a message type by entirely deleting a field, or commenting it out, future developers can reuse the field number when making their own updates to the type. This can cause severe issues, as described in [Consequences of Reusing Field Numbers](https://protobuf.dev/programming-guides/proto3/#consequences). To make sure this doesn’t happen, add your deleted field number to the `reserved` list.

​	如果您通过完全删除字段或将其注释掉来 [更新](https://protobuf.dev/programming-guides/proto3/#updating) 消息类型，未来的开发者可能会在更新类型时重复使用字段编号。这可能导致严重问题，如 [重复使用字段编号的后果](https://protobuf.dev/programming-guides/proto3/#consequences) 中所述。为确保不会发生这种情况，请将已删除的字段编号添加到 `reserved` 列表中。

The protoc compiler will generate error messages if any future developers try to use these reserved field numbers.

​	`protoc` 编译器会在未来的开发者尝试使用这些保留字段编号时生成错误消息。

```proto
message Foo {
  reserved 2, 15, 9 to 11;
}
```

Reserved field number ranges are inclusive (`9 to 11` is the same as `9, 10, 11`).

​	保留字段编号范围是 **闭区间**（`9 to 11` 等同于 `9, 10, 11`）。

#### 保留字段名称 Reserved Field Names

Reusing an old field name later is generally safe, except when using TextProto or JSON encodings where the field name is serialized. To avoid this risk, you can add the deleted field name to the `reserved` list.

​	在以后重新使用旧字段名称通常是安全的，除了在使用 TextProto 或 JSON 编码时，其中字段名称会被序列化。为避免此风险，您可以将已删除的字段名称添加到 `reserved` 列表中。

Reserved names affect only the protoc compiler behavior and not runtime behavior, with one exception: TextProto implementations may discard unknown fields (without raising an error like with other unknown fields) with reserved names at parse time (only the C++ and Go implementations do so today). Runtime JSON parsing is not affected by reserved names.

​	保留名称只影响 `protoc` 编译器行为，不影响运行时行为，唯一的例外是：TextProto 实现可能在解析时丢弃具有保留名称的未知字段（不会像其他未知字段那样报错，仅适用于 C++ 和 Go 实现）。运行时 JSON 解析不受保留名称影响。

```proto
message Foo {
  reserved 2, 15, 9 to 11;
  reserved "foo", "bar";
}
```

Note that you can’t mix field names and field numbers in the same `reserved` statement.

​	请注意，您不能在同一个 `reserved` 声明中混合字段名称和字段编号。

### 从 `.proto` 文件中生成的内容是什么？ What’s Generated from Your `.proto`?

When you run the [protocol buffer compiler](https://protobuf.dev/programming-guides/proto3/#generating) on a `.proto`, the compiler generates the code in your chosen language you’ll need to work with the message types you’ve described in the file, including getting and setting field values, serializing your messages to an output stream, and parsing your messages from an input stream.

​	当您对 `.proto` 文件运行 [Protocol Buffers 编译器](https://protobuf.dev/programming-guides/proto3/#generating) 时，编译器会根据文件中描述的消息类型生成所需的代码，以便在所选语言中处理这些消息类型，包括获取和设置字段值、将消息序列化到输出流以及从输入流解析消息。

- For **C++**, the compiler generates a `.h` and `.cc` file from each `.proto`, with a class for each message type described in your file.
  - 对于 **C++**，编译器从每个 `.proto` 文件生成 `.h` 和 `.cc` 文件，每个消息类型生成一个类。

- For **Java**, the compiler generates a `.java` file with a class for each message type, as well as a special `Builder` class for creating message class instances.
  - 对于 **Java**，编译器生成 `.java` 文件，每个消息类型生成一个类，并包含一个用于创建消息类实例的特殊 `Builder` 类。

- For **Kotlin**, in addition to the Java generated code, the compiler generates a `.kt` file for each message type with an improved Kotlin API. This includes a DSL that simplifies creating message instances, a nullable field accessor, and a copy function.
  - 对于 **Kotlin**，除了生成的 Java 代码外，编译器还为每个消息类型生成一个 `.kt` 文件，提供改进的 Kotlin API。这包括一个简化创建消息实例的 DSL、一个可空字段访问器以及一个复制函数。

- **Python** is a little different — the Python compiler generates a module with a static descriptor of each message type in your `.proto`, which is then used with a *metaclass* to create the necessary Python data access class at runtime.
  - 对于 **Python**，略有不同——Python 编译器生成一个模块，其中包含每个 `.proto` 的静态描述符，并使用 *元类* 在运行时创建所需的 Python 数据访问类。

- For **Go**, the compiler generates a `.pb.go` file with a type for each message type in your file.
  - 对于 **Go**，编译器生成一个 `.pb.go` 文件，每个消息类型生成一个类型。

- For **Ruby**, the compiler generates a `.rb` file with a Ruby module containing your message types.
  - 对于 **Ruby**，编译器生成一个 `.rb` 文件，包含一个 Ruby 模块及其消息类型。

- For **Objective-C**, the compiler generates a `pbobjc.h` and `pbobjc.m` file from each `.proto`, with a class for each message type described in your file.
  - 对于 **Objective-C**，编译器从每个 `.proto` 文件生成 `pbobjc.h` 和 `pbobjc.m` 文件，每个消息类型生成一个类。

- For **C#**, the compiler generates a `.cs` file from each `.proto`, with a class for each message type described in your file.
  - 对于 **C#**，编译器从每个 `.proto` 文件生成 `.cs` 文件，每个消息类型生成一个类。

- For **PHP**, the compiler generates a `.php` message file for each message type described in your file, and a `.php` metadata file for each `.proto` file you compile. The metadata file is used to load the valid message types into the descriptor pool.
  - 对于 **PHP**，编译器为每个 `.proto` 文件中描述的每个消息类型生成 `.php` 消息文件，并为每个 `.proto` 文件生成 `.php` 元数据文件。元数据文件用于将有效的消息类型加载到描述符池中。

- For **Dart**, the compiler generates a `.pb.dart` file with a class for each message type in your file.
  - 对于 **Dart**，编译器生成 `.pb.dart` 文件，每个消息类型生成一个类。


You can find out more about using the APIs for each language by following the tutorial for your chosen language. For even more API details, see the relevant [API reference](https://protobuf.dev/reference/).

​	您可以通过选择的语言教程了解更多关于 API 的使用信息。有关更多 API 详细信息，请参阅相关的 [API 参考](https://protobuf.dev/reference/)。

### 标量值类型 Scalar Value Types

A scalar message field can have one of the following types – the table shows the type specified in the `.proto` file, and the corresponding type in the automatically generated class:

​	标量消息字段可以具有以下类型之一。下表展示了 `.proto` 文件中指定的类型，以及自动生成类中对应的类型：

| .proto Type | Notes                                                        | C++ Type | Java/Kotlin Type[1] | Python Type[3]                  | Go Type | Ruby Type                      | C# Type    | PHP Type          | Dart Type | Rust Type   |
| ----------- | ------------------------------------------------------------ | -------- | ------------------- | ------------------------------- | ------- | ------------------------------ | ---------- | ----------------- | --------- | ----------- |
| double      |                                                              | double   | double              | float                           | float64 | Float                          | double     | float             | double    | f64         |
| float       |                                                              | float    | float               | float                           | float32 | Float                          | float      | float             | double    | f32         |
| int32       | Uses variable-length encoding. Inefficient for encoding negative numbers – if your field is likely to have negative values, use sint32 instead.使用变长编码。对负数编码效率较低——如果字段可能包含负数，建议使用 `sint32`。 | int32    | int                 | int                             | int32   | Fixnum or Bignum (as required) | int        | integer           | int       | i32         |
| int64       | Uses variable-length encoding. Inefficient for encoding negative numbers – if your field is likely to have negative values, use sint64 instead.使用变长编码。对负数编码效率较低——如果字段可能包含负数，建议使用 `sint64`。 | int64    | long                | int/long[4]                     | int64   | Bignum                         | long       | integer/string[6] | Int64     | i64         |
| uint32      | Uses variable-length encoding.使用变长编码。                 | uint32   | int[2]              | int/long[4]                     | uint32  | Fixnum or Bignum (as required) | uint       | integer           | int       | u32         |
| uint64      | Uses variable-length encoding.使用变长编码。                 | uint64   | long[2]             | int/long[4]                     | uint64  | Bignum                         | ulong      | integer/string[6] | Int64     | u64         |
| sint32      | Uses variable-length encoding. Signed int value. These more efficiently encode negative numbers than regular int32s.使用变长编码。有符号整型值。比普通的 `int32` 更高效地编码负数。 | int32    | int                 | int                             | int32   | Fixnum or Bignum (as required) | int        | integer           | int       | i32         |
| sint64      | Uses variable-length encoding. Signed int value. These more efficiently encode negative numbers than regular int64s.使用变长编码。有符号整型值。比普通的 `int64` 更高效地编码负数。 | int64    | long                | int/long[4]                     | int64   | Bignum                         | long       | integer/string[6] | Int64     | i64         |
| fixed32     | Always four bytes. More efficient than uint32 if values are often greater than 228.始终占用四字节。如果值通常大于 228，比 `uint32` 更高效。 | uint32   | int[2]              | int/long[4]                     | uint32  | Fixnum or Bignum (as required) | uint       | integer           | int       | u32         |
| fixed64     | Always eight bytes. More efficient than uint64 if values are often greater than 256.始终占用八字节。如果值通常大于 256，比 `uint64` 更高效。 | uint64   | long[2]             | int/long[4]                     | uint64  | Bignum                         | ulong      | integer/string[6] | Int64     | u64         |
| sfixed32    | Always four bytes.始终占用四字节。                           | int32    | int                 | int                             | int32   | Fixnum or Bignum (as required) | int        | integer           | int       | i32         |
| sfixed64    | Always eight bytes.始终占用八字节。                          | int64    | long                | int/long[4]                     | int64   | Bignum                         | long       | integer/string[6] | Int64     | i64         |
| bool        |                                                              | bool     | boolean             | bool                            | bool    | TrueClass/FalseClass           | bool       | boolean           | bool      | bool        |
| string      | A string must always contain UTF-8 encoded or 7-bit ASCII text, and cannot be longer than 232.字符串必须始终包含 UTF-8 编码或 7 位 ASCII 文本，且长度不得超过 2^32。 | string   | String              | str/unicode[5]                  | string  | String (UTF-8)                 | string     | string            | String    | ProtoString |
| bytes       | May contain any arbitrary sequence of bytes no longer than 232.可以包含长度不超过 2^32 的任意字节序列。 | string   | ByteString          | str (Python 2) bytes (Python 3) | []byte  | String (ASCII-8BIT)            | ByteString | string            | List      | ProtoBytes  |

[1] Kotlin uses the corresponding types from Java, even for unsigned types, to ensure compatibility in mixed Java/Kotlin codebases.

​	[1] Kotlin 使用与 Java 相对应的类型，即使是无符号类型，以确保在 Java/Kotlin 混合代码库中的兼容性。

[2] In Java, unsigned 32-bit and 64-bit integers are represented using their signed counterparts, with the top bit simply being stored in the sign bit.

​	[2] 在 Java 中，无符号 32 位和 64 位整数使用其对应的有符号类型表示，最高位存储在符号位中。

[3] In all cases, setting values to a field will perform type checking to make sure it is valid.

​	[3] 在所有情况下，为字段设置值时都会进行类型检查以确保其有效性。

[4] 64-bit or unsigned 32-bit integers are always represented as long when decoded, but can be an int if an int is given when setting the field. In all cases, the value must fit in the type represented when set. See [2].

​	[4] 解码时，64 位或无符号 32 位整数始终表示为 `long`，但如果设置字段时给定的值是 `int`，也可以是 `int`。无论如何，值必须适合设置时表示的类型。

[5] Python strings are represented as unicode on decode but can be str if an ASCII string is given (this is subject to change).

​	[5] Python 字符串在解码时表示为 `unicode`，但如果提供了 ASCII 字符串，也可以是 `str`（可能会发生变更）。

[6] Integer is used on 64-bit machines and string is used on 32-bit machines.

​	[6] 在 64 位机器上使用 `integer`，在 32 位机器上使用 `string`。

You can find out more about how these types are encoded when you serialize your message in [Protocol Buffer Encoding](https://protobuf.dev/programming-guides/encoding).

​	您可以在 [Protocol Buffer Encoding](https://protobuf.dev/programming-guides/encoding) 中了解更多关于这些类型序列化时的编码方式。

## 字段默认值 Default Field Values

When a message is parsed, if the encoded message bytes do not contain a particular field, accessing that field in the parsed object returns the default value for that field. The default values are type-specific:

​	当解析消息时，如果编码的消息字节不包含某字段，则访问该字段时会返回其默认值。默认值是类型特定的：

- For strings, the default value is the empty string.
  - 对于字符串，默认值为空字符串。

- For bytes, the default value is empty bytes.
  - 对于字节，默认值为空字节。

- For bools, the default value is false.
  - 对于布尔值，默认值为 `false`。

- For numeric types, the default value is zero.
  - 对于数值类型，默认值为 `0`。

- For message fields, the field is not set. Its exact value is language-dependent. See the [generated code guide](https://protobuf.dev/reference/) for details.
  - 对于消息字段，该字段未设置，其确切值依赖于具体语言。详细信息请参阅 [生成代码指南](https://protobuf.dev/reference/)。

- For enums, the default value is the **first defined enum value**, which must be 0. See [Enum Default Value](https://protobuf.dev/programming-guides/proto3/#enum-default).
  - 对于枚举，默认值为第一个定义的枚举值，且必须为 `0`。详见 [Enum Default Value](https://protobuf.dev/programming-guides/proto3/#enum-default)。


The default value for repeated fields is empty (generally an empty list in the appropriate language).

​	对于重复字段，默认值为空（通常为适当语言中的空列表）。

The default value for map fields is empty (generally an empty map in the appropriate language).

​	对于映射字段，默认值为空（通常为适当语言中的空映射）。

Note that for implicit-presence scalar fields, once a message is parsed there’s no way of telling whether that field was explicitly set to the default value (for example whether a boolean was set to `false`) or just not set at all: you should bear this in mind when defining your message types. For example, don’t have a boolean that switches on some behavior when set to `false` if you don’t want that behavior to also happen by default. Also note that if a scalar message field **is** set to its default, the value will not be serialized on the wire. If a float or double value is set to +0 it will not be serialized, but -0 is considered distinct and will be serialized.

​	注意，对于隐式存在的标量字段（implicit-presence scalar fields），一旦消息被解析，就无法判断该字段是被明确设置为默认值（例如布尔值被设置为 `false`）还是根本没有设置：在定义消息类型时应牢记这一点。例如，如果不希望某种行为默认发生，就不要使用布尔值 `false` 来切换行为。此外，请注意，如果标量消息字段被设置为默认值，该值将不会在序列化中写入。若浮点数或双精度浮点数值被设置为 `+0`，它将不会被序列化，但 `-0` 被视为不同值，会被序列化。

See the [generated code guide](https://protobuf.dev/reference/) for your chosen language for more details about how defaults work in generated code.

​	有关生成代码中默认值如何工作的更多详情，请参阅您选择语言的 [生成代码指南](https://protobuf.dev/reference/)。

## 枚举 Enumerations

When you’re defining a message type, you might want one of its fields to only have one of a predefined list of values. For example, let’s say you want to add a `corpus` field for each `SearchRequest`, where the corpus can be `UNIVERSAL`, `WEB`, `IMAGES`, `LOCAL`, `NEWS`, `PRODUCTS` or `VIDEO`. You can do this very simply by adding an `enum` to your message definition with a constant for each possible value.

​	定义消息类型时，您可能希望某些字段仅包含预定义值列表中的一个。例如，假设您希望为每个 `SearchRequest` 添加一个 `corpus` 字段，其中 corpus 可以是 `UNIVERSAL`、`WEB`、`IMAGES`、`LOCAL`、`NEWS`、`PRODUCTS` 或 `VIDEO`。您可以通过在消息定义中添加一个 `enum` 并为每个可能值定义一个常量来轻松实现这一点。

In the following example we’ve added an `enum` called `Corpus` with all the possible values, and a field of type `Corpus`:

​	以下示例中，我们添加了一个名为 `Corpus` 的 `enum`，包含所有可能的值，并定义了一个类型为 `Corpus` 的字段：

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

​	`SearchRequest.corpus` 字段的默认值为 `CORPUS_UNSPECIFIED`，因为这是枚举中定义的第一个值。

In proto3, the first value defined in an enum definition **must** have the value zero and should have the name `ENUM_TYPE_NAME_UNSPECIFIED` or `ENUM_TYPE_NAME_UNKNOWN`. This is because:

​	在 proto3 中，枚举定义的第一个值**必须**为零，并且应命名为 `ENUM_TYPE_NAME_UNSPECIFIED` 或 `ENUM_TYPE_NAME_UNKNOWN`。这是因为：

- There must be a zero value, so that we can use 0 as a numeric [default value](https://protobuf.dev/programming-guides/proto3/#default).
  - 必须有一个零值，以便我们可以使用 `0` 作为数值 [默认值](https://protobuf.dev/programming-guides/proto3/#default)。

- The zero value needs to be the first element, for compatibility with the [proto2](https://protobuf.dev/programming-guides/proto2) semantics where the first enum value is the default unless a different value is explicitly specified.
  - 零值需要是第一个元素，以兼容 [proto2](https://protobuf.dev/programming-guides/proto2) 的语义，其中第一个枚举值是默认值，除非显式指定了其他值。


It is also recommended that this first, default value have no semantic meaning other than “this value was unspecified”.

​	还建议将第一个默认值定义为无语义意义，仅表示“此值未指定”。

### 枚举值别名 Enum Value Aliases

You can define aliases by assigning the same value to different enum constants. To do this you need to set the `allow_alias` option to `true`. Otherwise, the protocol buffer compiler generates a warning message when aliases are found. Though all alias values are valid during deserialization, the first value is always used when serializing.

​	您可以通过为不同的枚举常量分配相同的值来定义别名。为此，需要将 `allow_alias` 选项设置为 `true`。否则，若发现别名，协议缓冲区编译器会生成警告信息。尽管反序列化期间所有别名值都是有效的，但序列化时始终使用第一个值。

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

Enumerator constants must be in the range of a 32-bit integer. Since `enum` values use [varint encoding](https://protobuf.dev/programming-guides/encoding) on the wire, negative values are inefficient and thus not recommended. You can define `enum`s within a message definition, as in the earlier example, or outside – these `enum`s can be reused in any message definition in your `.proto` file. You can also use an `enum` type declared in one message as the type of a field in a different message, using the syntax `_MessageType_._EnumType_`.

​	枚举器常量必须在 32 位整数范围内。由于枚举值在传输中使用 [varint 编码](https://protobuf.dev/programming-guides/encoding)，负值效率较低，因此不推荐使用负值。您可以在消息定义中定义 `enum`，如上例所示，也可以在外部定义——这些 `enum` 可以在 `.proto` 文件的任何消息定义中重用。您还可以在一个消息中声明的 `enum` 类型，用于另一个消息的字段，使用 `_MessageType_._EnumType_` 语法。

When you run the protocol buffer compiler on a `.proto` that uses an `enum`, the generated code will have a corresponding `enum` for Java, Kotlin, or C++, or a special `EnumDescriptor` class for Python that’s used to create a set of symbolic constants with integer values in the runtime-generated class.

​	运行协议缓冲区编译器处理使用 `enum` 的 `.proto` 文件时，生成的代码将为 Java、Kotlin 或 C++ 创建对应的 `enum`，或为 Python 创建一个特殊的 `EnumDescriptor` 类，用于在运行时生成的类中创建一组具有整数值的符号常量。

> Important
>
> The generated code may be subject to language-specific limitations on the number of enumerators (low thousands for one language). Review the limitations for the languages you plan to use.
>
> ​	生成的代码可能受特定语言对枚举数数量的限制（某些语言限制在数千以内）。请查看您计划使用的语言的限制。

During deserialization, unrecognized enum values will be preserved in the message, though how this is represented when the message is deserialized is language-dependent. In languages that support open enum types with values outside the range of specified symbols, such as C++ and Go, the unknown enum value is simply stored as its underlying integer representation. In languages with closed enum types such as Java, a case in the enum is used to represent an unrecognized value, and the underlying integer can be accessed with special accessors. In either case, if the message is serialized the unrecognized value will still be serialized with the message.

​	反序列化期间，未识别的枚举值会保留在消息中，但在消息被反序列化时的表示方式取决于语言。在支持开放枚举类型（值可以超出指定符号范围）的语言中（例如 C++ 和 Go），未知的枚举值以其基础整数表示存储。在具有封闭枚举类型的语言中（例如 Java），使用枚举中的某个值表示未识别值，可以通过特殊访问器获取基础整数值。无论哪种情况，若消息被序列化，未识别的值仍会随消息序列化。

> Important
>
> For information on how enums should work contrasted with how they currently work in different languages, see [Enum Behavior](https://protobuf.dev/programming-guides/enum).
>
> ​	关于枚举应如何工作及其在不同语言中的当前行为差异，请参阅 [枚举行为](https://protobuf.dev/programming-guides/enum)。

For more information about how to work with message `enum`s in your applications, see the [generated code guide](https://protobuf.dev/reference/) for your chosen language.

​	有关如何在应用程序中处理消息 `enum` 的更多信息，请参阅 [生成代码指南](https://protobuf.dev/reference/) 中与您选择语言相关的部分。

### 保留值 Reserved Values

If you [update](https://protobuf.dev/programming-guides/proto3/#updating) an enum type by entirely removing an enum entry, or commenting it out, future users can reuse the numeric value when making their own updates to the type. This can cause severe issues if they later load old instances of the same `.proto`, including data corruption, privacy bugs, and so on. One way to make sure this doesn’t happen is to specify that the numeric values (and/or names, which can also cause issues for JSON serialization) of your deleted entries are `reserved`. The protocol buffer compiler will complain if any future users try to use these identifiers. You can specify that your reserved numeric value range goes up to the maximum possible value using the `max` keyword.

​	如果您通过完全删除或注释掉枚举条目来 [更新](https://protobuf.dev/programming-guides/proto3/#updating) 枚举类型，则未来用户可能会在更新类型时重用数值。这可能导致严重问题，例如加载相同 `.proto` 的旧实例时出现数据损坏、隐私错误等。为确保不会发生这种情况，可以指定已删除条目的数值（和/或名称，这可能对 JSON 序列化产生问题）为 `reserved`。协议缓冲区编译器将警告任何未来用户若尝试使用这些标识符。您可以使用 `max` 关键字指定保留的数值范围一直到最大值。

```proto
enum Foo {
  reserved 2, 15, 9 to 11, 40 to max;
  reserved "FOO", "BAR";
}
```

Note that you can’t mix field names and numeric values in the same `reserved` statement.

​	注意，不能在同一条 `reserved` 声明中混合字段名称和数值。

## 使用其他消息类型 Using Other Message Types

You can use other message types as field types. For example, let’s say you wanted to include `Result` messages in each `SearchResponse` message – to do this, you can define a `Result` message type in the same `.proto` and then specify a field of type `Result` in `SearchResponse`:

​	您可以将其他消息类型用作字段类型。例如，假设您希望在每个 `SearchResponse` 消息中包含 `Result` 消息类型——为此，您可以在同一个 `.proto` 文件中定义 `Result` 消息类型，然后在 `SearchResponse` 中指定一个类型为 `Result` 的字段：

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

​	在上面的示例中，`Result` 消息类型与 `SearchResponse` 定义在同一个文件中——如果您想使用的消息类型已经定义在另一个 `.proto` 文件中，该怎么办？

You can use definitions from other `.proto` files by *importing* them. To import another `.proto`’s definitions, you add an import statement to the top of your file:

​	您可以通过 *import* 导入其他 `.proto` 文件中的定义。要导入另一个 `.proto` 文件中的定义，请在文件顶部添加一条 import 语句：

```proto
import "myproject/other_protos.proto";
```

By default, you can use definitions only from directly imported `.proto` files. However, sometimes you may need to move a `.proto` file to a new location. Instead of moving the `.proto` file directly and updating all the call sites in a single change, you can put a placeholder `.proto` file in the old location to forward all the imports to the new location using the `import public` notion.

​	默认情况下，您只能使用直接导入的 `.proto` 文件中的定义。不过，有时您可能需要将 `.proto` 文件移动到新位置，而无需直接移动文件并更新所有引用点。在这种情况下，您可以在旧位置放置一个占位 `.proto` 文件，通过 `import public` 语法将所有导入转发到新位置。

**Note that the public import functionality is not available in Java, Kotlin, TypeScript, JavaScript, GCL, as well as C++ targets that use protobuf static reflection.**

​	**注意，`import public` 功能在以下目标中不可用：Java、Kotlin、TypeScript、JavaScript、GCL，以及使用 Protobuf 静态反射的 C++。**

`import public` dependencies can be transitively relied upon by any code importing the proto containing the `import public` statement. For example:

​	通过 `import public` 声明的依赖项可以被任何导入包含该声明的 proto 文件的代码传递性地使用。例如：

```proto
// new.proto
// All definitions are moved here
// 所有定义已移动到此处
// old.proto
// This is the proto that all clients are importing.
// 所有客户端正在导入此 proto。
import public "new.proto";
import "other.proto";
// client.proto
import "old.proto";
// You use definitions from old.proto and new.proto, but not other.proto
// 您可以使用 old.proto 和 new.proto 的定义，但不能使用 other.proto 的定义
```

The protocol compiler searches for imported files in a set of directories specified on the protocol compiler command line using the `-I`/`--proto_path` flag. If no flag was given, it looks in the directory in which the compiler was invoked. In general you should set the `--proto_path` flag to the root of your project and use fully qualified names for all imports.

​	协议缓冲区编译器会在命令行中通过 `-I` 或 `--proto_path` 标志指定的目录中搜索导入文件。如果未提供标志，则它会在调用编译器的目录中查找。通常，您应该将 `--proto_path` 标志设置为项目的根目录，并为所有导入使用完全限定名。

### 使用 proto2 消息类型 Using proto2 Message Types

It’s possible to import [proto2](https://protobuf.dev/programming-guides/proto2) message types and use them in your proto3 messages, and vice versa. However, proto2 enums cannot be used directly in proto3 syntax (it’s okay if an imported proto2 message uses them).

​	可以导入 [proto2](https://protobuf.dev/programming-guides/proto2) 消息类型并在 proto3 消息中使用，反之亦然。不过，proto2 的枚举类型不能直接在 proto3 语法中使用（如果导入的 proto2 消息使用了它们则没问题）。

## 嵌套类型 Nested Types

You can define and use message types inside other message types, as in the following example – here the `Result` message is defined inside the `SearchResponse` message:

​	您可以在其他消息类型内定义并使用消息类型。例如，以下示例中，`Result` 消息定义在 `SearchResponse` 消息内：

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

​	如果您希望在父消息类型之外重用此消息类型，可以通过 `_Parent_._Type_` 的方式引用它：

```proto
message SomeOtherMessage {
  SearchResponse.Result result = 1;
}
```

You can nest messages as deeply as you like. In the example below, note that the two nested types named `Inner` are entirely independent, since they are defined within different messages:

​	您可以按任意深度嵌套消息类型。以下示例中，注意两个名为 `Inner` 的嵌套类型是完全独立的，因为它们定义在不同的消息中：

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

​	如果现有的消息类型不再满足您的需求——例如，您希望消息格式包含一个额外字段——但仍希望使用旧格式创建的代码，不用担心！在使用二进制线格式时，更新消息类型非常简单且不会破坏现有代码。

> Note
>
> If you use JSON or [proto text format](https://protobuf.dev/reference/protobuf/textformat-spec) to store your protocol buffer messages, the changes that you can make in your proto definition are different.
>
> ​	如果您使用 JSON 或 [proto 文本格式](https://protobuf.dev/reference/protobuf/textformat-spec) 存储协议缓冲区消息，则您在 proto 定义中可以进行的更改有所不同。

Check [Proto Best Practices](https://protobuf.dev/programming-guides/dos-donts) and the following rules:

​	检查 [Proto 最佳实践](https://protobuf.dev/programming-guides/dos-donts) 并遵循以下规则：

- Don’t change the field numbers for any existing fields. “Changing” the field number is equivalent to deleting the field and adding a new field with the same type. If you want to renumber a field, see the instructions for [deleting a field](https://protobuf.dev/programming-guides/proto3/#deleting).
  - **不要更改任何现有字段的字段编号**。“更改”字段编号等同于删除该字段并添加一个具有相同类型的新字段。如果需要重新编号字段，请参阅 [删除字段](https://protobuf.dev/programming-guides/proto3/#deleting) 的说明。

- If you add new fields, any messages serialized by code using your “old” message format can still be parsed by your new generated code. You should keep in mind the [default values](https://protobuf.dev/programming-guides/proto3/#default) for these elements so that new code can properly interact with messages generated by old code. Similarly, messages created by your new code can be parsed by your old code: old binaries simply ignore the new field when parsing. See the [Unknown Fields](https://protobuf.dev/programming-guides/proto3/#unknowns) section for details.
  - **如果添加新字段**，使用“旧”消息格式的代码序列化的任何消息仍可以被新的生成代码解析。需要注意这些字段的 [默认值](https://protobuf.dev/programming-guides/proto3/#default)，以便新代码能够正确与旧代码生成的消息交互。同样，由新代码创建的消息可以被旧代码解析：旧的二进制文件在解析时只会忽略新字段。有关详细信息，请参阅 [未知字段](https://protobuf.dev/programming-guides/proto3/#unknowns) 部分。

- Fields can be removed, as long as the field number is not used again in your updated message type. You may want to rename the field instead, perhaps adding the prefix “OBSOLETE_”, or make the field number [reserved](https://protobuf.dev/programming-guides/proto3/#fieldreserved), so that future users of your `.proto` can’t accidentally reuse the number.
  - **可以移除字段**，只要字段编号未在更新的消息类型中再次使用。可以考虑重命名该字段（例如添加前缀“OBSOLETE_”），或者将字段编号 [保留](https://protobuf.dev/programming-guides/proto3/#fieldreserved)，以防止 `.proto` 的未来用户意外重复使用该编号。

- `int32`, `uint32`, `int64`, `uint64`, and `bool` are all compatible – this means you can change a field from one of these types to another without breaking forwards- or backwards-compatibility. If a number is parsed from the wire which doesn’t fit in the corresponding type, you will get the same effect as if you had cast the number to that type in C++ (for example, if a 64-bit number is read as an int32, it will be truncated to 32 bits).
  - **`int32`、`uint32`、`int64`、`uint64` 和 `bool` 是兼容的**——这意味着可以在这些类型之间更改字段，而不会破坏向前或向后兼容性。如果从 wire 解析的数字与对应的类型不匹配，将产生与在 C++ 中将数字强制转换为该类型相同的效果（例如，如果读取了 64 位数字作为 `int32`，则会截断为 32 位）。

- `sint32` and `sint64` are compatible with each other but are *not* compatible with the other integer types.
  - **`sint32` 和 `sint64` 彼此兼容**，但与其他整数类型不兼容。

- `string` and `bytes` are compatible as long as the bytes are valid UTF-8.
  - **`string` 和 `bytes` 是兼容的**，前提是字节是有效的 UTF-8。

- Embedded messages are compatible with `bytes` if the bytes contain an encoded instance of the message.
  - **嵌套消息与 `bytes` 兼容**，前提是字节包含嵌套消息的编码实例。

- `fixed32` is compatible with `sfixed32`, and `fixed64` with `sfixed64`.
  - **`fixed32` 与 `sfixed32` 兼容，`fixed64` 与 `sfixed64` 兼容**。

- For `string`, `bytes`, and message fields, singular is compatible with `repeated`. Given serialized data of a repeated field as input, clients that expect this field to be singular will take the last input value if it’s a primitive type field or merge all input elements if it’s a message type field. Note that this is **not** generally safe for numeric types, including bools and enums. Repeated fields of numeric types are serialized in the [packed](https://protobuf.dev/programming-guides/encoding#packed) format by default, which will not be parsed correctly when a singular field is expected.
  - 对于 **`string`、`bytes` 和消息字段**，单值字段与 `repeated` 字段是兼容的。如果将序列化的 `repeated` 字段数据作为输入，则期望单值字段的客户端会取最后一个输入值（对于基本类型字段），或合并所有输入元素（对于消息类型字段）。但请注意，这对于包括布尔值和枚举在内的数字类型通常不安全。数字类型的 `repeated` 字段默认以 [packed](https://protobuf.dev/programming-guides/encoding#packed) 格式序列化，当期望单值字段时无法正确解析。

- `enum` is compatible with `int32`, `uint32`, `int64`, and `uint64` in terms of wire format (note that values will be truncated if they don’t fit). However, be aware that client code may treat them differently when the message is deserialized: for example, unrecognized proto3 `enum` values will be preserved in the message, but how this is represented when the message is deserialized is language-dependent. Int fields always just preserve their value.
  - **`enum` 在 wire 格式上与 `int32`、`uint32`、`int64` 和 `uint64` 兼容**（请注意，如果值不匹配，它们将被截断）。不过，客户端代码在消息反序列化时可能会以不同方式处理它们。例如，未识别的 proto3 `enum` 值将保留在消息中，但反序列化时的表示方式取决于语言。整数字段始终只保留其值。

- Changing a single `optional` field or extension into a member of a **new** `oneof` is binary compatible, however for some languages (notably, Go) the generated code’s API will change in incompatible ways. For this reason, Google does not make such changes in its public APIs, as documented in [AIP-180](https://google.aip.dev/180#moving-into-oneofs). With the same caveat about source-compatibility, moving multiple fields into a new `oneof` may be safe if you are sure that no code sets more than one at a time. Moving fields into an existing `oneof` is not safe. Likewise, changing a single field `oneof` to an `optional` field or extension is safe.
  - **将单个 `optional` 字段或扩展更改为新的 `oneof` 成员是二进制兼容的**，但对于某些语言（尤其是 Go），生成代码的 API 会发生不兼容的变化。因此，Google 在其公共 API 中不做此类更改（详见 [AIP-180](https://google.aip.dev/180#moving-into-oneofs)）。在确保没有代码同时设置多个字段的情况下，可以安全地将多个字段移入新的 `oneof`。但将字段移入现有的 `oneof` 并不安全。同样，将单字段 `oneof` 更改为 `optional` 字段或扩展是安全的。

- Changing a field between a `map<K, V>` and the corresponding `repeated` message field is binary compatible (see [Maps](https://protobuf.dev/programming-guides/proto3/#maps), below, for the message layout and other restrictions). However, the safety of the change is application-dependent: when deserializing and reserializing a message, clients using the `repeated` field definition will produce a semantically identical result; however, clients using the `map` field definition may reorder entries and drop entries with duplicate keys.
  - **在 `map<K, V>` 和对应的 `repeated` 消息字段之间更改是二进制兼容的**（参见 [Maps](https://protobuf.dev/programming-guides/proto3/#maps) 了解消息布局和其他限制）。但这种更改的安全性取决于应用程序：在反序列化和重新序列化消息时，使用 `repeated` 字段定义的客户端会生成语义相同的结果；但使用 `map` 字段定义的客户端可能会重新排序条目并丢弃重复键的条目。


### 未知字段 Unknown Fields

Unknown fields are well-formed protocol buffer serialized data representing fields that the parser does not recognize. For example, when an old binary parses data sent by a new binary with new fields, those new fields become unknown fields in the old binary.

​	未知字段是指解析器无法识别的字段，它们是格式良好的 Protocol Buffers 序列化数据。例如，当一个旧的二进制程序解析由包含新字段的新二进制程序发送的数据时，这些新字段会成为旧二进制程序中的未知字段。

Proto3 messages preserve unknown fields and includes them during parsing and in the serialized output, which matches proto2 behavior.

​	Proto3 消息会保留未知字段，并在解析和序列化输出时包含它们，这与 proto2 的行为一致。

#### 保留未知字段 Retaining Unknown Fields

Some actions can cause unknown fields to be lost. For example, if you do one of the following, unknown fields are lost:

​	某些操作可能会导致未知字段丢失。例如，执行以下操作之一时，未知字段会丢失：

- Serialize a proto to JSON.
  - 将 proto 序列化为 JSON。

- Iterate over all of the fields in a message to populate a new message.
  - 遍历消息中的所有字段以填充新消息。


To avoid losing unknown fields, do the following:

​	为了避免丢失未知字段，请执行以下操作：

- Use binary; avoid using text formats for data exchange.
  - 使用二进制格式；避免在数据交换中使用文本格式。

- Use message-oriented APIs, such as `CopyFrom()` and `MergeFrom()`, to copy data rather than copying field-by-field
  - 使用面向消息的 API，例如 `CopyFrom()` 和 `MergeFrom()`，以复制数据而不是逐字段复制。


TextFormat is a bit of a special case. Serializing to TextFormat prints unknown fields using their field numbers. But parsing TextFormat data back into a binary proto fails if there are entries that use field numbers.

​	TextFormat 是一个特殊情况。序列化为 TextFormat 时，使用字段编号打印未知字段。但如果 TextFormat 数据中有使用字段编号的条目，解析回二进制 proto 会失败。

## Any

The `Any` message type lets you use messages as embedded types without having their .proto definition. An `Any` contains an arbitrary serialized message as `bytes`, along with a URL that acts as a globally unique identifier for and resolves to that message’s type. To use the `Any` type, you need to [import](https://protobuf.dev/programming-guides/proto3/#other) `google/protobuf/any.proto`.

​	`Any` 消息类型允许您在没有 `.proto` 定义的情况下使用嵌入类型的消息。`Any` 包含一个任意序列化的消息作为 `bytes`，以及一个 URL，作为该消息类型的全局唯一标识符并解析到该类型。要使用 `Any` 类型，您需要 [import](https://protobuf.dev/programming-guides/proto3/#other) `google/protobuf/any.proto`：

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

​	不同的语言实现将支持运行时库助手以类型安全的方式打包和解包 `Any` 值。例如，在 Java 中，`Any` 类型具有特殊的 `pack()` 和 `unpack()` 访问器，而在 C++ 中有 `PackFrom()` 和 `UnpackTo()` 方法：

```cpp
// Storing an arbitrary message type in Any.
// 将任意消息类型存储在 Any 中
NetworkErrorDetails details = ...;
ErrorStatus status;
status.add_details()->PackFrom(details);

// Reading an arbitrary message from Any.
// 从 Any 中读取任意消息
ErrorStatus status = ...;
for (const google::protobuf::Any& detail : status.details()) {
  if (detail.Is<NetworkErrorDetails>()) {
    NetworkErrorDetails network_error;
    detail.UnpackTo(&network_error);
    ... processing network_error ...
  }
}
```

## Oneof

If you have a message with many singular fields and where at most one field will be set at the same time, you can enforce this behavior and save memory by using the oneof feature.

​	如果一个消息中有许多单值字段，但同一时间最多只有一个字段会被设置，您可以使用 `oneof` 功能来强制执行此行为并节省内存。

Oneof fields are like optional fields except all the fields in a oneof share memory, and at most one field can be set at the same time. Setting any member of the oneof automatically clears all the other members. You can check which value in a oneof is set (if any) using a special `case()` or `WhichOneof()` method, depending on your chosen language.

​	`oneof` 字段类似于可选字段，但 `oneof` 中的所有字段共享内存，并且同一时间最多只有一个字段可以被设置。设置 `oneof` 的任一成员会自动清除 `oneof` 的其他成员。您可以使用一个特殊的 `case()` 或 `WhichOneof()` 方法检查 `oneof` 中设置的值（如果有）。

Note that if *multiple values are set, the last set value as determined by the order in the proto will overwrite all previous ones*.

​	注意，如果 **多个值被设置，则按 proto 中的顺序，最后设置的值会覆盖所有先前的值**。

Field numbers for oneof fields must be unique within the enclosing message.

​	`oneof` 字段的字段编号在包含的消息中必须是唯一的。

### Using Oneof

To define a oneof in your `.proto` you use the `oneof` keyword followed by your oneof name, in this case `test_oneof`:

​	要在 `.proto` 中定义一个 `oneof`，可以使用 `oneof` 关键字，后跟 `oneof` 名称，例如 `test_oneof`：

```proto
message SampleMessage {
  oneof test_oneof {
    string name = 4;
    SubMessage sub_message = 9;
  }
}
```

You then add your oneof fields to the oneof definition. You can add fields of any type, except `map` fields and `repeated` fields. If you need to add a repeated field to a oneof, you can use a message containing the repeated field.

​	然后将 `oneof` 字段添加到定义中。可以添加任何类型的字段，但不能是 `map` 字段和 `repeated` 字段。如果需要向 `oneof` 添加一个 `repeated` 字段，可以使用包含该 `repeated` 字段的消息。

In your generated code, oneof fields have the same getters and setters as regular fields. You also get a special method for checking which value (if any) in the oneof is set. You can find out more about the oneof API for your chosen language in the relevant [API reference](https://protobuf.dev/reference/).

​	在生成的代码中，`oneof` 字段具有与常规字段相同的 getter 和 setter。您还可以使用一个特殊方法检查 `oneof` 中设置的值（如果有）。关于所选语言的 `oneof` API，请参阅相关 [API 参考](https://protobuf.dev/reference/)。

### Oneof Features

- Setting a oneof field will automatically clear all other members of the oneof. So if you set several oneof fields, only the *last* field you set will still have a value. 设置一个 `oneof` 字段会自动清除 `oneof` 的所有其他成员。因此，如果设置多个 `oneof` 字段，只有**最后**设置的字段会有值。

  ```cpp
  SampleMessage message;
  message.set_name("name");
  CHECK_EQ(message.name(), "name");
  // Calling mutable_sub_message() will clear the name field and will set
  // sub_message to a new instance of SubMessage with none of its fields set.
  // 调用 mutable_sub_message() 会清除 name 字段并将 sub_message 设置为一个新的 SubMessage 实例，其中没有设置任何字段
  message.mutable_sub_message();
  CHECK(message.name().empty());
  ```

- If the parser encounters multiple members of the same oneof on the wire, only the last member seen is used in the parsed message.

  - 如果解析器在 wire 数据中遇到同一 `oneof` 的多个成员，只有最后看到的成员会被解析到消息中。

- A oneof cannot be `repeated`.

  - `oneof` 不能是 `repeated`。

- Reflection APIs work for oneof fields.

  - Reflection API 可用于 `oneof` 字段。

- If you set a oneof field to the default value (such as setting an int32 oneof field to 0), the “case” of that oneof field will be set, and the value will be serialized on the wire.

  - 如果将一个 `oneof` 字段设置为默认值（例如将 `int32` 类型的 `oneof` 字段设置为 0），该 `oneof` 字段的“case”将被设置，并且该值会被序列化到 wire 数据中。

- If you’re using C++, make sure your code doesn’t cause memory crashes. The following sample code will crash because `sub_message` was already deleted by calling the `set_name()` method. 如果您使用的是 C++，请确保代码不会导致内存崩溃。以下示例代码会崩溃，因为调用 `set_name()` 方法后，`sub_message` 已被删除。

  ```cpp
  SampleMessage message;
  SubMessage* sub_message = message.mutable_sub_message();
  message.set_name("name");      // Will delete sub_message 会删除 sub_message
  sub_message->set_...            // Crashes here 这里会崩溃
  ```

- Again in C++, if you `Swap()` two messages with oneofs, each message will end up with the other’s oneof case: in the example below, `msg1` will have a `sub_message` and `msg2` will have a `name`. 同样在 C++ 中，如果对包含 `oneof` 的两个消息进行 `Swap()`，每个消息将交换 `oneof` 的内容。例如，下面的代码中，`msg1` 将包含 `sub_message`，而 `msg2` 将包含 `name`。

  ```cpp
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

​	在添加或移除 `oneof` 字段时需谨慎。如果检查一个 `oneof` 的值返回 `None`/`NOT_SET`，可能意味着该 `oneof` 未设置，或者它被设置为 `oneof` 中其他版本的字段。没有办法区分，因为无法知道 wire 数据中的未知字段是否是该 `oneof` 的成员。

#### 标签重用问题 Tag Reuse Issues

- **Move singular fields into or out of a oneof**: You may lose some of your information (some fields will be cleared) after the message is serialized and parsed. However, you can safely move a single field into a **new** oneof and may be able to move multiple fields if it is known that only one is ever set. See [Updating A Message Type](https://protobuf.dev/programming-guides/proto3/#updating) for further details.
  - **将单个字段移入或移出 `oneof`**：在消息序列化和解析后，可能会丢失部分信息（某些字段会被清除）。不过，可以安全地将单个字段移入**新的** `oneof`，如果已知只有一个字段会被设置，也可以移动多个字段。详情请参阅 [更新消息类型](https://protobuf.dev/programming-guides/proto3/#updating)。

- **Delete a oneof field and add it back**: This may clear your currently set oneof field after the message is serialized and parsed.
  - **删除 `oneof` 字段后再添加回**：这可能会在消息序列化和解析后清除当前设置的 `oneof` 字段。

- **Split or merge oneof**: This has similar issues to moving singular fields.
  - **拆分或合并 `oneof`**：这与移动单个字段的问题类似。


## Maps

If you want to create an associative map as part of your data definition, protocol buffers provides a handy shortcut syntax:

​	如果您希望在数据定义中创建关联的 map，Protocol Buffers 提供了一种便捷的语法：

```proto
map<key_type, value_type> map_field = N;
```

…where the `key_type` can be any integral or string type (so, any [scalar](https://protobuf.dev/programming-guides/proto3/#scalar) type except for floating point types and `bytes`). Note that neither enum nor proto messages are valid for `key_type`. The `value_type` can be any type except another map.

​	其中，`key_type` 可以是任何整型或字符串类型（即任何 [scalar](https://protobuf.dev/programming-guides/proto3/#scalar) 类型，但不包括浮点类型和 `bytes`）。注意，枚举和 proto 消息都不能作为 `key_type`。`value_type` 可以是除 `map` 之外的任何类型。

So, for example, if you wanted to create a map of projects where each `Project` message is associated with a string key, you could define it like this:

​	例如，如果希望创建一个项目 map，其中每个 `Project` 消息与一个字符串键关联，可以定义如下：

```proto
map<string, Project> projects = 3;
```

### Maps Features

- Map fields cannot be `repeated`.
  - `map` 字段不能是 `repeated`。

- Wire format ordering and map iteration ordering of map values is undefined, so you cannot rely on your map items being in a particular order.
  - wire 格式中的顺序和 `map` 值的迭代顺序未定义，因此不能依赖 `map` 项的特定顺序。

- When generating text format for a `.proto`, maps are sorted by key. Numeric keys are sorted numerically.
  - 为 `.proto` 生成文本格式时，`map` 会按键排序，数字键按数值排序。

- When parsing from the wire or when merging, if there are duplicate map keys the last key seen is used. When parsing a map from text format, parsing may fail if there are duplicate keys.
  - 从 wire 数据解析或合并时，如果存在重复的 `map` 键，则会使用最后看到的键。从文本格式解析 `map` 时，如果存在重复键，可能导致解析失败。

- If you provide a key but no value for a map field, the behavior when the field is serialized is language-dependent. In C++, Java, Kotlin, and Python the default value for the type is serialized, while in other languages nothing is serialized.
  - 如果为 `map` 字段提供键但没有值，字段序列化时的行为取决于语言。在 C++、Java、Kotlin 和 Python 中，序列化该类型的默认值，而在其他语言中不会序列化任何内容。

- No symbol `FooEntry` can exist in the same scope as a map `foo`, because `FooEntry` is already used by the implementation of the map.
  - `FooEntry` 符号不能存在于与 map `foo` 相同的作用域中，因为 `FooEntry` 已被用作 map 的实现。


The generated map API is currently available for all supported languages. You can find out more about the map API for your chosen language in the relevant [API reference](https://protobuf.dev/reference/).

​	生成的 map API 当前适用于所有支持的语言。有关所选语言的 map API 详情，请参阅相关 [API 参考](https://protobuf.dev/reference/)。

### 向后兼容性 Backwards Compatibility

The map syntax is equivalent to the following on the wire, so protocol buffers implementations that do not support maps can still handle your data:

​	`map` 的语法在 wire 数据中的表示等价于以下内容，因此不支持 `map` 的 Protocol Buffers 实现仍然可以处理您的数据：

```proto
message MapFieldEntry {
  key_type key = 1;
  value_type value = 2;
}

repeated MapFieldEntry map_field = N;
```

Any protocol buffers implementation that supports maps must both produce and accept data that can be accepted by the earlier definition.

​	支持 `map` 的任何 Protocol Buffers 实现必须能够生成和接受上述定义的数据。

## Packages

You can add an optional `package` specifier to a `.proto` file to prevent name clashes between protocol message types.

​	您可以在 `.proto` 文件中添加一个可选的 `package` 指定符，以防止协议消息类型之间的名称冲突。

```proto
package foo.bar;
message Open { ... }
```

You can then use the package specifier when defining fields of your message type:

​	然后，您可以在定义消息类型的字段时使用包名：

```proto
message Foo {
  ...
  foo.bar.Open open = 1;
  ...
}
```

The way a package specifier affects the generated code depends on your chosen language:

​	包名对生成代码的影响取决于您选择的语言：

- In **C++** the generated classes are wrapped inside a C++ namespace. For example, `Open` would be in the namespace `foo::bar`.
  - **C++**：生成的类会被封装在 C++ 的命名空间中。例如，`Open` 将位于 `foo::bar` 命名空间中。

- In **Java** and **Kotlin**, the package is used as the Java package, unless you explicitly provide an `option java_package` in your `.proto` file.
  - **Java** 和 **Kotlin**：包名被用作 Java 包，除非您在 `.proto` 文件中显式提供 `option java_package`。

- In **Python**, the `package` directive is ignored, since Python modules are organized according to their location in the file system.
  - **Python**：`package` 指令被忽略，因为 Python 模块是根据其在文件系统中的位置组织的。

- In **Go**, the `package` directive is ignored, and the generated `.pb.go` file is in the package named after the corresponding `go_proto_library` Bazel rule. For open source projects, you **must** provide either a `go_package` option or set the Bazel `-M` flag.
  - **Go**：`package` 指令被忽略，生成的 `.pb.go` 文件位于与相应 Bazel 规则 `go_proto_library` 命名一致的包中。对于开源项目，您**必须**提供 `go_package` 选项或设置 Bazel 的 `-M` 标志。

- In **Ruby**, the generated classes are wrapped inside nested Ruby namespaces, converted to the required Ruby capitalization style (first letter capitalized; if the first character is not a letter, `PB_` is prepended). For example, `Open` would be in the namespace `Foo::Bar`.
  - **Ruby**：生成的类会嵌套在 Ruby 的命名空间中，并转换为符合 Ruby 的命名风格（首字母大写；如果首字符不是字母，则添加 `PB_` 前缀）。例如，`Open` 将位于 `Foo::Bar` 命名空间中。

- In **PHP** the package is used as the namespace after converting to PascalCase, unless you explicitly provide an `option php_namespace` in your `.proto` file. For example, `Open` would be in the namespace `Foo\Bar`.
  - **PHP**：包名会被转换为 PascalCase，用作命名空间，除非您显式提供 `option php_namespace`。例如，`Open` 将位于 `Foo\Bar` 命名空间中。

- In **C#** the package is used as the namespace after converting to PascalCase, unless you explicitly provide an `option csharp_namespace` in your `.proto` file. For example, `Open` would be in the namespace `Foo.Bar`.
  - **C#**：包名会被转换为 PascalCase，用作命名空间，除非您显式提供 `option csharp_namespace`。例如，`Open` 将位于 `Foo.Bar` 命名空间中。


Note that even when the `package` directive does not directly affect the generated code, for example in Python, it is still strongly recommended to specify the package for the `.proto` file, as otherwise it may lead to naming conflicts in descriptors and make the proto not portable for other languages.

​	即使 `package` 指令不会直接影响生成代码（例如在 Python 中），仍然强烈建议为 `.proto` 文件指定包名，否则可能导致描述符的命名冲突，并使 proto 在其他语言中不可移植。

### 包和名称解析 Packages and Name Resolution

Type name resolution in the protocol buffer language works like C++: first the innermost scope is searched, then the next-innermost, and so on, with each package considered to be “inner” to its parent package. A leading ‘.’ (for example, `.foo.bar.Baz`) means to start from the outermost scope instead.

​	Protocol Buffer 语言中的类型名称解析类似于 C++：首先搜索最内层作用域，然后逐级向外搜索，每个包被认为是其父包的“内层”。以 `.` 开头（例如 `.foo.bar.Baz`）表示从最外层作用域开始。

The protocol buffer compiler resolves all type names by parsing the imported `.proto` files. The code generator for each language knows how to refer to each type in that language, even if it has different scoping rules.

​	Protocol Buffer 编译器通过解析导入的 `.proto` 文件解析所有类型名称。每种语言的代码生成器都知道如何在该语言中引用每种类型，即使它们的作用域规则不同。

## 定义服务 Defining Services

If you want to use your message types with an RPC (Remote Procedure Call) system, you can define an RPC service interface in a `.proto` file and the protocol buffer compiler will generate service interface code and stubs in your chosen language. So, for example, if you want to define an RPC service with a method that takes your `SearchRequest` and returns a `SearchResponse`, you can define it in your `.proto` file as follows:

​	如果希望在 RPC（远程过程调用）系统中使用消息类型，可以在 `.proto` 文件中定义 RPC 服务接口，Protocol Buffer 编译器会为您选择的语言生成服务接口代码和存根。例如，如果希望定义一个 RPC 服务，包含一个接受 `SearchRequest` 并返回 `SearchResponse` 的方法，可以在 `.proto` 文件中定义如下：

```proto
service SearchService {
  rpc Search(SearchRequest) returns (SearchResponse);
}
```

The most straightforward RPC system to use with protocol buffers is [gRPC](https://grpc.io/): a language- and platform-neutral open source RPC system developed at Google. gRPC works particularly well with protocol buffers and lets you generate the relevant RPC code directly from your `.proto` files using a special protocol buffer compiler plugin.

​	最直接的与 Protocol Buffers 配合使用的 RPC 系统是 [gRPC](https://grpc.io/)：一个语言和平台无关的开源 RPC 系统，由 Google 开发。gRPC 与 Protocol Buffers 配合使用效果非常好，并允许您直接从 `.proto` 文件生成相关的 RPC 代码，使用一个特殊的 Protocol Buffer 编译器插件。

If you don’t want to use gRPC, it’s also possible to use protocol buffers with your own RPC implementation. You can find out more about this in the [Proto2 Language Guide](https://protobuf.dev/programming-guides/proto2#services).

​	如果您不想使用 gRPC，也可以将 Protocol Buffers 与您自己的 RPC 实现结合使用。您可以在 [Proto2 语言指南](https://protobuf.dev/programming-guides/proto2#services) 中了解更多相关内容。

There are also a number of ongoing third-party projects to develop RPC implementations for Protocol Buffers. For a list of links to projects we know about, see the [third-party add-ons wiki page](https://github.com/protocolbuffers/protobuf/blob/master/docs/third_party.md).

​	此外，还有许多正在进行中的第三方项目致力于为 Protocol Buffers 开发 RPC 实现。有关我们已知项目的链接列表，请参阅 [第三方扩展 wiki 页面](https://github.com/protocolbuffers/protobuf/blob/master/docs/third_party.md)。

## JSON Mapping

The standard protobuf binary wire format is the preferred serialization format for communication between two systems that use protobufs. For communicating with systems that use JSON rather than protobuf wire format, Protobuf supports a canonical encoding in [JSON](https://protobuf.dev/programming-guides/json).

​	标准的 protobuf 二进制 wire 格式是两个使用 protobuf 系统之间通信的首选序列化格式。为了与使用 JSON 而不是 protobuf wire 格式的系统通信，Protobuf 支持一种规范的 [JSON 编码](https://protobuf.dev/programming-guides/json)。

## Options

Individual declarations in a `.proto` file can be annotated with a number of *options*. Options do not change the overall meaning of a declaration, but may affect the way it is handled in a particular context. The complete list of available options is defined in [`/google/protobuf/descriptor.proto`](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/descriptor.proto).

​	在 `.proto` 文件中，单个声明可以使用许多*选项*进行注释。选项不会改变声明的整体含义，但可能会影响其在特定上下文中的处理方式。完整的可用选项列表定义在 [`/google/protobuf/descriptor.proto`](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/descriptor.proto) 文件中。

Some options are file-level options, meaning they should be written at the top-level scope, not inside any message, enum, or service definition. Some options are message-level options, meaning they should be written inside message definitions. Some options are field-level options, meaning they should be written inside field definitions. Options can also be written on enum types, enum values, oneof fields, service types, and service methods; however, no useful options currently exist for any of these.

​	有些选项是文件级选项，应该写在顶层作用域，而不是消息、枚举或服务定义内。有些选项是消息级选项，应该写在消息定义内。有些选项是字段级选项，应该写在字段定义内。选项也可以用于枚举类型、枚举值、oneof 字段、服务类型和服务方法；然而，目前没有针对这些的有用选项。

Here are a few of the most commonly used options:

​	以下是一些常用选项：

- `java_package` (file option): The package you want to use for your generated Java/Kotlin classes. If no explicit `java_package` option is given in the `.proto` file, then by default the proto package (specified using the “package” keyword in the `.proto` file) will be used. However, proto packages generally do not make good Java packages since proto packages are not expected to start with reverse domain names. If not generating Java or Kotlin code, this option has no effect. **`java_package`**（文件选项）：指定生成的 Java/Kotlin 类的包。如果 `.proto` 文件中没有显式指定 `java_package` 选项，则默认使用 `.proto` 文件中通过 `package` 关键字指定的 proto 包名。然而，proto 包通常不是好的 Java 包名，因为它们通常不以反向域名开头。如果不生成 Java 或 Kotlin 代码，此选项无效。

  ```proto
  option java_package = "com.example.foo";
  ```

- `java_outer_classname` (file option): The class name (and hence the file name) for the wrapper Java class you want to generate. If no explicit `java_outer_classname` is specified in the `.proto` file, the class name will be constructed by converting the `.proto` file name to camel-case (so `foo_bar.proto` becomes `FooBar.java`). If the `java_multiple_files` option is disabled, then all other classes/enums/etc. generated for the `.proto` file will be generated *within* this outer wrapper Java class as nested classes/enums/etc. If not generating Java code, this option has no effect. **`java_outer_classname`**（文件选项）：指定生成的包装 Java 类的类名（因此也是文件名）。如果 `.proto` 文件中未显式指定 `java_outer_classname`，则类名将通过将 `.proto` 文件名转换为驼峰式命名来构造（例如 `foo_bar.proto` 会变成 `FooBar.java`）。如果禁用了 `java_multiple_files` 选项，则为此 `.proto` 文件生成的所有其他类/枚举等将作为嵌套类/枚举包含在此包装类中。如果不生成 Java 代码，此选项无效。

  ```proto
  option java_outer_classname = "Ponycopter";
  ```

- `java_multiple_files` (file option): If false, only a single `.java` file will be generated for this `.proto` file, and all the Java classes/enums/etc. generated for the top-level messages, services, and enumerations will be nested inside of an outer class (see `java_outer_classname`). If true, separate `.java` files will be generated for each of the Java classes/enums/etc. generated for the top-level messages, services, and enumerations, and the wrapper Java class generated for this `.proto` file won’t contain any nested classes/enums/etc. This is a Boolean option which defaults to `false`. If not generating Java code, this option has no effect. **`java_multiple_files`**（文件选项）：如果设置为 `false`，则仅为此 `.proto` 文件生成一个 `.java` 文件，且为顶级消息、服务和枚举生成的所有 Java 类/枚举等将嵌套在一个外部类中（参见 `java_outer_classname`）。如果设置为 `true`，则为顶级消息、服务和枚举生成的每个 Java 类/枚举等会生成单独的 `.java` 文件，而为此 `.proto` 文件生成的包装 Java 类不会包含任何嵌套类/枚举等。此布尔选项默认为 `false`。如果不生成 Java 代码，此选项无效。

  ```proto
  option java_multiple_files = true;
  ```

- `optimize_for` (file option): Can be set to `SPEED`, `CODE_SIZE`, or `LITE_RUNTIME`. This affects the C++ and Java code generators (and possibly third-party generators) in the following ways: **`optimize_for`**（文件选项）：可以设置为 `SPEED`、`CODE_SIZE` 或 `LITE_RUNTIME`。这会以如下方式影响 C++ 和 Java 代码生成器（以及可能的第三方生成器）：

  - `SPEED` (default): The protocol buffer compiler will generate code for serializing, parsing, and performing other common operations on your message types. This code is highly optimized. 
    - **`SPEED`**（默认）：Protocol Buffer 编译器会为消息类型生成高度优化的序列化、解析和其他常用操作代码。

  - `CODE_SIZE`: The protocol buffer compiler will generate minimal classes and will rely on shared, reflection-based code to implement serialization, parsing, and various other operations. The generated code will thus be much smaller than with `SPEED`, but operations will be slower. Classes will still implement exactly the same public API as they do in `SPEED` mode. This mode is most useful in apps that contain a very large number of `.proto` files and do not need all of them to be blindingly fast.
    - **`CODE_SIZE`**：Protocol Buffer 编译器会生成最小的类，并依赖共享的基于反射的代码来实现序列化、解析和各种其他操作。生成的代码会比 `SPEED` 模式小得多，但操作速度会更慢。类仍然会实现与 `SPEED` 模式完全相同的公共 API。此模式对于包含大量 `.proto` 文件且不需要所有文件都非常快的应用最有用。

  - `LITE_RUNTIME`: The protocol buffer compiler will generate classes that depend only on the “lite” runtime library (`libprotobuf-lite` instead of `libprotobuf`). The lite runtime is much smaller than the full library (around an order of magnitude smaller) but omits certain features like descriptors and reflection. This is particularly useful for apps running on constrained platforms like mobile phones. The compiler will still generate fast implementations of all methods as it does in `SPEED` mode. Generated classes will only implement the `MessageLite` interface in each language, which provides only a subset of the methods of the full `Message` interface. **`LITE_RUNTIME`**：Protocol Buffer 编译器会生成仅依赖“精简”运行时库（`libprotobuf-lite` 而不是 `libprotobuf`）的类。精简运行时比完整库小得多（大约小一个数量级），但省略了某些功能，如描述符和反射。这对运行在受限平台（如手机）上的应用特别有用。编译器仍然会像 `SPEED` 模式一样生成所有方法的快速实现。生成的类仅实现每种语言的 `MessageLite` 接口，该接口仅提供完整 `Message` 接口的一部分方法。

  ```proto
  option optimize_for = CODE_SIZE;
  ```

- `cc_generic_services`, `java_generic_services`, `py_generic_services` (file options): **Generic services are deprecated.** Whether or not the protocol buffer compiler should generate abstract service code based on [services definitions](https://protobuf.dev/programming-guides/proto3/#services) in C++, Java, and Python, respectively. For legacy reasons, these default to `true`. However, as of version 2.3.0 (January 2010), it is considered preferable for RPC implementations to provide [code generator plugins](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.compiler.plugin.pb) to generate code more specific to each system, rather than rely on the “abstract” services. **`cc_generic_services`**、**`java_generic_services`**、**`py_generic_services`**（文件选项）：**通用服务已被弃用。** 指定 Protocol Buffer 编译器是否应基于 [服务定义](https://protobuf.dev/programming-guides/proto3/#services) 在 C++、Java 和 Python 中生成抽象服务代码。由于历史原因，这些选项默认值为 `true`。然而，自 2.3.0 版（2010 年 1 月）起，建议 RPC 实现提供[代码生成器插件](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.compiler.plugin.pb)，以生成更适合每个系统的代码，而不是依赖“抽象”服务。

  ```proto
  // This file relies on plugins to generate service code.
  // 此文件依赖插件来生成服务代码。
  option cc_generic_services = false;
  option java_generic_services = false;
  option py_generic_services = false;
  ```

- `cc_enable_arenas` (file option): Enables [arena allocation](https://protobuf.dev/reference/cpp/arenas) for C++ generated code.

  - **`cc_enable_arenas`**（文件选项）：为 C++ 生成的代码启用[内存池分配](https://protobuf.dev/reference/cpp/arenas)。

- `objc_class_prefix` (file option): Sets the Objective-C class prefix which is prepended to all Objective-C generated classes and enums from this .proto. There is no default. You should use prefixes that are between 3-5 uppercase characters as [recommended by Apple](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Conventions/Conventions.html#//apple_ref/doc/uid/TP40011210-CH10-SW4). Note that all 2 letter prefixes are reserved by Apple.

  - **`objc_class_prefix`**（文件选项）：设置前缀，添加到所有 Objective-C 生成的类和枚举的名称之前。没有默认值。您应使用 3-5 个大写字符的前缀，正如 [Apple 推荐](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Conventions/Conventions.html#//apple_ref/doc/uid/TP40011210-CH10-SW4)的那样。注意，所有两个字符的前缀都保留给 Apple。

- `packed` (field option): Defaults to `true` on a repeated field of a basic numeric type, causing a more compact [encoding](https://protobuf.dev/programming-guides/encoding#packed) to be used. To use unpacked wireformat, it can be set to `false`. This provides compatibility with parsers prior to version 2.3.0 (rarely needed) as shown in the following example: **`packed`**（字段选项）：默认值为 `true`，用于基本数值类型的重复字段，使其使用更紧凑的[编码](https://protobuf.dev/programming-guides/encoding#packed)。若要使用未打包的 wire 格式，可以将其设置为 `false`。这为 2.3.0 版之前的解析器提供兼容性（很少需要），例如：

  ```proto
  repeated int32 samples = 4 [packed = false];
  ```

- `deprecated` (field option): If set to `true`, indicates that the field is deprecated and should not be used by new code. In most languages this has no actual effect. In Java, this becomes a `@Deprecated` annotation. For C++, clang-tidy will generate warnings whenever deprecated fields are used. In the future, other language-specific code generators may generate deprecation annotations on the field’s accessors, which will in turn cause a warning to be emitted when compiling code which attempts to use the field. If the field is not used by anyone and you want to prevent new users from using it, consider replacing the field declaration with a [reserved](https://protobuf.dev/programming-guides/proto3/#fieldreserved) statement. **`deprecated`**（字段选项）：如果设置为 `true`，表示该字段已被弃用，不应在新代码中使用。在大多数语言中，这不会产生实际影响。在 Java 中，这会成为一个 `@Deprecated` 注解。在 C++ 中，`clang-tidy` 在使用被弃用的字段时会生成警告。在未来，其他特定语言的代码生成器可能会在字段访问器上生成弃用注解，从而在编译尝试使用该字段的代码时触发警告。如果该字段未被任何人使用，并且您想阻止新用户使用它，请考虑用一个 [reserved](https://protobuf.dev/programming-guides/proto3/#fieldreserved) 声明替代字段声明。

  ```proto
  int32 old_field = 6 [deprecated = true];
  ```

### 枚举值选项 Enum Value Options

Enum value options are supported. You can use the `deprecated` option to indicate that a value shouldn’t be used anymore. You can also create custom options using extensions.

​	支持枚举值选项。您可以使用 `deprecated` 选项指示某个值不应再被使用。您还可以使用扩展创建自定义选项。

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

​	以下是 C++ 中读取 `string_name` 选项的代码示例：

```cpp
const absl::string_view foo = proto2::GetEnumDescriptor<Data>()
    ->FindValueByName("DATA_DISPLAY")->options().GetExtension(string_name);
```

See [Custom Options](https://protobuf.dev/programming-guides/proto3/#customoptions) to see how to apply custom options to enum values and to fields.

​	请参阅[自定义选项](https://protobuf.dev/programming-guides/proto3/#customoptions)，了解如何将自定义选项应用于枚举值和字段。

### 自定义选项 Custom Options

Protocol Buffers also allows you to define and use your own options. Note that this is an **advanced feature** which most people don’t need. If you do think you need to create your own options, see the [Proto2 Language Guide](https://protobuf.dev/programming-guides/proto2#customoptions) for details. Note that creating custom options uses [extensions](https://protobuf.dev/programming-guides/proto2#extensions), which are permitted only for custom options in proto3.

​	Protocol Buffers 允许您定义和使用自己的选项。请注意，这是一项**高级功能**，大多数人并不需要。如果您认为需要创建自己的选项，请参阅 [Proto2 语言指南](https://protobuf.dev/programming-guides/proto2#customoptions) 了解详细信息。注意，创建自定义选项需要使用 [扩展](https://protobuf.dev/programming-guides/proto2#extensions)，在 proto3 中仅允许用于自定义选项。

### 选项保留 Option Retention

Options have a notion of *retention*, which controls whether an option is retained in the generated code. Options have *runtime retention* by default, meaning that they are retained in the generated code and are thus visible at runtime in the generated descriptor pool. However, you can set `retention = RETENTION_SOURCE` to specify that an option (or field within an option) must not be retained at runtime. This is called *source retention*.

​	选项具有一种*保留*概念，控制选项是否保留在生成的代码中。默认情况下，选项具有*运行时保留*（runtime retention），即它们会保留在生成的代码中，并在运行时通过生成的描述符池可见。然而，您可以设置 `retention = RETENTION_SOURCE`，指定选项（或选项中的字段）在运行时不被保留。这称为*源代码保留*（source retention）。

Option retention is an advanced feature that most users should not need to worry about, but it can be useful if you would like to use certain options without paying the code size cost of retaining them in your binaries. Options with source retention are still visible to `protoc` and `protoc` plugins, so code generators can use them to customize their behavior.

​	选项保留是一项高级功能，大多数用户不需要担心，但如果您希望使用某些选项而无需承担将它们保留在二进制文件中的代码大小成本，则这会非常有用。具有源代码保留的选项在 `protoc` 和其插件中仍然可见，因此代码生成器可以使用它们来自定义行为。

Retention can be set directly on an option, like this:

​	可以直接在选项上设置保留，例如：

```proto
extend google.protobuf.FileOptions {
  optional int32 source_retention_option = 1234
      [retention = RETENTION_SOURCE];
}
```

It can also be set on a plain field, in which case it takes effect only when that field appears inside an option:

​	也可以在普通字段上设置保留，在这种情况下，只有当该字段出现在选项中时，保留才会生效：

```proto
message OptionsMessage {
  int32 source_retention_field = 1 [retention = RETENTION_SOURCE];
}
```

You can set `retention = RETENTION_RUNTIME` if you like, but this has no effect since it is the default behavior. When a message field is marked `RETENTION_SOURCE`, its entire contents are dropped; fields inside it cannot override that by trying to set `RETENTION_RUNTIME`.

​	您可以将保留设置为 `retention = RETENTION_RUNTIME`，但这不会生效，因为这是默认行为。当消息字段被标记为 `RETENTION_SOURCE` 时，其整个内容会被丢弃；其中的字段无法通过尝试设置 `RETENTION_RUNTIME` 来覆盖这一行为。

> Note
>
> As of Protocol Buffers 22.0, support for option retention is still in progress and only C++ and Java are supported. Go has support starting from 1.29.0. Python support is complete but has not made it into a release yet.
>
> ​	截至 Protocol Buffers 22.0，选项保留的支持仍在进行中，目前仅支持 C++ 和 Java。Go 从 1.29.0 开始支持。Python 的支持已完成，但尚未发布到版本中。

### 选项目标 Option Targets

Fields have a `targets` option which controls the types of entities that the field may apply to when used as an option. For example, if a field has `targets = TARGET_TYPE_MESSAGE` then that field cannot be set in a custom option on an enum (or any other non-message entity). Protoc enforces this and will raise an error if there is a violation of the target constraints.

​	字段具有 `targets` 选项，用于控制字段在用作选项时适用于哪些实体类型。例如，如果字段具有 `targets = TARGET_TYPE_MESSAGE`，则该字段不能在枚举（或任何非消息实体）上作为自定义选项设置。`protoc` 会强制执行这一点，如果违反目标约束将引发错误。

At first glance, this feature may seem unnecessary given that every custom option is an extension of the options message for a specific entity, which already constrains the option to that one entity. However, option targets are useful in the case where you have a shared options message applied to multiple entity types and you want to control the usage of individual fields in that message. For example:

​	乍一看，这个功能似乎没有必要，因为每个自定义选项都是特定实体的选项消息的扩展，这已经将选项约束为该实体。然而，当您有一个共享的选项消息应用于多个实体类型，并希望控制该消息中各个字段的使用时，选项目标会很有用。例如：

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
// 正确：该字段允许用于文件选项
option (file_options).file_only_option = "abc";

message MyMessage {
  // OK: this field is allowed on both message and enum options
  // 正确：该字段允许用于消息和枚举选项
  option (message_options).message_and_enum_option = 42;
}

enum MyEnum {
  MY_ENUM_UNSPECIFIED = 0;
  // Error: file_only_option cannot be set on an enum.
  // 错误：file_only_option 不能用于枚举。
  option (enum_options).file_only_option = "xyz";
}
```

## 生成类 Generating Your Classes

To generate the Java, Kotlin, Python, C++, Go, Ruby, Objective-C, or C# code that you need to work with the message types defined in a `.proto` file, you need to run the protocol buffer compiler `protoc` on the `.proto` file. If you haven’t installed the compiler, [download the package](https://protobuf.dev/downloads) and follow the instructions in the README. For Go, you also need to install a special code generator plugin for the compiler; you can find this and installation instructions in the [golang/protobuf](https://github.com/golang/protobuf/) repository on GitHub.

​	要生成 Java、Kotlin、Python、C++、Go、Ruby、Objective-C 或 C# 代码以使用 `.proto` 文件中定义的消息类型，您需要对 `.proto` 文件运行 Protocol Buffer 编译器 `protoc`。如果尚未安装编译器，请[下载安装包](https://protobuf.dev/downloads)并按照 README 中的说明操作。对于 Go，还需要为编译器安装一个特殊的代码生成器插件；您可以在 GitHub 上的 [golang/protobuf](https://github.com/golang/protobuf/) 仓库中找到此插件及安装说明。

The Protocol Compiler is invoked as follows:

​	Protocol Buffer 编译器的使用方式如下：

```sh
protoc --proto_path=IMPORT_PATH --cpp_out=DST_DIR --java_out=DST_DIR --python_out=DST_DIR --go_out=DST_DIR --ruby_out=DST_DIR --objc_out=DST_DIR --csharp_out=DST_DIR path/to/file.proto
```

- `IMPORT_PATH` specifies a directory in which to look for `.proto` files when resolving `import` directives. If omitted, the current directory is used. Multiple import directories can be specified by passing the `--proto_path` option multiple times; they will be searched in order. `-I=_IMPORT_PATH_` can be used as a short form of `--proto_path`.

  - **`IMPORT_PATH`** 指定解析 `import` 指令时查找 `.proto` 文件的目录。如果省略，则使用当前目录。可以通过多次传递 `--proto_path` 选项指定多个导入目录；它们会按顺序被搜索。`-I=_IMPORT_PATH_` 是 `--proto_path` 的简写。

- You can provide one or more *output directives*: 您可以提供一个或多个*输出指令*：

  - `--cpp_out` generates C++ code in `DST_DIR`. See the [C++ generated code reference](https://protobuf.dev/reference/cpp/cpp-generated) for more.
    - `--cpp_out` 在 `DST_DIR` 中生成 C++ 代码。参见 [C++ 生成代码参考](https://protobuf.dev/reference/cpp/cpp-generated)。

  - `--java_out` generates Java code in `DST_DIR`. See the [Java generated code reference](https://protobuf.dev/reference/java/java-generated) for more.
    - `--java_out` 在 `DST_DIR` 中生成 Java 代码。参见 [Java 生成代码参考](https://protobuf.dev/reference/java/java-generated)。

  - `--kotlin_out` generates additional Kotlin code in `DST_DIR`. See the [Kotlin generated code reference](https://protobuf.dev/reference/kotlin/kotlin-generated) for more.
    - `--kotlin_out` 在 `DST_DIR` 中生成 Kotlin 附加代码。参见 [Kotlin 生成代码参考](https://protobuf.dev/reference/kotlin/kotlin-generated)。

  - `--python_out` generates Python code in `DST_DIR`. See the [Python generated code reference](https://protobuf.dev/reference/python/python-generated) for more.
    - `--python_out` 在 `DST_DIR` 中生成 Python 代码。参见 [Python 生成代码参考](https://protobuf.dev/reference/python/python-generated)。

  - `--go_out` generates Go code in `DST_DIR`. See the [Go generated code reference](https://protobuf.dev/reference/go/go-generated) for more.
    - `--go_out` 在 `DST_DIR` 中生成 Go 代码。参见 [Go 生成代码参考](https://protobuf.dev/reference/go/go-generated)。

  - `--ruby_out` generates Ruby code in `DST_DIR`. See the [Ruby generated code reference](https://protobuf.dev/reference/ruby/ruby-generated) for more.
    - `--ruby_out` 在 `DST_DIR` 中生成 Ruby 代码。参见 [Ruby 生成代码参考](https://protobuf.dev/reference/ruby/ruby-generated)。

  - `--objc_out` generates Objective-C code in `DST_DIR`. See the [Objective-C generated code reference](https://protobuf.dev/reference/objective-c/objective-c-generated) for more.
    - `--objc_out` 在 `DST_DIR` 中生成 Objective-C 代码。参见 [Objective-C 生成代码参考](https://protobuf.dev/reference/objective-c/objective-c-generated)。

  - `--csharp_out` generates C# code in `DST_DIR`. See the [C# generated code reference](https://protobuf.dev/reference/csharp/csharp-generated) for more.
    - `--csharp_out` 在 `DST_DIR` 中生成 C# 代码。参见 [C# 生成代码参考](https://protobuf.dev/reference/csharp/csharp-generated)。

  - `--php_out` generates PHP code in `DST_DIR`. See the [PHP generated code reference](https://protobuf.dev/reference/php/php-generated) for more.
    - `--php_out` 在 `DST_DIR` 中生成 PHP 代码。参见 [PHP 生成代码参考](https://protobuf.dev/reference/php/php-generated)。


  As an extra convenience, if the `DST_DIR` ends in `.zip` or `.jar`, the compiler will write the output to a single ZIP-format archive file with the given name. `.jar` outputs will also be given a manifest file as required by the Java JAR specification. Note that if the output archive already exists, it will be overwritten.

  ​	如果 `DST_DIR` 以 `.zip` 或 `.jar` 结尾，编译器会将输出写入具有指定名称的 ZIP 格式归档文件中。`.jar` 输出还会包含符合 Java JAR 规范的清单文件。注意，如果输出归档文件已存在，它会被覆盖。

- You must provide one or more `.proto` files as input. Multiple `.proto` files can be specified at once. Although the files are named relative to the current directory, each file must reside in one of the `IMPORT_PATH`s so that the compiler can determine its canonical name.

  - 您必须提供一个或多个 `.proto` 文件作为输入。可以一次性指定多个 `.proto` 文件。尽管文件是相对于当前目录命名的，但每个文件必须位于 `IMPORT_PATH` 的某个目录下，以便编译器可以确定其规范名称。


## 文件位置 File location

Prefer not to put `.proto` files in the same directory as other language sources. Consider creating a subpackage `proto` for `.proto` files, under the root package for your project.

​	尽量不要将 `.proto` 文件与其他语言源文件放在同一目录。建议在项目的根包下为 `.proto` 文件创建一个 `proto` 子包。

### 位置应与语言无关 Location Should be Language-agnostic

When working with Java code, it’s handy to put related `.proto` files in the same directory as the Java source. However, if any non-Java code ever uses the same protos, the path prefix will no longer make sense. So in general, put the protos in a related language-agnostic directory such as `//myteam/mypackage`.

​	在处理 Java 代码时，将相关 `.proto` 文件与 Java 源文件放在同一目录中会很方便。然而，如果有任何非 Java 代码需要使用这些 proto 文件，则路径前缀将不再有意义。因此，通常应将 proto 文件放在与语言无关的相关目录中，例如 `//myteam/mypackage`。

The exception to this rule is when it’s clear that the protos will be used only in a Java context, such as for testing.

​	此规则的例外情况是当明确 proto 文件仅在 Java 上下文中使用，例如用于测试。

## 支持的平台 Supported Platforms

For information about:

- the operating systems, compilers, build systems, and C++ versions that are supported, see [Foundational C++ Support Policy](https://opensource.google/documentation/policies/cplusplus-support).
  - 支持的操作系统、编译器、构建系统和 C++ 版本，请参见 [C++ 基础支持政策](https://opensource.google/documentation/policies/cplusplus-support)。
- the PHP versions that are supported, see [Supported PHP versions](https://cloud.google.com/php/getting-started/supported-php-versions).
  - 支持的 PHP 版本，请参见 [支持的 PHP 版本](https://cloud.google.com/php/getting-started/supported-php-versions)。
