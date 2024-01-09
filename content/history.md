+++
title = "History"
weight = 1020
description = "A brief history behind the creation of protocol buffers."
type = "docs"
+++

## History 历史

A brief history behind the creation of protocol buffers.

​	协议缓冲区创建背后的简短历史。



Understanding why protobuf was created and the decisions that changed it over time can help you to better use the features of the tool.

​	了解 protobuf 创建的原因以及随着时间推移而改变它的决策，可以帮助您更好地使用该工具的功能。

## 为什么发布 Protocol Buffers？ Why Did You Release Protocol Buffers? 

There are several reasons that we released Protocol Buffers.

​	我们发布 Protocol Buffers 有几个原因。

Protocol buffers are used by many projects inside Google. We had other projects we wanted to release as open source that use protocol buffers, so to do this, we needed to release protocol buffers first. In fact, bits of the technology had already found their way into the open; if you dig into the code for Google AppEngine, you might find some of it.

​	许多 Google 内部项目都使用 Protocol Buffers。我们还有其他项目想要作为开源项目发布，这些项目使用 Protocol Buffers，因此，要做到这一点，我们需要先发布 Protocol Buffers。事实上，这项技术的部分内容已经以开源形式发布；如果您深入研究 Google AppEngine 的代码，可能会发现其中的一些内容。

We wanted to provide public APIs that accept protocol buffers as well as XML, both because it is more efficient and because we convert that XML to protocol buffers on our end, anyway.

​	我们希望提供同时接受 Protocol Buffers 和 XML 的公共 API，原因是这两种方式都更高效，而且无论如何，我们都会将 XML 转换为 Protocol Buffers。

We thought that people outside Google might find protocol buffers useful. Getting protocol buffers into a form we were happy to release was a fun side project.

​	我们认为，Google 以外的人可能会发现 Protocol Buffers 很实用。将 Protocol Buffers 变成我们乐于发布的形式是一个有趣的副项目。

## 为什么第一个发布版本是第 2 版？第 1 版发生了什么？ Why Is the First Release Version 2? What Happened to Version 1? 

The initial version of protocol buffers (“Proto1”) was developed starting in early 2001 and evolved over the course of many years, sprouting new features whenever someone needed them and was willing to do the work to create them. Like anything created in such a way, it was a bit of a mess. We came to the conclusion that it would not be feasible to release the code as it was.

​	Protocol Buffers 的初始版本（“Proto1”）从 2001 年初开始开发，并在很多年里不断演变，每当有人需要新功能并愿意为此付出创建工作时，就会添加新功能。像任何以这种方式创建的东西一样，它有点乱。我们得出结论，按原样发布代码是不可行的。

Version 2 (“Proto2”) was a complete rewrite, though it kept most of the design and used many of the implementation ideas from Proto1. Some features were added, some removed. Most importantly, though, the code was cleaned up and did not have any dependencies on Google libraries that were not yet open-sourced.

​	版本 2（“Proto2”）是一次彻底的重写，尽管它保留了大部分设计并使用了 Proto1 的许多实现思路。添加了一些功能，删除了一些功能。但最重要的是，代码经过了清理，并且没有任何依赖尚未开源的 Google 库。

## 为什么叫“Protocol Buffers”？ Why the Name “Protocol Buffers”? 

The name originates from the early days of the format, before we had the protocol buffer compiler to generate classes for us. At the time, there was a class called `ProtocolBuffer` that actually acted as a buffer for an individual method. Users would add tag/value pairs to this buffer individually by calling methods like `AddValue(tag, value)`. The raw bytes were stored in a buffer that could then be written out once the message had been constructed.

​	这个名称源自该格式的早期，那时我们还没有协议缓冲区编译器为我们生成类。当时，有一个名为 `ProtocolBuffer` 的类，它实际上充当了单个方法的缓冲区。用户可以通过调用 `AddValue(tag, value)` 等方法将标签/值对逐个添加到此缓冲区。原始字节存储在一个缓冲区中，然后在构建消息后可以将其写出。

Since that time, the “buffers” part of the name has lost its meaning, but it is still the name we use. Today, people usually use the term “protocol message” to refer to a message in an abstract sense, “protocol buffer” to refer to a serialized copy of a message, and “protocol message object” to refer to an in-memory object representing the parsed message.

​	从那时起，名称中的“缓冲区”部分已经失去了其含义，但它仍然是我们使用的名称。如今，人们通常使用术语“协议消息”来指代抽象意义上的消息，“协议缓冲区”来指代消息的序列化副本，以及“协议消息对象”来指代表示已解析消息的内存中对象。

## Google 是否拥有任何有关 Protocol Buffers 的专利？ Does Google Have Any Patents on Protocol Buffers? 

Google currently has no issued patents on protocol buffers, and we are happy to address any concerns around protocol buffers and patents that people may have.

​	谷歌目前没有发布有关协议缓冲区的专利，我们很乐意解决人们可能对协议缓冲区和专利提出的任何担忧。

##  协议缓冲区与 ASN.1、COM、CORBA 和 Thrift 有何不同？ How Do Protocol Buffers Differ from ASN.1, COM, CORBA, and Thrift?

We think all of these systems have strengths and weaknesses. Google relies on protocol buffers internally and they are a vital component of our success, but that doesn’t mean they are the ideal solution for every problem. You should evaluate each alternative in the context of your own project.

​	我们认为所有这些系统都有优点和缺点。谷歌在内部依赖协议缓冲区，它们是我们成功的关键组成部分，但这并不意味着它们是解决每个问题的理想解决方案。您应该在您自己的项目背景下评估每个备选方案。

It is worth noting, though, that several of these technologies define both an interchange format and an RPC (remote procedure call) protocol. Protocol buffers are just an interchange format. They could easily be used for RPC—and, indeed, they do have limited support for defining RPC services—but they are not tied to any one RPC implementation or protocol.

​	不过，值得注意的是，其中一些技术既定义了交换格式，也定义了 RPC（远程过程调用）协议。协议缓冲区只是交换格式。它们可以很容易地用于 RPC——事实上，它们确实对定义 RPC 服务有限支持——但它们不绑定到任何一个 RPC 实现或协议。
