+++
title = "技术"
weight = 70
description = "描述了一些用于处理 Protocol Buffers 的常用设计模式。"
type = "docs"

+++

## Techniques 技术

Describes some commonly-used design patterns for dealing with Protocol Buffers.

​	描述了一些用于处理 Protocol Buffers 的常用设计模式。



You can also send design and usage questions to the [Protocol Buffers discussion group](http://groups.google.com/group/protobuf).

​	您还可以将设计和使用问题发送到 Protocol Buffers 讨论组。

## 通用文件名后缀 Common Filename Suffixes 

It is fairly common to write messages to files in several different formats. We recommend using the following file extensions for these files.

​	将消息写入多种格式的文件中非常常见。我们建议对这些文件使用以下文件扩展名。

| Content 内容                                                 | Extension 扩展 |
| ------------------------------------------------------------ | -------------- |
| [Text Format 文本格式](https://protobuf.dev/reference/protobuf/textformat-spec) | `.txtpb`       |
| [Wire Format 线格式](https://protobuf.dev/programming-guides/encoding) | `.binpb`       |
| [JSON Format JSON 格式](https://protobuf.dev/programming-guides/proto3#json) | `.json`        |

For Text Format specifically, `.textproto` is also fairly common, but we recommend `.txtpb` for its brevity.

​	对于文本格式， `.textproto` 也相当常见，但我们推荐使用 `.txtpb` ，因为它更简洁。

## 流式传输多条消息 Streaming Multiple Messages 

If you want to write multiple messages to a single file or stream, it is up to you to keep track of where one message ends and the next begins. The Protocol Buffer wire format is not self-delimiting, so protocol buffer parsers cannot determine where a message ends on their own. The easiest way to solve this problem is to write the size of each message before you write the message itself. When you read the messages back in, you read the size, then read the bytes into a separate buffer, then parse from that buffer. (If you want to avoid copying bytes to a separate buffer, check out the `CodedInputStream` class (in both C++ and Java) which can be told to limit reads to a certain number of bytes.)

​	如果您想将多条消息写入单个文件或流，则需要自行跟踪一条消息的结束位置和下一条消息的开始位置。协议缓冲区线格式不是自定界符的，因此协议缓冲区解析器无法自行确定消息的结束位置。解决此问题的最简单方法是在写入消息本身之前写入每条消息的大小。在读回消息时，您会读取大小，然后将字节读入单独的缓冲区，然后从该缓冲区进行解析。（如果您想避免将字节复制到单独的缓冲区，请查看 `CodedInputStream` 类（在 C++ 和 Java 中均有），可以告知该类将读取限制为一定数量的字节。）

## 大型数据集 Large Data Sets 

Protocol Buffers are not designed to handle large messages. As a general rule of thumb, if you are dealing in messages larger than a megabyte each, it may be time to consider an alternate strategy.

​	协议缓冲区并非设计用于处理大型消息。一般来说，如果您处理的消息每个都大于一兆字节，则可能需要考虑采用替代策略。

That said, Protocol Buffers are great for handling individual messages *within* a large data set. Usually, large data sets are a collection of small pieces, where each small piece is structured data. Even though Protocol Buffers cannot handle the entire set at once, using Protocol Buffers to encode each piece greatly simplifies your problem: now all you need is to handle a set of byte strings rather than a set of structures.

​	话虽如此，Protocol Buffers 非常适合处理大型数据集中的各个消息。通常，大型数据集是由小块数据组成的集合，其中每小块数据都是结构化数据。即使 Protocol Buffers 无法一次处理整个数据集，但使用 Protocol Buffers 对每小块数据进行编码可以极大地简化问题：现在您只需要处理一组字节字符串，而不是一组结构。

Protocol Buffers do not include any built-in support for large data sets because different situations call for different solutions. Sometimes a simple list of records will do while other times you want something more like a database. Each solution should be developed as a separate library, so that only those who need it need pay the costs.

​	Protocol Buffers 不包含对大型数据集的任何内置支持，因为不同的情况需要不同的解决方案。有时，简单的记录列表就足够了，而其他时候您可能需要更类似于数据库的东西。每个解决方案都应作为一个单独的库进行开发，这样只有需要它的人才需要支付成本。

## 自描述消息 Self-describing Messages 

Protocol Buffers do not contain descriptions of their own types. Thus, given only a raw message without the corresponding `.proto` file defining its type, it is difficult to extract any useful data.

​	Protocol Buffers 不包含其自身类型的描述。因此，仅给定一个原始消息而没有定义其类型的相应 `.proto` 文件，就很难提取任何有用的数据。

However, the contents of a .proto file can itself be represented using protocol buffers. The file `src/google/protobuf/descriptor.proto` in the source code package defines the message types involved. `protoc` can output a `FileDescriptorSet`—which represents a set of .proto files—using the `--descriptor_set_out` option. With this, you can define a self-describing protocol message like so:

​	然而，.proto 文件的内容本身可以使用协议缓冲区表示。源代码包中的文件 `src/google/protobuf/descriptor.proto` 定义了涉及的消息类型。 `protoc` 可以使用 `--descriptor_set_out` 选项输出一个 `FileDescriptorSet` —它表示一组 .proto 文件。有了这个，您可以定义一个自描述的协议消息，如下所示：

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

​	通过使用 `DynamicMessage` 等类（在 C++ 和 Java 中可用），您可以编写可以操作 `SelfDescribingMessage` 的工具。

All that said, the reason that this functionality is not included in the Protocol Buffer library is because we have never had a use for it inside Google.

​	尽管如此，此功能未包含在 Protocol Buffer 库中的原因是我们从未在 Google 内部使用过它。

This technique requires support for dynamic messages using descriptors. Check that your platforms support this feature before using self-describing messages.

​	此技术需要使用描述符对动态消息提供支持。在使用自描述消息之前，请检查您的平台是否支持此功能。
