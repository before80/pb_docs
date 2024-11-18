+++
title = "Proto 限制"
date = 2024-11-17T09:35:36+08:00
weight = 30
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

> 原文：[https://protobuf.dev/programming-guides/proto-limits/](https://protobuf.dev/programming-guides/proto-limits/)
>
> 收录该文档的时间：`2024-11-17T09:35:36+08:00`

# Proto Limits - Proto 限制

Covers the limits to number of supported elements in proto schemas.

​	介绍 proto 架构中支持元素数量的限制。

This topic documents the limits to the number of supported elements (fields, enum values, and so on) in proto schemas.

​	本文档记录了 proto 架构中支持元素（字段、枚举值等）数量的限制。

This information is a collection of discovered limitations by many engineers, but is not exhaustive and may be incorrect/outdated in some areas. As you discover limitations in your work, contribute those to this document to help others.

​	这些信息是众多工程师发现的限制集合，但可能并不全面，某些方面可能存在错误或过时。如果您在工作中发现了新的限制，欢迎贡献到本文档中以帮助他人。

## 字段数量 Number of Fields

Message with only singular proto fields (such as Boolean):

​	仅包含单个 proto 字段（例如布尔值）的消息：

- ~2100 fields (proto2)
  - ~2100 个字段（proto2）

- ~3100 (proto3 without using optional fields)
  - ~3100 个字段（proto3，未使用 optional 字段）


Empty message extended by singular fields (such as Boolean):

​	通过单个字段（例如布尔值）扩展的空消息：

- ~4100 fields (proto2)
  - ~4100 个字段（proto2）


Extensions are supported [only by proto2](https://protobuf.dev/programming-guides/version-comparison#extensionsany).

​	扩展字段[仅在 proto2 中受支持](https://protobuf.dev/programming-guides/version-comparison#extensionsany)。

To test this limitation, create a proto message with more than the upper bound number of fields and compile using a Java proto rule. The limit comes from JVM specs.

​	要测试此限制，可以创建一个超过字段上限数量的 proto 消息，并使用 Java proto 规则进行编译。此限制源于 JVM 规范。

## 枚举值的数量 Number of Values in an Enum

The lowest limit is ~1700 values, in Java . Other languages have different limits.

​	最低限制约为 ~1700 个值（在 Java 中）。其他语言的限制不同。

## 消息的总大小 Total Size of the Message

Any proto in serialized form must be <2GiB, as that is the maximum size supported by all implementations. It’s recommended to bound request and response sizes.

​	任何序列化形式的 proto 消息都必须小于 2GiB，这是所有实现支持的最大大小。建议限制请求和响应的大小。

## Proto 解组的深度限制 Depth Limit for Proto Unmarshaling

- Java: 100
- C++: 100
- Go: 10000 (there is a plan to reduce this to 100) （计划将其减少到 100）

If you try to unmarshal a message that is nested deeper than the depth limit, the unmarshaling will fail.

​	如果尝试解组嵌套深度超过深度限制的消息，解组将失败。
