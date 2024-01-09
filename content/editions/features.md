+++
title = "版本的功能设置"
weight = 43
description = "Protobuf 版本特性及其对 protobuf 行为的影响。"
type = "docs"

+++

> 原文网址： https://protobuf.dev/editions/features/

## Feature Settings for Editions 版本的功能设置

Protobuf Editions features and how they affect protobuf behavior.

​	Protobuf 版本特性及其对 protobuf 行为的影响。



This topic provides an overview of the features that are included in Edition 2023. Each subsequent edition’s features will be added to this topic. We announce new editions in [the News section](https://protobuf.dev/news).

​	本主题概述了 2023 年版本中包含的特性。此主题将添加每个后续版本的特性。我们在新闻部分宣布新版本。

## Prototiller

Prototiller is a command-line tool that converts proto2 and proto3 definition files to Editions syntax. It hasn’t been released yet, but is referenced throughout this topic.

​	Prototiller 是一个命令行工具，可将 proto2 和 proto3 定义文件转换为 Editions 语法。它尚未发布，但整个主题中都引用了它。

## 功能 Features 

The following sections include all of the behaviors that are configurable using features in Edition 2023. [Preserving proto2 or proto3 Behavior](https://protobuf.dev/editions/features/#preserving) shows how to override the default behaviors so that your proto definition files act like proto2 or proto3 files. For more information on how Editions and Features work together to set behavior, see [Protobuf Editions Overview](https://protobuf.dev/editions/overview).

​	以下部分包括所有可使用 2023 年版本中的功能配置的行为。保留 proto2 或 proto3 行为展示了如何覆盖默认行为，以便您的 proto 定义文件像 proto2 或 proto3 文件一样运行。有关版本和功能如何协同工作以设置行为的更多信息，请参阅 Protobuf 版本概述。

### `features.enum_type`

This feature sets the behavior for how enum values that aren’t contained within the defined set are handled.

​	此功能设置了对未包含在已定义集合中的枚举值进行处理的方式的行为。

This feature doesn’t impact proto3 files, so this section doesn’t have a before and after of a proto3 file.

​	此功能不影响 proto3 文件，因此此部分没有 proto3 文件的对比。

**Values available: 可用的值：**

- `CLOSED:` Closed enums store enum values that are out of range in the unknown field set.
- `CLOSED:` 封闭枚举将超出范围的枚举值存储在未知字段集中。
- `OPEN:` Open enums parse out of range values into their fields directly.
- `OPEN:` 开放枚举将超出范围的值直接解析到其字段中。

**Applicable to the following elements:** File, Enum

​	适用于以下元素：文件、枚举

**Default behavior in Edition 2023:** `OPEN`

​	2023 年版本中的默认行为： `OPEN`

**Behavior in proto2:** `CLOSED`

​	proto2 中的行为： `CLOSED`

**Behavior in proto3:** `OPEN`

​	proto3 中的行为： `OPEN`

The following code sample shows a proto2 file:

​	以下代码示例显示了一个 proto2 文件：

```proto
syntax = "proto2";

enum Foo {
  A = 2;
  B = 4;
  C = 6;
}
```

After running [Prototiller](https://protobuf.dev/editions/features/#prototiller), the equivalent code might look like this:

​	在运行 Prototiller 之后，等效代码可能如下所示：

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

​	此功能设置了跟踪字段存在或 protobuf 字段是否具有值的概念的行为。

**Values available: 可用的值：**

- `LEGACY_REQUIRED`: The field is required for parsing and serialization. Any explicitly-set value is serialized onto the wire (even if it is the same as the default value).
- `LEGACY_REQUIRED` ：该字段是解析和序列化的必需字段。任何显式设置的值都会序列化到数据线上（即使它与默认值相同）。
- `EXPLICIT`: The field has explicit presence tracking. Any explicitly-set value is serialized onto the wire (even if it is the same as the default value). For singular primitive fields, `has_*` functions are generated for fields set to `EXPLICIT`.
- `EXPLICIT` ：该字段具有显式存在跟踪。任何显式设置的值都会序列化到数据线上（即使它与默认值相同）。对于奇数基本字段，会为设置为 `EXPLICIT` 的字段生成 `has_*` 函数。
- `IMPLICIT`: The field has no presence tracking. The default value is not serialized onto the wire (even if it is explicitly set). `has_*` functions are not generated for fields set to `IMPLICIT`.
- `IMPLICIT` ：该字段没有存在跟踪。默认值不会序列化到数据线上（即使它被显式设置）。不会为设置为 `IMPLICIT` 的字段生成 `has_*` 函数。

**Applicable to the following elements:** File, Field

​	适用于以下元素：文件、字段

**Default value in the Edition 2023:** `EXPLICIT`

​	2023 年版的默认值： `EXPLICIT`

**Behavior in proto2:** `EXPLICIT`

​	proto2 中的行为： `EXPLICIT`

**Behavior in proto3:** `IMPLICIT` unless the field has the `optional` label, in which case it behaves like `EXPLICIT`. See [Presence in Proto3 APIs](https://protobuf.dev/programming-guides/field_presence#presence-in-proto3-apis) for more information.

​	proto3 中的行为： `IMPLICIT` 除非该字段具有 `optional` 标签，在这种情况下，其行为类似于 `EXPLICIT` 。有关更多信息，请参阅 Proto3 API 中的存在。

The following code sample shows a proto2 file:

​	以下代码示例显示了一个 proto2 文件：

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

​	以下显示了一个 proto3 文件：

```proto
syntax = "proto3"

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

​	请注意， `required` 和 `optional` 标签不再存在于 Editions 中，因为相应行为已通过 `field_presence` 功能显式设置。

### `features.json_format`

This feature sets the behavior for JSON parsing and serialization.

​	此功能设置 JSON 解析和序列化的行为。

This feature doesn’t impact proto3 files, so this section doesn’t have a before and after of a proto3 file. Editions behavior matches the behavior in proto3.

​	此功能不会影响 proto3 文件，因此本部分没有 proto3 文件的对比。

**Values available:
Editions 行为与 proto3 中的行为相匹配。**

- `ALLOW`: A runtime must allow JSON parsing and serialization. Checks are applied at the proto level to make sure that there is a well-defined mapping to JSON.
- `ALLOW` ：运行时必须允许 JSON 解析和序列化。在 proto 级别应用检查以确保存在与 JSON 的明确定义的映射。
- `LEGACY_BEST_EFFORT`: A runtime does the best it can to parse and serialize JSON. Certain protos are allowed that can result in unspecified behavior at runtime (such as many:1 or 1:many mappings).
- `LEGACY_BEST_EFFORT` ：运行时尽最大努力解析和序列化 JSON。允许某些 proto，这些 proto 可能会导致运行时行为未指定（例如多对一或一对多映射）。

**Applicable to the following elements:** File, Message, Enum

​	适用于以下元素：文件、消息、枚举

**Default behavior in Edition 2023:** `ALLOW`

​	2023 年版的默认行为： `ALLOW`

**Behavior in proto2:** `LEGACY_BEST_EFFORT`

​	proto2 中的行为： `LEGACY_BEST_EFFORT`

**Behavior in proto3:** `ALLOW`

​	proto3 中的行为： `ALLOW`

The following code sample shows a proto2 file:

​	以下代码示例显示 proto2 文件：

```proto
syntax = "proto2";

message Foo {
  // Warning only
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

​	此功能设置序列化时字段的编码行为。

This feature doesn’t impact proto3 files, so this section doesn’t have a before and after of a proto3 file.

​	此功能不会影响 proto3 文件，因此此部分没有 proto3 文件的对比。

**Values available: 可用值：**

- `LENGTH_PREFIXED`: Fields are encoded using the LEN wire type described in [Message Structure](https://protobuf.dev/programming-guides/encoding#structure).
- `LENGTH_PREFIXED` ：字段使用消息结构中描述的 LEN 线路类型进行编码。
- `DELIMITED`: Message-typed fields are encoded as [groups](https://protobuf.dev/programming-guides/proto2#groups).
- `DELIMITED` ：消息类型字段编码为组。

**Applicable to the following elements:** File, Field

​	适用于以下元素：文件、字段

**Default behavior in Edition 2023:** `LENGTH_PREFIXED`

​	2023 年版的默认行为： `LENGTH_PREFIXED`

**Behavior in proto2:** `LENGTH_PREFIXED` except for groups, which default to `DELIMITED`

​	proto2 中的行为： `LENGTH_PREFIXED` ，但组除外，组默认为 `DELIMITED`

**Behavior in proto3:** `LENGTH_PREFIXED`. Proto3 doesn’t support `DELIMITED`.

​	proto3 中的行为： `LENGTH_PREFIXED` 。Proto3 不支持 `DELIMITED` 。

The following code sample shows a proto2 file:

​	以下代码示例显示了一个 proto2 文件：

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
    optional int32 x = 1;
    repeated int32 y = 2;
  }
  Bar bar = 1 [features.message_encoding = DELIMITED];
}
```

### `features.repeated_field_encoding`

This feature is what the proto2/proto3 [`packed` option](https://protobuf.dev/programming-guides/encoding#packed) for `repeated` fields has been migrated to in Editions.

​	此功能是 `packed` 字段的 proto2/proto3 `repeated` 选项在版本中迁移到的功能。

**Values available: 可用值：**

- `PACKED`: `Repeated` fields of a primitive type are encoded as a single LEN record that contains each element concatenated.
- `PACKED` ： `Repeated` 原始类型的字段编码为包含每个连接元素的单个 LEN 记录。
- `EXPANDED`: `Repeated` fields are each encoded with the field number for each value.
- `EXPANDED` ： `Repeated` 每个字段都使用每个值的字段编号进行编码。

**Applicable to the following elements:** File, Field

​	适用于以下元素：文件、字段

**Default behavior in Edition 2023:** `PACKED`

​	Edition 2023 中的默认行为： `PACKED`

**Behavior in proto2:** `EXPANDED`

​	proto2 中的行为： `EXPANDED`

**Behavior in proto3:** `PACKED`

​	proto3 中的行为： `PACKED`

The following code sample shows a proto2 file:

​	以下代码示例显示 proto2 文件：

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

​	以下显示 proto3 文件：

```proto
syntax = "proto3"

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

​	此功能设置字符串的验证方式。它适用于除具有覆盖它的特定于语言的 `utf8_validation` 功能的语言之外的所有语言。有关特定于 Java 语言的功能，请参阅 `features.(pb.java).utf8_validation` 。此功能不会影响 proto3 文件，因此本部分没有 proto3 文件的对比。

This feature doesn’t impact proto3 files, so this section doesn’t have a before and after of a proto3 file.
可用的值：

**Values available:
：运行时应验证 UTF-8。这是默认的 proto3 行为。**

- `VERIFY`: The runtime should verify UTF-8. This is the default proto3 behavior.
  `VERIFY` ：字段的行为类似于线路上未验证的 字段。解析器可能以不可预测的方式处理此类字段，例如替换无效字符。这是默认的 proto2 行为。
- `NONE`: The field behaves like an unverified `bytes` field on the wire. Parsers may handle this type of field in an unpredictable way, such as replacing invalid characters. This is the default proto2 behavior.
- 适用于以下元素：文件、字段

**Applicable to the following elements:** File, Field

​	2023 年版的默认行为：

**Default behavior in Edition 2023:** `VERIFY`

​	proto2 中的行为： `VERIFY`

**Behavior in proto2:** `NONE`

​	proto3 中的行为： `NONE`

**Behavior in proto3:** `VERIFY`

The following code sample shows a proto2 file:

​	以下代码示例显示了一个 proto2 文件：

```proto
syntax = "proto2";

message MyMessage {
  string foo = 1;
}
```

After running Prototiller, the equivalent code might look like this:

​	在运行 Prototiller 后，等效代码可能如下所示：

```proto
edition = "2023";

message MyMessage {
  string foo = 1 [features.utf8_validation = NONE];
}
```

### 特定语言的功能 Language-specific Features 

Some features apply to specific languages, and not to the same protos in other languages. Using these features requires you to import the corresponding `*_features.proto` file from the language’s runtime. The examples in the following sections show these imports._

​	某些功能适用于特定语言，而不适用于其他语言中的相同 proto。使用这些功能需要您从该语言的运行时导入相应的 `*_features.proto` 文件。以下部分中的示例显示了这些导入。

#### `features.(pb.cpp/pb.java).legacy_closed_enum`

**Languages:** C++, Java

​	语言：C++、Java

This feature determines whether a field with an open enum type should be behave as if it was a closed enum. This allows editions to reproduce [non-conformant behavior](https://protobuf.dev/programming-guides/enum) in Java and C++ from proto2 and proto3.

​	此功能确定具有开放枚举类型的字段是否应表现得像封闭枚举一样。这允许版本在 Java 和 C++ 中从 proto2 和 proto3 再现不符合规范的行为。

This feature doesn’t impact proto3 files, and so this section doesn’t have a before and after of a proto3 file.

​	此功能不会影响 proto3 文件，因此本部分没有 proto3 文件的对比。

**Values available: 可用的值：**

- `true`: Treats the enum as closed regardless of [`enum_type`](https://protobuf.dev/editions/features/#enum_type).
- `true` ：无论 `enum_type` 如何，都将枚举视为封闭的。
- `false`: Respect whatever is set in the `enum_type`.
- `false` ：尊重 `enum_type` 中设置的任何内容。

**Applicable to the following elements:** File, Field

​	适用于以下元素：文件、字段

**Default behavior in Edition 2023:** `false`

​	Edition 2023 中的默认行为： `false`

**Behavior in proto2:** `true`

​	proto2 中的行为： `true`

**Behavior in proto3:** `false`

​	proto3 中的行为： `false`

The following code sample shows a proto2 file:

​	以下代码示例显示了一个 proto2 文件：

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

#### `features.(pb.java).utf8_validation`

**Languages:** Java

​	语言：Java

This language-specific feature enables you to override the file-level settings at the field level for Java only.

​	此特定于语言的功能使您能够仅针对 Java 在字段级别覆盖文件级设置。

This feature doesn’t impact proto3 files, and so this section doesn’t have a before and after of a proto3 file.

​	此功能不会影响 proto3 文件，因此本部分没有 proto3 文件的对比。

**Values available: 可用值：**

- `DEFAULT`: The behavior matches that set by [`features.utf8_validation`](https://protobuf.dev/editions/features/#utf8_validation).
- `DEFAULT` ：行为与 `features.utf8_validation` 设置的相匹配。
- `VERIFY`: Overrides the file-level `features.utf8_validation` setting to force it to `VERIFY` for Java only.
- `VERIFY` ：覆盖文件级 `features.utf8_validation` 设置，强制其仅对 Java `VERIFY` 。

**Applicable to the following elements:** Field, File

​	适用于以下元素：字段、文件

**Default behavior in Edition 2023:** `DEFAULT`

​	Edition 2023 中的默认行为： `DEFAULT`

**Behavior in proto2:** `DEFAULT`

​	proto2 中的行为： `DEFAULT`

**Behavior in proto3:** `DEFAULT`

​	proto3 中的行为： `DEFAULT`

The following code sample shows a proto2 file:

​	以下代码示例显示了一个 proto2 文件：

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

## 保留 proto2 或 proto3 行为 Preserving proto2 or proto3 Behavior 

You may want to move to the editions format but not deal with updates to the way that generated code behaves yet. This section shows the changes that the Prototiller tool makes to your .proto files to make Edition 2023 protos behave like a proto2 or proto3 file.

​	您可能想要迁移到版本格式，但暂时不处理生成代码行为方式的更新。本部分显示 Prototiller 工具对 .proto 文件所做的更改，以使 Edition 2023 协议像 proto2 或 proto3 文件一样运行。

After these changes are made at the file level, you get the proto2 or proto3 defaults. You can override at lower levels (message level, field level) to consider additional behavior differences (such as [required, proto3 optional](https://protobuf.dev/editions/features/#caveats)) or if you want your definition to only be *mostly* like proto2 or proto3.

​	在文件级别进行这些更改后，您将获得 proto2 或 proto3 默认值。您可以在较低级别（消息级别、字段级别）进行覆盖，以考虑其他行为差异（例如必需的、proto3 可选的），或者如果您希望您的定义仅像 proto2 或 proto3 一样。

We recommend using Prototiller unless you have a specific reason not to. To manually apply all of these instead of using Prototiller, add the content from the following sections to the top of your .proto file.

​	我们建议使用 Prototiller，除非您有具体原因不使用。要手动应用所有这些内容而不是使用 Prototiller，请将以下部分的内容添加到 .proto 文件的顶部。

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

​	如果您选择不使用 Prototiller，本部分显示您需要手动进行的更改。

Setting the file-level defaults shown in the previous section sets the default behaviors in most cases, but there are a few exceptions.

​	在上一部分中显示的文件级默认值设置在大多数情况下设置了默认行为，但有一些例外。

- `optional`: Remove all instances of the `optional` label and change the [`features.field_presence`](https://protobuf.dev/editions/features/#field_presence) to `EXPLICIT` if the file default is `IMPLICIT`.
- `optional` ：删除 `optional` 标签的所有实例，如果文件默认值为 `IMPLICIT` ，则将 `features.field_presence` 更改为 `EXPLICIT` 。
- `required`: Remove all instances of the `required` label and add the [`features.field_presence=LEGACY_REQUIRED`](https://protobuf.dev/editions/features/#field_presence) option at the field level.
- `required` ：移除 `required` 标签的所有实例，并在字段级别添加 `features.field_presence=LEGACY_REQUIRED` 选项。
- `groups`: Unwrap the `groups` into a separate message and add the `features.message_encoding = DELIMITED` option at the field level. See [`features.message_encoding`](https://protobuf.dev/editions/features/#message_encoding) for more on this.
- `groups` ：将 `groups` 解包到单独的消息中，并在字段级别添加 `features.message_encoding = DELIMITED` 选项。有关此内容的更多信息，请参阅 `features.message_encoding` 。
- `java_string_check_utf8`: Remove this file option and replace it with the [`features.(pb.java).utf8_validation`](https://protobuf.dev/editions/features/#java-utf8_validation). You’ll need to import Java features, as covered in [Language-specific Features](https://protobuf.dev/editions/features/#lang-specific).
- `java_string_check_utf8` ：移除此文件选项，并用 `features.(pb.java).utf8_validation` 替换它。您需要导入 Java 功能，如语言特定功能中所述。
- `packed`: For proto2 files converted to editions format, remove the `packed` field option and add `[features.repeated_field_encoding=PACKED]` at the field level when you don’t want the `EXPANDED` behavior that you set in [Proto2 Behavior](https://protobuf.dev/editions/features/#proto2-behavior). For proto3 files converted to editions format, add `[features.repeated_field_encoding=EXPANDED]` at the field level when you don’t want the default proto3 behavior.
- `packed` ：对于已转换为 editions 格式的 proto2 文件，当您不希望在 Proto2 行为中设置的 `EXPANDED` 行为时，请移除 `packed` 字段选项，并在字段级别添加 `[features.repeated_field_encoding=PACKED]` 。对于已转换为 editions 格式的 proto3 文件，当您不希望默认的 proto3 行为时，请在字段级别添加 `[features.repeated_field_encoding=EXPANDED]` 。
