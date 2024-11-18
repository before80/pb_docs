+++
title = "字段存在性"
date = 2024-11-17T09:35:36+08:00
weight = 110
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

> 原文：[https://protobuf.dev/programming-guides/field_presence/](https://protobuf.dev/programming-guides/field_presence/)
>
> 收录该文档的时间：`2024-11-17T09:35:36+08:00`

# Application Note: Field Presence - 应用说明：

Explains the various presence-tracking disciplines for protobuf fields. It also explains the behavior of explicit presence-tracking for singular proto3 fields with basic types.

​	解释了protobuf字段的各种存在性跟踪机制，以及对基本类型的proto3单字段显式存在性跟踪行为。

## 背景 Background

*Field presence* is the notion of whether a protobuf field has a value. There are two different manifestations of presence for protobufs: *implicit presence*, where the generated message API stores field values (only), and *explicit presence*, where the API also stores whether or not a field has been set.

​	**字段存在性**是指protobuf字段是否具有值的概念。对于protobuf，存在性有两种表现形式：**隐式存在性**，即生成的消息API仅存储字段值；**显式存在性**，即API还存储字段是否已被设置的信息。

Historically, proto2 has mostly followed *explicit presence*, while proto3 exposes only *implicit presence* semantics. Singular proto3 fields of basic types (numeric, string, bytes, and enums) which are defined with the `optional` label have *explicit presence*, like proto2 (this feature is enabled by default as release 3.15).

​	历史上，proto2主要采用**显式存在性**，而proto3仅提供**隐式存在性**语义。定义为`optional`标签的proto3基本类型（数字、字符串、字节和枚举）字段具有与proto2相同的**显式存在性**（此功能自3.15版本起默认启用）。

**NOTE:** We recommend always adding the `optional` label for proto3 basic types. This provides a smoother path to editions, which uses explicit presence by default.

​	**注意**：建议始终为proto3基本类型添加`optional`标签，这将为迁移到默认使用显式存在性的editions提供更平滑的路径。

### 存在性规则 Presence Disciplines

*Presence disciplines* define the semantics for translating between the *API representation* and the *serialized representation*. The *implicit presence* discipline relies upon the field value itself to make decisions at (de)serialization time, while the *explicit presence* discipline relies upon the explicit tracking state instead.

​	**存在性规则**定义了在*API表示形式*和*序列化表示形式*之间进行转换的语义。**隐式存在性**依赖字段值本身在（反）序列化时做出决策，而**显式存在性**则依赖显式跟踪状态。

### 在*标记-值流*（线格式）序列化中的存在性 Presence in *Tag-value stream* (Wire Format) Serialization

The wire format is a stream of tagged, *self-delimiting* values. By definition, the wire format represents a sequence of *present* values. In other words, every value found within a serialization represents a *present* field; furthermore, the serialization contains no information about not-present values.

​	线格式是一种包含标记的、**自我分隔**的值流。根据定义，线格式表示一系列**存在**的值。换句话说，序列化中找到的每个值都代表一个**存在**的字段；此外，序列化中不包含任何关于不存在值的信息。

The generated API for a proto message includes (de)serialization definitions which translate between API types and a stream of definitionally *present* (tag, value) pairs. This translation is designed to be forward- and backward-compatible across changes to the message definition; however, this compatibility introduces some (perhaps surprising) considerations when deserializing wire-formatted messages:

​	生成的proto消息API包含将API类型与定义性**存在**的(tag, value)对流之间进行翻译的(反)序列化定义。该翻译设计为在消息定义的更改过程中保持前向和后向兼容性；然而，这种兼容性在反序列化线格式消息时引入了一些（可能令人惊讶的）考虑：

- When serializing, fields with `implicit presence` are not serialized if they contain their default value. 在序列化时，对于具有`隐式存在性`的字段，如果其值为默认值，则不会被序列化。

  - For numeric types, the default is 0.
    - 对于数字类型，默认值是0。

  - For enums, the default is the zero-valued enumerator.
    - 对于枚举类型，默认值是零值枚举成员。

  - For strings, bytes, and repeated fields, the default is the zero-length value.
    - 对于字符串、字节和重复字段，默认值是零长度值。

  - For messages, the default is the language-specific null value.
    - 对于消息类型，默认值是语言特定的`null`值。

- “Empty” length-delimited values (such as empty strings) can be validly represented in serialized values: the field is “present,” in the sense that it appears in the wire format. However, if the generated API does not track presence, then these values may not be re-serialized; i.e., the empty field may be “not present” after a serialization round-trip. “空的”长度分隔值（例如空字符串）可以在序列化值中有效表示：字段被认为是“存在的”，因为它出现在线格式中。然而，如果生成的API不跟踪字段存在性，则这些值可能无法重新序列化；即，在序列化往返后，空字段可能变为“不存在”。

- When deserializing, duplicate field values may be handled in different ways depending on the field definition. 在反序列化时，重复字段值的处理方式取决于字段定义。

  - Duplicate `repeated` fields are typically appended to the field’s API representation. (Note that serializing a *packed* repeated field produces only one, length-delimited value in the tag stream.)
    - 对于重复字段`repeated`，重复值通常被追加到字段的API表示中。（注意：序列化一个**打包**的重复字段时，在标记流中只会生成一个长度分隔值。）
  - Duplicate `optional` field values follow the rule that “the last one wins.”
    - 对于可选字段`optional`，重复值遵循“最后一个值胜出”规则。

- `oneof` fields expose the API-level invariant that only one field is set at a time. However, the wire format may include multiple (tag, value) pairs which notionally belong to the `oneof`. Similar to `optional` fields, the generated API follows the “last one wins” rule. `oneof`字段在API级别强制每次只有一个字段被设置。然而，线格式可能包含多个从概念上属于`oneof`的(tag, value)对。与`optional`字段类似，生成的API遵循“最后一个值胜出”规则。

- Out-of-range values are not returned for enum fields in generated proto2 APIs. However, out-of-range values may be stored as *unknown fields* in the API, even though the wire-format tag was recognized. 对于生成的proto2 API，枚举字段的超范围值不会返回。然而，即使线格式标记被识别，超范围值仍可能作为*未知字段*存储在API中。

### *命名字段映射*格式中的存在性 Presence in *Named-field Mapping* Formats

Protobufs can be represented in human-readable, textual forms. Two notable formats are TextFormat (the output format produced by generated message `DebugString` methods) and JSON.

​	protobuf可以用人类可读的文本形式表示。其中两个显著的格式是TextFormat（由生成的消息`DebugString`方法生成的输出格式）和JSON。

These formats have correctness requirements of their own, and are generally stricter than *tagged-value stream* formats. However, TextFormat more closely mimics the semantics of the wire format, and does, in certain cases, provide similar semantics (for example, appending repeated name-value mappings to a repeated field). In particular, similar to the wire format, TextFormat only includes fields which are present.

​	这些格式有自己的正确性要求，通常比*标记-值流*格式更严格。然而，TextFormat更接近于线格式语义，在某些情况下提供类似的语义（例如，将重复的名称-值映射追加到重复字段）。特别是，类似于线格式，TextFormat仅包括存在的字段。

JSON is a much stricter format, however, and cannot validly represent some semantics of the wire format or TextFormat.

​	JSON格式更严格，无法有效表示线格式或TextFormat的一些语义。

- Notably, JSON *elements* are semantically unordered, and each member must have a unique name. This is different from TextFormat rules for repeated fields.
  - 特别是，JSON**元素**在语义上是无序的，每个成员必须具有唯一的名称。这与TextFormat的重复字段规则不同。

- JSON may include fields that are “not present,” unlike the implicit presence discipline for other formats: 与其他格式的**隐式存在性**规则不同，JSON可以包括“不存在”的字段：

  - JSON defines a `null` value, which may be used to represent a *defined but not-present field*.
    - JSON定义了一个`null`值，可用于表示**已定义但不存在的字段**。
  - Repeated field values may be included in the formatted output, even if they are equal to the default (an empty list).
    - 即使重复字段值为空列表，JSON仍可能包含在格式化输出中。

- Because JSON elements are unordered, there is no way to unambiguously interpret the “last one wins” rule. 由于JSON元素无序，因此无法明确解释“最后一个值胜出”规则：

  - In most cases, this is fine: JSON elements must have unique names: repeated field values are not valid JSON, so they do not need to be resolved as they are for TextFormat.
    - 通常，这没有问题：JSON元素必须具有唯一的名称，重复字段值在JSON中无效，因此不需要像在TextFormat中那样解析。
  - However, this means that it may not be possible to interpret `oneof` fields unambiguously: if multiple cases are present, they are unordered.
    - 然而，这意味着可能无法明确解释`oneof`字段：如果多个情况同时存在，它们是无序的。

In theory, JSON *can* represent presence in a semantic-preserving fashion. In practice, however, presence correctness can vary depending upon implementation choices, especially if JSON was chosen as a means to interoperate with clients not using protobufs.

​	从理论上讲，JSON**可以**以保留语义的方式表示字段存在性。然而，实际上，字段存在性的正确性可能因实现选择而异，尤其是在JSON被用作与未使用protobuf的客户端进行互操作的手段时。

### Proto2 API中的存在性 Presence in Proto2 APIs

This table outlines whether presence is tracked for fields in proto2 APIs (both for generated APIs and using dynamic reflection):

​	以下表格概述了proto2 API（包括生成的API和动态反射API）中字段是否跟踪存在性：

| Field type                                                   | 显式存在性 Explicit Presence |
| ------------------------------------------------------------ | ---------------------------- |
| Singular numeric (integer or floating point) 单一数字（整数或浮点） | ✔️                            |
| Singular enum 单一枚举                                       | ✔️                            |
| Singular string or bytes 单一字符串或字节                    | ✔️                            |
| Singular message 单一消息                                    | ✔️                            |
| Repeated 重复字段                                            |                              |
| Oneofs `oneof`字段                                           | ✔️                            |
| Maps 映射字段                                                |                              |

Singular fields (of all types) track presence explicitly in the generated API. The generated message interface includes methods to query presence of fields. For example, the field `foo` has a corresponding `has_foo` method. (The specific name follows the same language-specific naming convention as the field accessors.) These methods are sometimes referred to as “hazzers” within the protobuf implementation.

​	在生成的 API 中，单一字段（所有类型）显式地跟踪存在性。生成的消息接口包含查询字段存在性的方法。例如，字段 `foo` 对应的存在性方法为 `has_foo`。（具体名称遵循与字段访问器相同的语言特定命名约定。）在 protobuf 实现中，这些方法有时被称为“hazzers”。

Similar to singular fields, `oneof` fields explicitly track which one of the members, if any, contains a value. For example, consider this example `oneof`:

​	与单一字段类似，`oneof` 字段显式地跟踪是否有一个成员包含值。例如，以下是一个 `oneof` 字段示例：

```protobuf
oneof foo {
  int32 a = 1;
  float b = 2;
}
```

Depending on the target language, the generated API would generally include several methods:

​	根据目标语言，生成的 API 通常包括以下方法：

- A hazzer for the oneof: `has_foo`
  - 用于整个 `oneof` 的存在性方法：`has_foo`

- A *oneof case* method: `foo`
  - 用于返回 `oneof` 当前设置成员的方法：`foo`

- Hazzers for the members: `has_a`, `has_b`
  - 成员的存在性方法：`has_a`, `has_b`

- Getters for the members: `a`, `b`
  - 成员的访问器方法：`a`, `b`


Repeated fields and maps do not track presence: there is no distinction between an *empty* and a *not-present* repeated field.

​	对于重复字段和映射字段，不跟踪存在性：*空*字段与*不存在*字段没有区别。

### Proto3 API 中的存在性 Presence in Proto3 APIs

This table outlines whether presence is tracked for fields in proto3 APIs (both for generated APIs and using dynamic reflection):

​	下表列出了在 Proto3 API 中，字段是否跟踪存在性（包括生成的 API 和动态反射 API）：

| Field type                                                   | `optional` | 显式存在性 Explicit Presence |
| ------------------------------------------------------------ | ---------- | ---------------------------- |
| Singular numeric (integer or floating point) 单一数字（整数或浮点） | No         |                              |
| Singular numeric (integer or floating point) 单一数字（整数或浮点） | Yes        | ✔️                            |
| Singular enum 单一枚举                                       | No         |                              |
| Singular enum 单一枚举                                       | Yes        | ✔️                            |
| Singular string or bytes 单一字符串或字节                    | No         |                              |
| Singular string or bytes 单一字符串或字节                    | Yes        | ✔️                            |
| Singular message 单一消息                                    | No         | ✔️                            |
| Singular message 单一消息                                    | Yes        | ✔️                            |
| Repeated 重复字段                                            | N/A 不适用 |                              |
| Oneofs `oneof` 字段                                          | N/A 不适用 | ✔️                            |
| Maps 映射字段                                                | N/A 不适用 |                              |

Similar to proto2 APIs, proto3 does not track presence explicitly for repeated fields. Without the `optional` label, proto3 APIs do not track presence for basic types (numeric, string, bytes, and enums), either. Oneof fields affirmatively expose presence, although the same set of hazzer methods may not generated as in proto2 APIs.

​	与 Proto2 API 类似，Proto3 不显式跟踪重复字段的存在性。如果没有 `optional` 标签，Proto3 API 也不会跟踪基本类型（数字、字符串、字节和枚举）的存在性。然而，`oneof` 字段明确暴露存在性，但生成的 hazzer 方法可能与 Proto2 API 中不同。

Under the *implicit presence* discipline, the default value is synonymous with “not present” for purposes of serialization. To notionally “clear” a field (so it won’t be serialized), an API user would set it to the default value.

​	在 **隐式存在性** 规则下，对于序列化目的，默认值等同于“不存在”。为了“清除”字段（使其不被序列化），API 用户需要将其设置为默认值。

The default value for enum-typed fields under *implicit presence* is the corresponding 0-valued enumerator. Under proto3 syntax rules, all enum types are required to have an enumerator value which maps to 0. By convention, this is an `UNKNOWN` or similarly-named enumerator. If the zero value is notionally outside the domain of valid values for the application, this behavior can be thought of as tantamount to *explicit presence*.

​	对于枚举字段，在 **隐式存在性** 规则下，默认值是对应的零值枚举项。根据 Proto3 语法规则，所有枚举类型都必须有一个映射到 `0` 的枚举值。通常，这被命名为 `UNKNOWN` 或类似的名称。如果零值在应用中超出有效值的定义域，这种行为可被视为接近于 **显式存在性**。

### 语义差异 Semantic Differences

The *implicit presence* serialization discipline results in visible differences from the *explicit presence* tracking discipline, when the default value is set. For a singular field with numeric, enum, or string type:

​	**隐式存在性** 的序列化规则与 **显式存在性** 的跟踪规则在默认值设置上会导致明显的差异。对于单一字段（数字、枚举或字符串类型）：

- Implicit presence discipline: 隐式存在性规则：

  - Default values are not serialized.
    - 默认值不会被序列化。
  - Default values are *not* merged-from.
    - 默认值不会被合并。
  - To “clear” a field, it is set to its default value.
    - 若要“清除”字段，需要将其设置为默认值。
  - The default value may mean: 默认值可能表示：
    - the field was explicitly set to its default value, which is valid in the application-specific domain of values;
      - 字段被显式设置为默认值（在应用特定的值域中是有效的）；
    - the field was notionally “cleared” by setting its default; or
      - 字段通过设置为默认值被“清除”；
    - the field was never set.
      - 字段从未被设置。
  - `has_` methods are not generated (but see note after this list)
    - 通常不会生成 `has_` 方法（但 Dart 是一个例外）。

- Explicit presence discipline: 显式存在性规则：

  - Explicitly set values are always serialized, including default values.
    - 显式设置的值总是会被序列化，包括默认值。
  - Un-set fields are never merged-from.
    - 未设置的字段永远不会被合并。
  - Explicitly set fields – including default values – *are* merged-from.
    - 显式设置的字段（包括默认值）会被合并。
  - A generated `has_foo` method indicates whether or not the field `foo` has been set (and not cleared).
    - 生成的 `has_foo` 方法指示字段 `foo` 是否已被设置（且未被清除）。
  - A generated `clear_foo` method must be used to clear (i.e., un-set) the value.
    - 必须使用生成的 `clear_foo` 方法来清除（即取消设置）字段值。

**Note:** `Has_` methods are not generated for implicit members in most cases. The exception to this behavior is Dart, which generates `has_` methods with proto3 proto schema files.

​	**注意：** 对于大多数情况下，隐式成员不会生成 `Has_` 方法。例外是 Dart，它在 proto3 的 proto schema 文件中会生成 `has_` 方法。

### 合并的考虑 Considerations for Merging

Under the *implicit presence* rules, it is effectively impossible for a target field to merge-from its default value (using the protobuf’s API merging functions). This is because default values are skipped, similar to the *implicit presence* serialization discipline. Merging only updates the target (merged-to) message using the non-skipped values from the update (merged-from) message.

​	在 **隐式存在性** 规则下，目标字段无法从其默认值中合并（通过 protobuf 的 API 合并函数）。这是因为默认值会被跳过，类似于 **隐式存在性** 的序列化规则。合并仅使用更新消息（合并源）中的非跳过值更新目标消息（合并目标）。

The difference in merging behavior has further implications for protocols which rely on partial “patch” updates. If field presence is not tracked, then an update patch alone cannot represent an update to the default value, because only non-default values are merged-from.

​	这种合并行为的差异对依赖部分“补丁”更新的协议有进一步的影响。如果字段存在性未被跟踪，则仅凭更新补丁无法表示对默认值的更新，因为只有非默认值会被合并。

Updating to set a default value in this case requires some external mechanism, such as `FieldMask`. However, if presence *is* tracked, then all explicitly-set values – even default values – will be merged into the target.

​	若要在这种情况下更新默认值，需要一些外部机制，例如 `FieldMask`。但是，如果存在性 **被跟踪**，则所有显式设置的值——即使是默认值——都会被合并到目标中。

### 更改兼容性的考虑 Considerations for change-compatibility

Changing a field between *explicit presence* and *implicit presence* is a binary-compatible change for serialized values in wire format. However, the serialized representation of the message may differ, depending on which version of the message definition was used for serialization. Specifically, when a “sender” explicitly sets a field to its default value:

​	在字段之间切换 **显式存在性** 和 **隐式存在性** 对线格式的序列化值是二进制兼容的。然而，消息的序列化表示可能会有所不同，具体取决于序列化时使用的消息定义版本。例如，当“发送方”显式设置字段为其默认值时：

- The serialized value following *implicit presence* discipline does not contain the default value, even though it was explicitly set.
  - **隐式存在性** 规则下，序列化值不包含默认值，即使它已被显式设置。

- The serialized value following *explicit presence* discipline contains every “present” field, even if it contains the default value.
  - **显式存在性** 规则下，序列化值包含所有“存在”的字段，即使它包含默认值。


This change may or may not be safe, depending on the application’s semantics. For example, consider two clients with different versions of a message definition.

​	这种更改可能安全，也可能不安全，取决于应用的语义。例如，考虑两个客户端使用同一消息定义的不同版本。

Client A uses this definition of the message, which follows the *explicit presence* serialization discipline for field `foo`:

​	客户端 A 使用以下定义，其中字段 `foo` 遵循 **显式存在性** 的序列化规则：

```protobuf
syntax = "proto3";
message Msg {
  optional int32 foo = 1;
}
```

Client B uses a definition of the same message, except that it follows the *no presence* discipline:

​	客户端 B 使用相同消息的定义，但遵循 **无存在性** 规则：

```protobuf
syntax = "proto3";
message Msg {
  int32 foo = 1;
}
```

Now, consider a scenario where client A observes `foo`’s presence as the clients repeatedly exchange the “same” message by deserializing and reserializing:

​	假设客户端 A 和客户端 B 反复交换“相同”的消息，以下是可能的情况：

```protobuf
// Client A:
// 客户端 A:
Msg m_a;
m_a.set_foo(1);                  // non-default value 非默认值
assert(m_a.has_foo());           // OK
Send(m_a.SerializeAsString());   // to client B 发送给客户端 B

// Client B:
// 客户端 B:
Msg m_b;
m_b.ParseFromString(Receive());  // from client A 接收来自客户端 A 的消息
assert(m_b.foo() == 1);          // OK
Send(m_b.SerializeAsString());   // to client A 发送给客户端 A

// Client A: 
// 客户端 A:
m_a.ParseFromString(Receive());  // from client B 接收来自客户端 B 的消息
assert(m_a.foo() == 1);          // OK
assert(m_a.has_foo());           // OK
m_a.set_foo(0);                  // default value 默认值
Send(m_a.SerializeAsString());   // to client B 发送给客户端 B

// Client B:
// 客户端 B:
Msg m_b;
m_b.ParseFromString(Receive());  // from client A 接收来自客户端 A 的消息
assert(m_b.foo() == 0);          // OK
Send(m_b.SerializeAsString());   // to client A 发送给客户端 A

// Client A:
// 客户端 A:
m_a.ParseFromString(Receive());  // from client B 接收来自客户端 B 的消息
assert(m_a.foo() == 0);          // OK
assert(m_a.has_foo());           // FAIL 失败
```

If client A depends on *explicit presence* for `foo`, then a “round trip” through client B will be lossy from the perspective of client A. In the example, this is not a safe change: client A requires (by `assert`) that the field is present; even without any modifications through the API, that requirement fails in a value- and peer-dependent case.

​	如果客户端 A 依赖于字段 `foo` 的 **显式存在性**，则通过客户端 B 的“往返”将导致客户端 A 视角下的信息丢失。在这个例子中，这是不安全的更改：客户端 A 需要字段存在性（通过 `assert` 检查），但由于 API 的值和对等方之间的依赖性，该要求未被满足。

### 如何在 Proto3 中启用显式存在性  How to Enable *Explicit Presence* in Proto3

These are the general steps to use field tracking support for proto3:

​	以下是使用 proto3 的字段跟踪支持的常规步骤：

1. Add an `optional` field to a `.proto` file. 在 `.proto` 文件中为字段添加 `optional` 标签。
2. Run `protoc` (at least v3.15, or v3.12 using `--experimental_allow_proto3_optional` flag). 运行 `protoc`（至少版本为 v3.15，或者在 v3.12 中使用 `--experimental_allow_proto3_optional` 标志）。
3. Use the generated “hazzer” methods and “clear” methods in application code, instead of comparing or setting default values. 在应用代码中使用生成的 “hazzer” 方法 和 “clear” 方法，而不是比较或设置默认值。

### `.proto` 文件更改示例 `.proto` File Changes

This is an example of a proto3 message with fields which follow both *no presence* and *explicit presence* semantics:

​	以下是一个 proto3 消息示例，其中字段分别遵循 **无存在性** 和 **显式存在性** 的语义：

```protobuf
syntax = "proto3";
package example;

message MyMessage {
  // implicit presence:
  // 无存在性：
  int32 not_tracked = 1;

  // Explicit presence:
  // 显式存在性：
  optional int32 tracked = 2;
}
```

### `protoc` 调用 `protoc` Invocation

Presence tracking for proto3 messages is enabled by default [since v3.15.0](https://github.com/protocolbuffers/protobuf/releases/tag/v3.15.0) release, formerly up until [v3.12.0](https://github.com/protocolbuffers/protobuf/releases/tag/v3.12.0) the `--experimental_allow_proto3_optional` flag was required when using presence tracking with protoc.

​	自 [v3.15.0](https://github.com/protocolbuffers/protobuf/releases/tag/v3.15.0) 版本以来，默认启用了 proto3 消息的存在性跟踪支持。在 [v3.12.0](https://github.com/protocolbuffers/protobuf/releases/tag/v3.12.0) 版本之前，使用存在性跟踪需要 `--experimental_allow_proto3_optional` 标志。

### 使用生成的代码 Using the Generated Code

The generated code for proto3 fields with *explicit presence* (the `optional` label) will be the same as it would be in a proto2 file.

​	对于带有 **显式存在性** 的 proto3 字段（使用 `optional` 标签），生成的代码与 proto2 文件中的代码相同。

This is the definition used in the “implicit presence” examples below:	

​	这是“隐式存在性”示例中使用的定义：

```protobuf
syntax = "proto3";
package example;
message Msg {
  int32 foo = 1;
}
```

This is the definition used in the “explicit presence” examples below:	

​	这是“显式存在性”示例中使用的定义：

```protobuf
syntax = "proto3";
package example;
message Msg {
  optional int32 foo = 1;
}
```

In the examples, a function `GetProto` constructs and returns a message of type `Msg` with unspecified contents.	

​	在以下示例中，一个函数 `GetProto` 构造并返回一个类型为 `Msg` 的消息，其内容未指定。

#### C++ Example

Implicit presence:

​	隐式存在性：

```cpp
Msg m = GetProto();
if (m.foo() != 0) {
  // "Clear" the field: “清除”字段：
  m.set_foo(0);
} else {
  // Default value: field may not have been present. 默认值：字段可能不存在。
  m.set_foo(1);
}
```

Explicit presence:

​	显式存在性：

```c++
Msg m = GetProto();
if (m.has_foo()) {
  // Clear the field: 清除字段：
  m.clear_foo();
} else {
  // Field is not present, so set it. 字段不存在，因此设置它。
  m.set_foo(1);
}
```

#### C# Example

Implicit presence:

​	隐式存在性：

```c#
var m = GetProto();
if (m.Foo != 0) {
  // "Clear" the field: “清除”字段：
  m.Foo = 0;
} else {
  // Default value: field may not have been present. 默认值：字段可能不存在。
  m.Foo = 1;
}
```

Explicit presence:

​	显式存在性：

```c#
var m = GetProto();
if (m.HasFoo) {
  // Clear the field:  “清除”字段：
  m.ClearFoo();
} else {
  // Field is not present, so set it. 字段不存在，因此设置它。
  m.Foo = 1;
}
```

#### Go Example

Implicit presence:

​	隐式存在性：

```go
m := GetProto()
if m.Foo != 0 {
  // "Clear" the field: “清除”字段：
  m.Foo = 0
} else {
  // Default value: field may not have been present. 默认值：字段可能不存在。
  m.Foo = 1
}
```

Explicit presence:

​	显式存在性：

```go
m := GetProto()
if m.Foo != nil {
  // Clear the field: “清除”字段：
  m.Foo = nil
} else {
  // Field is not present, so set it. 字段不存在，因此设置它。
  m.Foo = proto.Int32(1)
}
```

#### Java Example

These examples use a `Builder` to demonstrate clearing. Simply checking presence and getting values from a `Builder` follows the same API as the message type.

​	这些示例使用 `Builder` 来演示清除字段。仅检查字段的存在性和获取字段值时，`Builder` 与消息类型 API 相同。

Implicit presence:

​	隐式存在性：

```java
Msg.Builder m = GetProto().toBuilder();
if (m.getFoo() != 0) {
  // "Clear" the field:
  m.setFoo(0);
} else {
  // Default value: field may not have been present. 默认值：字段可能不存在。
  m.setFoo(1);
}
```

Explicit presence:

​	显式存在性：

```java
Msg.Builder m = GetProto().toBuilder();
if (m.hasFoo()) {
  // Clear the field: “清除”字段：
  m.clearFoo()
} else {
  // Field is not present, so set it. 字段不存在，因此设置它。
  m.setFoo(1);
}
```

#### Python Example

Implicit presence:

​	隐式存在性：

```python
m = example.Msg()
if m.foo != 0:
  # "Clear" the field: “清除”字段：
  m.foo = 0
else:
  # Default value: field may not have been present. 默认值：字段可能不存在。
  m.foo = 1
```

Explicit presence:

​	显式存在性：

```python
m = example.Msg()
if m.HasField('foo'):
  # Clear the field:
  m.ClearField('foo')
else:
  # Field is not present, so set it. 字段不存在，因此设置它。
  m.foo = 1
```

#### Ruby Example

Implicit presence:

​	隐式存在性：

```ruby
m = Msg.new
if m.foo != 0
  # "Clear" the field: “清除”字段：
  m.foo = 0
else
  # Default value: field may not have been present. 默认值：字段可能不存在。
  m.foo = 1
end
```

Explicit presence:

​	显式存在性：

```ruby
m = Msg.new
if m.has_foo?
  # Clear the field:
  m.clear_foo
else
  # Field is not present, so set it. 字段不存在，因此设置它。
  m.foo = 1
end
```

#### Javascript Example

Implicit presence:

​	隐式存在性：

```js
var m = new Msg();
if (m.getFoo() != 0) {
  // "Clear" the field: “清除”字段：
  m.setFoo(0);
} else {
  // Default value: field may not have been present. 默认值：字段可能不存在。
  m.setFoo(1);
}
```

Explicit presence:

​	显式存在性：

```js
var m = new Msg();
if (m.hasFoo()) {
  // Clear the field:
  m.clearFoo()
} else {
  // Field is not present, so set it. 字段不存在，因此设置它。
  m.setFoo(1);
}
```

#### Objective-C Example

Implicit presence:

​	隐式存在性：

```objective-c
Msg *m = [[Msg alloc] init];
if (m.foo != 0) {
  // "Clear" the field: “清除”字段：
  m.foo = 0;
} else {
  // Default value: field may not have been present. 默认值：字段可能不存在。
  m.foo = 1;
}
```

Explicit presence:

​	显式存在性：

```objective-c
Msg *m = [[Msg alloc] init];
if (m.hasFoo()) {
  // Clear the field:
  [m clearFoo];
} else {
  // Field is not present, so set it. 字段不存在，因此设置它。
  [m setFoo:1];
}
```

## 速查表 Cheat sheet

**Proto2:**

Is field presence tracked?

​	字段存在性是否被跟踪？

| Field type                              | Tracked? 跟踪？ |
| --------------------------------------- | --------------- |
| Singular field 单一字段                 | yes             |
| Singular message field 单一消息字段     | yes             |
| Field in a oneof `oneof` 中的字段       | yes             |
| Repeated field & map 重复字段和映射字段 | no              |

**Proto3:**

Is field presence tracked?

​	字段存在性是否被跟踪？

| Field type                              | Tracked? 跟踪？                                       |
| --------------------------------------- | ----------------------------------------------------- |
| *Other* singular field 其他单一字段     | if defined as `optional` 如果定义为 `optional` 则跟踪 |
| Singular message field 单一消息字段     | yes                                                   |
| Field in a oneof `oneof` 中的字段       | yes                                                   |
| Repeated field & map 重复字段和映射字段 | no                                                    |

**Edition 2023:**

Is field presence tracked?

​	字段存在性是否被跟踪？

| Field type                                         | Tracked? 跟踪？ |
| -------------------------------------------------- | --------------- |
| Default                                            | yes             |
| `features.field_presence` set to `LEGACY_REQUIRED` | yes             |
| `features.field_presence` set to `IMPLICIT`        | no              |
| Repeated field & map 重复字段和映射字段            | no              |
