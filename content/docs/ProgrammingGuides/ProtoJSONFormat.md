+++
title = "ProtoJSON 格式"
date = 2024-11-17T09:35:36+08:00
weight = 70
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

> 原文：[https://protobuf.dev/programming-guides/json/](https://protobuf.dev/programming-guides/json/)
>
> 收录该文档的时间：`2024-11-17T09:35:36+08:00`

# ProtoJSON Format - ProtoJSON 格式

Covers how to use the Protobuf to JSON conversion utilities.

​	介绍如何使用 Protobuf 到 JSON 的转换工具。

Protobuf supports a canonical encoding in JSON, making it easier to share data with systems that do not support the standard protobuf binary wire format.

​	Protobuf 支持 JSON 的规范化编码，这使得与不支持标准 Protobuf 二进制线格式的系统共享数据变得更容易。

ProtoJSON Format is not as efficient as protobuf wire format. The converter uses more CPU to encode and decode messages and (except in rare cases) encoded messages consume more space. Furthermore, ProtoJSON format puts your field and enum value names into encoded messages making it much harder to change those names later. Removing fields is a breaking change that will trigger a parsing error. In short, there are many good reasons why Google prefers to use the standard wire format for virtually everything rather than ProtoJSON format.

​	ProtoJSON 格式的效率不如 Protobuf 线格式。转换器在编码和解码消息时会消耗更多的 CPU，并且（除非在极少数情况下）编码的消息占用的空间更大。此外，ProtoJSON 格式会将字段名和枚举值名包含在编码的消息中，这使得后续更改这些名称变得更加困难。删除字段是一种破坏性更改，会触发解析错误。简而言之，这些原因使得 Google 更倾向于在几乎所有情况下使用标准线格式，而不是 ProtoJSON 格式。

The encoding is described on a type-by-type basis in the table later in this topic.

​	编码在本主题后面的表格中按类型逐一描述。

When parsing JSON-encoded data into a protocol buffer, if a value is missing or if its value is `null`, it will be interpreted as the corresponding [default value]({{< ref "/docs/ProgrammingGuides/LanguageGuideeditions#default" >}}).

​	在将 JSON 编码数据解析为协议缓冲区时，如果缺少值或其值为 `null`，它将被解释为对应的[默认值]({{< ref "/docs/ProgrammingGuides/LanguageGuideeditions#default" >}})。

When generating JSON-encoded output from a protocol buffer, if a protobuf field has the default value and if the field doesn’t support field presence, it will be omitted from the output by default. An implementation may provide options to include fields with default values in the output.

​	在从协议缓冲区生成 JSON 编码的输出时，如果某个 Protobuf 字段具有默认值且不支持字段存在性，则默认情况下该字段将在输出中被省略。实现可以提供选项以在输出中包含具有默认值的字段。

Fields that have a value set and that support field presence always include the field value in the JSON-encoded output, even if it is the default value. For example, a proto3 field that is defined with the `optional` keyword supports field presence and if set, will always appear in the JSON output. A message type field in any edition of protobuf supports field presence and if set will appear in the output. Proto3 implicit-presence scalar fields will only appear in the JSON output if they are not set to the default value for that type.

​	设置了值且支持字段存在性的字段总是在 JSON 编码输出中包含字段值，即使它是默认值。例如，使用 `optional` 关键字定义的 proto3 字段支持字段存在性，如果被设置，将始终出现在 JSON 输出中。任何版本的 Protobuf 中的消息类型字段都支持字段存在性，如果设置，将出现在输出中。Proto3 的隐式存在性标量字段仅在未设置为该类型的默认值时出现在 JSON 输出中。

| Protobuf               | JSON          | JSON example                                | Notes                                                        |
| ---------------------- | ------------- | ------------------------------------------- | ------------------------------------------------------------ |
| message                | object        | `{"fooBar": v, "g": null, ...}`             | Generates JSON objects. Message field names are mapped to lowerCamelCase and become JSON object keys. If the `json_name` field option is specified, the specified value will be used as the key instead. Parsers accept both the lowerCamelCase name (or the one specified by the `json_name` option) and the original proto field name. `null` is an accepted value for all field types and treated as the default value of the corresponding field type. However, `null` cannot be used for the `json_name` value. For more on why, see [Stricter validation for json_name](https://protobuf.dev/news/2023-04-28#json-name).生成 JSON 对象。消息字段名称映射为小驼峰式命名并成为 JSON 对象的键。如果指定了 `json_name` 字段选项，则使用指定的值作为键。解析器同时接受小驼峰式名称（或 `json_name` 选项指定的名称）和原始 proto 字段名。`null` 是所有字段类型的接受值，并被视为相应字段类型的默认值。然而，`null` 不能用作 `json_name` 的值，具体原因请参阅 [json_name 更严格的验证](https://protobuf.dev/news/2023-04-28#json-name)。 |
| enum                   | string        | `"FOO_BAR"`                                 | The name of the enum value as specified in proto is used. Parsers accept both enum names and integer values. 使用 proto 中指定的枚举值名称。解析器同时接受枚举名称和整数值。 |
| map<K,V>               | object        | `{"k": v, ...}`                             | All keys are converted to strings. 所有键都被转换为字符串。  |
| repeated V             | array         | `[v, ...]`                                  | `null` is accepted as the empty list `[]`. `null` 被接受为空列表 `[]`。 |
| bool                   | true, false   | `true, false`                               |                                                              |
| string                 | string        | `"Hello World!"`                            |                                                              |
| bytes                  | base64 string | `"YWJjMTIzIT8kKiYoKSctPUB+"`                | JSON value will be the data encoded as a string using standard base64 encoding with paddings. Either standard or URL-safe base64 encoding with/without paddings are accepted. JSON 值将是使用标准 Base64 编码（带填充）编码的数据。标准或 URL 安全的 Base64 编码（带/不带填充）均被接受。 |
| int32, fixed32, uint32 | number        | `1, -10, 0`                                 | JSON value will be a decimal number. Either numbers or strings are accepted. Empty strings are invalid. JSON 值为十进制数字。数字或字符串均被接受。空字符串无效。 |
| int64, fixed64, uint64 | string        | `"1", "-10"`                                | JSON value will be a decimal string. Either numbers or strings are accepted. Empty strings are invalid. JSON 值为十进制字符串。数字或字符串均被接受。空字符串无效。 |
| float, double          | number        | `1.1, -10.0, 0, "NaN", "Infinity"`          | JSON value will be a number or one of the special string values "NaN", "Infinity", and "-Infinity". Either numbers or strings are accepted. Empty strings are invalid. Exponent notation is also accepted. JSON 值为数字或特殊字符串值 "NaN"、"Infinity" 和 "-Infinity"。数字或字符串均被接受。空字符串无效。指数表示法也被接受。 |
| Any                    | `object`      | `{"@type": "url", "f": v, ... }`            | If the `Any` contains a value that has a special JSON mapping, it will be converted as follows: `{"@type": xxx, "value": yyy}`. Otherwise, the value will be converted into a JSON object, and the `"@type"` field will be inserted to indicate the actual data type. 如果 `Any` 包含的值有特殊的 JSON 映射，它将被转换为 `{"@type": xxx, "value": yyy}`。否则，值将被转换为 JSON 对象，并插入 `"@type"` 字段以指示实际数据类型。 |
| Timestamp              | string        | `"1972-01-01T10:00:20.021Z"`                | Uses RFC 3339, where generated output will always be Z-normalized and uses 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. 使用 RFC 3339 格式，生成的输出始终为 Z 标准化，并使用 0、3、6 或 9 个小数位。接受非 "Z" 的偏移量。 |
| Duration               | string        | `"1.000340012s", "1s"`                      | Generated output always contains 0, 3, 6, or 9 fractional digits, depending on required precision, followed by the suffix "s". Accepted are any fractional digits (also none) as long as they fit into nano-seconds precision and the suffix "s" is required. 生成的输出始终包含 0、3、6 或 9 个小数位，具体取决于所需的精度，后缀为 "s"。接受任何小数位（包括无），但必须适合纳秒精度并且要求后缀为 "s"。 |
| Struct                 | `object`      | `{ ... }`                                   | Any JSON object. See `struct.proto`. 任意 JSON 对象。请参阅 `struct.proto`。 |
| Wrapper types          | various types | `2, "2", "foo", true, "true", null, 0, ...` | Wrappers use the same representation in JSON as the wrapped primitive type, except that `null` is allowed and preserved during data conversion and transfer. Wrapper 使用与包装的基本类型相同的 JSON 表现形式，除了允许和保留 `null` 以用于数据转换和传输。 |
| FieldMask              | string        | `"f.fooBar,h"`                              | See `field_mask.proto`. 请参阅 `field_mask.proto`。          |
| ListValue              | array         | `[foo, bar, ...]`                           |                                                              |
| Value                  | value         |                                             | Any JSON value. Check [google.protobuf.Value]({{< ref "/docs/ReferenceGuides/ProtocolBuffers/Well-KnownTypes#value" >}}) for details. 任意 JSON 值。请查看 [google.protobuf.Value]({{< ref "/docs/ReferenceGuides/ProtocolBuffers/Well-KnownTypes#value" >}}) 的详细信息。 |
| NullValue              | null          |                                             | JSON null                                                    |
| Empty                  | object        | `{}`                                        | An empty JSON object 一个空的 JSON 对象                      |

### JSON 选项 JSON Options

A conformant protobuf JSON implementation may provide the following options:

​	合规的 Protobuf JSON 实现可以提供以下选项：

- **Always emit fields without presence**: Fields that don’t support presence and that have their default value are omitted by default in JSON output (for example, an implicit presence integer with a 0 value, implicit presence string fields that are empty strings, and empty repeated and map fields). An implementation may provide an option to override this behavior and output fields with their default values.
  - **始终输出无存在性的字段**：默认情况下，在 JSON 输出中，未支持存在性且具有默认值的字段会被省略（例如，隐式存在性的整数值为 0，隐式存在性的字符串字段为空字符串，以及空的重复字段和映射字段）。实现可以提供选项来覆盖此行为，并输出具有默认值的字段。

​	As of v25.x, the C++, Java, and Python implementations are nonconformant, as this flag affects proto2 `optional` fields but not proto3 `optional` fields. A fix is planned for a future release.

​	从 v25.x 开始，C++、Java 和 Python 实现不合规，因为此标志会影响 proto2 的 `optional` 字段，但不会影响 proto3 的 `optional` 字段。计划在未来版本中修复。

- **Ignore unknown fields**: The protobuf JSON parser should reject unknown fields by default but may provide an option to ignore unknown fields in parsing.
  - **忽略未知字段**：默认情况下，Protobuf JSON 解析器应拒绝未知字段，但可以提供选项在解析中忽略未知字段。

- **Use proto field name instead of lowerCamelCase name**: By default the protobuf JSON printer should convert the field name to lowerCamelCase and use that as the JSON name. An implementation may provide an option to use proto field name as the JSON name instead. Protobuf JSON parsers are required to accept both the converted lowerCamelCase name and the proto field name.
  - **使用 proto 字段名而非小驼峰式名称**：默认情况下，Protobuf JSON 打印机应将字段名转换为小驼峰式并将其用作 JSON 名称。实现可以提供选项以使用 proto 字段名作为 JSON 名称。Protobuf JSON 解析器需要同时接受转换后的小驼峰式名称和 proto 字段名。

- **Emit enum values as integers instead of strings**: The name of an enum value is used by default in JSON output. An option may be provided to use the numeric value of the enum value instead.
  - **将枚举值作为整数而非字符串输出**：默认情况下，枚举值的名称在 JSON 输出中使用。可以提供选项以改用枚举值的数值。
