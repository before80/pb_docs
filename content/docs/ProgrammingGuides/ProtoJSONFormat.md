+++
title = "ProtoJSON Format"
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

# ProtoJSON Format

Covers how to use the Protobuf to JSON conversion utilities.



Protobuf supports a canonical encoding in JSON, making it easier to share data with systems that do not support the standard protobuf binary wire format.

ProtoJSON Format is not as efficient as protobuf wire format. The converter uses more CPU to encode and decode messages and (except in rare cases) encoded messages consume more space. Furthermore, ProtoJSON format puts your field and enum value names into encoded messages making it much harder to change those names later. Removing fields is a breaking change that will trigger a parsing error. In short, there are many good reasons why Google prefers to use the standard wire format for virtually everything rather than ProtoJSON format.

The encoding is described on a type-by-type basis in the table later in this topic.

When parsing JSON-encoded data into a protocol buffer, if a value is missing or if its value is `null`, it will be interpreted as the corresponding [default value](https://protobuf.dev/programming-guides/editions#default).

When generating JSON-encoded output from a protocol buffer, if a protobuf field has the default value and if the field doesn’t support field presence, it will be omitted from the output by default. An implementation may provide options to include fields with default values in the output.

Fields that have a value set and that support field presence always include the field value in the JSON-encoded output, even if it is the default value. For example, a proto3 field that is defined with the `optional` keyword supports field presence and if set, will always appear in the JSON output. A message type field in any edition of protobuf supports field presence and if set will appear in the output. Proto3 implicit-presence scalar fields will only appear in the JSON output if they are not set to the default value for that type.

| Protobuf               | JSON          | JSON example                                | Notes                                                        |
| ---------------------- | ------------- | ------------------------------------------- | ------------------------------------------------------------ |
| message                | object        | `{"fooBar": v, "g": null, ...}`             | Generates JSON objects. Message field names are mapped to lowerCamelCase and become JSON object keys. If the `json_name` field option is specified, the specified value will be used as the key instead. Parsers accept both the lowerCamelCase name (or the one specified by the `json_name` option) and the original proto field name. `null` is an accepted value for all field types and treated as the default value of the corresponding field type. However, `null` cannot be used for the `json_name` value. For more on why, see [Stricter validation for json_name](https://protobuf.dev/news/2023-04-28#json-name). |
| enum                   | string        | `"FOO_BAR"`                                 | The name of the enum value as specified in proto is used. Parsers accept both enum names and integer values. |
| map<K,V>               | object        | `{"k": v, ...}`                             | All keys are converted to strings.                           |
| repeated V             | array         | `[v, ...]`                                  | `null` is accepted as the empty list `[]`.                   |
| bool                   | true, false   | `true, false`                               |                                                              |
| string                 | string        | `"Hello World!"`                            |                                                              |
| bytes                  | base64 string | `"YWJjMTIzIT8kKiYoKSctPUB+"`                | JSON value will be the data encoded as a string using standard base64 encoding with paddings. Either standard or URL-safe base64 encoding with/without paddings are accepted. |
| int32, fixed32, uint32 | number        | `1, -10, 0`                                 | JSON value will be a decimal number. Either numbers or strings are accepted. Empty strings are invalid. |
| int64, fixed64, uint64 | string        | `"1", "-10"`                                | JSON value will be a decimal string. Either numbers or strings are accepted. Empty strings are invalid. |
| float, double          | number        | `1.1, -10.0, 0, "NaN", "Infinity"`          | JSON value will be a number or one of the special string values "NaN", "Infinity", and "-Infinity". Either numbers or strings are accepted. Empty strings are invalid. Exponent notation is also accepted. |
| Any                    | `object`      | `{"@type": "url", "f": v, ... }`            | If the `Any` contains a value that has a special JSON mapping, it will be converted as follows: `{"@type": xxx, "value": yyy}`. Otherwise, the value will be converted into a JSON object, and the `"@type"` field will be inserted to indicate the actual data type. |
| Timestamp              | string        | `"1972-01-01T10:00:20.021Z"`                | Uses RFC 3339, where generated output will always be Z-normalized and uses 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. |
| Duration               | string        | `"1.000340012s", "1s"`                      | Generated output always contains 0, 3, 6, or 9 fractional digits, depending on required precision, followed by the suffix "s". Accepted are any fractional digits (also none) as long as they fit into nano-seconds precision and the suffix "s" is required. |
| Struct                 | `object`      | `{ ... }`                                   | Any JSON object. See `struct.proto`.                         |
| Wrapper types          | various types | `2, "2", "foo", true, "true", null, 0, ...` | Wrappers use the same representation in JSON as the wrapped primitive type, except that `null` is allowed and preserved during data conversion and transfer. |
| FieldMask              | string        | `"f.fooBar,h"`                              | See `field_mask.proto`.                                      |
| ListValue              | array         | `[foo, bar, ...]`                           |                                                              |
| Value                  | value         |                                             | Any JSON value. Check [google.protobuf.Value](https://protobuf.dev/reference/protobuf/google.protobuf#value) for details. |
| NullValue              | null          |                                             | JSON null                                                    |
| Empty                  | object        | `{}`                                        | An empty JSON object                                         |

### JSON Options

A conformant protobuf JSON implementation may provide the following options:

- **Always emit fields without presence**: Fields that don’t support presence and that have their default value are omitted by default in JSON output (for example, an implicit presence integer with a 0 value, implicit presence string fields that are empty strings, and empty repeated and map fields). An implementation may provide an option to override this behavior and output fields with their default values.

  As of v25.x, the C++, Java, and Python implementations are nonconformant, as this flag affects proto2 `optional` fields but not proto3 `optional` fields. A fix is planned for a future release.

- **Ignore unknown fields**: The protobuf JSON parser should reject unknown fields by default but may provide an option to ignore unknown fields in parsing.

- **Use proto field name instead of lowerCamelCase name**: By default the protobuf JSON printer should convert the field name to lowerCamelCase and use that as the JSON name. An implementation may provide an option to use proto field name as the JSON name instead. Protobuf JSON parsers are required to accept both the converted lowerCamelCase name and the proto field name.

- **Emit enum values as integers instead of strings**: The name of an enum value is used by default in JSON output. An option may be provided to use the numeric value of the enum value instead.
