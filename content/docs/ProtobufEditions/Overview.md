+++
title = "概述"
date = 2024-11-17T09:35:36+08:00
weight = 1
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

> 原文：[https://protobuf.dev/editions/overview/](https://protobuf.dev/editions/overview/)
>
> 收录该文档的时间：`2024-11-17T09:35:36+08:00`

# Protobuf Editions Overview - Protobuf Editions概述

An overview of the Protobuf Editions functionality.

​	Protobuf Editions功能概览。

Protobuf Editions replace the proto2 and proto3 designations that we have used for Protocol Buffers. Instead of adding `syntax = "proto2"` or `syntax = "proto3"` at the top of proto definition files, you use an edition number, such as `edition = "2023"`, to specify the default behaviors your file will have. Editions enable the language to evolve incrementally over time.

​	Protobuf Editions取代了我们之前在 Protocol Buffers 中使用的 proto2 和 proto3 标记。在 proto 定义文件顶部不再添加 `syntax = "proto2"` 或 `syntax = "proto3"`，而是使用类似 `edition = "2023"` 的版本号来指定文件的默认行为。版本允许语言随着时间的推移逐步演化。

Instead of the hardcoded behaviors that older versions have had, editions represent a collection of [features]({{< ref "/docs/ProtobufEditions/FeatureSettingsforEditions" >}}) with a default value (behavior) per feature. Features are options on a file, message, field, enum, and so on, that specify the behavior of protoc, the code generators, and protobuf runtimes. You can explicitly override a behavior at those different levels (file, message, field, …) when your needs don’t match the default behavior for the edition you’ve selected. You can also override your overrides. The [section later in this topic on lexical scoping](https://protobuf.dev/editions/overview/#scoping) goes into more detail on that.

​	相比于旧版本中硬编码的行为，版本表示一组带有默认值（行为）的[特性]({{< ref "/docs/ProtobufEditions/FeatureSettingsforEditions" >}})。特性是文件、消息、字段、枚举等上的选项，用于指定 protoc、代码生成器和 Protobuf 运行时的行为。当所选版本的默认行为不满足您的需求时，您可以在不同级别（文件、消息、字段等）显式覆盖行为，甚至可以覆盖先前的覆盖行为。有关详细信息，请参阅本文稍后关于[词法作用域](https://protobuf.dev/editions/overview/#scoping)的部分。

*The latest released edition is 2023.*

​	**目前发布的最新版本是 2023。**

## 特性的生命周期 Lifecycle of a Feature

Editions provide the fundamental increments for the lifecycle of a feature. Features have an expected lifecycle: introducing it, changing its default behavior, deprecating it, and then removing it. For example:

​	版本提供了特性生命周期的基本增量。特性通常具有以下生命周期：引入、改变默认行为、弃用，最后移除。例如：

1. Edition 2031 creates `feature.amazing_new_feature` with a default value of `false`. This value maintains the same behavior as all earlier editions. That is, it defaults to no impact. **Edition 2031** 创建了 `feature.amazing_new_feature`，其默认值为 `false`。该值保持与之前所有版本相同的行为，即默认无影响。

2. Developers update their .proto files to `edition = "2031"`. 开发者将其 `.proto` 文件更新为 `edition = "2031"`。

3. A later edition, such as edition 2033, switches the default of `feature.amazing_new_feature` from `false` to `true`. This is the desired behavior for all protos, and the reason that the protobuf team created the feature. 在稍后的版本（例如 **Edition 2033**）中，将 `feature.amazing_new_feature` 的默认值从 `false` 切换为 `true`。这是 Protobuf 团队创建该特性的预期行为。

   Using the Prototiller tool to migrate earlier versions of proto files to edition 2033 adds explicit `feature.amazing_new_feature = false` entries as needed to continue to retain the previous behavior. Developers remove these newly-added settings when they want the new behavior to apply to their .proto files.

   ​	使用 **Prototiller** 工具将早期版本的 `.proto` 文件迁移到 **Edition 2033** 时，会根据需要添加显式的 `feature.amazing_new_feature = false` 条目，以保留先前的行为。当开发者希望其 `.proto` 文件采用新行为时，可移除这些新添加的设置。

1. At some point, `feature.amazing_new_feature` is marked deprecated in an edition and removed in a later one. 在某个时间点，`feature.amazing_new_feature` 会在一个版本中标记为已弃用，并在稍后的版本中被移除。

   When a feature is removed, the code generators for that behavior and the runtime libraries that support it may also be removed. The timelines will be generous, though. Following the example in the earlier steps of the lifecycle, the deprecation might happen in edition 2034 but not be removed until edition 2036, roughly two years later. Removing a feature will always initiate a major version bump.
   
   ​	当特性被移除时，支持该行为的代码生成器和运行时库也可能被移除。但时间线会非常宽松。例如，在上述生命周期的早期步骤中，弃用可能发生在 **Edition 2034**，而完全移除可能要到 **Edition 2036**，大约两年后。移除特性将始终伴随着版本号的主要升级。

Because of this lifecycle, any `.proto` file that does not use deprecated features has a no-op upgrade from one edition to the next. You will have the full window of the Google migration plus the deprecation window to upgrade your code.

​	由于这一生命周期，任何未使用弃用特性的 `.proto` 文件在从一个edition升级到下一个edition时不会有任何影响。您将在 Google 提供的迁移窗口以及弃用窗口内完成代码升级。

The preceding lifecycle example used boolean values for the features, but features may also use enums. For example, `features.field_presence` has values `LEGACY_REQUIRED`, `EXPLICIT`, and `IMPLICIT.`

​	上述生命周期示例使用了布尔值特性，但特性也可以使用枚举值。例如，`features.field_presence` 具有 `LEGACY_REQUIRED`、`EXPLICIT` 和 `IMPLICIT` 三种值。

## 向 Protobuf Editions迁移 Migrating to Protobuf Editions

Editions won’t break existing binaries and don’t change a message’s binary, text, or JSON serialization format. The first edition is as minimally disruptive as possible. The first edition establishes the baseline and combines proto2 and proto3 definitions into a new single definition format.

​	editions不会破坏现有的二进制文件，也不会改变消息的二进制、文本或 JSON 序列化格式。第一个edition尽可能减少破坏性影响，建立基线并将 proto2 和 proto3 定义合并为一个新的定义格式。

When the subsequent editions are released, default behaviors for features may change. You can have Prototiller do a no-op transformation of your .proto file or you can choose to accept some or all of the new behaviors. Editions are planned to be released roughly once a year.

​	当后续editions发布时，特性的默认行为可能会发生变化。您可以让 **Prototiller** 工具对您的 `.proto` 文件进行无操作转换，或者选择接受部分或全部新行为。计划每年发布一个Editions。

### 从 Proto2 迁移到editions - Proto2 to Editions

This section shows a proto2 file, and what it might look like after running the Prototiller tool to change the definition files to use Protobuf Editions syntax.

​	以下展示了一个 proto2 文件及其在运行 **Prototiller** 工具后使用 Protobuf Editions语法的样子。

#### Proto2 syntax

```proto
// proto2 file
syntax = "proto2";

package com.example;

message Player {
  // in proto2, optional fields have explicit presence
  // 在 proto2 中，可选字段具有显式存在性
  optional string name = 1;
  // proto2 still supports the problematic "required" field rule
  // proto2 仍支持有问题的 "required" 字段规则
  required int32 id = 2;
  // in proto2 this is not packed by default
  // 在 proto2 中，默认不打包
  repeated int32 scores = 3;

  enum Handed {
    HANDED_UNSPECIFIED = 0;
    HANDED_LEFT = 1;
    HANDED_RIGHT = 2;
    HANDED_AMBIDEXTROUS = 3;
  }

  // in proto2 enums are closed
  // 在 proto2 中，枚举是封闭的
  optional Handed handed = 4;

  reserved "gender";
}
```

#### editions语法 Editions syntax

```proto
// Edition version of proto2 file
// proto2 文件的edition版本语法
edition = "2023";

package com.example;

message Player {
  // fields have explicit presence, so no explicit setting needed
  // 字段具有显式存在性，因此无需显式设置
  string name = 1;
  // to match the proto2 behavior, LEGACY_REQUIRED is set at the field level
  // 为匹配 proto2 行为，在字段级别设置 LEGACY_REQUIRED
  int32 id = 2 [features.field_presence = LEGACY_REQUIRED];
  // to match the proto2 behavior, EXPANDED is set at the field level
  // 为匹配 proto2 行为，在字段级别设置 EXPANDED
  repeated int32 scores = 3 [features.repeated_field_encoding = EXPANDED];

  enum Handed {
    // this overrides the default edition 2023 behavior, which is OPEN
    // 覆盖edition 2023 的默认行为（OPEN）
    option features.enum_type = CLOSED;
    HANDED_UNSPECIFIED = 0;
    HANDED_LEFT = 1;
    HANDED_RIGHT = 2;
    HANDED_AMBIDEXTROUS = 3;
  }

  Handed handed = 4;

  reserved gender;
}
```

### 从 Proto3 迁移到editions - Proto3 to Editions

This section shows a proto3 file, and what it might look like after running the Prototiller tool to change the definition files to use Protobuf Editions syntax.

​	以下展示了一个 proto3 文件及其在运行 **Prototiller** 工具后使用 Protobuf Editions语法的样子。

#### Proto3 syntax

```proto
// proto3 file
syntax = "proto3";

package com.example;

message Player {
  // in proto3, optional fields have explicit presence
  // 在 proto3 中，可选字段具有显式存在性
  optional string name = 1;
  // in proto3 no specified field rule defaults to implicit presence
  // 在 proto3 中，未指定字段规则默认为隐式存在性
  int32 id = 2;
  // in proto3 this is packed by default
  // 在 proto3 中，默认是打包的
  repeated int32 scores = 3;

  enum Handed {
    HANDED_UNSPECIFIED = 0;
    HANDED_LEFT = 1;
    HANDED_RIGHT = 2;
    HANDED_AMBIDEXTROUS = 3;
  }

  // in proto3 enums are open
  // 在 proto3 中，枚举是开放的
  optional Handed handed = 4;

  reserved "gender";
}
```

#### editions语法 Editions syntax

```proto
// Editions version of proto3 file
// proto3 文件的editions版本语法
edition = "2023";

package com.example;

message Player {
  // fields have explicit presence, so no explicit setting needed
  // 字段具有显式存在性，因此无需显式设置
  string name = 1;
  // to match the proto3 behavior, IMPLICIT is set at the field level
  // 为匹配 proto3 行为，在字段级别设置 IMPLICIT
  int32 id = 2 [features.field_presence = IMPLICIT];
  // PACKED is the default state, and is provided just for illustration
  // PACKED 是默认状态，此处仅作说明
  repeated int32 scores = 3 [features.repeated_field_encoding = PACKED];

  enum Handed {
    HANDED_UNSPECIFIED = 0;
    HANDED_LEFT = 1;
    HANDED_RIGHT = 2;
    HANDED_AMBIDEXTROUS = 3;
  }

  Handed handed = 4;

  reserved gender;
}
```



### 词法作用域 Lexical Scoping

Editions syntax supports lexical scoping, with a per-feature list of allowed targets. For example, in the first edition, features can be specified at only the file level or the lowest level of granularity. The implementation of lexical scoping enables you to set the default behavior for a feature across an entire file, and then override that behavior at the message, field, enum, enum value, oneof, service, or method level. Settings made at a higher level (file, message) apply when no setting is made within the same scope (field, enum value). Any features not explicitly set conform to the behavior defined in the edition version used for the .proto file.

​	editions语法支持词法作用域，并提供每个特性允许的目标列表。例如，在第一个edition中，特性可以仅在文件级别或最低粒度级别指定。词法作用域的实现使您可以为整个文件中的某个特性设置默认行为，然后在消息、字段、枚举、枚举值、oneof、服务或方法级别覆盖该行为。在较高级别（文件、消息）设置的内容适用于同一作用域内未设置的内容（字段、枚举值）。未显式设置的特性将遵循 `.proto` 文件使用的edition定义的行为。

The following code sample shows some features being set at the file, field, and enum level. The settings are in the highlighted lines:

​	以下代码示例显示了一些在文件、字段和枚举级别设置的特性。设置部分在高亮行中：

```proto
edition = "2023";

option features.enum_type = CLOSED;

message Person {
  string name = 1;
  int32 id = 2 [features.field_presence = IMPLICIT];

  enum Pay_Type {
    PAY_TYPE_UNSPECIFIED = 1;
    PAY_TYPE_SALARY = 2;
    PAY_TYPE_HOURLY = 3;
  }

  enum Employment {
    option features.enum_type = OPEN;
    EMPLOYMENT_UNSPECIFIED = 0;
    EMPLOYMENT_FULLTIME = 1;
    EMPLOYMENT_PARTTIME = 2;
  }
  Employment employment = 4;
}
```

In the preceding example, the presence feature is set to `IMPLICIT`; it would default to `EXPLICIT` if it wasn’t set. The `Pay_Type` `enum` will be `CLOSED`, as it applies the file-level setting. The `Employment` `enum`, though, will be `OPEN`, as it is set within the enum.

​	在上述示例中，存在性特性设置为 `IMPLICIT`；如果未设置，则默认值为 `EXPLICIT`。`Pay_Type` 枚举将是 `CLOSED`，因为它应用了文件级别的设置。而 `Employment` 枚举将是 `OPEN`，因为它在枚举内被显式设置。

### Prototiller

Currently, all conversions to editions format are handled by the Protobuf team.

​	目前，所有向editions格式的转换均由 Protobuf 团队处理。

When this shifts to a self-serve model, we will provide both a migration guide and migration tooling to ease the migration to and between editions. The tool, called Prototiller, will enable you to:

​	当转换转变为自助模型时，我们将提供迁移指南和工具以简化editions间的迁移。工具名为 **Prototiller**，它将支持：

- convert proto2 and proto3 definition files to the new editions syntax, at scale
  - 大规模将 proto2 和 proto3 定义文件转换为新editions语法

- migrate files from one edition to another
  - 在edition之间迁移文件

- manipulate proto files in other ways
  - 以其他方式操作 proto 文件


### 向后兼容性 Backward Compatibility

We are building Protobuf Editions to be as minimally disruptive as possible. For example, you can import proto2 and proto3 definitions into editions-based definition files, and vice versa:

​	我们构建 Protobuf Editions以尽可能减少破坏性影响。例如，您可以将 proto2 和 proto3 定义导入基于editions的定义文件，反之亦然：

```proto
// file myproject/foo.proto
syntax = "proto2";

enum Employment {
  EMPLOYMENT_UNSPECIFIED = 0;
  EMPLOYMENT_FULLTIME = 1;
  EMPLOYMENT_PARTTIME = 2;
}
// file myproject/edition.proto
edition = "2023";

import "myproject/foo.proto";
```

While the generated code changes when you move from proto2 or proto3 to editions, the wire format does not. You’ll still be able to access proto2 and proto3 data files or file streams using your editions-syntax proto definitions.

​	从 proto2 或 proto3 迁移到editions后，生成的代码会发生变化，但线格式不会。因此，您仍然可以使用editions语法的 proto 定义访问 proto2 和 proto3 数据文件或文件流。

### 语法变化 Grammar Changes

There are some grammar changes in editions compared to proto2 and proto3.

​	与 proto2 和 proto3 相比，editions中有一些语法变化。

**Syntax description.** Instead of the `syntax` element, you use an `edition` element:

​	**语法描述。** 不再使用 `syntax` 元素，而使用 `edition` 元素：



```proto
syntax = "proto2";
syntax = "proto3";
edition = "2028";
```

**Reserved names.** You no longer put field names and enum value names in quotation marks when reserving them:

​	**保留名称。** 保留字段名和枚举值名时，不再使用引号：

```proto
reserved foo, bar;
```

**Group syntax.** Group syntax, available in proto2, is removed in editions. The special wire-format that groups used is still available by using `DELIMITED` message encoding.

​	**组语法。** proto2 中可用的组语法在editions中被移除。组使用的特殊线格式仍然可通过 `DELIMITED` 消息编码实现。

**Required label.** The `required` label, available only in proto2, is unavailable in editions. The underlying functionality is still available (but [discouraged](https://protobuf.dev/programming-guides/required-considered-harmful)) by using `features.field_presence=LEGACY_REQUIRED`.

​	**必填标签。** proto2 中可用的 `required` 标签在editions中不可用。底层功能仍然可用（但已[不推荐](https://protobuf.dev/programming-guides/required-considered-harmful)），通过使用 `features.field_presence=LEGACY_REQUIRED` 实现。
