+++
title = "枚举行为"
weight = 55
description = "说明了枚举在 Protocol Buffers 中的当前工作方式与它们应该工作的方式之间的差异。"
type = "docs"

+++

> 原文网址： https://protobuf.dev/programming-guides/enum/

## Enum Behavior 枚举行为

Explains how enums currently work in Protocol Buffers vs. how they should work.

​	说明了枚举在 Protocol Buffers 中的当前工作方式与它们应该工作的方式之间的差异。



Enums behave differently in different language libraries. This topic covers the different behaviors as well as the plans to move protobufs to a state where they are consistent across all languages. If you’re looking for information on how to use enums in general, see the corresponding sections in the [proto2](https://protobuf.dev/programming-guides/proto#enum) and [proto3](https://protobuf.dev/programming-guides/proto3#enum) language guide topics.

​	枚举在不同的语言库中表现不同。本主题涵盖了不同的行为以及将 protobufs 移至所有语言中都一致的状态的计划。如果您正在寻找有关如何使用枚举的一般信息，请参阅 proto2 和 proto3 语言指南主题中的相应部分。

## 定义 Definitions 

Enums have two distinct flavors (*open* and *closed*). They behave identically except in their handling of unknown values. Practically, this means that simple cases work the same, but some corner cases have interesting implications.

​	枚举具有两种不同的类型（开放和封闭）。除了处理未知值的方式外，它们的行为完全相同。实际上，这意味着简单的情况工作方式相同，但一些特殊情况会产生有趣的影响。

For the purpose of explanation, let us assume we have the following `.proto` file (we are deliberately not specifying if this is a `syntax = "proto2"` or `syntax = "proto3"` file right now):

​	为了解释，让我们假设我们有以下 `.proto` 文件（我们现在故意不指定这是 `syntax = "proto2"` 还是 `syntax = "proto3"` 文件）：

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

​	开放和封闭之间的区别可以用一个问题概括：

> What happens when a program parses binary data that contains field 1 with the value `2`?
>
> ​	当程序解析包含值为 `2` 的字段 1 的二进制数据时会发生什么？

- **Open** enums will parse the value `2` and store it directly in the field. Accessor will report the field as being *set* and will return something that represents `2`.
- 开放枚举将解析值 `2` 并将其直接存储在字段中。访问器将报告该字段已设置，并将返回表示 `2` 的内容。
- **Closed** enums will parse the value `2` and store it in the message’s unknown field set. Accessors will report the field as being *unset* and will return the enum’s default value.
- 封闭枚举将解析值 `2` 并将其存储在消息的未知字段集中。访问器将报告该字段未设置，并将返回枚举的默认值。

## 封闭枚举的影响 Implications of *Closed* Enums 

The behavior of *closed* enums has unexpected consquences when parsing a repeated field. When a `repeated Enum` field is parsed all unknown values will be placed in the [unknown field](https://protobuf.dev/programming-guides/proto3/#unknowns) set. When it is serialized those unknown values will be written again, *but not in their original place in the list*. For example, given the `.proto` file:

​	解析重复字段时，封闭枚举的行为具有意外的后果。当解析 `repeated Enum` 字段时，所有未知值都将被置于未知字段集中。序列化时，这些未知值将再次被写入，但不会写入它们在列表中的原始位置。例如，给定 `.proto` 文件：

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

​	包含字段 1 的值 `[0, 2, 1, 2]` 的线格式将解析，以便重复字段包含 `[0, 1]` ，而值 `[2, 2]` 最终将存储为未知字段。重新序列化消息后，线格式将对应于 `[0, 1, 2, 2]` 。

Similarly, maps with *closed* enums for their value will place entire entries (key and value) in the unknown fields whenever the value is unknown.

​	同样，当值未知时，具有封闭枚举作为其值的映射会将整个条目（键和值）置于未知字段中。

## 历史 History 

Prior to the introduction of `syntax = "proto3"` all enums were *closed*. Proto3 introduced *open* enums specifically because of the unexpected behavior that *closed* enums cause.

​	在引入 `syntax = "proto3"` 之前，所有枚举都是封闭的。Proto3 特别引入了开放枚举，因为封闭枚举会导致意外行为。

## 规范 Specification 

The following specifies the behavior of conformant implementations for protobuf. Because this is subtle, many implementations are out of conformance. See [Known Issues](https://protobuf.dev/programming-guides/enum/#known-issues) for details on how different implementations behave.

​	以下指定了 protobuf 兼容实现的行为。由于这一点很微妙，因此许多实现都不符合规范。有关不同实现的行为的详细信息，请参阅已知问题。

- When a `proto2` file imports an enum defined in a `proto2` file, that enum should be treated as **closed**.
- 当 `proto2` 文件导入在 `proto2` 文件中定义的枚举时，该枚举应被视为已关闭。
- When a `proto3` file imports an enum defined in a `proto3` file, that enum should be treated as **open**.
- 当 `proto3` 文件导入在 `proto3` 文件中定义的枚举时，该枚举应被视为已打开。
- When a `proto3` file imports an enum defined in a `proto2` file, the `protoc` compiler will produce an error.
- 当 `proto3` 文件导入在 `proto2` 文件中定义的枚举时， `protoc` 编译器将产生错误。
- When a `proto2` file imports an enum defined in a `proto3` file, that enum should be treated as **open**.
- 当 `proto2` 文件导入在 `proto3` 文件中定义的枚举时，该枚举应被视为已打开。

## 已知问题 Known Issues 

### C++

All known C++ releases are out of conformance. When a `proto2` file imports an enum defined in a `proto3` file, C++ treats that field as a **closed** enum.

​	所有已知的 C++ 版本均不符合规范。当 `proto2` 文件导入在 `proto3` 文件中定义的枚举时，C++ 将该字段视为已关闭枚举。

### C#

All known C# releases are out of conformance. C# treats all enums as **open**.

​	所有已知的 C# 版本均不符合规范。C# 将所有枚举视为已打开。

### Java

All known Java releases are out of conformance. When a `proto2` file imports an enum defined in a `proto3` file, Java treats that field as a **closed** enum.

​	所有已知的 Java 版本均不符合规范。当 `proto2` 文件导入在 `proto3` 文件中定义的枚举时，Java 将该字段视为已关闭枚举。

> **NOTE:** Java’s handling of **open** enums has surprising edge cases. Given the following definitions:
>
> ​	注意：Java 对开放枚举的处理存在令人惊讶的边缘情况。给定以下定义：
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
> ​	Java 将生成方法 `Enum getName()` 和 `int getNameValue()` 。方法 `getName` 将为已知集合之外的值（例如 `2` ）返回 `Enum.UNRECOGNIZED` ，而 `getNameValue` 将返回 `2` 。
>
> Similarly, Java will generate methods `Builder setName(Enum value)` and `Builder setNameValue(int value)`. The method `setName` will throw an exception when passed `Enum.UNRECOGNIZED`, whereas `setNameValue` will accept `2`.
>
> ​	同样，Java 将生成方法 `Builder setName(Enum value)` 和 `Builder setNameValue(int value)` 。方法 `setName` 在传递 `Enum.UNRECOGNIZED` 时会抛出异常，而 `setNameValue` 将接受 `2` 。

### Kotlin

All known Kotlin releases are out of conformance. When a `proto2` file imports an enum defined in a `proto3` file, Kotlin treats that field as a **closed** enum.

​	所有已知的 Kotlin 版本都不符合规范。当 `proto2` 文件导入在 `proto3` 文件中定义的枚举时，Kotlin 将该字段视为封闭枚举。

Kotlin is built on Java and shares all of its oddities.

​	Kotlin 基于 Java 构建，并共享其所有怪异之处。

### Go

All known Go releases are out of conformance. Go treats all enums as **open**.

​	所有已知的 Go 版本都不符合规范。Go 将所有枚举视为开放枚举。

### JSPB

All known JSPB releases are out of conformance. JSPB treats all enums as **open**.

​	所有已知的 JSPB 版本都不符合规范。JSPB 将所有枚举视为开放枚举。

### PHP

PHP is conformant.

​	PHP 符合规范。

### Python

After 4.22.0, Python is conformant.

​	在 4.22.0 之后，Python 符合规范。

In 4.21.x, Python is conformant by default, but setting `PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=python` will cause it to be out of conformance.

​	在 4.21.x 中，Python 默认情况下符合规范，但设置 `PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=python` 将导致其不符合规范。

Before 4.21.0, Python is out of conformance.

​	在 4.21.0 之前，Python 不符合规范。

When a `proto2` file imports an enum defined in a `proto3` file, non-conformant Python versions treat that field as a **closed** enum.

​	当 `proto2` 文件导入在 `proto3` 文件中定义的枚举时，不符合规范的 Python 版本将该字段视为封闭枚举。

### Ruby

All known Ruby releases are out of conformance. Ruby treats all enums as **open**.

​	所有已知的 Ruby 版本都不符合规范。Ruby 将所有枚举视为开放枚举。

### Objective-C

After 22.0, Objective-C is conformant.

​	在 22.0 之后，Objective-C 符合规范。

Prior to 22.0, Objective-C was out of conformance. When a `proto2` file imported an enum defined in a `proto3` file, it would treat that field as a **closed** enum.

​	在 22.0 之前，Objective-C 不符合规范。当 `proto2` 文件导入在 `proto3` 文件中定义的枚举时，它会将该字段视为封闭枚举。

### Swift

Swift is conformant.

​	Swift 符合规范。

### Dart

Dart treats all enums as **closed**.

​	Dart 将所有枚举视为封闭枚举。
