+++
title = "Protocol Buffers"
weight = 5
toc_hide = "true"
description = "Protocol Buffers are language-neutral, platform-neutral extensible mechanisms for serializing structured data."
type = "docs"
no_list = "true"
+++

# Protocol Buffers

Protocol Buffers are language-neutral, platform-neutral extensible mechanisms for serializing structured data.

​	协议缓冲区是用于序列化结构化数据的与语言无关、与平台无关的可扩展机制。

```proto
message Person {
  optional string name = 1;
  optional int32 id = 2;
  optional string email = 3;
}
```

A proto definition.

​	proto 定义。

```java
// Java code
Person john = Person.newBuilder()
    .setId(1234)
    .setName("John Doe")
    .setEmail("jdoe@example.com")
    .build();
output = new FileOutputStream(args[0]);
john.writeTo(output);
```

Using a generated class to persist data.

​	使用生成的类来保存数据。

```cpp
// C++ code
Person john;
fstream input(argv[1],
    ios::in | ios::binary);
john.ParseFromIstream(&input);
id = john.id();
name = john.name();
email = john.email();
```

Using a generated class to parse persisted data.

​	使用生成的类来解析持久化数据。

## 协议缓冲区是什么？ What Are Protocol Buffers? 

Protocol buffers are Google’s language-neutral, platform-neutral, extensible mechanism for serializing structured data – think XML, but smaller, faster, and simpler. You define how you want your data to be structured once, then you can use special generated source code to easily write and read your structured data to and from a variety of data streams and using a variety of languages.

​	协议缓冲区是 Google 用于序列化结构化数据的与语言无关、与平台无关的可扩展机制——类似于 XML，但更小、更快、更简单。您只需定义一次想要如何构建数据结构，然后即可使用特殊生成的源代码轻松地将结构化数据写入各种数据流并从中读取数据，还可以使用各种语言。

## 选择您喜欢的语言 Pick Your Favorite Language 

Protocol buffers support generated code in C++, C#, Dart, Go, Java, Kotlin, Objective-C, Python, and Ruby. With proto3, you can also work with PHP.

​	协议缓冲区支持 C++、C#、Dart、Go、Java、Kotlin、Objective-C、Python 和 Ruby 中生成的代码。使用 proto3，您还可以使用 PHP。

## 如何开始？ How Do I Start? 

1. [Download and install](https://github.com/protocolbuffers/protobuf#protobuf-compiler-installation) the protocol buffer compiler.
2. 下载并安装协议缓冲区编译器。
3. Read the [overview](https://protobuf.dev/overview).
4. 阅读概述。
5. Try the [tutorial](https://protobuf.dev/getting-started) for your chosen language.
6. 尝试您所选语言的教程。
