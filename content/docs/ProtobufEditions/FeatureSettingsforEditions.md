+++
title = "editions特性设置"
date = 2024-11-17T09:35:36+08:00
weight = 10
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

> 原文：[https://protobuf.dev/editions/features/](https://protobuf.dev/editions/features/)
>
> 收录该文档的时间：`2024-11-17T09:35:36+08:00`

# Feature Settings for Editions - editions特性设置

Protobuf Editions features and how they affect protobuf behavior.

​	Protobuf Editions的特性及其如何影响 Protobuf 的行为。

This topic provides an overview of the features that are included in Edition 2023. Each subsequent edition’s features will be added to this topic. We announce new editions in [the News section]({{< ref "/docs/News" >}}).

​	本文概述了 **Edition 2023** 中包含的特性。每个后续editions的特性都会添加到本文中。我们会在 [新闻栏目]({{< ref "/docs/News" >}}) 中宣布新editions。

## Prototiller

Prototiller is a command-line tool that converts proto2 and proto3 definition files to Editions syntax. It hasn’t been released yet, but is referenced throughout this topic.

​	**Prototiller** 是一个命令行工具，用于将 proto2 和 proto3 定义文件转换为 Editions 语法。该工具尚未发布，但在本文中有多次提及。

## 特性 Features

The following sections include all of the behaviors that are configurable using features in Edition 2023. [Preserving proto2 or proto3 Behavior](https://protobuf.dev/editions/features/#preserving) shows how to override the default behaviors so that your proto definition files act like proto2 or proto3 files. For more information on how Editions and Features work together to set behavior, see [Protobuf Editions Overview]({{< ref "/docs/ProtobufEditions/Overview" >}}).

​	以下章节包括所有在 **Edition 2023** 中通过特性配置的行为。[保留 proto2 或 proto3 行为](https://protobuf.dev/editions/features/#preserving) 显示了如何覆盖默认行为，使您的 proto 定义文件表现得像 proto2 或 proto3 文件。有关 Editions 和 Features 如何协作设置行为的更多信息，请参见 [Protobuf Editions 概述]({{< ref "/docs/ProtobufEditions/Overview" >}})。

Each of the following sections has an entry for what scope the feature applies to. This can include file, enum, message, or field. The following sample shows a mock feature applied to each scope:

​	每个以下章节都列出了特性适用的范围，包括文件（file）、枚举（enum）、消息（message）或字段（field）。以下示例展示了一个应用于每种范围的 mock 特性：

```proto
edition = "2023";

// File-scope definition
// 文件范围定义
option features.bar = BAZ;

enum Foo {
  // Enum-scope definition
  // 枚举范围定义
  option features.bar = QUX;

  A = 1;
  B = 2;
}

message Corge {
  // Message-scope definition
  // 消息范围定义
  option features.bar = QUUX;

  // Field-scope definition
  // 字段范围定义
  Foo A = 1 [features.bar = GRAULT];
}
```

In this example, the setting `GRAULT` in the field-scope feature definition overrides the message-scope QUUX setting.

​	在此示例中，字段范围特性定义中的 `GRAULT` 设置覆盖了消息范围的 `QUUX` 设置。

### `features.enum_type`

This feature sets the behavior for how enum values that aren’t contained within the defined set are handled. See [Enum Behavior]({{< ref "/docs/ProgrammingGuides/EnumBehavior" >}}) for more information on open and closed enums.

​	此特性设置如何处理不在定义集中包含的枚举值的行为。有关开放和封闭枚举的更多信息，请参见 [枚举行为]({{< ref "/docs/ProgrammingGuides/EnumBehavior" >}})。

This feature doesn’t impact proto3 files, so this section doesn’t have a before and after of a proto3 file.

​	此特性不会影响 proto3 文件，因此没有提供 proto3 文件的前后对比。

**Values available:** 可用值：

- `CLOSED:` Closed enums store enum values that are out of range in the unknown field set.
  - `CLOSED:` 封闭枚举将超出范围的枚举值存储在未知字段集中。

- `OPEN:` Open enums parse out of range values into their fields directly.
  - `OPEN:` 开放枚举会将超出范围的值直接解析到其字段中。


**Applicable to the following scopes:** File, Enum

​	**适用范围：** 文件（File）、枚举（Enum）

**Default behavior in Edition 2023:** `OPEN`

​	**Edition 2023 的默认行为：** `OPEN`

**Behavior in proto2:** `CLOSED`

​	**proto2 的行为：** `CLOSED`

**Behavior in proto3:** `OPEN`

​	**proto3 的行为：** `OPEN`

The following code sample shows a proto2 file:

​	以下代码示例展示了一个 proto2 文件：

```proto
syntax = "proto2";

enum Foo {
  A = 2;
  B = 4;
  C = 6;
}
```

After running [Prototiller](https://protobuf.dev/editions/features/#prototiller), the equivalent code might look like this:

​	运行 [Prototiller](https://protobuf.dev/editions/features/#prototiller) 后，等效代码可能如下所示：

```proto
edition = "2023";

enum Foo {
  option features.enum_type = CLOSED;
  A = 2;
  B = 4;
  C = 6;
}
```

### `features.field_presence`

This feature sets the behavior for tracking field presence, or the notion of whether a protobuf field has a value.

​	此特性设置字段存在性（field presence）的跟踪行为，即 Protobuf 字段是否有值的概念。

**Values available:** 可用值：

- `LEGACY_REQUIRED`: The field is required for parsing and serialization. Any explicitly-set value is serialized onto the wire (even if it is the same as the default value).
  - `LEGACY_REQUIRED:` 字段在解析和序列化时是必须的。任何显式设置的值都会被序列化（即使与默认值相同）。

- `EXPLICIT`: The field has explicit presence tracking. Any explicitly-set value is serialized onto the wire (even if it is the same as the default value). For singular primitive fields, `has_*` functions are generated for fields set to `EXPLICIT`.
  - `EXPLICIT:` 字段具有显式的存在性跟踪。任何显式设置的值都会被序列化（即使与默认值相同）。对于单个原始字段，设置为 `EXPLICIT` 时会生成 `has_*` 函数。

- `IMPLICIT`: The field has no presence tracking. The default value is not serialized onto the wire (even if it is explicitly set). `has_*` functions are not generated for fields set to `IMPLICIT`.
  - `IMPLICIT:` 字段没有存在性跟踪。默认值不会被序列化（即使显式设置）。设置为 `IMPLICIT` 的字段不会生成 `has_*` 函数。


**Applicable to the following scopes:** File, Field

​	**适用范围：** 文件（File）、字段（Field）

**Default value in the Edition 2023:** `EXPLICIT`

​	**Edition 2023 的默认值：** `EXPLICIT`

**Behavior in proto2:** `EXPLICIT`

​	**proto2 的行为：** `EXPLICIT`

**Behavior in proto3:** `IMPLICIT` unless the field has the `optional` label, in which case it behaves like `EXPLICIT`. See [Presence in Proto3 APIs]({{< ref "/docs/ProgrammingGuides/FieldPresence#presence-in-proto3-apis" >}}) for more information.

​	**proto3 的行为：** 如果字段有 `optional` 标签，则行为类似于 `EXPLICIT`，否则为 `IMPLICIT`。更多信息请参见 [Proto3 API 中的存在性]({{< ref "/docs/ProgrammingGuides/FieldPresence#presence-in-proto3-apis" >}})。

The following code sample shows a proto2 file:

​	以下代码示例展示了一个 proto2 文件：

```proto
syntax = "proto2";

message Foo {
  required int32 x = 1;
  optional int32 y = 2;
  repeated int32 z = 3;
}
```

After running Prototiller, the equivalent code might look like this:

​	运行 Prototiller 后，等效代码可能如下所示：

```proto
edition = "2023";

message Foo {
  int32 x = 1 [features.field_presence = LEGACY_REQUIRED];
  int32 y = 2;
  repeated int32 z = 3;
}
```

The following shows a proto3 file:

​	以下代码展示了一个 proto3 文件：

```proto
syntax = "proto3";

message Bar {
  int32 x = 1;
  optional int32 y = 2;
  repeated int32 z = 3;
}
```

After running Prototiller, the equivalent code might look like this:

​	运行 Prototiller 后，等效代码可能如下所示：

```proto
edition = "2023";
option features.field_presence = IMPLICIT;

message Bar {
  int32 x = 1;
  int32 y = 2 [features.field_presence = EXPLICIT];
  repeated int32 z = 3;
}
```

Note that the `required` and `optional` labels no longer exist in Editions, as the corresponding behavior is set explicitly with the `field_presence` feature.

​	注意，在 Editions 中，不再存在 `required` 和 `optional` 标签，因为对应的行为可以通过 `field_presence` 特性显式设置。

### `features.json_format`

This feature sets the behavior for JSON parsing and serialization.

​	此特性设置 JSON 解析和序列化的行为。

This feature doesn’t impact proto3 files, so this section doesn’t have a before and after of a proto3 file. Editions behavior matches the behavior in proto3.

​	此特性不会影响 proto3 文件，因此没有提供 proto3 文件的前后对比。Edition 的行为与 proto3 的行为一致。

**Values available:** 可用值：

- `ALLOW`: A runtime must allow JSON parsing and serialization. Checks are applied at the proto level to make sure that there is a well-defined mapping to JSON.
  - `ALLOW:` 运行时必须允许 JSON 解析和序列化。在 proto 级别会应用检查，以确保有明确的 JSON 映射。

- `LEGACY_BEST_EFFORT`: A runtime does the best it can to parse and serialize JSON. Certain protos are allowed that can result in unspecified behavior at runtime (such as many:1 or 1:many mappings).
  - `LEGACY_BEST_EFFORT:` 运行时尽力解析和序列化 JSON。某些 proto 被允许，但可能在运行时会导致未定义的行为（如多对一或一对多映射）。


**Applicable to the following scopes:** File, Message, Enum

​	**适用范围：** 文件（File）、消息（Message）、枚举（Enum）

**Default behavior in Edition 2023:** `ALLOW`

​	**Edition 2023 的默认行为：** `ALLOW`

**Behavior in proto2:** `LEGACY_BEST_EFFORT`

​	**proto2 的行为：** `LEGACY_BEST_EFFORT`

**Behavior in proto3:** `ALLOW`

​	**proto3 的行为：** `ALLOW`

The following code sample shows a proto2 file:

​	以下代码示例展示了一个 proto2 文件：

```proto
syntax = "proto2";

message Foo {
  // Warning only
  // 仅警告
  string bar = 1;
  string bar_ = 2;
}
```

After running Prototiller, the equivalent code might look like this:

​	运行 Prototiller 后，等效代码可能如下所示：

```proto
edition = "2023";
features.json_format = LEGACY_BEST_EFFORT;

message Foo {
  string bar = 1;
  string bar_ = 2;
}
```

### `features.message_encoding`

This feature sets the behavior for encoding fields when serializing.

​	此特性设置序列化时编码字段的行为。

This feature doesn’t impact proto3 files, so this section doesn’t have a before and after of a proto3 file.

​	此特性不会影响 proto3 文件，因此没有提供 proto3 文件的前后对比。

Depending on the language, fields that are “group-like” may have some unexpected capitalization in generated code and in text-format, in order to provide backwards compatibility with proto2. Message fields are “group-like” if all of the following conditions are met:

​	根据语言的不同，某些“类组”字段可能在生成的代码和文本格式中会有意外的大写，以保持对 proto2 的向后兼容性。如果以下所有条件都满足，则消息字段为“类组”字段：

- Has `DELIMITED` message encoding specified
  - 指定了 `DELIMITED` 消息编码。

- Message type is defined in the same scope as the field
  - 消息类型定义在与字段相同的作用域中。

- Field name is exactly the lowercased type name
  - 字段名称与类型名称完全相同（小写）。


**Values available:** 可用值：

- `LENGTH_PREFIXED`: Fields are encoded using the LEN wire type described in [Message Structure]({{< ref "/docs/ProgrammingGuides/Encoding#structure" >}}).
  - `LENGTH_PREFIXED:` 使用 [消息结构]({{< ref "/docs/ProgrammingGuides/Encoding#structure" >}}) 中描述的 `LEN` 线类型对字段进行编码。

- `DELIMITED`: Message-typed fields are encoded as [groups]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2#groups" >}}).
  - `DELIMITED:` 消息类型字段以 [组]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2#groups" >}}) 的形式编码。


**Applicable to the following scopes:** File, Field

​	**适用范围：** 文件（File）、字段（Field）

**Default behavior in Edition 2023:** `LENGTH_PREFIXED`

​	**Edition 2023 的默认行为：** `LENGTH_PREFIXED`

**Behavior in proto2:** `LENGTH_PREFIXED` except for groups, which default to `DELIMITED`

​	**proto2 的行为：** 除组之外默认为 `LENGTH_PREFIXED`，组默认为 `DELIMITED`。

**Behavior in proto3:** `LENGTH_PREFIXED`. Proto3 doesn’t support `DELIMITED`.

​	**proto3 的行为：** `LENGTH_PREFIXED`，proto3 不支持 `DELIMITED`。

The following code sample shows a proto2 file:

​	以下代码示例展示了一个 proto2 文件：

```proto
syntax = "proto2";

message Foo {
  group Bar = 1 {
    optional int32 x = 1;
    repeated int32 y = 2;
  }
}
```

After running Prototiller, the equivalent code might look like this:

​	运行 Prototiller 后，等效代码可能如下所示：

```proto
edition = "2023";

message Foo {
  message Bar {
    int32 x = 1;
    repeated int32 y = 2;
  }
  Bar bar = 1 [features.message_encoding = DELIMITED];
}
```

### `features.repeated_field_encoding`

This feature is what the proto2/proto3 [`packed` option]({{< ref "/docs/ProgrammingGuides/Encoding#packed" >}}) for `repeated` fields has been migrated to in Editions.

​	此特性是 proto2/proto3 的 `packed` 选项在 Editions 中的迁移。

**Values available:**

- `PACKED`: `Repeated` fields of a primitive type are encoded as a single LEN record that contains each element concatenated.
  - `PACKED:` 原始类型的 `repeated` 字段被编码为一个包含所有元素的单个 `LEN` 记录。

- `EXPANDED`: `Repeated` fields are each encoded with the field number for each value.
  - `EXPANDED:` `repeated` 字段中的每个值都用字段编号进行编码。


**Applicable to the following scopes:** File, Field

​	**适用范围：** 文件（File）、字段（Field）

**Default behavior in Edition 2023:** `PACKED`

​	**Edition 2023 的默认行为：** `PACKED`

**Behavior in proto2:** `EXPANDED`

​	**proto2 的行为：** `EXPANDED`

**Behavior in proto3:** `PACKED`

​	**proto3 的行为：** `PACKED`

The following code sample shows a proto2 file:

​	以下代码示例展示了一个 proto2 文件：

```proto
syntax = "proto2";

message Foo {
  repeated int32 bar = 6 [packed=true];
  repeated int32 baz = 7;
}
```

After running Prototiller, the equivalent code might look like this:

​	运行 Prototiller 后，等效代码可能如下所示：

```proto
edition = "2023";
option features.repeated_field_encoding = EXPANDED;

message Foo {
  repeated int32 bar = 6 [features.repeated_field_encoding=PACKED];
  repeated int32 baz = 7;
}
```

The following shows a proto3 file:

​	以下代码展示了一个 proto3 文件：

```proto
syntax = "proto3";

message Foo {
  repeated int32 bar = 6;
  repeated int32 baz = 7 [packed=false];
}
```

After running Prototiller, the equivalent code might look like this:

​	运行 Prototiller 后，等效代码可能如下所示：

```proto
edition = "2023";

message Foo {
  repeated int32 bar = 6;
  repeated int32 baz = 7 [features.repeated_field_encoding=EXPANDED];
}
```

### `features.utf8_validation`

This feature sets how strings are validated. It applies to all languages except where there’s a language-specific `utf8_validation` feature that overrides it. See [`features.(pb.java).utf8_validation`](https://protobuf.dev/editions/features/#java-utf8_validation) for the Java-language-specific feature.

​	此特性设置字符串的验证方式。适用于除特定语言外的所有语言，如果某语言有语言特定的 `utf8_validation` 特性，则会覆盖此设置。有关 Java 的语言特定特性，请参见 [`features.(pb.java).utf8_validation`](https://protobuf.dev/editions/features/#java-utf8_validation)。

This feature doesn’t impact proto3 files, so this section doesn’t have a before and after of a proto3 file.

​	此特性不会影响 proto3 文件，因此没有提供 proto3 文件的前后对比。

**Values available:** 可用值：

- `VERIFY`: The runtime should verify UTF-8. This is the default proto3 behavior.
  - `VERIFY:` 运行时应验证 UTF-8。这是 proto3 的默认行为。

- `NONE`: The field behaves like an unverified `bytes` field on the wire. Parsers may handle this type of field in an unpredictable way, such as replacing invalid characters. This is the default proto2 behavior.
  - `NONE:` 字段在传输时的行为类似于未经验证的 `bytes` 字段。解析器可能以不可预测的方式处理此类字段，例如替换无效字符。这是 proto2 的默认行为。


**Applicable to the following scopes:** File, Field

​	**适用范围：** 文件（File）、字段（Field）

**Default behavior in Edition 2023:** `VERIFY`

​	**Edition 2023 的默认行为：** `VERIFY`

**Behavior in proto2:** `NONE`

​	**proto2 的行为：** `NONE`

**Behavior in proto3:** `VERIFY`

​	**proto3 的行为：** `VERIFY`

The following code sample shows a proto2 file:

​	以下代码示例展示了一个 proto2 文件：

```proto
syntax = "proto2";

message MyMessage {
  string foo = 1;
}
```

After running Prototiller, the equivalent code might look like this:

​	运行 Prototiller 后，等效代码可能如下所示：

```proto
edition = "2023";

message MyMessage {
  string foo = 1 [features.utf8_validation = NONE];
}
```

### 语言特定特性 Language-specific Features

Some features apply to specific languages, and not to the same protos in other languages. Using these features requires you to import the corresponding *_features.proto file from the language’s runtime. The examples in the following sections show these imports.

​	某些特性仅适用于特定语言，而不适用于其他语言中的相同 proto。使用这些特性需要导入对应语言运行时的 *_features.proto 文件。以下各节展示了这些导入。

#### `features.(pb.cpp/pb.java).legacy_closed_enum`

**Languages:** C++, Java

This feature determines whether a field with an open enum type should be behave as if it was a closed enum. This allows editions to reproduce [non-conformant behavior]({{< ref "/docs/ProgrammingGuides/EnumBehavior" >}}) in Java and C++ from proto2 and proto3.

​	此特性决定一个具有开放枚举类型的字段是否应表现为封闭枚举。这允许 Editions 重现 Java 和 C++ 中 proto2 和 proto3 的 [非标准行为]({{< ref "/docs/ProgrammingGuides/EnumBehavior" >}})。

This feature doesn’t impact proto3 files, and so this section doesn’t have a before and after of a proto3 file.

​	此特性不会影响 proto3 文件，因此没有提供 proto3 文件的前后对比。

**Values available: **可用值：

- `true`: Treats the enum as closed regardless of [`enum_type`](https://protobuf.dev/editions/features/#enum_type).
  - `true:` 无视 [`enum_type`](https://protobuf.dev/editions/features/#enum_type) 设置，将枚举视为封闭枚举。

- `false`: Respect whatever is set in the `enum_type`.
  - `false:` 遵循 `enum_type` 中的设置。


**Applicable to the following scopes:** File, Field

​	**适用范围：** 文件（File）、字段（Field）

**Default behavior in Edition 2023:** `false`

​	**Edition 2023 的默认行为：** `false`

**Behavior in proto2:** `true`

​	**proto2 的行为：** `true`

**Behavior in proto3:** `false`

​	**proto3 的行为：** `false`

The following code sample shows a proto2 file:

​	以下代码示例展示了一个 proto2 文件：

```proto
syntax = "proto2";

import "myproject/proto3file.proto";

message Msg {
  myproject.proto3file.Proto3Enum name = 1;
}
```

After running Prototiller, the equivalent code might look like this:

​	运行 Prototiller 后，等效代码可能如下所示：

```proto
edition = "2023";

import "myproject/proto3file.proto";

import "google/protobuf/cpp_features.proto";
import "google/protobuf/java_features.proto";

message Msg {
  myproject.proto3file.Proto3Enum name = 1 [
    features.(pb.cpp).legacy_closed_enum = true,
    features.(pb.java).legacy_closed_enum = true
  ];
}
```

#### `features.(pb.cpp).string_type`

**Languages:** C++

This feature determines how generated code should treat string fields. This replaces the `ctype` option from proto2 and proto3, and offers a new `string_view` feature. In Edition 2023, either `ctype` or `string_view` may be specified on a field, but not both.

​	此特性决定生成的代码应如何处理字符串字段。它取代了 proto2 和 proto3 的 `ctype` 选项，并提供了一个新的 `string_view` 功能。在 Edition 2023 中，字段上可以同时指定 `ctype` 或 `string_view`，但不能同时使用。

**Values available: **可用值：

- `VIEW`: Generates `string_view` accessors for the field. This will be the default in a future edition.
  - `VIEW:` 为字段生成 `string_view` 访问器。这将在未来edition中成为默认值。

- `CORD`: Generates `Cord` accessors for the field.
  - `CORD:` 为字段生成 `Cord` 访问器。

- `STRING`: Generates `string` accessors for the field.
  - `STRING:` 为字段生成 `string` 访问器。


**Applicable to the following scopes:** File, Field

​	**适用范围：** 文件（File）、字段（Field）

**Default behavior in Edition 2023:** `STRING`

​	**Edition 2023 的默认行为：** `STRING`

**Behavior in proto2:** `STRING`

​	**proto2 的行为：** `STRING`

**Behavior in proto3:** `STRING`

​	**proto3 的行为：** `STRING`

The following code sample shows a proto2 file:

​	以下代码示例展示了一个 proto2 文件：

```proto
syntax = "proto2";

message Foo {
  optional string bar = 6;
  optional string baz = 7 [ctype = CORD];
}
```

After running Prototiller, the equivalent code might look like this:

​	运行 Prototiller 后，等效代码可能如下所示：

```proto
edition = "2023";

import "google/protobuf/cpp_features.proto";

message Foo {
  string bar = 6;
  string baz = 7 [features.(pb.cpp).string_type = CORD];
}
```

The following shows a proto3 file:

​	以下代码展示了一个 proto3 文件：

```proto
syntax = "proto3"

message Foo {
  string bar = 6;
  string baz = 7 [ctype = CORD];
}
```

After running Prototiller, the equivalent code might look like this:

​	运行 Prototiller 后，等效代码可能如下所示：

```proto
edition = "2023";

import "google/protobuf/cpp_features.proto";

message Foo {
  string bar = 6;
  string baz = 7 [features.(pb.cpp).string_type = CORD];
}
```

#### `features.(pb.java).utf8_validation`

**Languages:** Java

This language-specific feature enables you to override the file-level settings at the field level for Java only.

​	此语言特定特性允许您在字段级别覆盖文件级设置，仅适用于 Java。

This feature doesn’t impact proto3 files, and so this section doesn’t have a before and after of a proto3 file.

​	此特性不会影响 proto3 文件，因此没有提供 proto3 文件的前后对比。

**Values available: **可用值：

- `DEFAULT`: The behavior matches that set by [`features.utf8_validation`](https://protobuf.dev/editions/features/#utf8_validation).
  - `DEFAULT:` 行为与 [`features.utf8_validation`](https://protobuf.dev/editions/features/#utf8_validation) 设置一致。

- `VERIFY`: Overrides the file-level `features.utf8_validation` setting to force it to `VERIFY` for Java only.
  - `VERIFY:` 覆盖文件级别的 `features.utf8_validation` 设置，仅对 Java 强制为 `VERIFY`。


**Applicable to the following scopes:** Field, File

​	**适用范围：** 字段（Field）、文件（File）

**Default behavior in Edition 2023:** `DEFAULT`

​	**Edition 2023 的默认行为：** `DEFAULT`

**Behavior in proto2:** `DEFAULT`

​	**proto2 的行为：** `DEFAULT`

**Behavior in proto3:** `DEFAULT`

​	**proto3 的行为：** `DEFAULT`

The following code sample shows a proto2 file:

​	以下代码示例展示了一个 proto2 文件：

```proto
syntax = "proto2";

option java_string_check_utf8=true;

message MyMessage {
  string foo = 1;
  string bar = 2;
}
```

After running Prototiller, the equivalent code might look like this:

​	运行 Prototiller 后，等效代码可能如下所示：

```proto
edition = "2023";

import "google/protobuf/java_features.proto";

option features.utf8_validation = NONE;
option features.(pb.java).utf8_validation = VERIFY;
message MyMessage {
  string foo = 1;
}
```

### 保留 proto2 或 proto3 行为 Preserving proto2 or proto3 Behavior

You may want to move to the editions format but not deal with updates to the way that generated code behaves yet. This section shows the changes that the Prototiller tool makes to your .proto files to make Edition 2023 protos behave like a proto2 or proto3 file.

​	您可能希望迁移到 Editions 格式，但暂时不想处理生成代码行为的变化。本节展示了 Prototiller 工具对 `.proto` 文件所做的更改，使 **Edition 2023** 的 proto 文件表现得像 proto2 或 proto3 文件。

After these changes are made at the file level, you get the proto2 or proto3 defaults. You can override at lower levels (message level, field level) to consider additional behavior differences (such as [required, proto3 optional](https://protobuf.dev/editions/features/#caveats)) or if you want your definition to only be *mostly* like proto2 or proto3.

​	在文件级别应用这些更改后，您将获得 proto2 或 proto3 的默认行为。您可以在较低级别（如消息级别、字段级别）覆盖以处理额外的行为差异（例如 [required、proto3 optional](https://protobuf.dev/editions/features/#caveats)）或如果您希望定义只“部分像” proto2 或 proto3。

We recommend using Prototiller unless you have a specific reason not to. To manually apply all of these instead of using Prototiller, add the content from the following sections to the top of your .proto file.

​	我们建议使用 Prototiller，除非您有特定原因不使用它。若不使用 Prototiller，请手动将以下内容添加到 `.proto` 文件顶部。

### Proto2 行为 Proto2 Behavior

```proto
edition = "2023";

import "google/protobuf/cpp_features.proto";
import "google/protobuf/java_features.proto";

option features.field_presence = EXPLICIT;
option features.enum_type = CLOSED;
option features.repeated_field_encoding = EXPANDED;
option features.json_format = LEGACY_BEST_EFFORT;
option features.utf8_validation = NONE;
option features.(pb.cpp).legacy_closed_enum = true;
option features.(pb.java).legacy_closed_enum = true;
```

### Proto3 行为 Proto3 Behavior

```proto
// proto3 behaviors
edition = "2023";

import "google/protobuf/cpp_features.proto";
import "google/protobuf/java_features.proto";

option features.field_presence = IMPLICIT;
option features.enum_type = OPEN;
// `packed=false` needs to be transformed to field-level repeated_field_encoding
// features in Editions syntax
option features.json_format = ALLOW;
option features.utf8_validation = VERIFY;
option features.(pb.cpp).legacy_closed_enum = false;
option features.(pb.java).legacy_closed_enum = false;
```

### 注意事项和例外 Caveats and Exceptions

This section shows the changes that you’ll need to make manually if you choose not to use Prototiller.

​	如果选择不使用 Prototiller，您需要手动处理以下更改：

Setting the file-level defaults shown in the previous section sets the default behaviors in most cases, but there are a few exceptions.

- `optional`: Remove all instances of the `optional` label and change the [`features.field_presence`](https://protobuf.dev/editions/features/#field_presence) to `EXPLICIT` if the file default is `IMPLICIT`. **`optional` 标签：** 删除所有 `optional` 标签，并将文件默认的 [`features.field_presence`](https://protobuf.dev/editions/features/#field_presence) 改为 `EXPLICIT`（如果文件默认值为 `IMPLICIT`）。
- `required`: Remove all instances of the `required` label and add the [`features.field_presence=LEGACY_REQUIRED`](https://protobuf.dev/editions/features/#field_presence) option at the field level. **`required` 标签：** 删除所有 `required` 标签，并在字段级别添加 [`features.field_presence=LEGACY_REQUIRED`](https://protobuf.dev/editions/features/#field_presence) 选项。
- `groups`: Unwrap the `groups` into a separate message and add the `features.message_encoding = DELIMITED` option at the field level. See [`features.message_encoding`](https://protobuf.dev/editions/features/#message_encoding) for more on this. **`groups`：** 将 `groups` 解包为单独的消息，并在字段级别添加 [`features.message_encoding = DELIMITED`](https://protobuf.dev/editions/features/#message_encoding) 选项。
- `java_string_check_utf8`: Remove this file option and replace it with the [`features.(pb.java).utf8_validation`](https://protobuf.dev/editions/features/#java-utf8_validation). You’ll need to import Java features, as covered in [Language-specific Features](https://protobuf.dev/editions/features/#lang-specific). **`java_string_check_utf8`：** 删除此文件选项，并将其替换为 [`features.(pb.java).utf8_validation`](https://protobuf.dev/editions/features/#java-utf8_validation)。需要导入 Java 特性，参见 [语言特定特性](https://protobuf.dev/editions/features/#lang-specific)。
- `packed`: For proto2 files converted to editions format, remove the `packed` field option and add `[features.repeated_field_encoding=PACKED]` at the field level when you don’t want the `EXPANDED` behavior that you set in [Proto2 Behavior](https://protobuf.dev/editions/features/#proto2-behavior).  For proto3 files converted to editions format, add `[features.repeated_field_encoding=EXPANDED]` at the field level when you don’t want the default proto3 behavior. 对于转换为 Editions 格式的 proto2 文件，删除 `packed` 字段选项，并在字段级别添加 `[features.repeated_field_encoding=PACKED]` 选项，如果不希望文件默认的 `EXPANDED` 行为。 对于转换为 Editions 格式的 proto3 文件，在字段级别添加 `[features.repeated_field_encoding=EXPANDED]` 选项，如果不希望 proto3 的默认行为。
