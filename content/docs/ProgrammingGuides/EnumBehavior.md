+++
title = "枚举行为"
date = 2024-11-17T09:35:36+08:00
weight = 50
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

> 原文：[https://protobuf.dev/programming-guides/enum/](https://protobuf.dev/programming-guides/enum/)
>
> 收录该文档的时间：`2024-11-17T09:35:36+08:00`

# Enum Behavior - 枚举行为

Explains how enums currently work in Protocol Buffers vs. how they should work.

​	解释 Protocol Buffers 中枚举当前的工作方式与理想工作方式之间的差异。

Enums behave differently in different language libraries. This topic covers the different behaviors as well as the plans to move protobufs to a state where they are consistent across all languages. If you’re looking for information on how to use enums in general, see the corresponding sections in the [proto2](https://protobuf.dev/programming-guides/proto2#enum) and [proto3](https://protobuf.dev/programming-guides/proto3#enum) language guide topics.

​	枚举在不同语言的库中表现不同。本主题涵盖这些不同的行为以及将 protobuf 移动到所有语言中一致状态的计划。如果您想了解如何使用枚举的一般信息，请参阅 [proto2](https://protobuf.dev/programming-guides/proto2#enum) 和 [proto3](https://protobuf.dev/programming-guides/proto3#enum) 语言指南的相应部分。

## 定义 Definitions

Enums have two distinct flavors (*open* and *closed*). They behave identically except in their handling of unknown values. Practically, this means that simple cases work the same, but some corner cases have interesting implications.

​	枚举有两种不同的风格（*开放* 和 *关闭*）。它们在处理未知值时的行为有所不同。实际上，这意味着简单的情况表现相同，但某些边界情况会产生有趣的影响。

For the purpose of explanation, let us assume we have the following `.proto` file (we are deliberately not specifying if this is a `syntax = "proto2"` or `syntax = "proto3"` file right now):

​	为了解释方便，我们假设有以下 `.proto` 文件（目前我们故意不指定这是 `syntax = "proto2"` 还是 `syntax = "proto3"` 的文件）：

```gdscript3
enum Enum {
  A = 0;
  B = 1;
}

message Msg {
  optional Enum enum = 1;
}
```

The distinction between *open* and *closed* can be encapsulated in a single question:

​	*开放* 和 *关闭* 的区别可以用一个问题概括：

> What happens when a program parses binary data that contains field 1 with the value `2`?
>
> ​	当程序解析包含字段 1 且值为 `2` 的二进制数据时会发生什么？

- **Open** enums will parse the value `2` and store it directly in the field. Accessor will report the field as being *set* and will return something that represents `2`.
  - **开放**枚举会解析值 `2` 并直接存储在字段中。访问器会报告该字段已被设置，并返回表示 `2` 的值。

- **Closed** enums will parse the value `2` and store it in the message’s unknown field set. Accessors will report the field as being *unset* and will return the enum’s default value.
  - **关闭**枚举会解析值 `2` 并将其存储在消息的未知字段集中。访问器会报告该字段未被设置，并返回枚举的默认值。


## *关闭* 枚举的影响 Implications of *Closed* Enums

The behavior of *closed* enums has unexpected consequences when parsing a repeated field. When a `repeated Enum` field is parsed all unknown values will be placed in the [unknown field](https://protobuf.dev/programming-guides/proto3/#unknowns) set. When it is serialized those unknown values will be written again, *but not in their original place in the list*. For example, given the `.proto` file:

​	当解析重复字段时，*关闭* 枚举的行为会产生意外后果。当解析 `repeated Enum` 字段时，所有未知值将被放入[未知字段集](https://protobuf.dev/programming-guides/proto3/#unknowns)。在序列化时，这些未知值会再次写入，但*不会保留它们在列表中的原始位置*。例如，给定以下 `.proto` 文件：

```gdscript3
enum Enum {
  A = 0;
  B = 1;
}

message Msg {
  repeated Enum r = 1;
}
```

A wire format containing the values `[0, 2, 1, 2]` for field 1 will parse so that the repeated field contains `[0, 1]` and the value `[2, 2]` will end up stored as an unknown field. After reserializing the message, the wire format will correspond to `[0, 1, 2, 2]`.

​	包含字段 1 值为 `[0, 2, 1, 2]` 的线格式会解析为重复字段包含 `[0, 1]`，而值 `[2, 2]` 将存储为未知字段。重新序列化消息后，线格式将对应 `[0, 1, 2, 2]`。

Similarly, maps with *closed* enums for their value will place entire entries (key and value) in the unknown fields whenever the value is unknown.

​	类似地，如果映射的值是 *关闭* 枚举，当值未知时，整个键值对（键和值）将被放入未知字段中。

## 历史 History

Prior to the introduction of `syntax = "proto3"` all enums were *closed*. Proto3 introduced *open* enums specifically because of the unexpected behavior that *closed* enums cause.

​	在引入 `syntax = "proto3"` 之前，所有枚举都是 *关闭* 的。Proto3 专门引入了 *开放* 枚举，以解决 *关闭* 枚举引发的意外行为。

## 规范 Specification

The following specifies the behavior of conformant implementations for protobuf. Because this is subtle, many implementations are out of conformance. See [Known Issues](https://protobuf.dev/programming-guides/enum/#known-issues) for details on how different implementations behave.

​	以下内容规范了 protobuf 的合规实现行为。由于该主题比较微妙，许多实现不符合规范。有关不同实现的行为详细信息，请参阅 [已知问题](https://protobuf.dev/programming-guides/enum/#known-issues)。

- When a `proto2` file imports an enum defined in a `proto2` file, that enum should be treated as **closed**.
  - 当 `proto2` 文件导入定义在 `proto2` 文件中的枚举时，该枚举应被视为 **关闭**。

- When a `proto3` file imports an enum defined in a `proto3` file, that enum should be treated as **open**.
  - 当 `proto3` 文件导入定义在 `proto3` 文件中的枚举时，该枚举应被视为 **开放**。

- When a `proto3` file imports an enum defined in a `proto2` file, the `protoc` compiler will produce an error.
  - 当 `proto3` 文件导入定义在 `proto2` 文件中的枚举时，`protoc` 编译器会产生错误。

- When a `proto2` file imports an enum defined in a `proto3` file, that enum should be treated as **open**.
  - 当 `proto2` 文件导入定义在 `proto3` 文件中的枚举时，该枚举应被视为 **开放**。


## 已知问题 Known Issues

### C++

All known C++ releases are out of conformance. When a `proto2` file imports an enum defined in a `proto3` file, C++ treats that field as a **closed** enum. Under editions, this behavior is represented by the deprecated field feature [`features.(pb.cpp).legacy_closed_enum`](https://protobuf.dev/editions/features#legacy_closed_enum). There are two options for moving to conformant behavior:

​	所有已知的 C++ 版本均不符合规范。当 `proto2` 文件导入定义在 `proto3` 文件中的枚举时，C++ 将该字段视为 **关闭** 枚举。这种行为由已弃用的字段特性 [`features.(pb.cpp).legacy_closed_enum`](https://protobuf.dev/editions/features#legacy_closed_enum) 表示。迁移到符合规范行为的两种方法：

- Remove the field feature. This is the recommended approach, but may cause runtime behavior changes. Without the feature, unrecognized integers will end up stored in the field cast to the enum type instead of being put into the unknown field set.
  - 删除字段特性。这是推荐的方法，但可能导致运行时行为变化。删除特性后，未识别的整数将存储在字段中，并被强制转换为枚举类型，而不是被放入未知字段集中。

- Change the enum to closed. This is discouraged, and can cause runtime behavior if *anybody else* is using the enum. Unrecognized integers will end up in the unknown field set instead of those fields.
  - 将枚举更改为关闭。不推荐这样做，因为如果**任何其他用户**使用了该枚举，他们可能会遇到运行时行为变化。未识别的整数将存储在未知字段集中，而不是对应字段中。


### C#

All known C# releases are out of conformance. C# treats all enums as **open**.

​	所有已知的 C# 版本均不符合规范。C# 将所有枚举视为 **开放**。

### Java

All known Java releases are out of conformance. When a `proto2` file imports an enum defined in a `proto3` file, Java treats that field as a **closed** enum.

​	所有已知的 Java 版本均不符合规范。当 `proto2` 文件导入一个定义在 `proto3` 文件中的枚举时，Java 会将该字段视为 **关闭** 枚举。

Under editions, this behavior is represented by the deprecated field feature [`features.(pb.java).legacy_closed_enum`](https://protobuf.dev/editions/features#legacy_closed_enum)). There are two options for moving to conformant behavior:

​	在版本管理（editions）中，这种行为由已弃用的字段特性 [`features.(pb.java).legacy_closed_enum`](https://protobuf.dev/editions/features#legacy_closed_enum) 表示。迁移到符合规范的行为有两种选择：

- Remove the field feature. This may cause runtime behavior changes. Without the feature, unrecognized integers will end up stored in the field and the `UNRECOGNIZED` value will be returned by the enum getter. Before, these values would be put into the unknown field set.
  - **删除字段特性。**这可能会导致运行时行为变化。没有该特性时，未识别的整数将存储在字段中，并通过枚举 getter 返回 `UNRECOGNIZED` 值。此前，这些值会被放入未知字段集。

- Change the enum to closed. If *anybody else* is using it they may see runtime behavior changes. Unrecognized integers will end up in the unknown field set instead of those fields.
  - **将枚举更改为关闭。**如果*其他任何用户*正在使用该枚举，他们可能会看到运行时行为变化。未识别的整数将存储在未知字段集中，而不是字段中。

> **NOTE:** Java’s handling of **open** enums has surprising edge cases. Given the following definitions:
>
> ​	Java 对 **开放** 枚举的处理有一些令人惊讶的边界情况。以下定义为例：
>
> ```gdscript3
> syntax = "proto3";
> 
> enum Enum {
> A = 0;
> B = 1;
> }
> 
> message Msg {
> repeated Enum name = 1;
> }
> ```
>
> Java will generate the methods `Enum getName()` and `int getNameValue()`. The method `getName` will return `Enum.UNRECOGNIZED` for values outside the known set (such as `2`), whereas `getNameValue` will return `2`.
>
> ​	Java 会生成方法 `Enum getName()` 和 `int getNameValue()`。对于超出已知集合的值（例如 `2`），`getName` 方法将返回 `Enum.UNRECOGNIZED`，而 `getNameValue` 方法将返回 `2`。
>
> Similarly, Java will generate methods `Builder setName(Enum value)` and `Builder setNameValue(int value)`. The method `setName` will throw an exception when passed `Enum.UNRECOGNIZED`, whereas `setNameValue` will accept `2`.
>
> ​	类似地，Java 会生成方法 `Builder setName(Enum value)` 和 `Builder setNameValue(int value)`。当传入 `Enum.UNRECOGNIZED` 时，`setName` 方法会抛出异常，而 `setNameValue` 方法会接受 `2`。

### Kotlin

All known Kotlin releases are out of conformance. When a `proto2` file imports an enum defined in a `proto3` file, Kotlin treats that field as a **closed** enum.

​	所有已知的 Kotlin 版本均不符合规范。当 `proto2` 文件导入一个定义在 `proto3` 文件中的枚举时，Kotlin 会将该字段视为 **关闭** 枚举。

Kotlin is built on Java and shares all of its oddities.

​	Kotlin 基于 Java，因此与其共享所有特性。

### Go

All known Go releases are out of conformance. Go treats all enums as **open**.

​	所有已知的 Go 版本均不符合规范。Go 将所有枚举视为 **开放**。

### JSPB

All known JSPB releases are out of conformance. JSPB treats all enums as **open**.

​	所有已知的 JSPB 版本均不符合规范。JSPB 将所有枚举视为 **开放**。

### PHP

PHP is conformant.

​	PHP 符合规范。

### Python

After 4.22.0, Python is conformant.

​	**4.22.0** 及以后版本符合规范。

In 4.21.x, Python is conformant by default, but setting `PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=python` will cause it to be out of conformance.

​	在 **4.21.x** 中，Python 默认符合规范，但设置 `PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=python` 会导致不符合规范。

Before 4.21.0, Python is out of conformance.

​	**4.21.0** 之前的版本不符合规范。

When a `proto2` file imports an enum defined in a `proto3` file, non-conformant Python versions treat that field as a **closed** enum.

​	当 `proto2` 文件导入一个定义在 `proto3` 文件中的枚举时，不符合规范的 Python 版本会将该字段视为 **关闭** 枚举。

### Ruby

All known Ruby releases are out of conformance. Ruby treats all enums as **open**.

​	所有已知的 Ruby 版本均不符合规范。Ruby 将所有枚举视为 **开放**。

### Objective-C

After 22.0, Objective-C is conformant.

​	**22.0** 及以后版本符合规范。

Prior to 22.0, Objective-C was out of conformance. When a `proto2` file imported an enum defined in a `proto3` file, it would treat that field as a **closed** enum.

​	在 **22.0** 之前的版本中，Objective-C 不符合规范。当 `proto2` 文件导入一个定义在 `proto3` 文件中的枚举时，它会将该字段视为 **关闭** 枚举。

### Swift

Swift is conformant.

​	Swift 符合规范。

### Dart

Dart treats all enums as **closed**.

​	Dart 将所有枚举视为 **关闭**。
