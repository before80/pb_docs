+++
title = "实现editions支持"
date = 2024-11-17T09:35:36+08:00
weight = 20
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

> 原文：[https://protobuf.dev/editions/implementation/](https://protobuf.dev/editions/implementation/)
>
> 收录该文档的时间：`2024-11-17T09:35:36+08:00`

# Implementing Editions Support - 实现editions支持

Instructions for implementing Editions support in runtimes and plugins.

​	为运行时和插件实现editions支持的指导。

This topic explains how to implement editions in new runtimes and generators.

​	本文解释了如何在新的运行时和代码生成器中实现editions支持。

## 概述 Overview

### Edition 2023

The first edition released is Edition 2023, which is designed to unify proto2 and proto3 syntax. The features we’ve added to cover the difference in behaviors are detailed in [Feature Settings for Editions]({{< ref "/docs/ProtobufEditions/FeatureSettingsforEditions" >}}).

​	发布的第一个版本是 **Edition 2023**，旨在统一 proto2 和 proto3 语法。我们添加了新特性以覆盖行为差异，详情请参阅 [Editions特性设置]({{< ref "/docs/ProtobufEditions/FeatureSettingsforEditions" >}})。

### 特性定义 Feature Definition

In addition to *supporting* editions and the global features we’ve defined, you may want to define your own features to leverage the infrastructure. This will allow you to define arbitrary features that can be used by your generators and runtimes to gate new behaviors. The first step is to claim an extension number for the `FeatureSet` message in descriptor.proto above 9999. You can send a pull-request to us in GitHub, and it will be included in our next release (see, for example, [#15439](https://github.com/protocolbuffers/protobuf/pull/15439)).

​	除了**支持**版本和我们定义的全局特性之外，您还可以定义自己的特性以利用基础设施。这允许您定义可以由生成器和运行时使用的任意特性，以控制新的行为。第一步是为 `descriptor.proto` 中的 `FeatureSet` 消息申请 9999 以上的扩展编号。您可以通过 GitHub 向我们提交 Pull Request（例如：[#15439](https://github.com/protocolbuffers/protobuf/pull/15439)），它将包含在下一个版本中。

Once you have your extension number, you can create your features proto (similar to [cpp_features.proto](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/cpp_features.proto)). These will typically look something like:

​	获得扩展编号后，您可以创建自己的特性 proto（类似于 [cpp_features.proto](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/cpp_features.proto)）。特性文件通常如下所示：

```proto
edition = "2023";

package foo;

import "google/protobuf/descriptor.proto";

extend google.protobuf.FeatureSet {
  MyFeatures features = <extension #>;
}

message MyFeatures {
  enum FeatureValue {
    FEATURE_VALUE_UNKNOWN = 0;
    VALUE1 = 1;
    VALUE2 = 2;
  }

  FeatureValue feature_value = 1 [
    targets = TARGET_TYPE_FIELD,
    targets = TARGET_TYPE_FILE,
    feature_support = {
      edition_introduced: EDITION_2023,
      edition_deprecated: EDITION_2024,
      deprecation_warning: "Feature will be removed in 2025",
      edition_removed: EDITION_2025,
    },
    edition_defaults = { edition: EDITION_LEGACY, value: "VALUE1" },
    edition_defaults = { edition: EDITION_2024, value: "VALUE2" }
  ];
}
```

Here we’ve defined a new enum feature `foo.feature_value` (currently only boolean and enum types are supported). In addition to defining the values it can take, you also need to specify how it can be used:

​	在这里，我们定义了一个新的枚举特性 `foo.feature_value`（目前仅支持布尔型和枚举型特性）。除了定义其可取的值之外，还需指定其使用方式：

- **Targets** - specifies the type of proto descriptors this feature can be attached to. This controls where users can explicitly specify the feature. Every type must be explicitly listed.
  - **目标（Targets）：** 指定此特性可附加的 proto 描述符类型。这决定了用户可以显式设置特性的具体位置。每种类型都必须显式列出。

- **Feature support** - specifies the lifetime of this feature relative to edition. You must specify the edition it was introduced in, and it will not be allowed before then. You can optionally deprecate or remove the feature in later editions.
  - **特性支持（Feature support）：** 指定特性相对于版本的生命周期。必须声明特性引入的版本，并且在此版本之前不允许使用。可以选择在后续版本中弃用或移除特性。

- **Edition defaults** - specifies any changes to the default value of the feature. This must cover every supported edition, but you can leave out any edition where the default didn’t change. Note that `EDITION_PROTO2` and `EDITION_PROTO3` can be specified here to provide defaults for the “legacy” editions (see [Legacy Editions](https://protobuf.dev/editions/implementation/#legacy_editions)).
  - **Edition默认值（Edition defaults）：** 指定特性的默认值变化。必须覆盖每个支持的版本，但可以省略未发生默认值变化的版本。`EDITION_PROTO2` 和 `EDITION_PROTO3` 可在此处指定，为“旧版”提供默认值（参见 [旧版支持](https://protobuf.dev/editions/implementation/#legacy_editions)）。


#### 什么是特性？ What is a Feature?

Features are designed to provide a mechanism for ratcheting down bad behavior over time, on edition boundaries. While the timeline for actually removing a feature may be years (or decades) in the future, the desired goal of any feature should be eventual removal. When a bad behavior is identified, you can introduce a new feature that guards the fix. In the next edition (or possibly later), you would flip the default value while still allowing users to retain their old behavior when upgrading. At some point in the future you would mark the feature deprecated, which would trigger a custom warning to any users overriding it. In a later edition, you would then mark it removed, preventing users from overriding it anymore (but the default value would still apply). Until support for that last edition is dropped in a breaking release, the feature will remain usable for protos stuck on older editions, giving them time to migrate.

​	特性旨在提供一种机制，用于在版本边界上逐步淘汰不良行为。尽管移除某一特性的时间线可能需要数年甚至数十年，但任何特性的最终目标都应是**移除**。当发现某一不良行为时，可以引入一个新特性来控制修复。在下一个版本（或更晚）中，可以切换该特性的默认值，同时允许用户在升级时保留旧行为。在将来的某个版本中，特性会被标记为弃用，从而对任何覆盖此特性的用户发出警告。在更晚的版本中，特性会被标记为移除，不再允许用户覆盖它（但默认值仍然适用）。直到破坏性版本移除最后一个支持版本之前，特性仍可用于停留在旧版上的 proto，给予迁移时间。

Flags that control optional behaviors you have no intention of removing are better implemented as [custom options](https://protobuf.dev/programming-guides/proto2/#customoptions). This is related to the reason we’ve restricted features to be either boolean or enum types. Any behavior controlled by a (relatively) unbounded number of values probably isn’t a good fit for the editions framework, since it’s not realistic to eventually turn down that many different behaviors.

​	对于无意移除的可选行为控制标志，更好的实现方式是 [自定义选项](https://protobuf.dev/programming-guides/proto2/#customoptions)。这是我们限制特性为布尔型或枚举型的原因之一。由相对无限数量的值控制的行为可能并不适合版本框架，因为难以最终关闭如此多的不同行为。

One caveat to this is behaviors related to wire boundaries. Using language-specific features to control serialization or parsing behavior can be dangerous, since any other language could be on the other side. Wire-format changes should always be controlled by global features in `descriptor.proto`, which can be respected by every runtime uniformly.

​	**注意：** 与线格式边界相关的行为可能较危险。使用语言特定特性控制序列化或解析行为可能存在风险，因为另一端可能使用不同语言。线格式更改应始终由 `descriptor.proto` 中的全局特性控制，以便所有运行时一致遵循。

### 代码生成器 Generators

Generators written in C++ get a lot for free, because they use the C++ runtime. They don’t need to handle [Feature Resolution](https://protobuf.dev/editions/implementation/#feature_resolution) themselves, and if they need any feature extensions they can register them in `GetFeatureExtensions` in their CodeGenerator. They can generally use `GetResolvedSourceFeatures` for access to resolved features for a descriptor in codegen and `GetUnresolvedSourceFeatures` for access to their own unresolved features.

​	使用 C++ 编写的生成器可以自动获得许多支持功能，因为它们使用 C++ 运行时。无需自行处理 [特性解析](https://protobuf.dev/editions/implementation/#feature_resolution)，如果需要任何特性扩展，可以在 `CodeGenerator` 的 `GetFeatureExtensions` 中注册。通常可使用 `GetResolvedSourceFeatures` 获取代码生成中描述符的已解析特性，以及 `GetUnresolvedSourceFeatures` 获取未解析的特性。

Plugins written in the same language as the runtime they generate code for may need some custom bootstrapping for their feature definitions.

​	与目标运行时使用相同语言编写的插件可能需要为其特性定义进行一些自定义引导。

#### 显式支持 Explicit Support

Generators must specify exactly which editions they support. This allows you to safely add support for an edition after it’s been released, on your own schedule. Protoc will reject any editions protos sent to generators that don’t include `FEATURE_SUPPORTS_EDITIONS` in the `supported_features` field of their `CodeGeneratorResponse`. Additionally, we have `minimum_edition` and `maximum_edition` fields for specifying your precise support window. Once you’ve defined all of the code and feature changes for a new edition, you can bump `maximum_edition` to advertise this support.

​	生成器必须明确指定支持的版本。这允许您在版本发布后按照自己的时间表安全地添加支持。Protoc 将拒绝发送到不包含 `FEATURE_SUPPORTS_EDITIONS` 的 `CodeGeneratorResponse` 的生成器的版本 proto。此外，我们还提供了 `minimum_edition` 和 `maximum_edition` 字段，用于指定支持的具体版本范围。一旦为新版本定义了所有代码和特性更改，便可提升 `maximum_edition` 来表明支持。

#### 代码生成测试 Codegen Tests

We have a set of codegen tests that can be used to lock down that Edition 2023 produces no unexpected functional changes. These have been very useful in languages like C++ and Java, where a significant amount of the functionality is in gencode. On the other hand, in languages like Python, where the gencode is basically just a collection of serialized descriptors, these are not quite as useful.

​	我们提供了一组代码生成测试，用于锁定 **Edition 2023** 不会产生意外的功能变化。这些测试在 C++ 和 Java 等语言中非常有用，因为大量功能在生成代码中实现。而在 Python 等语言中，由于生成代码主要是序列化描述符的集合，这些测试的作用较小。

This infrastructure is not reusable yet, but is planned to be in a future release. At that point you will be able to use them to verify that migrating to editions doesn’t have any unexpected codegen changes.

​	此测试基础设施尚不可重用，但计划在未来版本中实现。届时，您可以使用这些测试验证迁移到版本是否导致意外的代码生成更改。

### 运行时 Runtimes

Runtimes without reflection or dynamic messages should not need to do anything to implement Editions. All of that logic should be handled by the code generator.

​	对于不支持反射或动态消息的运行时，无需额外操作来实现版本支持，所有逻辑均由代码生成器处理。

Languages *with* reflection but *without* dynamic messages need resolved features, but may optionally choose to handle it in their generator only. This can be done by passing both resolved and unresolved feature sets to the runtime during codegen. This avoids re-implementing [Feature Resolution](https://protobuf.dev/editions/implementation/#feature_resolution) in the runtime with the main downside being efficiency, since it will create a unique feature set for every descriptor.

​	支持反射但不支持动态消息的语言可能需要解析特性，但也可以选择仅在生成器中处理。这可以通过在代码生成期间向运行时传递已解析和未解析的特性集来实现，从而避免在运行时重新实现 [特性解析](https://protobuf.dev/editions/implementation/#feature_resolution)。这种方式的主要缺点是效率较低，因为它会为每个描述符创建唯一的特性集。

Languages with dynamic messages must fully implement Editions, because they need to be able to build descriptors at runtime.

​	支持动态消息的语言必须完全实现版本支持，因为它们需要在运行时构建描述符。

#### 语法反射 Syntax Reflection

The first step in implementing Editions in a runtime with reflection is to remove all direct checks of the `syntax` keyword. All of these should be moved to finer-grained feature helpers, which can continue to use `syntax` if necessary.

​	在支持反射的运行时中实现版本的第一步是移除对 `syntax` 关键字的所有直接检查。所有这些检查都应迁移到更细粒度的特性助手中，如果有必要，这些助手仍然可以使用 `syntax`。

The following feature helpers should be implemented on descriptors, with language-appropriate naming:

​	以下特性助手应在描述符上实现，并根据语言使用适当的命名：

- `FieldDescriptor::has_presence` \- Whether or not a field has explicit presence `FieldDescriptor::has_presence` - 字段是否具有显式的存在性
  - Repeated fields *never* have presence
    - 重复字段（repeated fields）*从不*具有存在性
  - Message, extension, and oneof fields *always* have explicit presence
    - 消息、扩展和 oneof 字段*始终*具有显式存在性
  - Everything else has presence iff `field_presence` is not `IMPLICIT`
    - 其他字段仅在 `field_presence` 不为 `IMPLICIT` 时具有存在性

- `FieldDescriptor::is_required` - Whether or not a field is required
  - `FieldDescriptor::is_required` - 字段是否为必填字段

- `FieldDescriptor::requires_utf8_validation` - Whether or not a field should be checked for utf8 validity
  - `FieldDescriptor::requires_utf8_validation` - 字段是否应检查 UTF-8 有效性

- `FieldDescriptor::is_packed` - Whether or not a repeated field has packed encoding
  - `FieldDescriptor::is_packed` - 重复字段是否具有打包编码

- `FieldDescriptor::is_delimited` - Whether or not a message field has delimited encoding
  - `FieldDescriptor::is_delimited` - 消息字段是否具有分隔编码

- `EnumDescriptor::is_closed` - Whether or not a field is closed
  - `EnumDescriptor::is_closed` - 字段是否为封闭枚举


**Note:** In most languages, the message encoding feature is still currently signaled by `TYPE_GROUP` and required fields still have `LABEL_REQUIRED` set. This is not ideal, and was done to make downstream migrations easier. Eventually, these should be migrated to the appropriate helpers and `TYPE_MESSAGE/LABEL_OPTIONAL`.

​	**注意：** 在大多数语言中，消息编码特性仍通过 `TYPE_GROUP` 信号表示，必填字段仍设置为 `LABEL_REQUIRED`。虽然这不是理想的做法，但为了便于下游迁移，当前保留了这些信号。最终，这些信号应迁移到适当的助手中，例如 `TYPE_MESSAGE/LABEL_OPTIONAL`。

Downstream users should migrate to these new helpers instead of using syntax directly. The following class of existing descriptor APIs should ideally be deprecated and eventually removed, since they leak syntax information:

​	下游用户应迁移到这些新的助手，而不是直接使用 `syntax`。以下现有的描述符 API 类别理想情况下应被弃用并最终移除，因为它们泄露了语法信息：

- `FileDescriptor` syntax  `FileDescriptor` 语法

- Proto3 optional APIs  Proto3 可选 API
  - `FieldDescriptor::has_optional_keyword`
  - `OneofDescriptor::is_synthetic`
  - `Descriptor::*real_oneof*` - should be renamed to just “oneof” and the existing “oneof” helpers should be removed, since they leak information about synthetic oneofs (which don’t exist in editions).
    - `Descriptor::*real_oneof*` - 应重命名为 “oneof”，并移除现有的 “oneof” 助手，因为它们泄露了合成 oneof 的信息（合成 oneof 在版本中并不存在）。
- Group type 组类型
  - The `TYPE_GROUP` enum value should be removed, replaced with the `is_delimited` helper.
    - 应移除 `TYPE_GROUP` 枚举值，改用 `is_delimited` 助手。
- Required label 必填标签
  - The `LABEL_REQUIRED` enum value should be removed, replaced with the `is_required` helper.
    - 应移除 `LABEL_REQUIRED` 枚举值，改用 `is_required` 助手。

There are many classes of user code where these checks exist but *aren’t* hostile to editions. For example, code that needs to handle proto3 `optional` specially because of its synthetic oneof implementation won’t be hostile to editions as long as the polarity is something like `syntax == "proto3"` (rather than checking `syntax != "proto2"`).

​	在许多用户代码中，这些检查存在但并*不*与版本冲突。例如，需要特殊处理 proto3 `optional` 的代码，由于其实现基于合成 oneof，只要极性条件类似于 `syntax == "proto3"`（而不是检查 `syntax != "proto2"`），就不会与版本冲突。

If it’s not possible to remove these APIs entirely, they should be deprecated and discouraged.

​	如果完全移除这些 API 不可行，它们应被弃用并标注为不推荐使用。

#### 特性可见性 Feature Visibility

As discussed in [editions-feature-visibility](https://github.com/protocolbuffers/protobuf/blob/main/docs/design/editions/editions-feature-visibility.md), feature protos should remain an internal detail of any Protobuf implementation. The *behaviors* they control should be exposed via descriptor methods, but the protos themselves should not. Notably, this means that any options that are exposed to the users need to have their `features` fields stripped out.

​	正如 [特性可见性](https://github.com/protocolbuffers/protobuf/blob/main/docs/design/editions/editions-feature-visibility.md) 中讨论的那样，特性 proto 应保持为任何 Protobuf 实现的内部细节。它们控制的*行为*应通过描述符方法暴露，但 proto 本身不应暴露。值得注意的是，这意味着任何暴露给用户的选项需要移除其 `features` 字段。

The one case where we permit features to leak out is when serializing descriptors. The resulting descriptor protos should be a faithful representation of the original proto files, and should contain *unresolved features* inside of the options.

​	唯一允许特性泄露的情况是序列化描述符时。生成的描述符 proto 应是原始 proto 文件的忠实表示，并应在选项中包含*未解析的特性*。

#### 旧版支持 Legacy Editions

As discussed more in [legacy-syntax-editions](https://github.com/protocolbuffers/protobuf/blob/main/docs/design/editions/legacy-syntax-editions.md), a great way to get early coverage of your editions implementation is to unify proto2, proto3, and editions. This effectively migrates proto2 and proto3 to editions under the hood, and makes all of the helpers implemented in [Syntax Reflection](https://protobuf.dev/editions/implementation/#syntax_reflection) use features exclusively (instead of branching on syntax). This can be done by inserting a *feature inference* phase into [Feature Resolution](https://protobuf.dev/editions/implementation/#feature_resolution), where various aspects of the proto file can inform what features are appropriate. These features can then be merged into the parent’s features to get the resolved feature set.

​	正如 [旧版语法支持](https://github.com/protocolbuffers/protobuf/blob/main/docs/design/editions/legacy-syntax-editions.md) 中更详细讨论的那样，为实现版本支持的早期覆盖范围，一个有效的方法是统一 proto2、proto3 和版本支持。这实际上在底层将 proto2 和 proto3 迁移到版本中，并使 [语法反射](https://protobuf.dev/editions/implementation/#syntax_reflection) 中实现的所有助手仅使用特性（而不是分支语法）。这可以通过在 [特性解析](https://protobuf.dev/editions/implementation/#feature_resolution) 中插入一个*特性推断*阶段来实现，此阶段可以根据 proto 文件的各个方面推断适当的特性。这些特性然后可以合并到父特性中以获取解析后的特性集。

While we provide reasonable defaults for proto2/proto3 already, for edition 2023 the following additional inferences are required:

​	虽然我们已经为 proto2/proto3 提供了合理的默认值，但对于 Edition 2023，以下附加推断是必需的：

- required - we infer `LEGACY_REQUIRED` presence when a field has `LABEL_REQUIRED`
  - 必填字段 - 如果字段具有 `LABEL_REQUIRED`，我们推断其存在性为 `LEGACY_REQUIRED`

- groups - we infer `DELIMITED` message encoding when a field has `TYPE_GROUP`
  - 组类型 - 如果字段具有 `TYPE_GROUP`，我们推断其消息编码为 `DELIMITED`

- packed - we infer `PACKED` encoding when the `packed` option is true
  - 打包字段 - 如果 `packed` 选项为 true，我们推断其编码为 `PACKED`

- expanded - we infer `EXPANDED` encoding when a proto3 field has `packed` explicitly set to false
  - 扩展字段 - 如果 proto3 字段显式设置了 `packed=false`，我们推断其编码为 `EXPANDED`


#### 合规性测试 Conformance Tests

Editions-specific conformance tests have been added, but need to be opted-in to. A `--maximum_edition 2023` flag can be passed to the runner to enable these. You will need to configure your testee binary to handle the following new message types:

​	已经添加了版本特定的合规性测试，但需要手动启用。可以向运行程序传递 `--maximum_edition 2023` 标志以启用这些测试。您需要配置测试二进制文件以处理以下新消息类型：

- `protobuf_test_messages.editions.proto2.TestAllTypesProto2` - Identical to the old proto2 message, but transformed to edition 2023
  - `protobuf_test_messages.editions.proto2.TestAllTypesProto2` - 与旧 proto2 消息相同，但已转换为 Edition 2023

- `protobuf_test_messages.editions.proto3.TestAllTypesProto3` - Identical to the old proto3 message, but transformed to edition 2023
  - `protobuf_test_messages.editions.proto3.TestAllTypesProto3` - 与旧 proto3 消息相同，但已转换为 Edition 2023

- `protobuf_test_messages.editions.TestAllTypesEdition2023` - Used to cover edition-2023-specific test cases
  - `protobuf_test_messages.editions.TestAllTypesEdition2023` - 用于覆盖 Edition 2023 的特定测试用例


### 特性解析 Feature Resolution

Editions use lexical scoping to define features, meaning that any non-C++ code that needs to implement editions support will need to reimplement our *feature resolution* algorithm. However, the bulk of the work is handled by protoc itself, which can be configured to output an intermediate `FeatureSetDefaults` message. This message contains a “compilation” of a set of feature definition files, laying out the default feature values in every edition.

​	版本使用词法作用域定义特性，这意味着任何需要实现版本支持的非 C++ 代码都需要重新实现我们的*特性解析*算法。然而，大部分工作由 protoc 本身处理，它可以配置为输出一个中间的 `FeatureSetDefaults` 消息。此消息包含一组特性定义文件的“编译”，列出了每个版本中的默认特性值。

For example, the feature definition above would compile to the following defaults between proto2 and edition 2025 (in text-format notation):

​	例如，上述特性定义将在 proto2 和 Edition 2025 之间编译为以下默认值（以文本格式表示）：

```fallback
defaults {
  edition: EDITION_PROTO2
  overridable_features { [foo.features] {} }
  fixed_features {
    // Global feature defaults…
    // 全局特性默认值…
    [foo.features] { feature_value: VALUE1 }
  }
}
defaults {
  edition: EDITION_PROTO3
  overridable_features { [foo.features] {} }
  fixed_features {
    // Global feature defaults…
     // 全局特性默认值…
    [foo.features] { feature_value: VALUE1 }
  }
}
defaults {
  edition: EDITION_2023
  overridable_features {
    // Global feature defaults…
    // 全局特性默认值…
    [foo.features] { feature_value: VALUE1 }
  }
}
defaults {
  edition: EDITION_2024
  overridable_features {
    // Global feature defaults…
    // 全局特性默认值…
    [foo.features] { feature_value: VALUE2 }
  }
}
defaults {
  edition: EDITION_2025
  overridable_features {
    // Global feature defaults…
    // 全局特性默认值…
  }
  fixed_features { [foo.features] { feature_value: VALUE2 } }
}
minimum_edition: EDITION_PROTO2
maximum_edition: EDITION_2025
```

Global feature defaults are left out for compactness, but they would also be present. This object contains an ordered list of every edition with a unique set of defaults (some editions may end up not being present) within the specified range. Each set of defaults is split into *overridable* and *fixed* features. The former are supported features for the edition that can be freely overridden by users. The fixed features are those which haven’t yet been introduced or have been removed, and can’t be overridden by users.

​	为简洁起见，省略了全局特性默认值，但它们同样会存在。此对象包含每个版本的有序列表，每个版本具有唯一的默认值集（某些版本可能不会出现）。每个默认值集分为*可覆盖特性*和*固定特性*。前者是支持版本的可自由覆盖的特性，后者是尚未引入或已被移除且用户无法覆盖的特性。

We provide a Bazel rule for compiling these intermediate objects:

​	我们提供了一个 Bazel 规则，用于编译这些中间对象：

```gdscript3
load("@com_google_protobuf//editions:defaults.bzl", "compile_edition_defaults")

compile_edition_defaults(
    name = "my_defaults",
    srcs = ["//some/path:lang_features_proto"],
    maximum_edition = "PROTO2",
    minimum_edition = "2024",
)
```

The output `FeatureSetDefaults` can be embedded into a raw string literal in whatever language you need to do feature resolution in. We also provide an `embed_edition_defaults` macro to do this:

​	输出的 `FeatureSetDefaults` 可以嵌入为原始字符串字面量，用于您需要进行特性解析的任何语言中。我们还提供了一个 `embed_edition_defaults` 宏来实现此功能：

```fallback
embed_edition_defaults(
    name = "embed_my_defaults",
    defaults = ":my_defaults",
    output = "my_defaults.h",
    placeholder = "DEFAULTS_DATA",
    template = "my_defaults.h.template",
)
```

Alternatively, you can invoke protoc directly (outside of Bazel) to generate this data:

​	或者，您可以直接调用 protoc（不依赖 Bazel）生成此数据：

```fallback
protoc --edition_defaults_out=defaults.binpb --edition_defaults_minimum=PROTO2 --edition_defaults_maximum=2023 <feature files...>
```

Once the defaults message is hooked up and parsed by your code, feature resolution for a file descriptor at a given edition follows a simple algorithm:

​	一旦默认值消息被连接并由您的代码解析，对特定版本的文件描述符进行特性解析遵循以下简单算法：

1. Validate that the edition is in the appropriate range [`minimum_edition`, `maximum_edition`] 验证版本是否在适当的范围内 [`minimum_edition`, `maximum_edition`]
2. Binary-search the ordered `defaults` field for the highest entry less than or equal to the edition 对有序的 `defaults` 字段进行二分搜索，找到小于或等于该版本的最高条目
3. Merge `overridable_features` into `fixed_features` from the selected defaults 将选定默认值中的 `overridable_features` 合并到 `fixed_features` 中
4. Merge any explicit features set on the descriptor (the `features` field in the file options) 合并描述符上显式设置的任何特性（文件选项中的 `features` 字段）

From there, you can recursively resolve features for all other descriptors: 从此处，您可以递归解析其他描述符的特性：

1. Initialize to the parent descriptor’s feature set 初始化为父描述符的特性集
2. Merge any explicit features set on the descriptor (the `features` field in the options) 合并描述符上显式设置的任何特性（选项中的 `features` 字段）

For determining the “parent” descriptor, you can reference our [C++ implementation](https://github.com/protocolbuffers/protobuf/blob/27.x/src/google/protobuf/descriptor.cc#L1129). This is straightforward in most cases, but extensions are a bit surprising because their parent is the enclosing scope rather than the extendee. Oneofs also need to be considered as the parent of their fields.

​	要确定“父”描述符，您可以参考我们的 [C++ 实现](https://github.com/protocolbuffers/protobuf/blob/27.x/src/google/protobuf/descriptor.cc#L1129)。大多数情况下这很简单，但扩展的父级是封闭作用域而不是扩展对象，因此需要特别注意。一元也需要被视为其字段的父级。

#### 合规性测试 Conformance Tests

In a future release, we plan to add conformance tests to verify feature resolution cross-language. Until then, our regular [conformance tests](https://protobuf.dev/editions/implementation/#conformance_tests) do give partial coverage, and our [example inheritance unit tests](https://github.com/protocolbuffers/protobuf/blob/27.x/python/google/protobuf/internal/descriptor_test.py#L1386) can be ported to provide more comprehensive coverage.

​	在未来版本中，我们计划添加跨语言验证特性解析的合规性测试。在此之前，我们的常规 [合规性测试](https://protobuf.dev/editions/implementation/#conformance_tests) 提供部分覆盖范围，而我们的 [示例继承单元测试](https://github.com/protocolbuffers/protobuf/blob/27.x/python/google/protobuf/internal/descriptor_test.py#L1386) 可移植以提供更全面的覆盖范围。

### 示例 Examples

Below are some real examples of how we implemented editions support in our runtimes and plugins.

​	以下是我们在运行时和插件中实现版本支持的一些真实示例。

#### Java

- [#14138](https://github.com/protocolbuffers/protobuf/pull/14138) - Bootstrap compiler with C++ gencode for Java features proto
  - [#14138](https://github.com/protocolbuffers/protobuf/pull/14138) - 使用 C++ 生成的 Java 特性 proto 引导编译器

- [#14377](https://github.com/protocolbuffers/protobuf/pull/14377) - Use features in Java, Kotlin, and Java Lite code generators, including codegen tests
  - [#14377](https://github.com/protocolbuffers/protobuf/pull/14377) - 在 Java、Kotlin 和 Java Lite 代码生成器中使用特性，包括代码生成测试

- [#15210](https://github.com/protocolbuffers/protobuf/pull/15210) - Use features in Java full runtimes covering Java features bootstrap, feature resolution, and legacy editions, along with unit-tests and conformance testing
  - [#15210](https://github.com/protocolbuffers/protobuf/pull/15210) - 在完整的 Java 运行时中使用特性，涵盖 Java 特性引导、特性解析和旧版本支持，以及单元测试和合规性测试

#### Pure Python

- [#14546](https://github.com/protocolbuffers/protobuf/pull/14546) - Setup codegen tests in advance

  - [#14546](https://github.com/protocolbuffers/protobuf/pull/14546) - 提前设置代码生成测试

- [#14547](https://github.com/protocolbuffers/protobuf/pull/14547) - Fully implements editions in one shot, along with unit-tests and conformance testing

  - [#14547](https://github.com/protocolbuffers/protobuf/pull/14547) - 一次性完全实现版本支持，包括单元测试和合规性测试

  

#### 𝛍pb

- [#14638](https://github.com/protocolbuffers/protobuf/pull/14638) - First pass at editions implementation covering feature resolution and legacy editions

  - [#14638](https://github.com/protocolbuffers/protobuf/pull/14638) - 首次实现版本支持，涵盖特性解析和旧版本支持

- [#14667](https://github.com/protocolbuffers/protobuf/pull/14667) - Added more complete handling of field label/type, support for upb’s code generator, and some tests

  - [#14667](https://github.com/protocolbuffers/protobuf/pull/14667) - 增加字段标签/类型的完整处理，支持 upb 的代码生成器，并添加部分测试

- [#14678](https://github.com/protocolbuffers/protobuf/pull/14678) - Hooks up upb to the Python runtime, with more unit tests and conformance tests

  - [#14678](https://github.com/protocolbuffers/protobuf/pull/14678) - 将 upb 连接到 Python 运行时，并增加更多单元测试和合规性测试

  

#### Ruby

- [#16132](https://github.com/protocolbuffers/protobuf/pull/16132) - Hook up upb/Java to all four Ruby runtimes for full editions support
  - [#16132](https://github.com/protocolbuffers/protobuf/pull/16132) - 将 upb/Java 连接到所有四个 Ruby 运行时，实现完整的版本支持
