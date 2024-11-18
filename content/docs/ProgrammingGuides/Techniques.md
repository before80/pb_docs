+++
title = "技术技巧"
date = 2024-11-17T09:35:36+08:00
weight = 80
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

> 原文：[https://protobuf.dev/programming-guides/techniques/](https://protobuf.dev/programming-guides/techniques/)
>
> 收录该文档的时间：`2024-11-17T09:35:36+08:00`

# Techniques - 技术技巧

Describes some commonly-used design patterns for dealing with Protocol Buffers.

​	介绍处理 Protocol Buffers 的一些常用设计模式。

You can also send design and usage questions to the [Protocol Buffers discussion group](http://groups.google.com/group/protobuf).

​	您还可以将设计和使用问题发送到 [Protocol Buffers 讨论组](http://groups.google.com/group/protobuf)。

## 常见文件名后缀 Common Filename Suffixes

It is fairly common to write messages to files in several different formats. We recommend using the following file extensions for these files.

​	在不同格式的文件中写入消息非常常见。我们建议使用以下文件扩展名：

| Content                                                      | Extension |
| ------------------------------------------------------------ | --------- |
| [Text Format](https://protobuf.dev/reference/protobuf/textformat-spec) | `.txtpb`  |
| [Wire Format](https://protobuf.dev/programming-guides/encoding) | `.binpb`  |
| [JSON Format](https://protobuf.dev/programming-guides/proto3#json) | `.json`   |

For Text Format specifically, `.textproto` is also fairly common, but we recommend `.txtpb` for its brevity.

​	对于文本格式，`.textproto` 也很常见，但我们推荐使用 `.txtpb`，因为它更简洁。

## 流式处理多个消息 Streaming Multiple Messages

If you want to write multiple messages to a single file or stream, it is up to you to keep track of where one message ends and the next begins. The Protocol Buffer wire format is not self-delimiting, so protocol buffer parsers cannot determine where a message ends on their own. The easiest way to solve this problem is to write the size of each message before you write the message itself. When you read the messages back in, you read the size, then read the bytes into a separate buffer, then parse from that buffer. (If you want to avoid copying bytes to a separate buffer, check out the `CodedInputStream` class (in both C++ and Java) which can be told to limit reads to a certain number of bytes.)

​	如果您希望将多个消息写入一个文件或流，您需要自己管理每条消息的起始和结束位置。Protocol Buffer 的线缆格式不是自我定界的，因此 Protocol Buffer 解析器无法自行确定消息的结束位置。最简单的解决方法是先写入每条消息的大小，然后写入消息本身。读取消息时，先读取大小，再将字节读入单独的缓冲区，最后从该缓冲区解析消息。（如果您希望避免将字节复制到单独的缓冲区，可以查看 `CodedInputStream` 类（在 C++ 和 Java 中均可使用），它允许限制读取的字节数。）

## 处理大型数据集 Large Data Sets

Protocol Buffers are not designed to handle large messages. As a general rule of thumb, if you are dealing in messages larger than a megabyte each, it may be time to consider an alternate strategy.

​	Protocol Buffers 设计上不适合处理大型消息。一般来说，如果单条消息大于 1MB，您可能需要考虑其他策略。

That said, Protocol Buffers are great for handling individual messages *within* a large data set. Usually, large data sets are a collection of small pieces, where each small piece is structured data. Even though Protocol Buffers cannot handle the entire set at once, using Protocol Buffers to encode each piece greatly simplifies your problem: now all you need is to handle a set of byte strings rather than a set of structures.

​	尽管如此，Protocol Buffers 非常适合处理大型数据集中的单个消息。通常，大型数据集由多个小数据片段组成，每个小片段是结构化数据。尽管 Protocol Buffers 无法一次性处理整个数据集，但使用 Protocol Buffers 对每个片段进行编码可以大大简化问题：您只需处理一组字节字符串，而不是一组结构。

Protocol Buffers do not include any built-in support for large data sets because different situations call for different solutions. Sometimes a simple list of records will do while other times you want something more like a database. Each solution should be developed as a separate library, so that only those who need it need pay the costs.

​	Protocol Buffers 没有内置对大型数据集的支持，因为不同情况需要不同的解决方案。有时，一个简单的记录列表就够了，而其他时候，您可能需要类似数据库的功能。每种解决方案都应开发为独立库，这样只有需要它的人才需支付相应成本。

## 自描述消息 Self-describing Messages

Protocol Buffers do not contain descriptions of their own types. Thus, given only a raw message without the corresponding `.proto` file defining its type, it is difficult to extract any useful data.

​	Protocol Buffers 不包含其自身类型的描述。因此，仅凭一个未定义类型的原始消息，难以提取任何有用数据。

However, the contents of a .proto file can itself be represented using protocol buffers. The file `src/google/protobuf/descriptor.proto` in the source code package defines the message types involved. `protoc` can output a `FileDescriptorSet`—which represents a set of .proto files—using the `--descriptor_set_out` option. With this, you can define a self-describing protocol message like so:

​	然而，`.proto` 文件的内容本身可以用 Protocol Buffers 表示。源码包中的 `src/google/protobuf/descriptor.proto` 定义了相关消息类型。`protoc` 可以使用 `--descriptor_set_out` 选项输出一个 `FileDescriptorSet`，它表示一组 `.proto` 文件。基于此，您可以定义如下的自描述协议消息：

```proto
syntax = "proto3";

import "google/protobuf/any.proto";
import "google/protobuf/descriptor.proto";

message SelfDescribingMessage {
  // Set of FileDescriptorProtos which describe the type and its dependencies.
  google.protobuf.FileDescriptorSet descriptor_set = 1;

  // The message and its type, encoded as an Any message.
  google.protobuf.Any message = 2;
}
```

By using classes like `DynamicMessage` (available in C++ and Java), you can then write tools which can manipulate `SelfDescribingMessage`s.

​	通过使用类似 `DynamicMessage`（可用于 C++ 和 Java）的类，您可以编写工具来操作 `SelfDescribingMessage`。

All that said, the reason that this functionality is not included in the Protocol Buffer library is because we have never had a use for it inside Google.

​	尽管如此，Protocol Buffer 库中没有包含这种功能的原因是，在 Google 内部我们从未需要过此功能。

This technique requires support for dynamic messages using descriptors. Check that your platforms support this feature before using self-describing messages.

​	此技术需要使用描述符支持动态消息。在使用自描述消息之前，请确认您的平台支持该功能。
