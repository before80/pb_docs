+++
title = "应用说明：字段存在"
weight = 85
linkTitle = "Field Presence"
description = "解释了 protobuf 字段的各种存在跟踪规则。它还解释了具有基本类型的奇数 proto3 字段的显式存在跟踪的行为。"
type = "docs"

+++

## Application Note: Field Presence 应用说明：字段存在

Explains the various presence-tracking disciplines for protobuf fields. It also explains the behavior of explicit presence-tracking for singular proto3 fields with basic types.

​	解释了 protobuf 字段的各种存在跟踪规则。它还解释了具有基本类型的奇数 proto3 字段的显式存在跟踪的行为。



## 背景 Background 

*Field presence* is the notion of whether a protobuf field has a value. There are two different manifestations of presence for protobufs: *no presence*, where the generated message API stores field values (only), and *explicit presence*, where the API also stores whether or not a field has been set.

​	字段存在是指 protobuf 字段是否具有值。protobuf 的存在有两种不同的表现形式：不存在，其中生成的 message API 仅存储字段值；显式存在，其中 API 还存储字段是否已设置。

Historically, proto2 has mostly followed *explicit presence*, while proto3 exposes only *no presence* semantics. Singular proto3 fields of basic types (numeric, string, bytes, and enums) which are defined with the `optional` label have *explicit presence*, like proto2 (this feature is enabled by default as release 3.15).

​	从历史上看，proto2 主要遵循显式存在，而 proto3 仅公开不存在语义。使用 `optional` 标签定义的基本类型（数字、字符串、字节和枚举）的奇数 proto3 字段具有显式存在，如 proto2（此功能在版本 3.15 中默认启用）。

### 存在约束 Presence Disciplines 

*Presence disciplines* define the semantics for translating between the *API representation* and the *serialized representation*. The *no presence* discipline relies upon the field value itself to make decisions at (de)serialization time, while the *explicit presence* discipline relies upon the explicit tracking state instead.

​	存在约束定义了在 API 表示和序列化表示之间进行转换时的语义。无存在约束依赖于字段值本身在 (反)序列化时做出决策，而显式存在约束则依赖于显式跟踪状态。

### 标签值流 (线格式) 序列化中的存在 Presence in *Tag-value stream* (Wire Format) Serialization 

The wire format is a stream of tagged, *self-delimiting* values. By definition, the wire format represents a sequence of *present* values. In other words, every value found within a serialization represents a *present* field; furthermore, the serialization contains no information about not-present values.

​	线格式是带标签的自限定值流。根据定义，线格式表示一系列存在的值。换句话说，序列化中找到的每个值都表示一个存在字段；此外，序列化不包含有关不存在的值的任何信息。

The generated API for a proto message includes (de)serialization definitions which translate between API types and a stream of definitionally *present* (tag, value) pairs. This translation is designed to be forward- and backward-compatible across changes to the message definition; however, this compatibility introduces some (perhaps surprising) considerations when deserializing wire-formatted messages:

​	针对 proto 消息生成的 API 包括 (反)序列化定义，这些定义在 API 类型和一系列定义上存在 (标签、值) 对之间进行转换。此转换旨在向前和向后兼容消息定义的更改；但是，这种兼容性在反序列化线格式消息时引入了一些（可能令人惊讶的）注意事项：

- When serializing, fields with no presence are not serialized if they contain their default value.

- 
  在序列化时，如果字段没有存在且包含其默认值，则不会序列化这些字段。
  
  - For numeric types, the default is 0.
  - 对于数字类型，默认值为 0。
  - For enums, the default is the zero-valued enumerator.
  - 对于枚举，默认值是零值枚举器。
  - For strings, bytes, and repeated fields, the default is the zero-length value.
  - 对于字符串、字节和重复字段，默认值是零长度值。
  - For messages, the default is the language-specific null value.
  - 对于消息，默认值是特定于语言的空值。
  
- “Empty” length-delimited values (such as empty strings) can be validly represented in serialized values: the field is “present,” in the sense that it appears in the wire format. However, if the generated API does not track presence, then these values may not be re-serialized; i.e., the empty field may be “not present” after a serialization round-trip.
  
- “空”长度分隔值（例如空字符串）可以有效地表示在序列化值中：该字段是“存在的”，从它出现在线格式中这一点来看。但是，如果生成的 API 不跟踪存在，则这些值可能不会被重新序列化；即，空字段在序列化往返后可能“不存在”。
  
- When deserializing, duplicate field values may be handled in different ways depending on the field definition.

- 
  在反序列化时，重复的字段值可能会根据字段定义以不同的方式处理。
  
  - Duplicate `repeated` fields are typically appended to the field’s API representation. (Note that serializing a *packed* repeated field produces only one, length-delimited value in the tag stream.)
  - 重复的 `repeated` 字段通常附加到字段的 API 表示形式。（请注意，序列化一个打包的重复字段只会在标记流中产生一个长度分隔值。）
  - Duplicate `optional` field values follow the rule that “the last one wins.”
  - 重复的 `optional` 字段值遵循“最后一个获胜”的规则。
  
- `oneof` fields expose the API-level invariant that only one field is set at a time. However, the wire format may include multiple (tag, value) pairs which notionally belong to the `oneof`. Similar to `optional` fields, the generated API follows the “last one wins” rule.
  
- `oneof` 字段公开 API 级不变性，即一次只能设置一个字段。但是，线格式可能包含多个名义上属于 `oneof` 的 (tag, value) 对。与 `optional` 字段类似，生成的 API 遵循“最后一个获胜”规则。
  
- Out-of-range values are not returned for enum fields in generated proto2 APIs. However, out-of-range values may be stored as *unknown fields* in the API, even though the wire-format tag was recognized.
  
- 在生成的 proto2 API 中，枚举字段不会返回超出范围的值。但是，即使识别出线格式标记，超出范围的值也可能作为未知字段存储在 API 中。

###  命名字段映射格式中的存在 Presence in *Named-field Mapping* Formats

Protobufs can be represented in human-readable, textual forms. Two notable formats are TextFormat (the output format produced by generated message `DebugString` methods) and JSON.

​	Protobufs 可以用人类可读的文本形式表示。两种值得注意的格式是 TextFormat（由生成的 message `DebugString` 方法生成的输出格式）和 JSON。

These formats have correctness requirements of their own, and are generally stricter than *tagged-value stream* formats. However, TextFormat more closely mimics the semantics of the wire format, and does, in certain cases, provide similar semantics (for example, appending repeated name-value mappings to a repeated field). In particular, similar to the wire format, TextFormat only includes fields which are present.

​	这些格式有其自身的正确性要求，并且通常比标记值流格式更严格。但是，TextFormat 更接近于线格式的语义，并且在某些情况下确实提供了类似的语义（例如，将重复的名称-值映射追加到重复字段）。特别是，与线格式类似，TextFormat 仅包含存在字段。

JSON is a much stricter format, however, and cannot validly represent some semantics of the wire format or TextFormat.

​	JSON 是一种更严格的格式，但无法有效地表示线格式或 TextFormat 的某些语义。

- Notably, JSON *elements* are semantically unordered, and each member must have a unique name. This is different from TextFormat rules for repeated fields.
  
- 值得注意的是，JSON 元素在语义上是无序的，每个成员都必须具有唯一名称。这不同于 TextFormat 对重复字段的规则。
  
- JSON may include fields that are “not present,” unlike the no presence discipline for other formats:

- 
JSON 可以包含“不存在”的字段，这与其他格式的无存在原则不同：
  
- JSON defines a `null` value, which may be used to represent a *defined but not-present field*.
  - JSON 定义了一个 `null` 值，该值可用于表示已定义但不存在的字段。
- Repeated field values may be included in the formatted output, even if they are equal to the default (an empty list).
  - 即使重复字段值等于默认值（空列表），也可以将其包含在格式化输出中。

- Because JSON elements are unordered, there is no way to unambiguously interpret the “last one wins” rule.

- 
  由于 JSON 元素是无序的，因此无法明确解释“最后一个获胜”规则。
  
  - In most cases, this is fine: JSON elements must have unique names: repeated field values are not valid JSON, so they do not need to be resolved as they are for TextFormat.
  - 在大多数情况下，这是可以的：JSON 元素必须具有唯一名称：重复字段值不是有效的 JSON，因此无需像 TextFormat 那样解析它们。
  - However, this means that it may not be possible to interpret `oneof` fields unambiguously: if multiple cases are present, they are unordered.
  - 但是，这意味着可能无法明确解释 `oneof` 字段：如果存在多个案例，则它们是无序的。

In theory, JSON *can* represent presence in a semantic-preserving fashion. In practice, however, presence correctness can vary depending upon implementation choices, especially if JSON was chosen as a means to interoperate with clients not using protobufs.

​	理论上，JSON 可以以保留语义的方式表示存在。然而，在实践中，存在正确性可能会因实现选择而异，特别是如果选择 JSON 作为与不使用 protobuf 的客户端进行互操作的手段。

### Proto2 API 中的存在 Presence in Proto2 APIs 

This table outlines whether presence is tracked for fields in proto2 APIs (both for generated APIs and using dynamic reflection):

​	此表概述了在 proto2 API 中是否跟踪字段的存在（对于生成的 API 和使用动态反射）：

| Field type 字段类型                                          | Explicit Presence 显式存在 |
| ------------------------------------------------------------ | -------------------------- |
| Singular numeric (integer or floating point) 单数数字（整数或浮点数） | ✔️                          |
| Singular enum 单数枚举                                       | ✔️                          |
| Singular string or bytes 单数字符串或字节                    | ✔️                          |
| Singular message 单数消息                                    | ✔️                          |
| Repeated 重复                                                |                            |
| Oneofs                                                       | ✔️                          |
| Maps 映射                                                    |                            |

Singular fields (of all types) track presence explicitly in the generated API. The generated message interface includes methods to query presence of fields. For example, the field `foo` has a corresponding `has_foo` method. (The specific name follows the same language-specific naming convention as the field accessors.) These methods are sometimes referred to as “hazzers” within the protobuf implementation.

​	所有类型的单数字段在生成的 API 中明确跟踪存在性。生成的 message 接口包含用于查询字段存在性的方法。例如，字段 `foo` 有一个对应的 `has_foo` 方法。（具体名称遵循与字段访问器相同的语言特定命名约定。）这些方法在 protobuf 实现中有时称为“hazzers”。

Similar to singular fields, `oneof` fields explicitly track which one of the members, if any, contains a value. For example, consider this example `oneof`:

​	与单数字段类似， `oneof` 字段明确跟踪其中哪个成员（如果有）包含值。例如，考虑此示例 `oneof` ：

```protobuf
oneof foo {
  int32 a = 1;
  float b = 2;
}
```

Depending on the target language, the generated API would generally include several methods:

​	根据目标语言，生成的 API 通常会包含几种方法：

- A hazzer for the oneof: `has_foo`
- oneof 的 hazzer： `has_foo`
- A *oneof case* method: `foo`
- oneof case 方法： `foo`
- Hazzers for the members: `has_a`, `has_b`
- 成员的 Hazzers： `has_a` , `has_b`
- Getters for the members: `a`, `b`
- 成员的 Getters： `a` , `b`

Repeated fields and maps do not track presence: there is no distinction between an *empty* and a *not-present* repeated field.

​	重复字段和映射不跟踪存在：空重复字段和不存在的重复字段之间没有区别。

### Proto3 API 中的存在 Presence in Proto3 APIs 

This table outlines whether presence is tracked for fields in proto3 APIs (both for generated APIs and using dynamic reflection):

​	此表概述了 proto3 API 中的字段是否存在跟踪（对于生成的 API 和使用动态反射）：

| Field type 字段类型                                          | `optional` | Explicit Presence 显式存在 |
| ------------------------------------------------------------ | ---------- | -------------------------- |
| Singular numeric (integer or floating point) 单数数字（整数或浮点数） | No         |                            |
| Singular enum 单数枚举                                       | No         |                            |
| Singular string or bytes 单数字符串或字节                    | No         |                            |
| Singular numeric (integer or floating point) 单数数字（整数或浮点数） | Yes        | ✔️                          |
| Singular enum 单数枚举                                       | Yes        | ✔️                          |
| Singular string or bytes 单数字符串或字节                    | Yes        | ✔️                          |
| Singular message 单数消息                                    | Yes        | ✔️                          |
| Singular message 单数消息                                    | No         | ✔️                          |
| Repeated                                                     | N/A        |                            |
| Oneofs                                                       | N/A        | ✔️                          |
| Maps 映射                                                    | N/A        |                            |

Similar to proto2 APIs, proto3 does not track presence explicitly for repeated fields. Without the `optional` label, proto3 APIs do not track presence for basic types (numeric, string, bytes, and enums), either. Oneof fields affirmatively expose presence, although the same set of hazzer methods may not generated as in proto2 APIs.

​	与 proto2 API 类似，proto3 不会明确跟踪 repeated 字段的存在。如果没有 `optional` 标签，proto3 API 也不会跟踪基本类型（数字、字符串、字节和枚举）的存在。Oneof 字段明确地公开存在，尽管可能不会像在 proto2 API 中那样生成相同的 hazzer 方法集。

Under the *no presence* discipline, the default value is synonymous with “not present” for purposes of serialization. To notionally “clear” a field (so it won’t be serialized), an API user would set it to the default value.

​	在没有存在的情况下，默认值与“不存在”同义，用于序列化。为了从概念上“清除”字段（因此不会序列化），API 用户会将其设置为默认值。

The default value for enum-typed fields under *no presence* is the corresponding 0-valued enumerator. Under proto3 syntax rules, all enum types are required to have an enumerator value which maps to 0. By convention, this is an `UNKNOWN` or similarly-named enumerator. If the zero value is notionally outside the domain of valid values for the application, this behavior can be thought of as tantamount to *explicit presence*.

​	在没有存在的情况下，枚举类型字段的默认值是对应的 0 值枚举器。根据 proto3 语法规则，所有枚举类型都必须具有映射到 0 的枚举器值。根据惯例，这是一个 `UNKNOWN` 或类似命名的枚举器。如果零值在概念上超出了应用程序的有效值域，则可以将此行为视为等同于显式存在。

## 语义差异 Semantic Differences 

The *no presence* serialization discipline results in visible differences from the *explicit presence* tracking discipline, when the default value is set. For a singular field with numeric, enum, or string type:

​	当设置默认值时，无表示序列化规则会导致与显式表示跟踪规则可见的差异。对于具有数字、枚举或字符串类型的单数字段：

- No presence discipline:

- 
  无表示规则：

  - Default values are not serialized.
    
  - 默认值不会序列化。
    
  - Default values are *not* merged-from.
    
  - 默认值不会合并。
    
  - To “clear” a field, it is set to its default value.
    
  - 要“清除”字段，请将其设置为其默认值。
    
  - The default value may mean:

  - 
    默认值可能意味着：
    
    - the field was explicitly set to its default value, which is valid in the application-specific domain of values;
    - 字段已明确设置为其默认值，这在特定于应用程序的值域中有效；
    - the field was notionally “cleared” by setting its default; or
    - 通过设置其默认值在概念上“清除”了该字段；或
    - the field was never set.
    - 从未设置该字段。

- Explicit presence discipline:

- 
  显式表示规则：
  
  - Explicitly set values are always serialized, including default values.
  - 显式设置的值总是被序列化，包括默认值。
  - Un-set fields are never merged-from.
  - 未设置的字段永远不会被合并。
  - Explicitly set fields – including default values – *are* merged-from.
  - 显式设置的字段（包括默认值）会被合并。
  - A generated `has_foo` method indicates whether or not the field `foo` has been set (and not cleared).
  - 生成的 `has_foo` 方法指示字段 `foo` 是否已设置（且未清除）。
  - A generated `clear_foo` method must be used to clear (i.e., un-set) the value.
  - 必须使用生成的 `clear_foo` 方法来清除（即取消设置）该值。

### 合并注意事项 Considerations for Merging 

Under the *no presence* rules, it is effectively impossible for a target field to merge-from its default value (using the protobuf’s API merging functions). This is because default values are skipped, similar to the *no presence* serialization discipline. Merging only updates the target (merged-to) message using the non-skipped values from the update (merged-from) message.

​	根据无存在规则，目标字段实际上不可能从其默认值合并（使用 protobuf 的 API 合并函数）。这是因为默认值被跳过，类似于无存在序列化规则。合并仅使用更新（合并自）消息中的非跳过值来更新目标（合并到）消息。

The difference in merging behavior has further implications for protocols which rely on partial “patch” updates. If field presence is not tracked, then an update patch alone cannot represent an update to the default value, because only non-default values are merged-from.

​	合并行为的差异对依赖于部分“修补程序”更新的协议有进一步的影响。如果未跟踪字段存在，则更新修补程序本身无法表示对默认值的更新，因为只有非默认值才会被合并。

Updating to set a default value in this case requires some external mechanism, such as `FieldMask`. However, if presence *is* tracked, then all explicitly-set values – even default values – will be merged into the target.

​	在这种情况下，更新以设置默认值需要一些外部机制，例如 `FieldMask` 。但是，如果跟踪存在，则所有显式设置的值（甚至是默认值）都将合并到目标中。

### 考虑更改兼容性 Considerations for change-compatibility 

Changing a field between *explicit presence* and *no presence* is a binary-compatible change for serialized values in wire format. However, the serialized representation of the message may differ, depending on which version of the message definition was used for serialization. Specifically, when a “sender” explicitly sets a field to its default value:

​	在显式存在和不存在之间更改字段是线格式中序列化值的二进制兼容更改。但是，消息的序列化表示形式可能会有所不同，具体取决于用于序列化的消息定义的版本。具体来说，当“发送者”将字段显式设置为其默认值时：

- The serialized value following *no presence* discipline does not contain the default value, even though it was explicitly set.
- 遵循无存在规则的序列化值不包含默认值，即使该值已显式设置。
- The serialized value following *explicit presence* discipline contains every “present” field, even if it contains the default value.
- 遵循显式存在规则的序列化值包含每个“存在”字段，即使它包含默认值。

This change may or may not be safe, depending on the application’s semantics. For example, consider two clients with different versions of a message definition.

​	此更改可能是安全的，也可能不安全，具体取决于应用程序的语义。例如，考虑两个具有不同版本的消息定义的客户端。

Client A uses this definition of the message, which follows the *explicit presence* serialization discipline for field `foo`:

​	客户端 A 使用此消息定义，该定义遵循字段 `foo` 的显式存在序列化规则：

```protobuf
syntax = "proto3";
message Msg {
  optional int32 foo = 1;
}
```

Client B uses a definition of the same message, except that it follows the *no presence* discipline:

​	客户端 B 使用相同消息的定义，但它遵循无表示原则：

```protobuf
syntax = "proto3";
message Msg {
  int32 foo = 1;
}
```

Now, consider a scenario where client A observes `foo`’s presence as the clients repeatedly exchange the “same” message by deserializing and reserializing:

​	现在，考虑一种场景，其中客户端 A 观察 `foo` 的表示，因为客户端通过反序列化和重新序列化反复交换“相同”消息：

```protobuf
// Client A:
Msg m_a;
m_a.set_foo(1);                  // non-default value
assert(m_a.has_foo());           // OK
Send(m_a.SerializeAsString());   // to client B

// Client B:
Msg m_b;
m_b.ParseFromString(Receive());  // from client A
assert(m_b.foo() == 1);          // OK
Send(m_b.SerializeAsString());   // to client A

// Client A:
m_a.ParseFromString(Receive());  // from client B
assert(m_a.foo() == 1);          // OK
assert(m_a.has_foo());           // OK
m_a.set_foo(0);                  // default value
Send(m_a.SerializeAsString());   // to client B

// Client B:
Msg m_b;
m_b.ParseFromString(Receive());  // from client A
assert(m_b.foo() == 0);          // OK
Send(m_b.SerializeAsString());   // to client A

// Client A:
m_a.ParseFromString(Receive());  // from client B
assert(m_a.foo() == 0);          // OK
assert(m_a.has_foo());           // FAIL
```

If client A depends on *explicit presence* for `foo`, then a “round trip” through client B will be lossy from the perspective of client A. In the example, this is not a safe change: client A requires (by `assert`) that the field is present; even without any modifications through the API, that requirement fails in a value- and peer-dependent case.

​	如果客户端 A 依赖 `foo` 的显式表示，那么从客户端 A 的角度来看，通过客户端 B 的“往返”将会有所损失。在此示例中，这不是一个安全的更改：客户端 A 要求（通过 `assert` ）该字段存在；即使不通过 API 进行任何修改，该要求也会在值和对等依赖的情况下失败。

## 如何在 Proto3 中启用显式表示 How to Enable *Explicit Presence* in Proto3 

These are the general steps to use field tracking support for proto3:

​	以下是使用字段跟踪支持的常规步骤适用于 proto3：

1. Add an `optional` field to a `.proto` file.
2. 将 `optional` 字段添加到 `.proto` 文件。
3. Run `protoc` (at least v3.15, or v3.12 using `--experimental_allow_proto3_optional` flag).
4. 运行 `protoc` （至少 v3.15 或使用 `--experimental_allow_proto3_optional` 标志的 v3.12）。
5. Use the generated “hazzer” methods and “clear” methods in application code, instead of comparing or setting default values.
6. 在应用程序代码中使用生成的“hazzer”方法和“clear”方法，而不是比较或设置默认值。

### `.proto` 文件更改 `.proto` File Changes 

This is an example of a proto3 message with fields which follow both *no presence* and *explicit presence* semantics:

​	这是一个 proto3 消息的示例，其中字段遵循无表示和显式表示语义：

```protobuf
syntax = "proto3";
package example;

message MyMessage {
  // No presence:
  int32 not_tracked = 1;

  // Explicit presence:
  optional int32 tracked = 2;
}
```

### `protoc` 调用 `protoc` Invocation 

Presence tracking for proto3 messages is enabled by default [since v3.15.0](https://github.com/protocolbuffers/protobuf/releases/tag/v3.15.0) release, formerly up until [v3.12.0](https://github.com/protocolbuffers/protobuf/releases/tag/v3.12.0) the `--experimental_allow_proto3_optional` flag was required when using presence tracking with protoc.

​	自 v3.15.0 版本发布以来，默认情况下启用了 proto3 消息的存在跟踪，以前在 v3.12.0 之前，在使用 protoc 进行存在跟踪时需要 `--experimental_allow_proto3_optional` 标志。

###  使用生成的代码 Using the Generated Code

The generated code for proto3 fields with *explicit presence* (the `optional` label) will be the same as it would be in a proto2 file.

​	具有显式存在（ `optional` 标签）的 proto3 字段的生成代码将与 proto2 文件中的代码相同。

This is the definition used in the “no presence” examples below:

​	这是在下面的“无存在”示例中使用的定义：

```protobuf
syntax = "proto3";
package example;
message Msg {
  int32 foo = 1;
}
```

This is the definition used in the “explicit presence” examples below:

​	这是在下面的“显式存在”示例中使用的定义：

```protobuf
syntax = "proto3";
package example;
message Msg {
  optional int32 foo = 1;
}
```

In the examples, a function `GetProto` constructs and returns a message of type `Msg` with unspecified contents.

​	在示例中，函数 `GetProto` 构造并返回类型为 `Msg` 的消息，内容未指定。

#### C++ 示例 C++ Example 

No presence:

​	无存在：

```c++
Msg m = GetProto();
if (m.foo() != 0) {
  // "Clear" the field:
  m.set_foo(0);
} else {
  // Default value: field may not have been present.
  m.set_foo(1);
}
```

Explicit presence:

​	显式存在：

```c++
Msg m = GetProto();
if (m.has_foo()) {
  // Clear the field:
  m.clear_foo();
} else {
  // Field is not present, so set it.
  m.set_foo(1);
}
```

#### C# 示例 C# Example 

No presence:

​	无存在：

```c#
var m = GetProto();
if (m.Foo != 0) {
  // "Clear" the field:
  m.Foo = 0;
} else {
  // Default value: field may not have been present.
  m.Foo = 1;
}
```

Explicit presence:

​	显式存在：

```c#
var m = GetProto();
if (m.HasFoo) {
  // Clear the field:
  m.ClearFoo();
} else {
  // Field is not present, so set it.
  m.Foo = 1;
}
```

#### Go 示例 Go Example 

No presence:

​	无存在：

```go
m := GetProto()
if m.Foo != 0 {
  // "Clear" the field:
  m.Foo = 0
} else {
  // Default value: field may not have been present.
  m.Foo = 1
}
```

Explicit presence:

​	显式存在：

```go
m := GetProto()
if m.Foo != nil {
  // Clear the field:
  m.Foo = nil
} else {
  // Field is not present, so set it.
  m.Foo = proto.Int32(1)
}
```

#### Java Example Java 示例

These examples use a `Builder` to demonstrate clearing. Simply checking presence and getting values from a `Builder` follows the same API as the message type.

​	这些示例使用 `Builder` 来演示清除。只需检查存在并从 `Builder` 获取值，即可遵循与消息类型相同的 API。

No presence:

​	无存在：

```java
Msg.Builder m = GetProto().toBuilder();
if (m.getFoo() != 0) {
  // "Clear" the field:
  m.setFoo(0);
} else {
  // Default value: field may not have been present.
  m.setFoo(1);
}
```

Explicit presence:

​	显式存在：

```java
Msg.Builder m = GetProto().toBuilder();
if (m.hasFoo()) {
  // Clear the field:
  m.clearFoo()
} else {
  // Field is not present, so set it.
  m.setFoo(1);
}
```

#### Python 示例 Python Example 

No presence:

​	无存在：

```python
m = example.Msg()
if m.foo != 0:
  # "Clear" the field:
  m.foo = 0
else:
  # Default value: field may not have been present.
  m.foo = 1
```

Explicit presence:

​	显式存在：

```python
m = example.Msg()
if m.HasField('foo'):
  # Clear the field:
  m.ClearField('foo')
else:
  # Field is not present, so set it.
  m.foo = 1
```

#### Ruby 示例 Ruby Example 

No presence:

​	无存在：

```ruby
m = Msg.new
if m.foo != 0
  # "Clear" the field:
  m.foo = 0
else
  # Default value: field may not have been present.
  m.foo = 1
end
```

Explicit presence:

​	显式存在：

```ruby
m = Msg.new
if m.has_foo?
  # Clear the field:
  m.clear_foo
else
  # Field is not present, so set it.
  m.foo = 1
end
```

#### Javascript 示例 Javascript Example 

No presence:

​	无存在：

```js
var m = new Msg();
if (m.getFoo() != 0) {
  // "Clear" the field:
  m.setFoo(0);
} else {
  // Default value: field may not have been present.
  m.setFoo(1);
}
```

Explicit presence:

​	显式存在：

```js
var m = new Msg();
if (m.hasFoo()) {
  // Clear the field:
  m.clearFoo()
} else {
  // Field is not present, so set it.
  m.setFoo(1);
}
```

#### Objective-C 示例 Objective-C Example 

No presence:

​	无状态：

```objective-c
Msg *m = [[Msg alloc] init];
if (m.foo != 0) {
  // "Clear" the field:
  m.foo = 0;
} else {
  // Default value: field may not have been present.
  m.foo = 1;
}
```

Explicit presence:

​	显式状态：

```objective-c
Msg *m = [[Msg alloc] init];
if (m.hasFoo()) {
  // Clear the field:
  [m clearFoo];
} else {
  // Field is not present, so set it.
  [m setFoo:1];
}
```

## 备忘单 Cheat sheet 

**Proto2:**

Is field presence tracked?
是否跟踪字段状态？

| Field type 字段类型                 | Tracked? 是否跟踪？ |
| ----------------------------------- | ------------------- |
| Singular field 单数字段             | yes                 |
| Singular message field 单数消息字段 | yes                 |
| Field in a oneof oneof 中的字段     | yes                 |
| Repeated field & map 重复字段和映射 | no                  |

**Proto3:**

Is field presence tracked?

​	是否跟踪字段状态？

| Field type 字段类型                 | Tracked? 是否跟踪？                            |
| ----------------------------------- | ---------------------------------------------- |
| Singular message field 单一消息字段 | yes                                            |
| Field in a oneof oneof 中的字段     | yes                                            |
| *Other* singular field 其他单一字段 | if defined as `optional` 如果定义为 `optional` |
| Repeated field & map 重复字段和映射 | no                                             |

Edition 2023:

Is field presence tracked?

​	是否跟踪字段状态？

| Field type 字段类型                                          | Tracked? 是否跟踪？ |
| ------------------------------------------------------------ | ------------------- |
| Default 默认值                                               | yes                 |
| `features.field_presence` set to `LEGACY_REQUIRED` `features.field_presence` 设置为 `LEGACY_REQUIRED` | yes                 |
| `features.field_presence` set to `IMPLICIT` `features.field_presence` 设置为 `IMPLICIT` | no                  |
| Repeated field & map 重复字段和映射                          | no                  |
