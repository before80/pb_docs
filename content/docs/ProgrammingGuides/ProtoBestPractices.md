+++
title = "Proto 最佳实践"
date = 2024-11-17T09:35:36+08:00
weight = 150
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

> 原文：[https://protobuf.dev/programming-guides/dos-donts/](https://protobuf.dev/programming-guides/dos-donts/)
>
> 收录该文档的时间：`2024-11-17T09:35:36+08:00`

# Proto Best Practices - Proto 最佳实践

Shares vetted best practices for authoring Protocol Buffers.

​	提供编写 Protocol Buffers 的验证最佳实践。

Clients and servers are never updated at exactly the same time - even when you try to update them at the same time. One or the other may get rolled back. Don’t assume that you can make a breaking change and it’ll be okay because the client and server are in sync.

​	客户端和服务器永远无法在完全相同的时间更新——即使您尝试同时更新它们也是如此。某一方可能会被回滚。不要假设可以进行破坏性更改，并认为客户端和服务器同步就没问题。



## **不要**重用标签号 **Don’t** Re-use a Tag Number

Never re-use a tag number. It messes up deserialization. Even if you think no one is using the field, don’t re-use a tag number. If the change was live ever, there could be serialized versions of your proto in a log somewhere. Or there could be old code in another server that will break.

​	绝不要重用标签号。这会破坏反序列化。即使您认为没有人使用该字段，也不要重用标签号。如果更改曾经上线，可能会在日志中存在已序列化的 `.proto` 版本，或者其他旧服务器中的代码会因此崩溃。

## **务必**为已删除字段保留标签号 **Do** Reserve Tag	 Numbers for Deleted Fields

When you delete a field that’s no longer used, reserve its tag number so that no one accidentally re-uses it in the future. Just `reserved 2, 3;` is enough. No type required (lets you trim dependencies!). You can also reserve names to avoid recycling now-deleted field names: `reserved "foo", "bar";`.

​	当您删除一个不再使用的字段时，请保留其标签号，以防将来有人意外重用它。例如，`reserved 2, 3;` 就足够了，不需要指定类型（这也让您可以减少依赖！）。您还可以保留名称以避免回收已删除的字段名称，例如：`reserved "foo", "bar";`。

## **务必**为已删除的枚举值保留编号 **Do** Reserve Numbers for Deleted Enum Values

When you delete an enum value that’s no longer used, reserve its number so that no one accidentally re-uses it in the future. Just `reserved 2, 3;` is enough. You can also reserve names to avoid recycling now-deleted value names: `reserved "FOO", "BAR";`.

​	当您删除一个不再使用的枚举值时，请保留其编号，以防将来有人意外重用。例如，`reserved 2, 3;` 就足够了。您也可以保留名称以避免回收已删除的值名称，例如：`reserved "FOO", "BAR";`。

## **不要**更改字段类型 **Don’t** Change the Type of a Field

Almost never change the type of a field; it’ll mess up deserialization, same as re-using a tag number. The [protobuf docs]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2#updating" >}}) outline a small number of cases that are okay (for example, going between `int32`, `uint32`, `int64` and `bool`). However, changing a field’s message type **will break** unless the new message is a superset of the old one.

​	几乎永远不要更改字段的类型；这会像重用标签号一样破坏反序列化。[Protobuf 文档]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2#updating" >}}) 概述了少数允许的情况（例如 `int32`、`uint32`、`int64` 和 `bool` 之间的转换）。但是，更改字段的消息类型 **会导致破坏**，除非新消息是旧消息的超集。

## **不要**添加必填字段 **Don’t** Add a Required Field

Never add a required field, instead add `// required` to document the API contract. Required fields are considered harmful by so many they were removed from proto3 completely. Make all fields optional or repeated. You never know how long a message type is going to last and whether someone will be forced to fill in your required field with an empty string or zero in four years when it’s no longer logically required but the proto still says it is.

​	永远不要添加必填字段，而是使用 `// required` 注释来记录 API 契约。许多人认为必填字段有害，因此它们在 proto3 中被完全移除。将所有字段设为可选或重复字段。您无法预见一个消息类型会存在多久，或者四年后当字段逻辑上不再必填但 `.proto` 文件仍然要求它时，会迫使某人填写空字符串或零值。

For proto3 there are no `required` fields, so this advice does not apply.

​	对于 proto3，没有 `required` 字段，因此此建议不适用。

## **不要**创建包含大量字段的消息 **Don’t** Make a Message with Lots of Fields

Don’t make a message with “lots” (think: hundreds) of fields. In C++ every field adds roughly 65 bits to the in-memory object size whether it’s populated or not (8 bytes for the pointer and, if the field is declared as optional, another bit in a bitfield that keeps track of whether the field is set). When your proto grows too large, the generated code may not even compile (for example, in Java there is a hard limit on the size of a method ).

​	不要创建包含“大量”（例如数百个）字段的消息。在 C++ 中，每个字段无论是否被填充，都会为内存对象大小增加约 65 位（8 字节的指针以及用于跟踪字段是否被设置的 bitfield 中的一位）。当您的 proto 变得过大时，生成的代码甚至可能无法编译（例如，Java 对方法大小有硬性限制）。

## **务必**在枚举中包含未指定值 **Do** Include an Unspecified Value in an Enum

Enums should include a default `FOO_UNSPECIFIED` value as the first value in the declaration . When new values are added to a proto2 enum, old clients will see the field as unset and the getter will return the default value or the first-declared value if no default exists . For consistent behavior with [proto enums]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2#enum" >}}), the first declared enum value should be a default `FOO_UNSPECIFIED` value and should use tag 0. It may be tempting to declare this default as a semantically meaningful value but as a general rule, do not, to aid in the evolution of your protocol as new enum values are added over time. All enum values declared under a container message are in the same C++ namespace, so prefix the unspecified value with the enum’s name to avoid compilation errors. If you’ll never need cross-language constants, an `int32` will preserve unknown values and generates less code. Note that [proto enums]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2#enum" >}}) require the first value to be zero and can round-trip (deserialize, serialize) an unknown enum value.

​	枚举应在声明中包含一个默认的 `FOO_UNSPECIFIED` 值作为第一个值。当向 proto2 枚举添加新值时，旧客户端会将字段视为未设置，并且 Getter 将返回默认值或第一个声明的值（如果没有默认值）。为了与 [proto 枚举]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2#enum" >}}) 保持一致，第一个声明的枚举值应为默认的 `FOO_UNSPECIFIED` 值，并使用标签 0。尽管将默认值声明为一个有语义意义的值可能很诱人，但通常不建议这样做，因为这有助于在随着时间推移添加新枚举值时更容易演进协议。所有声明在容器消息中的枚举值都位于同一个 C++ 命名空间中，因此需要为未指定的值加上枚举名称的前缀以避免编译错误。如果您永远不需要跨语言的常量，可以使用 `int32` 保存未知值，同时生成的代码更少。需要注意的是 [proto 枚举]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2#enum" >}}) 要求第一个值为零，并且能够对未知枚举值进行完整的序列化和反序列化（即回路处理）。

## **不要**为枚举值使用 C/C++ 宏常量 **Don’t** Use C/C++ Macro Constants for Enum Values

Using words that have already been defined by the C++ language - specifically, in its headers such as `math.h`, may cause compilation errors if the `#include` statement for one of those headers appears before the one for `.proto.h`. Avoid using macro constants such as “`NULL`,” “`NAN`,” and “`DOMAIN`” as enum values.

​	使用 C++ 语言中已定义的保留字（特别是在 `math.h` 等头文件中）可能会导致编译错误。例如，如果某个头文件的 `#include` 语句出现在 `.proto.h` 文件之前，可能会产生问题。避免使用像 `NULL`、`NAN` 和 `DOMAIN` 这样的宏常量作为枚举值。

## **务必**使用公认类型和通用类型 **Do** Use Well-Known Types and Common Types

Using the following common, shared types is strongly encouraged. E.g., do not use `int32 timestamp_seconds_since_epoch` or `int64 timeout_millis` in your code when a perfectly suitable common type already exists!

​	强烈建议使用以下常见的共享类型。例如，不要在代码中使用 `int32 timestamp_seconds_since_epoch` 或 `int64 timeout_millis`，而是使用适合的通用类型！

- [`duration`](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/duration.proto) is a signed, fixed-length span of time (for example, 42s).
  - [`duration`](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/duration.proto): 一个有符号的固定长度时间跨度（例如 42s）。

- [`timestamp`](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/timestamp.proto) is a point in time independent of any time zone or calendar (for example, 2017-01-15T01:30:15.01Z).
  - [`timestamp`](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/timestamp.proto): 一个与时区或日历无关的时间点（例如 2017-01-15T01:30:15.01Z）。

- [`interval`](https://github.com/googleapis/googleapis/blob/master/google/type/interval.proto) is a time interval independent of time zone or calendar (for example, 2017-01-15T01:30:15.01Z - 2017-01-16T02:30:15.01Z).
  - [`interval`](https://github.com/googleapis/googleapis/blob/master/google/type/interval.proto): 一个与时区或日历无关的时间间隔（例如 2017-01-15T01:30:15.01Z - 2017-01-16T02:30:15.01Z）。

- [`date`](https://github.com/googleapis/googleapis/blob/master/google/type/date.proto) is a whole calendar date (for example, 2005-09-19).
  - [`date`](https://github.com/googleapis/googleapis/blob/master/google/type/date.proto): 一个完整的日历日期（例如 2005-09-19）。

- [`month`](https://github.com/googleapis/googleapis/blob/master/google/type/month.proto) is a month of year (for example, April).
  - [`month`](https://github.com/googleapis/googleapis/blob/master/google/type/month.proto): 一年的月份（例如 4 月）。

- [`dayofweek`](https://github.com/googleapis/googleapis/blob/master/google/type/dayofweek.proto) is a day of week (for example, Monday).
  - [`dayofweek`](https://github.com/googleapis/googleapis/blob/master/google/type/dayofweek.proto): 一周中的某一天（例如星期一）。

- [`timeofday`](https://github.com/googleapis/googleapis/blob/master/google/type/timeofday.proto) is a time of day (for example, 10:42:23).
  - [`timeofday`](https://github.com/googleapis/googleapis/blob/master/google/type/timeofday.proto): 一天中的某个时间（例如 10:42:23）。

- [`field_mask`](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/field_mask.proto) is a set of symbolic field paths (for example, f.b.d).
  - [`field_mask`](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/field_mask.proto): 一组符号字段路径（例如 f.b.d）。

- [`postal_address`](https://github.com/googleapis/googleapis/blob/master/google/type/postal_address.proto) is a postal address (for example, 1600 Amphitheatre Parkway Mountain View, CA 94043 USA).
  - [`postal_address`](https://github.com/googleapis/googleapis/blob/master/google/type/postal_address.proto): 一个邮政地址（例如 1600 Amphitheatre Parkway Mountain View, CA 94043 USA）。

- [`money`](https://github.com/googleapis/googleapis/blob/master/google/type/money.proto) is an amount of money with its currency type (for example, 42 USD).
  - [`postal_address`](https://github.com/googleapis/googleapis/blob/master/google/type/postal_address.proto): 一个邮政地址（例如 1600 Amphitheatre Parkway Mountain View, CA 94043 USA）。

- [`latlng`](https://github.com/googleapis/googleapis/blob/master/google/type/latlng.proto) is a latitude/longitude pair (for example, 37.386051 latitude and -122.083855 longitude).
  - [`latlng`](https://github.com/googleapis/googleapis/blob/master/google/type/latlng.proto): 一个纬度/经度对（例如纬度 37.386051， 经度 -122.083855）。

- [`color`](https://github.com/googleapis/googleapis/blob/master/google/type/color.proto) is a color in the RGBA color space.
  - [`color`](https://github.com/googleapis/googleapis/blob/master/google/type/color.proto): 一个 RGBA 颜色空间的颜色。




## **务必**在独立文件中定义消息类型 **Do** Define Message Types in Separate Files

When defining a proto schema, you should have a single message, enum, extension, service, or group of cyclic dependencies per file. This makes refactoring easier. Moving files when they’re separated is much easier than extracting messages from a file with other messages. Following this practice also helps to keep the proto schema files smaller, which enhances maintainability.

​	在定义 proto 架构时，每个文件中应只包含一个消息、枚举、扩展、服务或循环依赖组。这样可以使重构更加容易。分离文件时，移动文件比从包含其他消息的文件中提取消息要容易得多。遵循此实践还可以使 proto 架构文件更小，从而提高可维护性。

If they will be widely used outside of your project, consider putting them in their own file with no dependencies. Then it’s easy for anyone to use those types without introducing the transitive dependencies in your other proto files.

​	如果这些类型会在项目外部被广泛使用，请考虑将它们放在没有依赖项的独立文件中。这样任何人都可以轻松使用这些类型，而不会引入其他 proto 文件的传递依赖。

For more on this topic, see [1-1-1 Rule]({{< ref "/docs/ProgrammingGuides/1-1-1BestPractice" >}}).

​	有关更多信息，请参阅 [1-1-1 规则]({{< ref "/docs/ProgrammingGuides/1-1-1BestPractice" >}})。



## **不要**更改字段的默认值 **Don’t** Change the Default Value of a Field

Almost never change the default value of a proto field. This causes version skew between clients and servers. A client reading an unset value will see a different result than a server reading the same unset value when their builds straddle the proto change. Proto3 removed the ability to set default values.

​	几乎不要更改 proto 字段的默认值。这会导致客户端和服务器之间的版本差异。当客户端读取未设置的值时，看到的结果可能与服务器读取同一值时不同，而它们的构建横跨该 proto 更改。Proto3 已移除设置默认值的功能。

## **不要**从重复字段切换到标量字段 **Don’t** Go from Repeated to Scalar

Although it won’t cause crashes, you’ll lose data. For JSON, a mismatch in repeatedness will lose the whole *message*. For numeric proto3 fields and proto2 `packed` fields, going from repeated to scalar will lose all data in that *field*. For non-numeric proto3 fields and un-annotated proto2 fields, going from repeated to scalar will result in the last deserialized value “winning.”

​	尽管不会导致崩溃，但会导致数据丢失。在 JSON 中，重复性的错配会丢失整个 *消息*。在 proto3 的数字字段和 proto2 的 `packed` 字段中，从重复切换到标量字段会丢失该字段的所有数据。在 proto3 的非数字字段和未标注的 proto2 字段中，从重复切换到标量字段会导致最后反序列化的值“获胜”。

Going from scalar to repeated is OK in proto2 and in proto3 with `[packed=false]` because for binary serialization the scalar value becomes a one-element list .

​	在 proto2 中以及在 proto3 中使用 `[packed=false]` 时，从标量切换到重复字段是可以的，因为对于二进制序列化，标量值会变为单元素列表。



## **务必**遵循生成代码的样式指南 **Do** Follow the Style Guide for Generated Code

Proto generated code is referred to in normal code. Ensure that options in `.proto` file do not result in generation of code which violate the style guide. For example:

​	proto 生成的代码会在普通代码中引用。确保 `.proto` 文件中的选项不会导致生成的代码违反样式指南。例如：

- `java_outer_classname` should follow https://google.github.io/styleguide/javaguide.html#s5.2.2-class-names
  - `java_outer_classname` 应遵循 [Java 类命名规范](https://google.github.io/styleguide/javaguide.html#s5.2.2-class-names)。

- `java_package` and `java_alt_package` should follow https://google.github.io/styleguide/javaguide.html#s5.2.1-package-names
  - `java_package` 和 `java_alt_package` 应遵循 [Java 包命名规范](https://google.github.io/styleguide/javaguide.html#s5.2.1-package-names)。

- `package`, although used for Java when `java_package` is not present, always directly corresponds to C++ namespace and thus should follow https://google.github.io/styleguide/cppguide.html#Namespace_Names. If these style guides conflict, use `java_package` for Java.
  - `package` 虽然在没有 `java_package` 时会用于 Java，但始终直接对应于 C++ 命名空间，因此应遵循 [C++ 命名空间规范](https://google.github.io/styleguide/cppguide.html#Namespace_Names)。如果这些样式指南存在冲突，应优先考虑为 Java 设置 `java_package`。

- `ruby_package` should be in the form `Foo::Bar::Baz` rather than `Foo.Bar.Baz`.
  - `ruby_package` 应采用 `Foo::Bar::Baz` 的形式，而非 `Foo.Bar.Baz`。




## **不要**使用文本格式消息进行数据交换 **Don’t** use Text Format Messages for Interchange

Text-based serialization formats like text format and JSON represent fields and enum values as strings. As a result, deserialization of protocol buffers in these formats using old code will fail when a field or enum value is renamed, or when a new field or enum value or extension is added. Use binary serialization when possible for data interchange, and use text format for human editing and debugging only.

​	像文本格式和 JSON 这样的基于文本的序列化格式会将字段和枚举值表示为字符串。因此，当字段或枚举值被重命名，或者添加新字段、枚举值或扩展时，使用旧代码反序列化这些格式的协议缓冲区将失败。尽可能使用二进制序列化进行数据交换，仅将文本格式用于人工编辑和调试。

If you use protos converted to JSON in your API or for storing data, you may not be able to safely rename fields or enums at all.

​	如果您在 API 或数据存储中使用转换为 JSON 的 proto，您可能完全无法安全地重命名字段或枚举。

## **绝不要**依赖序列化的跨构建稳定性 **Never** Rely on Serialization Stability Across Builds

The stability of proto serialization is not guaranteed across binaries or across builds of the same binary. Do not rely on it when, for example, building cache keys.

​	proto 序列化的稳定性在不同的二进制文件或同一二进制文件的不同构建之间没有保证。例如，在构建缓存键时，不要依赖序列化的稳定性。

## **不要**将 Java Protos 生成到与其他代码相同的 Java 包中 **Don’t** Generate Java Protos in the Same Java Package as Other Code

Generate Java proto sources into a separate package from your hand-written Java sources. The `package`, `java_package` and `java_alt_api_package` options control [where the generated Java sources are emitted]({{< ref "/docs/ReferenceGuides/Java/GeneratedCodeGuide#package" >}}). Make sure hand-written Java source code does not also live in that same package. A common practice is to generate your protos into a `proto` subpackage in your project that **only** contains those protos (that is, no hand-written source code).

​	将 Java proto 源文件生成到与手写 Java 源代码不同的包中。`package`、`java_package` 和 `java_alt_api_package` 选项控制 [生成的 Java 源文件的路径]({{< ref "/docs/ReferenceGuides/Java/GeneratedCodeGuide#package" >}})。确保手写 Java 源代码不位于相同的包中。一种常见的做法是将 proto 生成到项目中的 `proto` 子包中，该子包 **仅** 包含这些 proto 文件（即没有手写源代码）。

## 避免将语言关键字用作字段名称 Avoid Using Language Keywords for Field Names

If the name of a message, field, enum, or enum value is a keyword in the language that reads from/writes to that field, then protobuf may change the field name, and may have different ways to access them than normal fields. For example, see [this warning about Python]({{< ref "/docs/ReferenceGuides/Python/GeneratedCodeGuide#keyword-conflicts" >}}).

​	如果消息、字段、枚举或枚举值的名称是读取/写入该字段的语言中的关键字，则 protobuf 可能会更改字段名称，并且访问这些字段的方式可能不同于普通字段。例如，请参阅 [Python 的相关警告]({{< ref "/docs/ReferenceGuides/Python/GeneratedCodeGuide#keyword-conflicts" >}})。

You should also avoid using keywords in your file paths, as this can also cause problems.

​	您还应该避免在文件路径中使用关键字，因为这也可能引发问题。

## 附录 Appendix

### API 最佳实践 API Best Practices

This document lists only changes that are extremely likely to cause breakage. For higher-level guidance on how to craft proto APIs that grow gracefully see [API Best Practices]({{< ref "/docs/ProgrammingGuides/APIBestPractices" >}}).

​	本文档仅列出非常可能导致破坏的更改。有关如何设计能够平稳扩展的 Proto API 的高级指导，请参阅 [API 最佳实践]({{< ref "/docs/ProgrammingGuides/APIBestPractices" >}})。
