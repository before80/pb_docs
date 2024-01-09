+++
title = "扩展声明"
weight = 83
description = "详细描述了扩展声明是什么、为什么需要它们以及如何使用它们。"
type = "docs"

+++

## Extension Declarations 扩展声明

Describes in detail what extension declarations are, why we need them, and how we use them.

​	详细描述了扩展声明是什么、为什么需要它们以及如何使用它们。



## 简介 Introduction 

This page describes in detail what extension declarations are, why we need them, and how we use them.

​	此页面详细描述了扩展声明是什么、为什么需要它们以及如何使用它们。

**NOTE:** Extension declarations are mostly used in proto2, as proto3 does not support extensions at this time (except for [declaring custom options](https://protobuf.dev/programming-guides/proto3/#customoptions)).

​	注意：扩展声明主要用于 proto2，因为 proto3 目前不支持扩展（除了声明自定义选项）。

If you need an introduction to extensions, read this [extensions guide](https://protobuf.dev/programming-guides/proto2/#extensions)

​	如果您需要扩展简介，请阅读此扩展指南

## 动机 Motivation 

Extension declarations aim to strike a happy medium between regular fields and extensions. Like extensions, they avoid creating a dependency on the message type of the field, which therefore results in a leaner build graph and smaller binaries in environments where unused messages are difficult or impossible to strip. Like regular fields, the field name/number appear in the enclosing message, which makes it easier to avoid conflicts and see a convenient listing of what fields are declared.

​	扩展声明旨在常规字段和扩展之间取得平衡。与扩展一样，它们避免创建对字段的消息类型的依赖，因此在难以或无法剥离未使用的消息的环境中，可生成更精简的构建图和更小的二进制文件。与常规字段一样，字段名称/编号显示在封闭消息中，这使得避免冲突和查看已声明字段的便捷列表变得更加容易。

Listing the occupied extension numbers with extension declarations makes it easier for users to pick an available extension number and to avoid conflicts.

​	使用扩展声明列出已占用的扩展编号，使用户能够更轻松地选择可用的扩展编号并避免冲突。

## 用法 Usage 

Extension declarations are an option of extension ranges. Like forward declarations in C++, you can declare the field type, field name, and cardinality (singular or repeated) of an extension field without importing the `.proto` file containing the full extension definition:

​	扩展声明是扩展范围的一个选项。与 C++ 中的前向声明一样，您可以在不导入包含完整扩展定义的 `.proto` 文件的情况下声明扩展字段的字段类型、字段名称和基数（单数或重复）。

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
- 如果范围的大小允许，可以在单个扩展范围内定义具有不同扩展号的多个 `declaration` 。
- If there is any declaration for the extension range, *all* extensions of the range must also be declared. This prevents non-declared extensions from being added, and enforces that any new extensions use declarations for the range.
- 如果对扩展范围有任何声明，则也必须声明该范围的所有扩展。这可防止添加未声明的扩展，并强制任何新扩展使用该范围的声明。
- The given message type (`.logs.proto.ValidationAnnotations`) does not need to have been previously defined or imported. We check only that it is a valid name that could potentially be defined in another `.proto` file.
- 给定的消息类型 ( `.logs.proto.ValidationAnnotations` ) 无需预先定义或导入。我们仅检查它是否是一个有效名称，该名称可能在另一个 `.proto` 文件中定义。
- When this or another `.proto` file defines an extension of this message (`Foo`) with this name or number, we enforce that the number, type, and full name of the extension match what is forward-declared here.
- 当此 `.proto` 文件或另一个 `.proto` 文件使用此名称或编号定义此消息 ( `Foo` ) 的扩展时，我们强制扩展的编号、类型和全名与此处前向声明的内容匹配。

**WARNING:** Avoid using declarations for extension range groups such as `extensions 4, 999`. It is unclear which extension range the declarations apply to, and it is currently unsupported.

​	警告：避免对扩展范围组（例如 `extensions 4, 999` ）使用声明。目前尚不清楚声明适用于哪个扩展范围，并且目前不支持此操作。

The extension declarations expect two extension fields with different packages:

​	扩展声明期望具有不同包的两个扩展字段：

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

### 保留的声明 Reserved Declarations 

An extension declaration can be marked `reserved: true` to indicate that it is no longer actively used and the extension definition has been deleted. **Do not delete the extension declaration or edit its `type` or `full_name` value**.

​	可以将扩展声明标记为 `reserved: true` 以指示不再主动使用它，并且已删除扩展定义。请勿删除扩展声明或编辑其 `type` 或 `full_name` 值。

This `reserved` tag is separate from the reserved keyword for regular fields and **does not require breaking up the extension range**.

​	此 `reserved` 标记与常规字段的保留关键字分开，并且不需要分解扩展范围。

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

​	使用声明中为 `reserved` 的数字的扩展字段定义将无法编译。

## 在 descriptor.proto 中的表示 Representation in descriptor.proto 

Extension declaration is represented in descriptor.proto as fields in `proto2.ExtensionRangeOptions`:

​	扩展声明在 descriptor.proto 中表示为 `proto2.ExtensionRangeOptions` 中的字段：

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

​	扩展声明不会从常规字段查找函数（如 `Descriptor::FindFieldByName()` 或 `Descriptor::FindFieldByNumber()` ）返回。与扩展一样，它们可以通过扩展查找例程（如 `DescriptorPool::FindExtensionByName()` ）发现。这是一个明确的选择，反映了声明不是定义，并且没有足够的信息来返回完整的 `FieldDescriptor` 。

Declared extensions still behave like regular extensions from the perspective of TextFormat and JSON. It also means that migrating an existing field to a declared extension will require first migrating any reflective use of that field.

​	已声明的扩展从 TextFormat 和 JSON 的角度来看仍然表现得像常规扩展。这也意味着将现有字段迁移到已声明的扩展将需要首先迁移对该字段的任何反射使用。

## 使用扩展声明分配数字 Use Extension Declarations to Allocate Numbers 

Extensions use field numbers just like ordinary fields do, so it is important for each extension to be assigned a number that is unique within the parent message. We recommend using extension declarations to declare the field number and type for each extension in the parent message. The extension declarations serve as a registry of all the parent message’s extensions, and protoc will enforce that there are no field number conflicts. When you add a new extension, choose the number by just incrementing by one the previously added extension number. Whenever you delete an extension, make sure to mark the field number `reserved` to eliminate the risk of accidentally reusing it.

​	扩展使用字段编号，就像普通字段一样，因此，为每个扩展分配一个在父消息中唯一的编号非常重要。我们建议使用扩展声明来声明父消息中每个扩展的字段编号和类型。扩展声明用作所有父消息扩展的注册表，protoc 将强制执行，确保没有字段编号冲突。添加新扩展时，只需将之前添加的扩展编号加一即可选择编号。每当删除扩展时，请务必标记字段编号 `reserved` ，以消除意外重复使用它的风险。

This convention is only a recommendation–the protobuf team does not have the ability or desire to force anyone to adhere to it for every extendable message. If you as the owner of an extendable proto do not want to coordinate extension numbers through extension declarations, you can choose to provide coordination through other means. Be very careful, though, because accidental reuse of an extension number can cause serious problems.

​	此约定只是一个建议——protobuf 团队无权也无意强迫任何人将其用于每个可扩展消息。如果您作为可扩展 proto 的所有者不想通过扩展声明协调扩展编号，可以选择通过其他方式提供协调。但是，请务必小心，因为意外重复使用扩展编号可能会导致严重问题。

One way to sidestep the issue would be to avoid extensions entirely and use [`google.protobuf.Any`](https://protobuf.dev/programming-guides/proto3/#any) instead. This could be a good choice for APIs that front storage or for pass-through systems where the client cares about the contents of the proto but the system receiving it does not.

​	规避此问题的一种方法是完全避免使用扩展，而改用 `google.protobuf.Any` 。对于前端存储的 API 或客户端关心 proto 内容但接收它的系统不关心的直通系统，这可能是一个不错的选择。

### 重复使用扩展号的后果 Consequences of Reusing an Extension Number 

An extension is a field defined outside the container message; usually in a separate .proto file. This distribution of definitions makes it easy for two developers to accidentally create different definitions for the same extension field number.

​	扩展是在容器消息外部定义的字段；通常在单独的 .proto 文件中。这种定义分布使得两位开发者很容易意外地为同一个扩展字段号创建不同的定义。

The consequences of changing an extension definition are the same for extensions and standard fields. Reusing a field number introduces an ambiguity in how a proto should be decoded from the wire format. The protobuf wire format is lean and doesn’t provide a good way to detect fields encoded using one definition and decoded using another.

​	更改扩展定义的后果对于扩展和标准字段来说是相同的。重复使用字段号会在从线路格式解码 proto 的方式中引入歧义。protobuf 线路格式精简，无法很好地检测使用一种定义编码并使用另一种定义解码的字段。

This ambiguity can manifest in a short time frame, such as a client using one extension definition and a server using another communicating .

​	这种歧义可能会在短时间内显现，例如客户端使用一种扩展定义，而服务器使用另一种扩展定义进行通信。

This ambiguity can also manifest over a longer time frame, such as storing data encoded using one extension definition and later retrieving and decoding using the second extension definition. This long-term case can be difficult to diagnose if the first extension definition was deleted after the data was encoded and stored.

​	这种歧义也可能在较长时间内显现，例如，存储使用一种扩展定义编码的数据，然后使用第二种扩展定义检索和解码数据。如果在对数据进行编码和存储后删除了第一个扩展定义，则这种长期情况可能难以诊断。

The outcome of this can be:

​	其结果可能是：

1. A parse error (best case scenario).
2. 解析错误（最佳情况）。
3. Leaked PII / SPII – if PII or SPII is written using one extension definition and read using another extension definition.
4. 泄露的 PII/SPII - 如果使用一种扩展定义写入 PII 或 SPII，并使用另一种扩展定义读取。
5. Data Corruption – if data is read using the “wrong” definition, modified and rewritten.
6. 数据损坏 - 如果使用“错误的”定义读取数据，修改并重写。

Data definition ambiguity is almost certainly going to cost someone time for debugging at a minimum. It could also cause data leaks or corruption that takes months to clean up.

​	数据定义歧义几乎肯定会让某人至少花费时间进行调试。它还可能导致数据泄露或损坏，需要数月时间才能清理干净。

## 使用技巧 Usage Tips 

### 切勿删除扩展声明 Never Delete an Extension Declaration 

Deleting an extension declaration opens the door to accidental reuse in the future. If the extension is no longer processed and the definition is deleted, the extension declaration can be [marked reserved](https://protobuf.dev/programming-guides/extension_declarations/#reserved).

​	删除扩展声明会为将来意外重复使用打开大门。如果不再处理扩展并且删除了定义，则可以将扩展声明标记为保留。

### 切勿将 `reserved` 列表中的字段名称或编号用于新的扩展声明 Never Use a Field Name or Number from the `reserved` List for a New Extension Declaration 

Reserved numbers may have been used for fields or other extensions in the past.

​	保留的数字过去可能已用于字段或其他扩展。

Using the `full_name` of a reserved field is not recommended due to the possibility of ambiguity when using textproto.

​	由于在使用 textproto 时可能出现歧义，因此不建议使用保留字段的 `full_name` 。

### 切勿更改现有扩展声明的类型 Never change the type of an existing extension declaration 

Changing the extension field’s type can result in data corruption.

​	更改扩展字段的类型可能会导致数据损坏。

If the extension field is of an enum or message type, and that enum or message type is being renamed, updating the declaration name is required and safe. To avoid breakages, the update of the type, the extension field definition, and extension declaration should all happen in a single commit.

​	如果扩展字段为枚举或消息类型，并且该枚举或消息类型正在重命名，则需要且可以安全地更新声明名称。为了避免中断，类型、扩展字段定义和扩展声明的更新都应在一个提交中进行。

### 重命名扩展字段时要小心 Use Caution When Renaming an Extension Field 

While renaming an extension field is fine for the wire format, it can break JSON and TextFormat parsing.

​	虽然重命名扩展字段对数据线格式来说很好，但它可能会破坏 JSON 和 TextFormat 解析。
