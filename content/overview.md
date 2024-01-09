+++
title = "概述"
weight = 10
description = "Protocol Buffers are a language-neutral, platform-neutral extensible mechanism for serializing structured data."
type = "docs"

+++

## Overview 概述

Protocol Buffers are a language-neutral, platform-neutral extensible mechanism for serializing structured data.

​	协议缓冲区是一种语言中立、平台中立的可扩展机制，用于序列化结构化数据。

It’s like JSON, except it’s smaller and faster, and it generates native language bindings. You define how you want your data to be structured once, then you can use special generated source code to easily write and read your structured data to and from a variety of data streams and using a variety of languages.

​	它类似于 JSON，但它更小、更快，并且它生成本机语言绑定。您只需定义一次想要如何构建数据，然后即可使用特殊生成的源代码轻松地将结构化数据写入各种数据流并从中读取，还可以使用各种语言。

Protocol buffers are a combination of the definition language (created in `.proto` files), the code that the proto compiler generates to interface with data, language-specific runtime libraries, and the serialization format for data that is written to a file (or sent across a network connection).

​	协议缓冲区是定义语言（在 `.proto` 文件中创建）、proto 编译器生成的用于与数据交互的代码、特定于语言的运行时库以及写入文件（或通过网络连接发送）的数据的序列化格式的组合。

## 协议缓冲区解决了哪些问题？ What Problems do Protocol Buffers Solve? 

Protocol buffers provide a serialization format for packets of typed, structured data that are up to a few megabytes in size. The format is suitable for both ephemeral network traffic and long-term data storage. Protocol buffers can be extended with new information without invalidating existing data or requiring code to be updated.

​	协议缓冲区为大小不超过几兆字节的类型化结构化数据包提供序列化格式。该格式适用于临时网络流量和长期数据存储。协议缓冲区可以扩展新信息，而不会使现有数据无效或要求更新代码。

Protocol buffers are the most commonly-used data format at Google. They are used extensively in inter-server communications as well as for archival storage of data on disk. Protocol buffer *messages* and *services* are described by engineer-authored `.proto` files. The following shows an example `message`:

​	协议缓冲区是 Google 中最常用的数据格式。它们被广泛用于服务器间通信以及磁盘上的数据归档存储。协议缓冲区消息和服务由工程师编写的 `.proto` 文件描述。以下显示了一个示例 `message` ：

```proto
message Person {
  optional string name = 1;
  optional int32 id = 2;
  optional string email = 3;
}
```

The proto compiler is invoked at build time on `.proto` files to generate code in various programming languages (covered in [Cross-language Compatibility](https://protobuf.dev/overview/#cross-lang) later in this topic) to manipulate the corresponding protocol buffer. Each generated class contains simple accessors for each field and methods to serialize and parse the whole structure to and from raw bytes. The following shows you an example that uses those generated methods:

​	在构建时调用 `.proto` 文件上的 proto 编译器，以生成各种编程语言中的代码（稍后在本主题的跨语言兼容性中介绍），以操作相应的协议缓冲区。每个生成的类都包含每个字段的简单访问器以及将整个结构序列化和解析为原始字节的方法。以下显示了一个使用这些生成方法的示例：

```java
Person john = Person.newBuilder()
    .setId(1234)
    .setName("John Doe")
    .setEmail("jdoe@example.com")
    .build();
output = new FileOutputStream(args[0]);
john.writeTo(output);
```

Because protocol buffers are used extensively across all manner of services at Google and data within them may persist for some time, maintaining backwards compatibility is crucial. Protocol buffers allow for the seamless support of changes, including the addition of new fields and the deletion of existing fields, to any protocol buffer without breaking existing services. For more on this topic, see [Updating Proto Definitions Without Updating Code](https://protobuf.dev/overview/#updating-defs), later in this topic.

​	由于协议缓冲区在 Google 的各种服务中广泛使用，并且其中的数据可能会持续存在一段时间，因此保持向后兼容性至关重要。协议缓冲区允许无缝支持更改，包括添加新字段和删除现有字段，而不会中断现有服务。有关此主题的更多信息，请参阅本主题后面的“在不更新代码的情况下更新 Proto 定义”。

## 使用协议缓冲区的优势是什么？ What are the Benefits of Using Protocol Buffers? 

Protocol buffers are ideal for any situation in which you need to serialize structured, record-like, typed data in a language-neutral, platform-neutral, extensible manner. They are most often used for defining communications protocols (together with gRPC) and for data storage.

​	协议缓冲区适用于任何需要以语言中立、平台中立、可扩展的方式序列化结构化、类似记录的类型化数据的情况。它们最常用于定义通信协议（与 gRPC 一起使用）和数据存储。

Some of the advantages of using protocol buffers include:

​	使用协议缓冲区的一些优点包括：

- Compact data storage
- 紧凑的数据存储
- Fast parsing
- 快速解析
- Availability in many programming languages
- 可在多种编程语言中使用
- Optimized functionality through automatically-generated classes
- 通过自动生成的类优化功能

### 跨语言兼容性 Cross-language Compatibility 

The same messages can be read by code written in any supported programming language. You can have a Java program on one platform capture data from one software system, serialize it based on a `.proto` definition, and then extract specific values from that serialized data in a separate Python application running on another platform.

​	任何受支持的编程语言编写的代码都可以读取相同的消息。您可以在一个平台上的 Java 程序中捕获来自一个软件系统的数据，根据 `.proto` 定义对其进行序列化，然后在另一个平台上运行的单独 Python 应用程序中从该序列化数据中提取特定值。

The following languages are supported directly in the protocol buffers compiler, protoc:

​	协议缓冲区编译器 protoc 直接支持以下语言：

- [C++](https://protobuf.dev/reference/cpp/cpp-generated#invocation)
- [C#](https://protobuf.dev/reference/csharp/csharp-generated#invocation)
- [Java](https://protobuf.dev/reference/java/java-generated#invocation)
- [Kotlin](https://protobuf.dev/reference/kotlin/kotlin-generated#invocation)
- [Objective-C](https://protobuf.dev/reference/objective-c/objective-c-generated#invocation)
- [PHP](https://protobuf.dev/reference/php/php-generated#invocation)
- [Python](https://protobuf.dev/reference/python/python-generated#invocation)
- [Ruby](https://protobuf.dev/reference/ruby/ruby-generated#invocation)

The following languages are supported by Google, but the projects’ source code resides in GitHub repositories. The protoc compiler uses plugins for these languages:

​	Google 支持以下语言，但项目的源代码位于 GitHub 存储库中。protoc 编译器对这些语言使用插件：

- [Dart](https://github.com/google/protobuf.dart)
- [Go](https://github.com/protocolbuffers/protobuf-go)

Additional languages are not directly supported by Google, but rather by other GitHub projects. These languages are covered in [Third-Party Add-ons for Protocol Buffers](https://github.com/protocolbuffers/protobuf/blob/master/docs/third_party.md).

​	其他语言并未得到 Google 的直接支持，而是由其他 GitHub 项目支持。这些语言在 Protocol Buffers 的第三方加载项中有所介绍。

### 跨项目支持 Cross-project Support 

You can use protocol buffers across projects by defining `message` types in `.proto` files that reside outside of a specific project’s code base. If you’re defining `message` types or enums that you anticipate will be widely used outside of your immediate team, you can put them in their own file with no dependencies.

​	您可以在项目间使用协议缓冲区，方法是在位于特定项目代码库之外的 `.proto` 文件中定义 `message` 类型。如果您正在定义您预计将在您的直接团队之外广泛使用的 `message` 类型或枚举，您可以将它们放在它们自己的文件中，而无需依赖项。

A couple of examples of proto definitions widely-used within Google are [`timestamp.proto`](https://github.com/protocolbuffers/protobuf/blob/master/src/google/protobuf/timestamp.proto) and [`status.proto`](https://github.com/googleapis/googleapis/blob/master/google/rpc/status.proto).

​	Google 内部广泛使用的 proto 定义示例包括 `timestamp.proto` 和 `status.proto` 。

### 更新 Proto 定义而不更新代码 Updating Proto Definitions Without Updating Code 

It’s standard for software products to be backward compatible, but it is less common for them to be forward compatible. As long as you follow some [simple practices](https://protobuf.dev/programming-guides/proto3/#updating) when updating `.proto` definitions, old code will read new messages without issues, ignoring any newly added fields. To the old code, fields that were deleted will have their default value, and deleted repeated fields will be empty. For information on what “repeated” fields are, see [Protocol Buffers Definition Syntax](https://protobuf.dev/overview/#syntax) later in this topic.

​	软件产品向后兼容是标准做法，但它们向前兼容的情况较少。只要您在更新 `.proto` 定义时遵循一些简单的做法，旧代码就会在不出现问题的情况下读取新消息，并忽略任何新添加的字段。对于旧代码，已删除的字段将具有其默认值，已删除的重复字段将为空。有关“重复”字段的详细信息，请参阅本主题后面的“协议缓冲区定义语法”。

New code will also transparently read old messages. New fields will not be present in old messages; in these cases protocol buffers provide a reasonable default value.

​	新代码也将透明地读取旧消息。新字段不会出现在旧消息中；在这些情况下，协议缓冲区会提供合理默认值。

### 协议缓冲区在哪些情况下不合适？ When are Protocol Buffers not a Good Fit? 

Protocol buffers do not fit all data. In particular:

​	协议缓冲区并不适合所有数据。特别是：

- Protocol buffers tend to assume that entire messages can be loaded into memory at once and are not larger than an object graph. For data that exceeds a few megabytes, consider a different solution; when working with larger data, you may effectively end up with several copies of the data due to serialized copies, which can cause surprising spikes in memory usage.
- 协议缓冲区倾向于假设整个消息可以一次加载到内存中，并且不比对象图大。对于超过几兆字节的数据，请考虑不同的解决方案；在处理较大的数据时，由于序列化副本，您实际上可能会得到多个数据副本，这可能会导致内存使用量意外激增。
- When protocol buffers are serialized, the same data can have many different binary serializations. You cannot compare two messages for equality without fully parsing them.
- 当协议缓冲区被序列化时，相同的数据可以有许多不同的二进制序列化。您无法在不完全解析它们的情况下比较两个消息的相等性。
- Messages are not compressed. While messages can be zipped or gzipped like any other file, special-purpose compression algorithms like the ones used by JPEG and PNG will produce much smaller files for data of the appropriate type.
- 消息未压缩。虽然消息可以像任何其他文件一样被压缩或 gzip 压缩，但 JPEG 和 PNG 使用的专用压缩算法会为适当类型的数据生成更小的文件。
- Protocol buffer messages are less than maximally efficient in both size and speed for many scientific and engineering uses that involve large, multi-dimensional arrays of floating point numbers. For these applications, [FITS](https://en.wikipedia.org/wiki/FITS) and similar formats have less overhead.
- 对于涉及大型多维浮点数数组的许多科学和工程用途，协议缓冲区消息在大小和速度方面都低于最大效率。对于这些应用程序，FITS 和类似格式的开销更小。
- Protocol buffers are not well supported in non-object-oriented languages popular in scientific computing, such as Fortran and IDL.
- 协议缓冲区在科学计算中流行的非面向对象语言（如 Fortran 和 IDL）中不受良好支持。
- Protocol buffer messages don’t inherently self-describe their data, but they have a fully reflective schema that you can use to implement self-description. That is, you cannot fully interpret one without access to its corresponding `.proto` file.
- 协议缓冲区消息本身并不描述其数据，但它们具有可用于实现自描述的完全反射模式。也就是说，如果没有访问其对应的 `.proto` 文件，您就无法完全解释一个文件。
- Protocol buffers are not a formal standard of any organization. This makes them unsuitable for use in environments with legal or other requirements to build on top of standards.
- 协议缓冲区不是任何组织的正式标准。这使得它们不适合在具有法律或其他要求以构建在标准之上的环境中使用。

## 谁在使用协议缓冲区？ Who Uses Protocol Buffers? 

Many projects use protocol buffers, including the following:

​	许多项目使用协议缓冲区，包括以下内容：

- [gRPC](https://grpc.io/)
- [Google Cloud](https://cloud.google.com/)
- [Envoy Proxy](https://www.envoyproxy.io/)

## 协议缓冲区如何工作？ How do Protocol Buffers Work? 

The following diagram shows how you use protocol buffers to work with your data.

​	下图显示了如何使用协议缓冲区处理数据。



![img](./overview_img/protocol-buffers-concepts.png)

**Figure 1. Protocol buffers workflow**

​	图 1. 协议缓冲区工作流



The code generated by protocol buffers provides utility methods to retrieve data from files and streams, extract individual values from the data, check if data exists, serialize data back to a file or stream, and other useful functions.

​	协议缓冲区生成的代码提供了实用方法，用于从文件和流中检索数据、从数据中提取各个值、检查数据是否存在、将数据序列化回文件或流，以及其他有用功能。

The following code samples show you an example of this flow in Java. As shown earlier, this is a `.proto` definition:

​	以下代码示例向您展示了 Java 中此流程的一个示例。如前所示，这是一个 `.proto` 定义：

```proto
message Person {
  optional string name = 1;
  optional int32 id = 2;
  optional string email = 3;
}
```

Compiling this `.proto` file creates a `Builder` class that you can use to create new instances, as in the following Java code:

​	编译此 `.proto` 文件会创建一个 `Builder` 类，您可以使用该类创建新实例，如下面的 Java 代码所示：

```java
Person john = Person.newBuilder()
    .setId(1234)
    .setName("John Doe")
    .setEmail("jdoe@example.com")
    .build();
output = new FileOutputStream(args[0]);
john.writeTo(output);
```

You can then deserialize data using the methods protocol buffers creates in other languages, like C++:

​	然后，您可以使用协议缓冲区在其他语言（如 C++）中创建的方法反序列化数据：

```cpp
Person john;
fstream input(argv[1], ios::in | ios::binary);
john.ParseFromIstream(&input);
int id = john.id();
std::string name = john.name();
std::string email = john.email();
```

## 协议缓冲区定义语法 Protocol Buffers Definition Syntax 

When defining `.proto` files, you can specify that a field is either `optional` or `repeated` (proto2 and proto3) or leave it set to the default, implicit presence, in proto3. (The option to set a field to `required` is absent in proto3 and strongly discouraged in proto2. For more on this, see “Required is Forever” in [Specifying Field Rules](https://protobuf.dev/programming-guides/proto3#specifying-field-rules).)

​	在定义 `.proto` 文件时，您可以指定某个字段是 `optional` 或 `repeated` （proto2 和 proto3），或者在 proto3 中将其保留为默认的隐式存在。（在 proto3 中没有将字段设置为 `required` 的选项，在 proto2 中强烈不建议这样做。有关此内容的更多信息，请参阅“Required is Forever”中的指定字段规则。）

After setting the optionality/repeatability of a field, you specify the data type. Protocol buffers support the usual primitive data types, such as integers, booleans, and floats. For the full list, see [Scalar Value Types](https://protobuf.dev/programming-guides/proto3#scalar).

​	设置字段的可选性/可重复性后，您需要指定数据类型。协议缓冲区支持常用的基本数据类型，例如整数、布尔值和浮点数。有关完整列表，请参阅标量值类型。

A field can also be of:

​	字段还可以是：

- A `message` type, so that you can nest parts of the definition, such as for repeating sets of data.
- `message` 类型，以便您可以嵌套定义的部分，例如用于重复的数据集。
- An `enum` type, so you can specify a set of values to choose from.
- `enum` 类型，以便您可以指定要从中选择的数值集。
- A `oneof` type, which you can use when a message has many optional fields and at most one field will be set at the same time.
- `oneof` 类型，当消息具有许多可选字段且最多只能同时设置一个字段时，可以使用此类型。
- A `map` type, to add key-value pairs to your definition.
- `map` 类型，以便向定义中添加键值对。

In proto2, messages can allow **extensions** to define fields outside of the message, itself. For example, the protobuf library’s internal message schema allows extensions for custom, usage-specific options.

​	在 proto2 中，消息可以允许扩展在消息本身之外定义字段。例如，protobuf 库的内部消息架构允许针对特定用途的自定义选项进行扩展。

For more information about the options available, see the language guide for [proto2](https://protobuf.dev/programming-guides/proto2) or [proto3](https://protobuf.dev/programming-guides/proto3).

​	有关可用选项的更多信息，请参阅 proto2 或 proto3 的语言指南。

After setting optionality and field type, you choose a name for the field. There are some things to keep in mind when setting field names:

​	设置可选性和字段类型后，您需要为字段选择一个名称。在设置字段名称时，需要牢记以下几点：

- It can sometimes be difficult, or even impossible, to change field names after they’ve been used in production.
- 在生产中使用字段名称后，有时很难更改，甚至不可能更改。
- Field names cannot contain dashes. For more on field name syntax, see [Message and Field Names](https://protobuf.dev/programming-guides/style#message-field-names).
- 字段名称不能包含破折号。有关字段名称语法的更多信息，请参阅消息和字段名称。
- Use pluralized names for repeated fields.
- 对重复字段使用复数名称。

After assigning a name to the field, you assign a field number. Field numbers cannot be repurposed or reused. If you delete a field, you should reserve its field number to prevent someone from accidentally reusing the number.

​	在为字段分配名称后，您分配一个字段编号。字段编号不能重新使用或重复使用。如果您删除了一个字段，您应该保留其字段编号，以防止有人意外地重复使用该编号。

## 其他数据类型支持 Additional Data Type Support 

Protocol buffers support many scalar value types, including integers that use both variable-length encoding and fixed sizes. You can also create your own composite data types by defining messages that are, themselves, data types that you can assign to a field. In addition to the simple and composite value types, several [common types](https://protobuf.dev/programming-guides/dos-donts#common) are published.

​	协议缓冲区支持许多标量值类型，包括使用可变长度编码和固定大小的整数。您还可以通过定义消息来创建自己的复合数据类型，这些消息本身就是您可以分配给字段的数据类型。除了简单和复合值类型之外，还发布了几种常见类型。

## 历史 History 

To read about the history of the protocol buffers project, see [History of Protocol Buffers](https://protobuf.dev/history).

​	要了解协议缓冲区项目的历史，请参阅协议缓冲区历史。

## 协议缓冲区开源理念 Protocol Buffers Open Source Philosophy 

Protocol buffers were open sourced in 2008 as a way to provide developers outside of Google with the same benefits that we derive from them internally. We support the open source community through regular updates to the language as we make those changes to support our internal requirements. While we accept select pull requests from external developers, we cannot always prioritize feature requests and bug fixes that don’t conform to Google’s specific needs.

​	协议缓冲区于 2008 年开源，目的是向 Google 外部的开发人员提供与我们在内部获得的相同好处。我们通过定期更新语言来支持开源社区，因为我们对这些更改以支持我们的内部要求。虽然我们接受外部开发人员选择的拉取请求，但我们无法始终优先考虑不符合 Google 特定需求的功能请求和错误修复。

## 开发者社区 Developer Community 

To be alerted to upcoming changes in Protocol Buffers and to connect with protobuf developers and users, [join the Google Group](https://groups.google.com/g/protobuf).

​	要了解协议缓冲区的即将进行的更改并与 protobuf 开发人员和用户联系，请加入 Google 群组。

## 其他资源[ ](https://protobuf.dev/overview/#additional-resources) Additional Resources 

- [Protocol Buffers GitHub
  协议缓冲区 GitHub](https://github.com/protocolbuffers/protobuf/)
