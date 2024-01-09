+++
title = "Protobuf 版本概述"
weight = 42
description = "Protobuf 版本功能的概述"
type = "docs"

+++

> 原文网址： https://protobuf.dev/editions/overview/

## Protobuf Editions Overview - Protobuf 版本概述

An overview of the Protobuf Editions functionality.

​	Protobuf 版本功能的概述。



Protobuf Editions replace the proto2 and proto3 designations that we have used for Protocol Buffers. Instead of adding `syntax = "proto2"` or `syntax = "proto3"` at the top of proto definition files, you use an edition number, such as `edition = "2024"`, to specify the default behaviors your file will have. Editions enable the language to evolve incrementally over time.

​	Protobuf 版本取代了我们用于 Protocol Buffers 的 proto2 和 proto3 名称。您无需在 proto 定义文件的顶部添加 `syntax = "proto2"` 或 `syntax = "proto3"` ，而是使用版本号（例如 `edition = "2024"` ）来指定文件的默认行为。版本使语言能够随着时间的推移逐渐演变。

Instead of the hardcoded behaviors that older versions have had, editions represent a collection of [features](https://protobuf.dev/editions/features) with a default value (behavior) per feature. Features are options on a file, message, field, enum, and so on, that specify the behavior of protoc, the code generators, and protobuf runtimes. You can explicitly override a behavior at those different levels (file, message, field, …) when your needs don’t match the default behavior for the edition you’ve selected. You can also override your overrides. The [section later in this topic on lexical scoping](https://protobuf.dev/editions/overview/#scoping) goes into more detail on that.
版本不是具有硬编码行为的旧版本，而是表示具有一组具有默认值（行为）的特性。特性是文件、消息、字段、枚举等上的选项，用于指定 protoc、代码生成器和 protobuf 运行时的行为。当您的需求与您选择的版本默认行为不匹配时，您可以在这些不同级别（文件、消息、字段等）显式覆盖行为。您还可以覆盖您的覆盖。本主题后面有关词法作用域的部分对此进行了更详细的介绍。

## 特性的生命周期 Lifecycle of a Feature 

Editions provide the fundamental increments for the lifecycle of a feature. Features have an expected lifecycle: introducing it, changing its default behavior, deprecating it, and then removing it. For example:

​	版本为特性的生命周期提供了基本增量。特性具有预期的生命周期：引入它、更改其默认行为、弃用它，然后将其删除。例如：

1. Edition 2031 creates `feature.amazing_new_feature` with a default value of `false`. This value maintains the same behavior as all earlier editions. That is, it defaults to no impact.

2. 2031 版创建 `feature.amazing_new_feature` ，其默认值为 `false` 。此值与所有早期版本保持相同行为。也就是说，它默认为无影响。

3. Developers update their .proto files to `edition = "2031"`.

4. 开发人员将他们的 .proto 文件更新为 `edition = "2031"` 。

5. A later edition, such as edition 2033, switches the default of `feature.amazing_new_feature` from `false` to `true`. This is the desired behavior for all protos, and the reason that the protobuf team created the feature.

6. 更高版本（例如 2033 版）将 `feature.amazing_new_feature` 的默认值从 `false` 切换为 `true` 。这是所有 proto 的预期行为，也是 protobuf 团队创建此功能的原因。

   Using the Prototiller tool to migrate earlier versions of proto files to edition 2033 adds explicit `feature.amazing_new_feature = false` entries as needed to continue to retain the previous behavior. Developers remove these newly-added settings when they want the new behavior to apply to their .proto files.

   使用 Prototiller 工具将早期版本的 proto 文件迁移到 2033 版会根据需要添加显式 `feature.amazing_new_feature = false` 条目，以继续保留之前的行为。当开发人员希望将新行为应用到他们的 .proto 文件时，他们会移除这些新添加的设置。

7. At some point, `feature.amazing_new_feature` is marked deprecated in an edition and removed in a later one.

8. 在某个时间点， `feature.amazing_new_feature` 在某个版本中被标记为已弃用，并在更高版本中被移除。

   When a feature is removed, the code generators for that behavior and the runtime libraries that support it may also be removed. The timelines will be generous, though. Following the example in the earlier steps of the lifecycle, the deprecation might happen in edition 2034 but not be removed until edition 2036, roughly two years later. Removing a feature will always initiate a major version bump.

   移除某个功能时，该行为的代码生成器和支持它的运行时库也可能会被移除。不过，时间表会很宽裕。按照生命周期早期步骤中的示例，弃用可能会在 2034 版中发生，但直到大约两年后的 2036 版中才会被移除。移除某个功能始终会启动一次重大版本升级。

Because of this lifecycle, any `.proto` file that does not use deprecated features has a no-op upgrade from one edition to the next. You will have the full window of the Google migration plus the deprecation window to upgrade your code.

​	由于此生命周期，任何不使用已弃用功能的 `.proto` 文件都可以从一个版本无操作升级到下一个版本。您将拥有完整的 Google 迁移窗口以及弃用窗口来升级您的代码。

The preceding lifecycle example used boolean values for the features, but features may also use enums. For example, `features.field_presence` has values `LEGACY_REQUIRED`, `EXPLICIT`, and `IMPLICIT.`

​	前面的生命周期示例对这些功能使用了布尔值，但这些功能也可以使用枚举。例如， `features.field_presence` 具有值 `LEGACY_REQUIRED` 、 `EXPLICIT` 和 `IMPLICIT.`

## 迁移到 Protobuf 版本 Migrating to Protobuf Editions 

Editions won’t break existing binaries and don’t change a message’s binary, text, or JSON serialization format. The first edition is as minimally disruptive as possible. The first edition establishes the baseline and combines proto2 and proto3 definitions into a new single definition format.

​	版本不会破坏现有二进制文件，也不会更改消息的二进制、文本或 JSON 序列化格式。第一个版本尽可能地减少破坏。第一个版本建立基线，并将 proto2 和 proto3 定义组合成新的单一定义格式。

When the subsequent editions are released, default behaviors for features may change. You can have Prototiller do a no-op transformation of your .proto file or you can choose to accept some or all of the new behaviors. Editions are planned to be released roughly once a year.

​	发布后续版本时，功能的默认行为可能会发生变化。您可以让 Prototiller 对您的 .proto 文件执行无操作转换，也可以选择接受部分或全部新行为。计划每年发布一次版本。

### Proto2 到版本 Proto2 to Editions 

This section shows a proto2 file, and what it might look like after running the Prototiller tool to change the definition files to use Protobuf Editions syntax.

​	本部分显示了一个 proto2 文件，以及在运行 Prototiller 工具将定义文件更改为使用 Protobuf 版本语法后它的样子。

#### Proto2 语法 Proto2 syntax 

```proto
// proto2 file
syntax = "proto2";

message Player {
  // in proto2, optional fields have explicit presence
  optional string name = 1;
  // proto2 still supports the problematic "required" field rule
  required int32 id = 2;
  // in proto2 this is not packed by default
  repeated int32 scores = 3;

  enum Handed {
    HANDED_UNSPECIFIED = 0;
    HANDED_LEFT = 1;
    HANDED_RIGHT = 2;
    HANDED_AMBIDEXTROUS = 3;
  }

  // in proto2 enums are closed
  optional Handed handed = 4;
}
```

#### Editions 语法 Editions syntax 

```proto
// Edition version of proto2 file
edition = "2023";

message Player {
  // fields have explicit presence, so no explicit setting needed
  string name = 1;
  // to match the proto2 behavior, LEGACY_REQUIRED is set at the field level
  int32 id = 2 [features.field_presence = LEGACY_REQUIRED];
  // to match the proto2 behavior, EXPANDED is set at the field level
  repeated int32 scores = 3 [features.repeated_field_encoding = EXPANDED];

  enum Handed {
    // this overrides the default edition 2023 behavior, which is OPEN
    option features.enum_type = CLOSED;
    HANDED_UNSPECIFIED = 0;
    HANDED_LEFT = 1;
    HANDED_RIGHT = 2;
    HANDED_AMBIDEXTROUS = 3;
  }

  Handed handed = 4;
}
```

### Proto3 到 Editions Proto3 to Editions 

This section shows a proto3 file, and what it might look like after running the Prototiller tool to change the definition files to use Protobuf Editions syntax.

​	本部分展示了一个 proto3 文件，以及在运行 Prototiller 工具将定义文件更改为使用 Protobuf Editions 语法后它的可能样子。

#### Proto3 语法 Proto3 syntax 

```proto
// proto3 file
syntax = "proto3";

message Player {
  // in proto3, optional fields have explicit presence
  optional string name = 1;
  // in proto3 no specified field rule defaults to implicit presence
  int32 id = 2;
  // in proto3 this is packed by default
  repeated int32 scores = 3;

  enum Handed {
    HANDED_UNSPECIFIED = 0;
    HANDED_LEFT = 1;
    HANDED_RIGHT = 2;
    HANDED_AMBIDEXTROUS = 3;
  }

  // in proto3 enums are open
  optional Handed handed = 4;
}
```

#### Editions 语法 Editions syntax 

```proto
// Editions version of proto3 file
edition = "2023";

message Player {
  // fields have explicit presence, so no explicit setting needed
  string name = 1;
  // to match the proto3 behavior, IMPLICIT is set at the field level
  int32 id = 2 [features.field_presence = IMPLICIT];
  // PACKED is the default state, and is provided just for illustration
  repeated int32 scores = 3 [features.repeated_field_encoding = PACKED];

  enum Handed {
    HANDED_UNSPECIFIED = 0;
    HANDED_LEFT = 1;
    HANDED_RIGHT = 2;
    HANDED_AMBIDEXTROUS = 3;
  }

  Handed handed = 4;
}
```



### 词法作用域 Lexical Scoping 

Editions syntax supports lexical scoping, with a per-feature list of allowed targets. For example, in the first edition, features can be specified at only the file level or the lowest level of granularity. The implementation of lexical scoping enables you to set the default behavior for a feature across an entire file, and then override that behavior at the message, field, enum, enum value, oneof, service, or method level. Settings made at a higher level (file, message) apply when no setting is made within the same scope (field, enum value). Any features not explicitly set conform to the behavior defined in the edition version used for the .proto file.

​	Editions 语法支持词法作用域，其中包含允许目标的按功能列出的列表。例如，在第一个版本中，只能在文件级别或最低粒度级别指定功能。词法作用域的实现使您能够为整个文件设置功能的默认行为，然后在消息、字段、枚举、枚举值、oneof、服务或方法级别覆盖该行为。当在同一作用域（字段、枚举值）内未进行任何设置时，将在较高级别（文件、消息）进行的设置适用。任何未明确设置的功能都符合用于 .proto 文件的版本中定义的行为。

The following code sample shows some features being set at the file, message, and enum level. The settings are in the highlighted lines:

​	以下代码示例显示了一些在文件、消息和枚举级别设置的功能。设置位于高亮显示的行中：

```proto
edition = "2023";

option features.enum_type = CLOSED;

message Person {
  string name = 1;
  int32 id = 2 [features.presence = IMPLICIT];

  enum Pay_Type {
    PAY_TYPE_UNSPECIFIED = 1,
    PAY_TYPE_SALARY = 2,
    PAY_TYPE_HOURLY = 3
  }

  enum Employment {
    option features.enum_type = OPEN;
    EMPLOYMENT_UNSPECIFIED = 0,
    EMPLOYMENT_FULLTIME = 1,
    EMPLOYMENT_PARTTIME = 2,
  }
  Employment employment = 4;
}
```

In the preceding example, the presence feature is set to `IMPLICIT`; it would default to `EXPLICIT` if it wasn’t set. The `Pay_Type` `enum` will be `CLOSED`, as it applies the file-level setting. The `Employment` `enum`, though, will be `OPEN`, as it is set within the enum.

​	在前面的示例中，presence 功能设置为 `IMPLICIT` ；如果未设置，则默认为 `EXPLICIT` 。 `Pay_Type` `enum` 将为 `CLOSED` ，因为它应用了文件级设置。 `Employment` `enum` 将为 `OPEN` ，因为它在枚举中设置。

### Prototiller

We provide both a migration guide and migration tooling that ease the migration to and between editions. The tool, called Prototiller, will enable you to:

​	我们提供了迁移指南和迁移工具，可以轻松地进行版本之间的迁移。该工具名为 Prototiller，它使您能够：

- convert proto2 and proto3 definition files to the new editions syntax, at scale
- 大规模地将 proto2 和 proto3 定义文件转换为新版本的语法
- migrate files from one edition to another
- 将文件从一个版本迁移到另一个版本
- manipulate proto files in other ways
- 以其他方式操作 proto 文件

### 向后兼容性 Backward Compatibility 

We are building Protobuf Editions to be as minimally disruptive as possible. For example, you can import proto2 and proto3 definitions into editions-based definition files, and vice versa:

​	我们正在构建 Protobuf 版本，以尽可能地减少中断。例如，您可以将 proto2 和 proto3 定义导入基于版本的定义文件，反之亦然：

```proto
// file myproject/foo.proto
syntax = "proto2";

enum Employment {
  EMPLOYMENT_UNSPECIFIED = 0,
  EMPLOYMENT_FULLTIME = 1,
  EMPLOYMENT_PARTTIME = 2,
}
// file myproject/edition.proto
edition = "2023";

import "myproject/foo.proto";
```

While the generated code changes when you move from proto2 or proto3 to editions, the wire format does not. You’ll still be able to access proto2 and proto3 data files or file streams using your editions-syntax proto definitions.

​	当您从 proto2 或 proto3 转到版本时，生成的代码会发生变化，但数据线格式不会发生变化。您仍然可以使用版本语法 proto 定义访问 proto2 和 proto3 数据文件或文件流。
