+++
title = "迁移指南"
weight = 920
description = "对库版本所做的重大更改列表，以及如何更新代码以适应这些更改。"
aliases = "/programming-guides/migration/"
type = "docs"

+++

> 原文网址：  https://protobuf.dev/support/migration/

## Migration Guide 迁移指南

A list of the breaking changes made to versions of the libraries, and how to update your code to accommodate the changes.

​	对库版本所做的重大更改列表，以及如何更新代码以适应这些更改。



## v22.0 中的编译器更改 Compiler Changes in v22.0 

### JSON 字段名称冲突 JSON Field Name Conflicts 

Source of changes: [PR #11349](https://github.com/protocolbuffers/protobuf/pull/11349), [PR #10750](https://github.com/protocolbuffers/protobuf/pull/10750)

更改来源：PR #11349、PR #10750

We’ve made some subtle changes in how we handle field name conflicts with respect to JSON mappings. In proto3, we’ve partially loosened the restrictions and only give errors when field names produce case-sensitive JSON mappings (camel case of the original name). We now also check the `json_name` option, and give errors for case-sensitive conflicts. In proto2, we’ve tightened restrictions a bit and will give errors if two `json_name` specifications conflict. If implicit JSON mappings (camel case) have conflicts, we will give warnings in proto2.

​	我们对处理与 JSON 映射相关的字段名称冲突的方式进行了一些细微的更改。在 proto3 中，我们部分放宽了限制，仅当字段名称产生区分大小写的 JSON 映射（原始名称的驼峰式）时才给出错误。我们现在还检查 `json_name` 选项，并针对区分大小写的冲突给出错误。在 proto2 中，我们收紧了限制，如果两个 `json_name` 规范冲突，我们将给出错误。如果隐式 JSON 映射（驼峰式）发生冲突，我们将在 proto2 中给出警告。

We’ve provided a temporary message/enum option for restoring the legacy behavior. If renaming the conflicting fields isn’t an option you can take immediately, set the `deprecated_legacy_json_field_conflicts` option on the specific message/enum. This option will be removed in a future release, but gives you more time to migrate.

​	我们提供了一个临时消息/枚举选项来恢复旧行为。如果您无法立即重命名冲突字段，请在特定消息/枚举上设置 `deprecated_legacy_json_field_conflicts` 选项。此选项将在未来版本中删除，但会为您提供更多迁移时间。

## v22.0 中的 C++ API 更改 C++ API Changes in v22.0 

4.22.0 has breaking changes for C++ runtime and protoc, as [announced in August](https://protobuf.dev/news/2022-08-03#cpp-changes).

​	4.22.0 对 C++ 运行时和 protoc 做出了重大更改，如 8 月份宣布的。

### Autotools 停用 Autotools Turndown 

Source of changes: [PR #10132](https://github.com/protocolbuffers/protobuf/pull/10132)

​	更改来源：PR #10132

In v22.0, we removed all Autotools support from the protobuf compiler and the C++ runtime. If you’re using Autotools to build either of these, you must migrate to [CMake](http://cmake.org/) or [Bazel](http://bazel.build/). We have some [dedicated instructions](https://github.com/protocolbuffers/protobuf/blob/main/cmake/README.md) for setting up protobuf with CMake.

​	在 v22.0 中，我们从 protobuf 编译器和 C++ 运行时中删除了所有 Autotools 支持。如果您使用 Autotools 来构建其中任何一个，您必须迁移到 CMake 或 Bazel。我们有一些专门的说明，用于使用 CMake 设置 protobuf。

### Abseil 依赖项 Abseil Dependency 

Source of changes: [PR #10416](https://github.com/protocolbuffers/protobuf/pull/10416)

​	更改来源：PR #10416

With v22.0, we’ve taken on an explicit dependency on [Abseil](https://github.com/abseil/abseil-cpp). This allowed us to remove most of our [stubs](https://github.com/protocolbuffers/protobuf/tree/21.x/src/google/protobuf/stubs), which were branched from old internal code that later became Abseil. There are a number of subtle behavior changes, but most should be transparent to users. Some notable changes include:

​	在 v22.0 中，我们明确依赖于 Abseil。这使我们能够删除大多数存根，这些存根是从旧的内部代码中分支出来的，后来这些代码变成了 Abseil。有一些细微的行为更改，但大多数对用户来说应该是透明的。一些值得注意的更改包括：

- **string_view** - `absl::string_view` has replaced `const std::string&` in many of our APIs. This is most-commonly used for input arguments, where there should be no noticeable change for users. In a few cases (such as virtual method arguments or return types) users may need to make an explicit change to use the new signature.

- string_view - `absl::string_view` 在我们的许多 API 中替换了 `const std::string&` 。这最常用于输入参数，用户应该不会注意到任何变化。在少数情况下（例如虚拟方法参数或返回类型），用户可能需要进行显式更改以使用新签名。

- **tables** - Instead of STL sets/maps, we now use Abseil’s `flat_hash_map`, `flat_hash_set`, `btree_map`, and `btree_set`. These are more efficient and allow for [heterogeneous lookup](https://abseil.io/tips/144). This should be mostly invisible to users, but may cause some subtle behavior changes related to table ordering.

- 表格 - 现在，我们使用 Abseil 的 `flat_hash_map` 、 `flat_hash_set` 、 `btree_map` 和 `btree_set` ，而不是 STL 集/映射。这些更加高效，并允许异构查找。这对于用户来说应该基本上是不可见的，但可能会导致与表格排序相关的某些细微的行为变化。

- **logging** - Abseil’s [logging library](https://abseil.io/docs/cpp/guides/logging) is very similar to our old logging code, with just a slightly different spelling (for example, `ABSL_CHECK` instead of `GOOGLE_CHECK`). The biggest difference is that it doesn’t support exceptions, and will now always crash when `FATAL` assertions fail. (Previously we had a `PROTOBUF_USE_EXCEPTIONS` flag to switch to exceptions.) Since these only occur when serious issues are encountered, we feel unconditional crashing is a suitable response.

- 日志记录 - Abseil 的日志记录库与我们旧的日志记录代码非常相似，只是拼写略有不同（例如， `ABSL_CHECK` 而不是 `GOOGLE_CHECK` ）。最大的区别在于它不支持异常，并且现在当 `FATAL` 断言失败时，它将始终崩溃。（以前我们有一个 `PROTOBUF_USE_EXCEPTIONS` 标志来切换到异常。）由于这些仅在遇到严重问题时才会发生，我们认为无条件崩溃是一种合适的响应。

  Source of logging changes: [PR #11623](https://github.com/protocolbuffers/protobuf/pull/11623)

  日志记录更改的来源：PR #11623

- **Build dependency** - A new build dependency can always cause breakages for downstream users. We require [Abseil LTS 20230117](https://github.com/abseil/abseil-cpp/releases/tag/20230117.rc1) or later to build.

- 构建依赖项 - 新的构建依赖项总是可能导致下游用户的中断。我们需要 Abseil LTS 20230117 或更高版本才能构建。

  - For Bazel builds, Abseil will be automatically downloaded and built at a pinned LTS release when [`protobuf_deps`](https://github.com/protocolbuffers/protobuf/blob/main/protobuf_deps.bzl) is run from your `WORKSPACE`. This should be transparent, but if you depend on an older version of Abseil, you’ll need to upgrade your dependency.
  - 对于 Bazel 构建，当从 `WORKSPACE` 运行 `protobuf_deps` 时，Abseil 将在固定的 LTS 版本中自动下载并构建。这应该是透明的，但如果您依赖于旧版本的 Abseil，则需要升级您的依赖项。
  - For CMake builds, we will first look for an existing Abseil installation pulled in by the top-level CMake configuration (see [instructions](https://github.com/abseil/abseil-cpp/blob/master/CMake/README.md#traditional-cmake-set-up)). Otherwise, if `protobuf_ABSL_PROVIDER` is set to `module` (its default) we will attempt to build and link Abseil from our git [submodule](https://github.com/protocolbuffers/protobuf/tree/main/third_party). If `protobuf_ABSL_PROVIDER` is set to `package`, we will look for a pre-installed system version of Abseil. 
  - 对于 CMake 构建，我们将首先查找由顶级 CMake 配置提取的现有 Abseil 安装（请参阅说明）。否则，如果 `protobuf_ABSL_PROVIDER` 设置为 `module` （其默认值），我们将尝试从我们的 git 子模块构建并链接 Abseil。如果 `protobuf_ABSL_PROVIDER` 设置为 `package` ，我们将查找预先安装的 Abseil 系统版本。



###  GetCurrentTime 方法中的更改 Changes in GetCurrentTime Method

  On Windows, `GetCurrentTime()` is the name of a macro provided by the system. Prior to v22.x, Protobuf incorrectly removed the macro definition for `GetCurrentTime()`. That made the macro unusable for Windows developers after including `<protobuf/util/time_util.h>`. Starting with v22.x, Protobuf preserves the macro definition. This may break customer code relying on the previous behavior, such as if they use the expression [`google::protobuf::util::TimeUtil::GetCurrentTime()`](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.util.time_util.md#TimeUtil).

​	  在 Windows 上， `GetCurrentTime()` 是系统提供的宏的名称。在 v22.x 之前，Protobuf 错误地删除了 `GetCurrentTime()` 的宏定义。这使得在包含 `<protobuf/util/time_util.h>` 之后，Windows 开发人员无法使用该宏。从 v22.x 开始，Protobuf 保留了宏定义。这可能会破坏依赖于以前行为的客户代码，例如如果他们使用表达式 `google::protobuf::util::TimeUtil::GetCurrentTime()` 。

  To migrate your app to the new behavior, change your code to do one of the following:

​	  要将您的应用迁移到新行为，请更改您的代码以执行以下操作之一：

  - if the `GetCurrent` macro is defined, explicitly undefine the `GetCurrentTime` macro
  - 如果定义了 `GetCurrent` 宏，则明确取消定义 `GetCurrentTime` 宏
  - prevent the macro expansion by using `(google::protobuf::util::TimeUtil::GetCurrentTime)()` or a similar expression
  - 使用 `(google::protobuf::util::TimeUtil::GetCurrentTime)()` 或类似表达式来防止宏展开

#### 示例：取消定义宏 Example: Undefining the macro 

Use this approach if you don’t use the macro from Windows.

​	如果您不从 Windows 使用宏，请使用此方法。

Before:

​	之前：

```cpp
#include <google/protobuf/util/time_util.h>

void F() {
  auto time = google::protobuf::util::TimeUtil::GetCurrentTime();
}
```

After:

​	之后：

```cpp
#include <google/protobuf/util/time_util.h>
#ifdef GetCurrent
#undef GetCurrentTime
#endif

void F() {
  auto time = google::protobuf::util::TimeUtil::GetCurrentTime();
}
```

**示例 2：防止宏展开 Example 2: Preventing macro expansion**

Before:

​	之前：

```cpp
#include <google/protobuf/util/time_util.h>

void F() {
  auto time = google::protobuf::util::TimeUtil::GetCurrentTime();
}
```

After:

​	之后：

```cpp
#include <google/protobuf/util/time_util.h>

void F() {
  auto time = (google::protobuf::util::TimeUtil::GetCurrentTime)();
}
```

## C++20 支持 C++20 Support 

Source of changes: [PR #10796](https://github.com/protocolbuffers/protobuf/pull/10796)
更改来源：PR #10796

To support C++20, we’ve reserved the new [keywords](https://en.cppreference.com/w/cpp/keyword) in C++ generated protobuf code. As with other reserved keywords, if you use them for any fields, enums, or messages, we will add an underscore suffix to make them valid C++. For example, a `concept` field will generate a `concept_()` getter. In the scenario where you have existing protos that use these keywords, you’ll need to update the C++ code that references them to add the appropriate underscores.

​	为了支持 C++20，我们在 C++ 生成的 protobuf 代码中保留了新关键字。与其他保留关键字一样，如果您将它们用于任何字段、枚举或消息，我们将添加一个下划线后缀以使其成为有效的 C++。例如， `concept` 字段将生成 `concept_()` getter。在您有使用这些关键字的现有协议的情况下，您需要更新引用它们的 C++ 代码以添加适当的下划线。

### Final Classes

Source of changes: [PR #11604](https://github.com/protocolbuffers/protobuf/pull/11604)
更改来源：PR #11604

As part of a larger effort to harden assumptions made in the Protobuf library, we’ve marked some classes `final` that were never intended to be inherited from. There are no known use cases for inheriting from these, and doing so would likely cause problems. There is no mitigation if your code is inheriting from these classes, but if you think you have some valid reason for the inheritance you’re using, you can [open an issue](https://github.com/protocolbuffers/protobuf/issues).

​	作为强化 Protobuf 库中假设的一部分，我们标记了一些从未打算继承的类 `final` 。没有已知的继承这些类的用例，这样做可能会导致问题。如果您的代码继承自这些类，则没有缓解措施，但如果您认为您有使用继承的一些有效理由，则可以打开一个问题。

### 容器静态断言 Container Static Assertions 

Source of changes: [PR #11550](https://github.com/protocolbuffers/protobuf/pull/11550)
更改来源：PR #11550

As part of a larger effort to harden assumptions made in the Protobuf library, we’ve added static assertions to the `Map`, `RepeatedField`, and `RepeatedPtrField` containers. These ensure that you’re using these containers with only expected types, as [covered in our documentation](https://protobuf.dev/programming-guides/proto/#maps). If you hit these static assertions, you should migrate your code to use Abseil or STL containers. `std::vector` is a good drop-in replacement for repeated field containers, and `std::unordered_map` or `absl::flat_hash_map` for `Map` (the former gives similar pointer stability, while the latter is more efficient).

​	作为强化 Protobuf 库中假设的更大努力的一部分，我们已向 `Map` 、 `RepeatedField` 和 `RepeatedPtrField` 容器添加了静态断言。这些断言确保您仅将这些容器与预期类型一起使用，如我们的文档中所述。如果您遇到这些静态断言，您应迁移您的代码以使用 Abseil 或 STL 容器。 `std::vector` 是重复字段容器的良好替代品，而 `std::unordered_map` 或 `absl::flat_hash_map` 是 `Map` 的良好替代品（前者提供类似的指针稳定性，而后者则更有效）。

### 已清除元素弃用 Cleared Element Deprecation 

Source of changes: [PR #11588](https://github.com/protocolbuffers/protobuf/pull/11588), [PR #11639](https://github.com/protocolbuffers/protobuf/pull/11639)
更改来源：PR #11588、PR #11639

The `RepeatedPtrField` API around “cleared fields” has been deprecated, and will be fully removed in a later breaking release. This was originally added as an optimization for reusing elements after they’ve been cleared, but ended up not working well. If you’re using this API, you should consider migrating to arenas for better memory reuse.

​	围绕“已清除字段”的 `RepeatedPtrField` API 已弃用，并且将在以后的重大版本中完全删除。这最初被添加为在清除元素后重新使用它们的优化，但最终效果不佳。如果您正在使用此 API，您应考虑迁移到竞技场以实现更好的内存重用。

### UnsafeArena 弃用 UnsafeArena Deprecation 

Source of changes: [PR #10325](https://github.com/protocolbuffers/protobuf/pull/10325)
更改来源：PR #10325

As part of a larger effort to remove arena-unsafe APIs, we’ve hidden `RepeatedField::UnsafeArenaSwap`. This is the only one we’ve removed so far, but in later releases we will continue to remove them and provide helpers to handle efficient borrowing patterns between arenas. Within a single arena (or the stack/heap), `Swap` is just as efficient as `UnsafeArenaSwap`. The benefit is that it won’t cause invalid memory operations if you accidentally call it across different arenas.

​	作为移除竞技场不安全 API 的更大努力的一部分，我们隐藏了 `RepeatedField::UnsafeArenaSwap` 。这是我们到目前为止移除的唯一一个，但在以后的版本中，我们将继续移除它们并提供助手来处理竞技场之间的有效借用模式。在单个竞技场（或堆栈/堆）中， `Swap` 与 `UnsafeArenaSwap` 一样有效。好处是，如果您不小心在不同竞技场之间调用它，它不会导致无效的内存操作。

### Map 对升级 Map Pair Upgrades 

Source of changes: [PR #11625](https://github.com/protocolbuffers/protobuf/pull/11625)
更改来源：PR #11625

For v22.0 we’ve started cleaning up the `Map` API to make it more consistent with Abseil and STL. Notably, we’ve replaced the `MapPair` class with an alias to `std::pair`. This should be transparent for most users, but if you were using the class directly you may need to update your code.

​	对于 v22.0，我们已经开始清理 `Map` API，使其与 Abseil 和 STL 更加一致。值得注意的是，我们用 `std::pair` 的别名替换了 `MapPair` 类。对于大多数用户来说，这应该是透明的，但如果您直接使用该类，则可能需要更新您的代码。

### 新的 JSON 解析器 {:#json-parser} New JSON Parser {:#json-parser} 

Source of changes: [PR #10729](https://github.com/protocolbuffers/protobuf/pull/10729)
更改来源：PR #10729

We have rewritten the C++ JSON parser this release. It should be mostly a hidden change, but inevitably some undocumented quirks my have changed; test accordingly. Parsing documents that are not valid RFC-8219 JSON (such as those that are missing quotes or using non-standard bools) is deprecated and will be removed in a future release. The serialization order of fields is now guaranteed to match the field number order, where before it was less deterministic.

​	我们在此版本中重写了 C++ JSON 解析器。它应该主要是一个隐藏的更改，但不可避免地，一些未记录的怪癖可能已经改变；相应地进行测试。解析不是有效的 RFC-8219 JSON 的文档（例如缺少引号或使用非标准布尔值的文档）已被弃用，并将在未来版本中删除。字段的序列化顺序现在保证与字段编号顺序匹配，而之前它不太确定。

As part of this migration, all of the files under [util/internal](https://github.com/protocolbuffers/protobuf/tree/21.x/src/google/protobuf/util/internal) have been deleted. These were used in the old parser, and were never intended to be used externally.

​	作为此迁移的一部分，util/internal 下的所有文件均已删除。这些文件用于旧解析器，并且从不打算在外部使用。

### `Arena::Init`

Source of changes: [PR #10623](https://github.com/protocolbuffers/protobuf/pull/10623)
更改来源：PR #10623

The `Init` method in `Arena` was code that didn’t do anything, and has now been removed. If you were calling this method, you likely meant to call the `Arena` constructor directly with a set of `ArenaOptions`. You should either delete the call or migrate to that constructor.

​	中的 `Init` 方法是没有任何作用的代码，现在已将其删除。如果您正在调用此方法，您可能打算直接使用一组 `ArenaOptions` 调用 `Arena` 构造函数。您应该删除调用或迁移到该构造函数。

### 错误收集器迁移 ErrorCollector Migration 

Source of changes: [PR #11555](https://github.com/protocolbuffers/protobuf/pull/11555)
更改来源：PR #11555

As part of our Abseil migration, we’re moving from `const std::string&` to `absl::string_view`. For our three error collector classes, this can’t be done without breaking existing code. For v22.0, we’ve decided to release both variants, and rename the methods from `AddError` and `AddWarning` to `RecordError` and `RecordWarning`. The old signature has been marked deprecated, and will be slightly less efficient (due to string copies), but will otherwise still work. You should migrate these to the new version, as the `Add*` methods will be removed in a later breaking release.

​	作为 Abseil 迁移的一部分，我们正在从 `const std::string&` 迁移到 `absl::string_view` 。对于我们的三个错误收集器类，如果不中断现有代码，则无法完成此操作。对于 v22.0，我们决定发布这两个变体，并将方法从 `AddError` 和 `AddWarning` 重命名为 `RecordError` 和 `RecordWarning` 。旧签名已被标记为已弃用，并且效率会稍低（由于字符串副本），但除此之外仍可正常工作。您应该将这些迁移到新版本，因为 `Add*` 方法将在以后的中断版本中删除。
