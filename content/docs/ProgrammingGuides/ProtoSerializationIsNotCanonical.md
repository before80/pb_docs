+++
title = "Proto 序列化不是规范化的"
date = 2024-11-17T09:35:36+08:00
weight = 120
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

> 原文：[https://protobuf.dev/programming-guides/serialization-not-canonical/](https://protobuf.dev/programming-guides/serialization-not-canonical/)
>
> 收录该文档的时间：`2024-11-17T09:35:36+08:00`

# Proto Serialization Is Not Canonical - Proto 序列化不是规范化的

Explains how serialization works and why it is not canonical.

​	解释序列化的工作原理以及为何其不是规范化的。

Many people want a serialized proto to canonically represent the contents of that proto. Use cases include:

​	许多人希望序列化后的 proto 能够规范地表示 proto 的内容。常见的用例包括：

- using a serialized proto as a key in a hash table
  - 在哈希表中将序列化的 proto 作为键

- taking a fingerprint or checksum of a serialized proto
  - 对序列化的 proto 取指纹或校验和

- comparing serialized payloads as a way of checking message equality
  - 比较序列化的负载以检查消息是否相等


Unfortunately, *protobuf serialization is not (and cannot be) canonical*. There are a few notable exceptions, such as MapReduce, but in general you should generally think of proto serialization as unstable. This page explains why.

​	然而，**protobuf 序列化并不是（且不能是）规范化的**。虽然有一些例外情况（如 MapReduce），但通常情况下，proto 序列化应被认为是不稳定的。本文解释了原因。

## 确定性并非规范化 Deterministic is not Canonical

Deterministic serialization is not canonical. The serializer can generate different output for many reasons, including but not limited to the following variations:

​	确定性序列化并不等于规范化。序列化器可能因多种原因生成不同的输出，包括但不限于以下变化：

1. The protobuf schema changes in any way. protobuf 架构的任何更改。
2. The application being built changes in any way. 应用构建的任何变化。
3. The binary is built with different flags (eg. opt vs. debug). 二进制文件使用不同的编译标志（如优化 vs. 调试）。
4. The protobuf library is updated. protobuf 库的更新。

This means that hashes of serialized protos are fragile and not stable across time or space.

​	这意味着序列化 proto 的哈希在时间或空间上是脆弱且不稳定的。

There are many reasons why the serialized output can change. The above list is not exhaustive. Some of them are inherent difficulties in the problem space that would make it inefficient or impossible to guarantee canonical serialization even if we wanted to. Others are things we intentionally leave undefined to allow for optimization opportunities.

​	序列化输出可能更改的原因有很多。上述列表并不详尽。一些原因源于问题空间的固有困难，即使我们希望，也无法保证序列化的规范性。其他原因则是我们故意保留了未定义的行为以便于优化。

## 稳定序列化的固有障碍 Inherent Barriers to Stable Serialization

Protobuf objects preserve unknown fields to provide forward and backward compatibility. Unknown fields cannot be canonically serialized:

​	Protobuf 对象保留未知字段以提供向前和向后兼容性。未知字段无法规范地序列化：

1. Unknown fields can’t distinguish between bytes and sub-messages, as both have the same wire type. This makes it impossible to canonicalize messages stored in the unknown field set. If we were going to canonicalize, we would need to recurse into unknown submessages to sort their fields by field number, but we don’t have enough information to do this. 未知字段无法区分字节和子消息，因为二者的线缆类型相同。这使得规范化存储在未知字段集中的消息变得不可能。如果要规范化，我们需要递归进入未知子消息并按字段编号对其字段进行排序，但我们没有足够的信息来完成这项操作。
2. Unknown fields are always serialized after known fields, for efficiency. But canonical serialization would require interleaving unknown fields with known fields by field number. This would cause efficiency and code size overheads for everybody, even people who do not use the feature. 出于效率考虑，未知字段总是序列化在已知字段之后。但规范化序列化需要按字段编号将未知字段与已知字段交错。这将为所有用户带来效率和代码大小的开销，即使他们不使用该功能。

## 有意保留未定义行为的原因 Things Intentionally Left Undefined

Even if canonical serialization was feasible (that is, if we could solve the unknown field problem), we intentionally leave serialization order undefined to allow for more optimization opportunities:

​	即使规范化序列化是可行的（例如，如果我们能够解决未知字段问题），我们仍然故意将序列化顺序保留为未定义状态，以便于更多的优化机会：

1. If we can prove a field is never used in a binary, we can remove it from the schema completely and process it as an unknown field. This saves substantial code size and CPU cycles. 如果可以证明某个字段在二进制中从未使用过，我们可以完全从架构中移除它，并将其作为未知字段处理。这可以显著节省代码大小和 CPU 周期。
2. There may be opportunities to optimize by serializing vectors of the same field together, even though this would break field number order. 可以通过将同一字段的向量一起序列化来进行优化，即使这会打破字段编号的顺序。

To leave room for optimizations like this, we want to intentionally scramble field order in some configurations, so that applications do not inappropriately depend on field order.

​	为了给这样的优化留有余地，我们希望在某些配置中故意打乱字段顺序，以避免应用程序错误地依赖字段顺序。
