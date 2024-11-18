+++
title = "扩展声明"
date = 2024-11-17T09:35:36+08:00
weight = 100
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

> 原文：[https://protobuf.dev/programming-guides/extension_declarations/](https://protobuf.dev/programming-guides/extension_declarations/)
>
> 收录该文档的时间：`2024-11-17T09:35:36+08:00`

# Extension Declarations - 扩展声明

Describes in detail what extension declarations are, why we need them, and how we use them.

​	详细描述了扩展声明的定义、必要性及使用方式。

## 简介 Introduction

This page describes in detail what extension declarations are, why we need them, and how we use them.

​	本文详细描述了扩展声明的定义、必要性及使用方式。

**NOTE:** Proto3 does not support extensions (except for [declaring custom options](https://protobuf.dev/programming-guides/proto3/#customoptions)). Extensions are fully supported in proto2 and editions though.

​	**注意**：Proto3 不支持扩展（除了 [自定义选项声明](https://protobuf.dev/programming-guides/proto3/#customoptions)）。然而，Proto2 和版本管理（editions）完全支持扩展。

If you need an introduction to extensions, read this [extensions guide](https://protobuf.dev/programming-guides/proto2/#extensions)

​	如果需要扩展的基础介绍，请阅读 [扩展指南](https://protobuf.dev/programming-guides/proto2/#extensions)。

## 动机 Motivation

Extension declarations aim to strike a happy medium between regular fields and extensions. Like extensions, they avoid creating a dependency on the message type of the field, which therefore results in a leaner build graph and smaller binaries in environments where unused messages are difficult or impossible to strip. Like regular fields, the field name/number appear in the enclosing message, which makes it easier to avoid conflicts and see a convenient listing of what fields are declared.

​	扩展声明旨在平衡常规字段和扩展字段的优缺点。像扩展字段一样，它们避免了字段与消息类型之间的依赖，从而简化了构建图并减少了二进制大小，尤其是在无法移除未使用消息的环境中。像常规字段一样，扩展字段的名称和编号会显示在包含的消息中，这使得避免冲突和查看已声明字段列表更加容易。

Listing the occupied extension numbers with extension declarations makes it easier for users to pick an available extension number and to avoid conflicts.

​	通过扩展声明列出已占用的扩展编号，用户可以更轻松地选择可用的扩展编号并避免冲突。

## Usage

Extension declarations are an option of extension ranges. Like forward declarations in C++, you can declare the field type, field name, and cardinality (singular or repeated) of an extension field without importing the `.proto` file containing the full extension definition:

​	扩展声明是扩展范围的一个选项。类似于 C++ 中的前向声明，可以声明扩展字段的字段类型、字段名称和基数（单一或重复），而无需导入包含完整扩展定义的 `.proto` 文件：

```proto
syntax = "proto2";

message Foo {
  extensions 4 to 1000 [
    declaration = {
      number: 4,
      full_name: ".my.package.event_annotations",
      type: ".logs.proto.ValidationAnnotations",
      repeated: true },
    declaration = {
      number: 999,
      full_name: ".foo.package.bar",
      type: "int32"}];
}
```

This syntax has the following semantics:

​	此语法具有以下语义：

- Multiple `declaration`s with distinct extension numbers can be defined in a single extension range if the size of the range allows.
  - 如果范围允许，可以在单个扩展范围内定义多个具有不同扩展编号的 `declaration`。

- If there is any declaration for the extension range, *all* extensions of the range must also be declared. This prevents non-declared extensions from being added, and enforces that any new extensions use declarations for the range.
  - 如果扩展范围中存在任何声明，则该范围的**所有**扩展都必须声明。这可以防止添加未声明的扩展，并强制所有新扩展在范围内使用声明。

- The given message type (`.logs.proto.ValidationAnnotations`) does not need to have been previously defined or imported. We check only that it is a valid name that could potentially be defined in another `.proto` file.
  - 给定的消息类型（如 `.logs.proto.ValidationAnnotations`）不需要事先定义或导入。只需检查它是否是另一个 `.proto` 文件中可能定义的有效名称。

- When this or another `.proto` file defines an extension of this message (`Foo`) with this name or number, we enforce that the number, type, and full name of the extension match what is forward-declared here.
  - 当此文件或另一个 `.proto` 文件为该消息（`Foo`）定义扩展，并具有相同的名称或编号时，会强制检查扩展的编号、类型和全名是否与此处的前向声明匹配。


**WARNING:** Avoid using declarations for extension range groups such as `extensions 4, 999`. It is unclear which extension range the declarations apply to, and it is currently unsupported.

​	**警告**：避免为类似 `extensions 4, 999` 的扩展范围组使用声明。目前尚不清楚声明适用于哪个扩展范围，因此当前不支持此操作。

The extension declarations expect two extension fields with different packages:

​	扩展声明期望以下两个具有不同包的扩展字段：

```proto
package my.package;
extend Foo {
  repeated logs.proto.ValidationAnnotations event_annotations = 4;
}
package foo.package;
extend Foo {
  optional int32 bar = 999;
}
```

### 保留声明 Reserved Declarations

An extension declaration can be marked `reserved: true` to indicate that it is no longer actively used and the extension definition has been deleted. **Do not delete the extension declaration or edit its `type` or `full_name` value**.

​	扩展声明可以通过标记 `reserved: true` 来表示该扩展不再被使用且扩展定义已被删除。**不要删除扩展声明或编辑其 `type` 或 `full_name` 值**。

This `reserved` tag is separate from the reserved keyword for regular fields and **does not require breaking up the extension range**.

​	此 `reserved` 标记独立于常规字段的保留关键字，**不需要分隔扩展范围**。

```proto
syntax = "proto2";

message Foo {
  extensions 4 to 1000 [
    declaration = {
      number: 500,
      full_name: ".my.package.event_annotations",
      type: ".logs.proto.ValidationAnnotations",
      reserved: true }];
}
```

An extension field definition using a number that is `reserved` in the declaration will fail to compile.

​	如果扩展字段定义使用了声明中标记为 `reserved` 的编号，则会编译失败。

## 在 descriptor.proto 中的表示 Representation in descriptor.proto

Extension declaration is represented in descriptor.proto as fields in `proto2.ExtensionRangeOptions`:

​	扩展声明在 `descriptor.proto` 中表示为 `proto2.ExtensionRangeOptions` 的字段：

```proto
message ExtensionRangeOptions {
  message Declaration {
    optional int32 number = 1;
    optional string full_name = 2;
    optional string type = 3;
    optional bool reserved = 5;
    optional bool repeated = 6;
  }
  repeated Declaration declaration = 2;
}
```

## 反射字段查找 Reflection Field Lookup

Extension declarations are *not* returned from the normal field lookup functions like `Descriptor::FindFieldByName()` or `Descriptor::FindFieldByNumber()`. Like extensions, they are discoverable by extension lookup routines like `DescriptorPool::FindExtensionByName()`. This is an explicit choice that reflects the fact that declarations are not definitions and do not have enough information to return a full `FieldDescriptor`.

​	扩展声明**不会**通过诸如 `Descriptor::FindFieldByName()` 或 `Descriptor::FindFieldByNumber()` 等常规字段查找函数返回。与扩展类似，可以通过扩展查找例程（如 `DescriptorPool::FindExtensionByName()`）发现它们。这是一个明确的设计选择，因为声明不是定义，并且没有足够的信息返回完整的 `FieldDescriptor`。

Declared extensions still behave like regular extensions from the perspective of TextFormat and JSON. It also means that migrating an existing field to a declared extension will require first migrating any reflective use of that field.

​	从 TextFormat 和 JSON 的角度看，已声明的扩展仍然像常规扩展一样工作。这也意味着将现有字段迁移到已声明的扩展需要首先迁移该字段的所有反射使用。

## 使用扩展声明分配编号 Use Extension Declarations to Allocate Numbers

Extensions use field numbers just like ordinary fields do, so it is important for each extension to be assigned a number that is unique within the parent message. We recommend using extension declarations to declare the field number and type for each extension in the parent message. The extension declarations serve as a registry of all the parent message’s extensions, and protoc will enforce that there are no field number conflicts. When you add a new extension, choose the next available number usually by just incrementing by one the previously added extension number.

​	扩展字段与普通字段一样使用字段编号，因此每个扩展必须分配一个在父消息中唯一的编号。建议使用扩展声明为父消息中的每个扩展声明字段编号和类型。扩展声明充当父消息所有扩展的注册表，`protoc` 将确保不存在字段编号冲突。当添加新扩展时，通常只需将之前扩展编号递增一即可选择下一个可用编号。

**TIP:** There is a [special guidance](https://protobuf.dev/programming-guides/extension_declarations/#message-set) for `MessageSet` which provides a script to help pick the next available number.

​	**提示**：针对 `MessageSet` 的[特殊指导](https://protobuf.dev/programming-guides/extension_declarations/#message-set)提供了一个脚本来帮助选择下一个可用编号。

Whenever you delete an extension, make sure to mark the field number `reserved` to eliminate the risk of accidentally reusing it.

​	每次删除扩展时，请务必将字段编号标记为 `reserved`，以消除意外重用的风险。

This convention is only a recommendation–the protobuf team does not have the ability or desire to force anyone to adhere to it for every extendable message. If you as the owner of an extendable proto do not want to coordinate extension numbers through extension declarations, you can choose to provide coordination through other means. Be very careful, though, because accidental reuse of an extension number can cause serious problems.

​	这种约定仅是一个推荐——protobuf 团队无权强制要求所有可扩展消息的所有者遵守此规则。如果您作为可扩展 proto 的所有者，不希望通过扩展声明协调扩展编号，则可以选择通过其他方式进行协调。但请务必小心，因为意外重用扩展编号可能导致严重问题。

One way to sidestep the issue would be to avoid extensions entirely and use [`google.protobuf.Any`](https://protobuf.dev/programming-guides/proto3/#any) instead. This could be a good choice for APIs that front storage or for pass-through systems where the client cares about the contents of the proto but the system receiving it does not.

​	一种避免此问题的方法是完全避免使用扩展，而改用 [`google.protobuf.Any`](https://protobuf.dev/programming-guides/proto3/#any)。对于 API 前端存储或客户端关心 proto 内容而接收系统不关心的传递系统，这可能是一个不错的选择。

### 重用扩展编号的后果 Consequences of Reusing an Extension Number

An extension is a field defined outside the container message; usually in a separate .proto file. This distribution of definitions makes it easy for two developers to accidentally create different definitions for the same extension field number.

​	扩展是定义在容器消息之外的字段，通常位于单独的 `.proto` 文件中。这种分布式定义使两个开发者很容易意外地为同一扩展字段编号创建不同的定义。

The consequences of changing an extension definition are the same for extensions and standard fields. Reusing a field number introduces an ambiguity in how a proto should be decoded from the wire format. The protobuf wire format is lean and doesn’t provide a good way to detect fields encoded using one definition and decoded using another.

​	更改扩展定义的后果与标准字段相同。重用字段编号会引入关于如何从线格式解码 proto 的歧义。protobuf 线格式精简，不提供有效方式来检测使用一个定义编码但使用另一个定义解码的字段。

This ambiguity can manifest in a short time frame, such as a client using one extension definition and a server using another communicating .

​	这种歧义可能在短期内显现，例如客户端使用一种扩展定义，而服务器使用另一种定义。

This ambiguity can also manifest over a longer time frame, such as storing data encoded using one extension definition and later retrieving and decoding using the second extension definition. This long-term case can be difficult to diagnose if the first extension definition was deleted after the data was encoded and stored.

​	这种歧义也可能在长期内显现，例如使用一种扩展定义编码并存储数据，稍后使用第二种扩展定义检索和解码数据。如果第一个扩展定义在数据编码和存储后被删除，这种长期情况可能难以诊断。

The outcome of this can be:

​	结果可能包括：

1. A parse error (best case scenario). **解析错误**（最佳情况）。
2. Leaked PII / SPII – if PII or SPII is written using one extension definition and read using another extension definition. **PII/SPII 泄漏**——如果使用一种扩展定义写入 PII 或 SPII，并使用另一种扩展定义读取。
3. Data Corruption – if data is read using the “wrong” definition, modified and rewritten. **数据损坏**——如果使用“错误”的定义读取数据、修改并重写。

Data definition ambiguity is almost certainly going to cost someone time for debugging at a minimum. It could also cause data leaks or corruption that takes months to clean up.

​	定义数据的歧义几乎肯定会导致调试成本，并可能导致数据泄漏或损坏，清理这些问题可能需要数月时间。

## 使用提示 Usage Tips

### 不要删除扩展声明 Never Delete an Extension Declaration

Deleting an extension declaration opens the door to accidental reuse in the future. If the extension is no longer processed and the definition is deleted, the extension declaration can be [marked reserved](https://protobuf.dev/programming-guides/extension_declarations/#reserved).

​	删除扩展声明会导致未来可能意外重用该编号。如果扩展不再处理且定义已被删除，可以将扩展声明[标记为保留](https://protobuf.dev/programming-guides/extension_declarations/#reserved)。

### 不要对新扩展声明使用 `reserved` 列表中的字段名称或编号 Never Use a Field Name or Number from the `reserved` List for a New Extension Declaration

Reserved numbers may have been used for fields or other extensions in the past.

​	保留的编号可能在过去用于字段或其他扩展。

Using the `full_name` of a reserved field is not recommended due to the possibility of ambiguity when using textproto.

​	不推荐使用保留字段的 `full_name`，因为在使用 textproto 时可能会引发歧义。

### 不要更改现有扩展声明的类型 Never change the type of an existing extension declaration

Changing the extension field’s type can result in data corruption.

​	更改扩展字段的类型可能导致数据损坏。

If the extension field is of an enum or message type, and that enum or message type is being renamed, updating the declaration name is required and safe. To avoid breakages, the update of the type, the extension field definition, and extension declaration should all happen in a single commit.

​	如果扩展字段为枚举或消息类型，而该枚举或消息类型正在被重命名，则更新声明名称是必须的且安全的。为避免中断，类型、扩展字段定义和扩展声明的更新应在单个提交中完成。

### 重命名扩展字段时需谨慎 Use Caution When Renaming an Extension Field

While renaming an extension field is fine for the wire format, it can break JSON and TextFormat parsing.

​	虽然重命名扩展字段对线格式没有影响，但可能会破坏 JSON 和 TextFormat 解析。
