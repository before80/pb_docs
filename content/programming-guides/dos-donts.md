+++
title = "Proto 最佳实践"
weight = 90
description = "分享编写 Protocol Buffers 的经过验证的最佳实践。"
type = "docs"

+++

> 原文网址： https://protobuf.dev/programming-guides/dos-donts/

## Proto Best Practices - Proto 最佳实践

Shares vetted best practices for authoring Protocol Buffers.

​	分享编写 Protocol Buffers 的经过验证的最佳实践。



Clients and servers are never updated at exactly the same time - even when you try to update them at the same time. One or the other may get rolled back. Don’t assume that you can make a breaking change and it’ll be okay because the client and server are in sync.

​	客户端和服务器永远不会在完全相同的时间更新 - 即使您尝试同时更新它们。其中一个可能会回滚。不要假设您可以进行重大更改，并且由于客户端和服务器同步，因此不会出现问题。



## 不要重复使用标记号 **Don’t** Re-use a Tag Number 

Never re-use a tag number. It messes up deserialization. Even if you think no one is using the field, don’t re-use a tag number. If the change was live ever, there could be serialized versions of your proto in a log somewhere. Or there could be old code in another server that will break.

​	切勿重复使用标记号。它会搞乱反序列化。即使您认为没有人使用该字段，也不要重复使用标记号。如果更改已上线，则您的 proto 的序列化版本可能存在于某个日志中。或者另一个服务器中可能存在会中断的旧代码。



## 确实要为已删除的字段保留标记号 Do Reserve Tag Numbers for Deleted Fields 

When you delete a field that’s no longer used, reserve its tag number so that no one accidentally re-uses it in the future. Just `reserved 2, 3;` is enough. No type required (lets you trim dependencies!). You can also reserve names to avoid recycling now-deleted field names: `reserved "foo", "bar";`.

​	当您删除不再使用的字段时，请保留其标记号，以便将来没有人意外地重复使用它。只需 `reserved 2, 3;` 就足够了。不需要类型（让您减少依赖项！）。您还可以保留名称以避免回收现已删除的字段名称： `reserved "foo", "bar";` 。



## 确实要为已删除的枚举值保留数字 Do Reserve Numbers for Deleted Enum Values 

When you delete an enum value that’s no longer used, reserve its number so that no one accidentally re-uses it in the future. Just `reserved 2, 3;` is enough. You can also reserve names to avoid recycling now-deleted value names: `reserved "FOO", "BAR";`.

​	删除不再使用的枚举值时，保留其编号，以便将来没有人意外地重新使用它。只需 `reserved 2, 3;` 就足够了。您还可以保留名称以避免回收现已删除的值名称： `reserved "FOO", "BAR";` 。



## 不要更改字段的类型 Don’t Change the Type of a Field 

Almost never change the type of a field; it’ll mess up deserialization, same as re-using a tag number. The [protobuf docs](https://protobuf.dev/programming-guides/proto2#updating) outline a small number of cases that are okay (for example, going between `int32`, `uint32`, `int64` and `bool`). However, changing a field’s message type **will break** unless the new message is a superset of the old one.

​	几乎从不更改字段的类型；它会搞乱反序列化，就像重新使用标签号一样。protobuf 文档概述了一些可以的情况（例如，在 `int32` 、 `uint32` 、 `int64` 和 `bool` 之间切换）。但是，更改字段的消息类型会中断，除非新消息是旧消息的超集。



## 不要添加必填字段 Don’t Add a Required Field 

Never add a required field, instead add `// required` to document the API contract. Required fields are considered harmful by so many they were removed from proto3 completely. Make all fields optional or repeated. You never know how long a message type is going to last and whether someone will be forced to fill in your required field with an empty string or zero in four years when it’s no longer logically required but the proto still says it is.

​	切勿添加必填字段，而是添加 `// required` 来记录 API 契约。必填字段被认为是有害的，因此它们已从 proto3 中完全删除。使所有字段可选或重复。您永远不知道消息类型会持续多久，也不知道是否有人会在四年后被迫用空字符串或零来填写您的必填字段，而那时它在逻辑上不再需要，但 proto 仍然说需要。

For proto3 there are no `required` fields, so this advice does not apply.

​	对于 proto3，没有 `required` 字段，因此此建议不适用。



## 不要创建具有大量字段的消息 Don’t Make a Message with Lots of Fields 

Don’t make a message with “lots” (think: hundreds) of fields. In C++ every field adds roughly 65 bits to the in-memory object size whether it’s populated or not (8 bytes for the pointer and, if the field is declared as optional, another bit in a bitfield that keeps track of whether the field is set). When your proto grows too large, the generated code may not even compile (for example, in Java there is a hard limit on the size of a method ).

​	不要创建包含“大量”（想想：数百个）字段的消息。在 C++ 中，无论字段是否已填充，每个字段都会向内存中对象的尺寸添加大约 65 位（指针占 8 个字节，如果字段声明为可选字段，则在位字段中再添加一位，用于跟踪该字段是否已设置）。当您的 proto 增长过大时，生成的代码甚至可能无法编译（例如，在 Java 中，方法的大小存在严格限制）。



## 在枚举中包含一个未指定的值 Do Include an Unspecified Value in an Enum 

Enums should include a default `FOO_UNSPECIFIED` value as the first value in the declaration . When new values are added to a proto2 enum, old clients will see the field as unset and the getter will return the default value or the first-declared value if no default exists . For consistent behavior with [proto enums](https://protobuf.dev/programming-guides/proto2#enum), the first declared enum value should be a default `FOO_UNSPECIFIED` value and should use tag 0. It may be tempting to declare this default as a semantically meaningful value but as a general rule, do not, to aid in the evolution of your protocol as new enum values are added over time. All enum values declared under a container message are in the same C++ namespace, so prefix the unspecified value with the enum’s name to avoid compilation errors. If you’ll never need cross-language constants, an `int32` will preserve unknown values and generates less code. Note that [proto enums](https://protobuf.dev/programming-guides/proto2#enum) require the first value to be zero and can round-trip (deserialize, serialize) an unknown enum value.

​	枚举应包含一个默认 `FOO_UNSPECIFIED` 值作为声明中的第一个值。当向 proto2 枚举添加新值时，旧客户端会将该字段视为未设置，并且如果不存在默认值，则 getter 将返回默认值或第一个声明的值。为了与 proto 枚举保持一致的行为，第一个声明的枚举值应为默认 `FOO_UNSPECIFIED` 值，并应使用标记 0。将此默认值声明为语义有意义的值可能很诱人，但作为一般规则，不要这样做，以帮助您的协议随着时间的推移添加新的枚举值而不断发展。在容器消息下声明的所有枚举值都在同一个 C++ 命名空间中，因此请使用枚举的名称作为前缀来避免编译错误。如果您永远不需要跨语言常量， `int32` 将保留未知值并生成更少的代码。请注意，proto 枚举要求第一个值为零，并且可以往返（反序列化、序列化）未知枚举值。



## 不要对枚举值使用 C/C++ 宏常量 Don’t Use C/C++ Macro Constants for Enum Values 

Using words that have already been defined by the C++ language - specifically, in its headers such as `math.h`, may cause compilation errors if the `#include` statement for one of those headers appears before the one for `.proto.h`. Avoid using macro constants such as “`NULL`,” “`NAN`,” and “`DOMAIN`” as enum values.

​	使用 C++ 语言已定义的单词（特别是在其头文件中，例如 `math.h` ）可能会导致编译错误，如果其中一个头文件的 `#include` 语句出现在 `.proto.h` 的语句之前。避免将宏常量（例如“ `NULL` ”、“ `NAN` ”和“ `DOMAIN` ”）用作枚举值。



## 确实使用众所周知的类型和常见类型 Do Use Well-Known Types and Common Types 

Embedding the following common, shared types is strongly encouraged. Do not use `int32 timestamp_seconds_since_epoch` or `int64 timeout_millis` in your code when a perfectly suitable common type already exists!

​	强烈建议嵌入以下常见的共享类型。当已经存在完全合适的常见类型时，请勿在代码中使用 `int32 timestamp_seconds_since_epoch` 或 `int64 timeout_millis` ！



### 众所周知的类型 Well-Known Types 

- [duration](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/duration.proto) is a signed, fixed-length span of time (for example, 42s).
- duration 是一个有符号的固定长度时间跨度（例如，42s）。
- [timestamp](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/timestamp.proto) is a point in time independent of any time zone or calendar (for example, 2017-01-15T01:30:15.01Z).
- timestamp 是一个独立于任何时区或日历的时间点（例如，2017-01-15T01:30:15.01Z）。
- [field_mask](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/field_mask.proto) is a set of symbolic field paths (for example, f.b.d).
- field_mask 是一组符号字段路径（例如，f.b.d）。



### 常见类型 Common Types 

- [`Duration`](https://github.com/protocolbuffers/protobuf/blob/master/src/google/protobuf/duration.proto) is a signed, fixed-length span of time, such as 42s.
- `Duration` 是一个有符号的固定长度时间跨度，例如 42s。
- [`Timestamp`](https://github.com/protocolbuffers/protobuf/blob/master/src/google/protobuf/timestamp.proto) is a point in time independent of any time zone or calendar, such as 2017-01-15T01:30:15.01Z.
- `Timestamp` 是一个独立于任何时区或日历的时间点，例如 2017-01-15T01:30:15.01Z。
- [`interval`](https://github.com/googleapis/googleapis/blob/master/google/type/interval.proto) is a time interval independent of time zone or calendar (for example, 2017-01-15T01:30:15.01Z - 2017-01-16T02:30:15.01Z).
- `interval` 是一个独立于时区或日历的时间间隔（例如，2017-01-15T01:30:15.01Z - 2017-01-16T02:30:15.01Z）。
- [`date`](https://github.com/googleapis/googleapis/blob/master/google/type/date.proto) is a whole calendar date (for example, 2005-09-19).
- `date` 是一个完整的日历日期（例如，2005-09-19）。
- [`dayofweek`](https://github.com/googleapis/googleapis/blob/master/google/type/dayofweek.proto) is a day of week (for example, Monday).
- `dayofweek` 是一个星期几（例如，星期一）。
- [`timeofday`](https://github.com/googleapis/googleapis/blob/master/google/type/timeofday.proto) is a time of day (for example, 10:42:23).
- `timeofday` 是一个一天中的时间（例如，10:42:23）。
- [`latlng`](https://github.com/googleapis/googleapis/blob/master/google/type/latlng.proto) is a latitude/longitude pair (for example, 37.386051 latitude and -122.083855 longitude).
- `latlng` 是一个纬度/经度对（例如，纬度 37.386051 和经度 -122.083855）。
- [`money`](https://github.com/googleapis/googleapis/blob/master/google/type/money.proto) is an amount of money with its currency type (for example, 42 USD).
- `money` 是一个带有其货币类型的金额（例如，42 美元）。
- [`postal_address`](https://github.com/googleapis/googleapis/blob/master/google/type/postal_address.proto) is a postal address (for example, 1600 Amphitheatre Parkway Mountain View, CA 94043 USA).
- `postal_address` 是一个邮政地址（例如，1600 Amphitheatre Parkway Mountain View, CA 94043 USA）。
- [`color`](https://github.com/googleapis/googleapis/blob/master/google/type/color.proto) is a color in the RGBA color space.
- `color` 是 RGBA 颜色空间中的一个颜色。
- [`month`](https://github.com/googleapis/googleapis/blob/master/google/type/month.proto) is a month of year (for example, April).
- `month` 是一个月份（例如，4 月）。



## 在单独的文件中定义广泛使用的消息类型 Do Define Widely-used Message Types in Separate Files 

If you’re defining message types or enums that you hope/fear/expect to be widely used outside your immediate team, consider putting them in their own file with no dependencies. Then it’s easy for anyone to use those types without introducing the transitive dependencies in your other proto files.

​	如果你正在定义消息类型或枚举，你希望/担心/期望它们在你的直接团队之外被广泛使用，请考虑将它们放在它们自己的文件中，而没有任何依赖项。这样，任何人都可以轻松地使用这些类型，而不会在你的其他 proto 文件中引入传递依赖项。



## 不要更改字段的默认值 Don’t Change the Default Value of a Field 

Almost never change the default value of a proto field. This causes version skew between clients and servers. A client reading an unset value will see a different result than a server reading the same unset value when their builds straddle the proto change. Proto3 removed the ability to set default values.

​	几乎不要更改 proto 字段的默认值。这会导致客户端和服务器之间的版本偏差。当构建跨越 proto 更改时，读取未设置值的客户端将看到与读取相同未设置值时服务器看到的结果不同。Proto3 删除了设置默认值的功能。



## 不要从 Repeated 改为 Scalar - Don’t Go from Repeated to Scalar 

Although it won’t cause crashes, you’ll lose data. For JSON, a mismatch in repeatedness will lose the whole *message*. For numeric proto3 fields and proto2 `packed` fields, going from repeated to scalar will lose all data in that *field*. For non-numeric proto3 fields and un-annotated proto2 fields, going from repeated to scalar will result in the last deserialized value “winning.”

​	虽然不会导致崩溃，但您会丢失数据。对于 JSON，重复性不匹配将丢失整个消息。对于数字 proto3 字段和 proto2 `packed` 字段，从 repeated 改为 scalar 将丢失该字段中的所有数据。对于非数字 proto3 字段和未注释的 proto2 字段，从 repeated 改为 scalar 将导致最后一个反序列化的值“获胜”。

Going from scalar to repeated is OK in proto2 and in proto3 with `[packed=false]` because for binary serialization the scalar value becomes a one-element list .

​	在 proto2 中从 scalar 改为 repeated 是可以的，在 proto3 中使用 `[packed=false]` 也是可以的，因为对于二进制序列化，标量值会变成一个元素列表。



## 请遵循生成代码的风格指南 Do Follow the Style Guide for Generated Code 

Proto generated code is referred to in normal code. Ensure that options in `.proto` file do not result in generation of code which violate the style guide. For example:

​	在普通代码中引用了 Proto 生成的代码。确保 `.proto` 文件中的选项不会导致生成违反风格指南的代码。例如：

- `java_outer_classname` should follow https://google.github.io/styleguide/javaguide.html#s5.2.2-class-names
- `java_outer_classname` 应遵循 https://google.github.io/styleguide/javaguide.html#s5.2.2-class-names
- `java_package` and `java_alt_package` should follow https://google.github.io/styleguide/javaguide.html#s5.2.1-package-names
- `java_package` ， `java_alt_package` 应遵循 https://google.github.io/styleguide/javaguide.html#s5.2.1-package-names
- `package`, although used for Java when `java_package` is not present, always directly corresponds to C++ namespace and thus should follow https://google.github.io/styleguide/cppguide.html#Namespace_Names. If these style guides conflict, use `java_package` for Java.
- `package` ，虽然在 `java_package` 不存在时用于 Java，但始终直接对应于 C++ 命名空间，因此应遵循 https://google.github.io/styleguide/cppguide.html#Namespace_Names。如果这些风格指南冲突，请对 Java 使用 `java_package` 。



## 不要将文本格式消息用于交换 Don’t use Text Format Messages for Interchange 

Text-based serialization formats like text format and JSON represent fields and enum values as strings. As a result, deserialization of protocol buffers in these formats using old code will fail when a field or enum value is renamed, or when a new field or enum value or extension is added. Use binary serialization when possible for data interchange, and use text format for human editing and debugging only.

​	文本格式和 JSON 等基于文本的序列化格式将字段和枚举值表示为字符串。因此，在字段或枚举值重命名时，或添加新的字段、枚举值或扩展时，使用旧代码对这些格式中的协议缓冲区进行反序列化将会失败。在可能的情况下，对数据交换使用二进制序列化，仅对人工编辑和调试使用文本格式。

If you use protos converted to JSON in your API or for storing data, you may not be able to safely rename fields or enums at all.

​	如果您在 API 中或用于存储数据时使用转换为 JSON 的协议，则可能根本无法安全地重命名字段或枚举。



## 切勿依赖于跨版本构建的序列化稳定性 Never Rely on Serialization Stability Across Builds 

The stability of proto serialization is not guaranteed across binaries or across builds of the same binary. Do not rely on it when, for example, building cache keys.

​	跨二进制文件或同一二进制文件的不同版本，proto 序列化的稳定性无法得到保证。例如，在构建缓存键时，不要依赖它。



## **不要在与其他代码相同的 Java 包中生成 Java Proto Don’t Generate Java Protos in the Same Java Package as Other Code** 

Generate Java proto sources into a separate package from your hand-written Java sources. The `package`, `java_package` and `java_alt_api_package` options control [where the generated Java sources are emitted](https://protobuf.dev/reference/java/java-generated#package). Make sure hand-written Java source code does not also live in that same package. A common practice is to generate your protos into a `proto` subpackage in your project that **only** contains those protos (that is, no hand-written source code).

​	将 Java proto 源代码生成到与手工编写的 Java 源代码不同的包中。 `package` 、 `java_package` 和 `java_alt_api_package` 选项控制生成的 Java 源代码的发出位置。确保手工编写的 Java 源代码也不存在于同一包中。一种常见做法是将 proto 生成到项目中的 `proto` 子包中，该子包仅包含这些 proto（即，没有手工编写的源代码）。

## 避免将语言关键字用作字段名称 Avoid Using Language Keywords for Field Names 

If the name of a message, field, enum, or enum value is a keyword in the language that reads from/writes to that field, then protobuf may change the field name, and may have different ways to access them than normal fields. For example, see [this warning about Python](https://protobuf.dev/reference/python/python-generated#keyword-conflicts).

​	如果消息、字段、枚举或枚举值名称是读取/写入该字段的语言中的关键字，则 protobuf 可能会更改字段名称，并且可能具有不同于普通字段的访问方式。例如，请参阅有关 Python 的此警告。

You should also avoid using keywords in your file paths, as this can also cause problems.

​	您还应避免在文件路径中使用关键字，因为这也会导致问题。

## 附录 Appendix 

### 最佳实践 API Best Practices API 

This document lists only changes that are extremely likely to cause breakage. For higher-level guidance on how to craft proto APIs that grow gracefully see [API Best Practices](https://protobuf.dev/programming-guides/api).

​	此文档仅列出极有可能导致中断的更改。有关如何精心设计可优雅增长的 proto API 的更高级指导，请参阅 API 最佳做法。
