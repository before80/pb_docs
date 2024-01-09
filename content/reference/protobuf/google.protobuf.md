+++
title = "Protocol Buffers Well-Known Types"
weight = 830
linkTitle = "Well-Known Types"
description = "API documentation for the google.protobuf package."
type = "docs"
+++

# Protocol Buffers Well-Known Types Protocol Buffers 众所周知的类型

API documentation for the google.protobuf package.
google.protobuf 包的 API 文档。



## Index 索引

- [`Any`](https://protobuf.dev/reference/protobuf/google.protobuf/#any) (message)
  `Any` （消息）
- [`Api`](https://protobuf.dev/reference/protobuf/google.protobuf/#api) (message)
  `Api` （消息）
- [`BoolValue`](https://protobuf.dev/reference/protobuf/google.protobuf/#bool-value) (message)
  `BoolValue` （消息）
- [`BytesValue`](https://protobuf.dev/reference/protobuf/google.protobuf/#bytes-value) (message)
  `BytesValue` （消息）
- [`DoubleValue`](https://protobuf.dev/reference/protobuf/google.protobuf/#double-value) (message)
  `DoubleValue` （消息）
- [`Duration`](https://protobuf.dev/reference/protobuf/google.protobuf/#duration) (message)
  `Duration` （消息）
- [`Empty`](https://protobuf.dev/reference/protobuf/google.protobuf/#empty) (message)
  `Empty` （消息）
- [`Enum`](https://protobuf.dev/reference/protobuf/google.protobuf/#enum) (message)
  `Enum` （消息）
- [`EnumValue`](https://protobuf.dev/reference/protobuf/google.protobuf/#enum-value) (message)
  `EnumValue` （消息）
- [`Field`](https://protobuf.dev/reference/protobuf/google.protobuf/#field) (message)
  `Field` （消息）
- [`Field.Cardinality`](https://protobuf.dev/reference/protobuf/google.protobuf/#field-cardinality) (enum)
  `Field.Cardinality` （枚举）
- [`Field.Kind`](https://protobuf.dev/reference/protobuf/google.protobuf/#field-kind) (enum)
  `Field.Kind` （枚举）
- [`FieldMask`](https://protobuf.dev/reference/protobuf/google.protobuf/#field-mask) (message)
  `FieldMask` （消息）
- [`FloatValue`](https://protobuf.dev/reference/protobuf/google.protobuf/#float-value) (message)
  `FloatValue` （消息）
- [`Int32Value`](https://protobuf.dev/reference/protobuf/google.protobuf/#int32-value) (message)
  `Int32Value` （消息）
- [`Int64Value`](https://protobuf.dev/reference/protobuf/google.protobuf/#int64-value) (message)
  `Int64Value` （消息）
- [`ListValue`](https://protobuf.dev/reference/protobuf/google.protobuf/#list-value) (message)
  `ListValue` （消息）
- [`Method`](https://protobuf.dev/reference/protobuf/google.protobuf/#method) (message)
  `Method` （消息）
- [`Mixin`](https://protobuf.dev/reference/protobuf/google.protobuf/#mixin) (message)
  `Mixin` （消息）
- [`NullValue`](https://protobuf.dev/reference/protobuf/google.protobuf/#null-value) (enum)
  `NullValue` （枚举）
- [`Option`](https://protobuf.dev/reference/protobuf/google.protobuf/#option) (message)
  `Option` （消息）
- [`SourceContext`](https://protobuf.dev/reference/protobuf/google.protobuf/#source-context) (message)
  `SourceContext` （消息）
- [`StringValue`](https://protobuf.dev/reference/protobuf/google.protobuf/#string-value) (message)
  `StringValue` （消息）
- [`Struct`](https://protobuf.dev/reference/protobuf/google.protobuf/#struct) (message)
  `Struct` （消息）
- [`Syntax`](https://protobuf.dev/reference/protobuf/google.protobuf/#syntax) (enum)
  `Syntax` （枚举）
- [`Timestamp`](https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp) (message)
  `Timestamp` （消息）
- [`Type`](https://protobuf.dev/reference/protobuf/google.protobuf/#type) (message)
  `Type` （消息）
- [`UInt32Value`](https://protobuf.dev/reference/protobuf/google.protobuf/#uint32-value) (message)
  `UInt32Value` （消息）
- [`UInt64Value`](https://protobuf.dev/reference/protobuf/google.protobuf/#uint64-value) (message)
  `UInt64Value` （消息）
- [`Value`](https://protobuf.dev/reference/protobuf/google.protobuf/#value) (message)
  `Value` （消息）

## Any

`Any` contains an arbitrary serialized message along with a URL that describes the type of the serialized message.
`Any` 包含一个任意序列化的消息以及一个描述序列化消息类型的 URL。

#### JSON

The JSON representation of an `Any` value uses the regular representation of the deserialized, embedded message, with an additional field `@type` which contains the type URL. Example:
`Any` 值的 JSON 表示使用反序列化、嵌入式消息的常规表示，并带有包含类型 URL 的附加字段 `@type` 。示例：

```proto
package google.profile;
message Person {
  string first_name = 1;
  string last_name = 2;
}
{
  "@type": "type.googleapis.com/google.profile.Person",
  "firstName": <string>,
  "lastName": <string>
}
```

If the embedded message type is well-known and has a custom JSON representation, that representation will be embedded adding a field `value` which holds the custom JSON in addition to the `@type` field. Example (for message `google.protobuf.Duration`):
如果嵌入式消息类型是众所周知的并且具有自定义 JSON 表示，则该表示将被嵌入，除了 `@type` 字段之外，还会添加一个包含自定义 JSON 的字段 `value` 。示例（对于消息 `google.protobuf.Duration` ）：

```json
{
  "@type": "type.googleapis.com/google.protobuf.Duration",
  "value": "1.212s"
}
```

| Field name 字段名称 | Type 类型 | Description 说明                                             |
| ------------------- | --------- | ------------------------------------------------------------ |
| `type_url`          | `string`  | A URL/resource name whose content describes the type of the serialized message. 一个 URL/资源名称，其内容描述序列化消息的类型。For URLs which use the schema `http`, `https`, or no schema, the following restrictions and interpretations apply: 对于使用架构 `http` 、 `https` 或无架构的 URL，适用以下限制和解释：If no schema is provided, `https` is assumed. 如果未提供架构，则假定为 `https` 。The last segment of the URL's path must represent the fully qualified name of the type (as in `path/google.protobuf.Duration`). URL 路径的最后一部分必须表示类型的完全限定名称（如 `path/google.protobuf.Duration` 中）。An HTTP GET on the URL must yield a `google.protobuf.Type` value in binary format, or produce an error. 对 URL 执行 HTTP GET 必须以二进制格式产生 `google.protobuf.Type` 值，或产生错误。Applications are allowed to cache lookup results based on the URL, or have them precompiled into a binary to avoid any lookup. Therefore, binary compatibility needs to be preserved on changes to types. (Use versioned type names to manage breaking changes.) 允许应用程序根据 URL 缓存查找结果，或将它们预编译成二进制文件以避免任何查找。因此，在更改类型时需要保留二进制兼容性。（使用版本化类型名称来管理重大更改。）Schemas other than `http`, `https` (or the empty schema) might be used with implementation specific semantics. 除 `http` 、 `https` （或空架构）之外的架构可能会与特定于实现的语义一起使用。 |
| `value`             | `bytes`   | Must be valid serialized data of the above specified type. 必须是上述指定类型的有效序列化数据。 |

## Api

Api is a light-weight descriptor for a protocol buffer service.
Api 是协议缓冲区服务的轻量级描述符。

| Field name 字段名称 | Type 类型       | Description 说明                                             |
| ------------------- | --------------- | ------------------------------------------------------------ |
| `name`              | `string`        | The fully qualified name of this api, including package name followed by the api's simple name. 此 api 的完全限定名称，包括包名称后跟 api 的简单名称。 |
| `methods`           | `Method`        | The methods of this api, in unspecified order. 此 api 的方法，顺序不指定。 |
| `options`           | `Option`        | Any metadata attached to the API. 附加到 API 的任何元数据。  |
| `version`           | `string`        | A version string for this api. If specified, must have the form `major-version.minor-version`, as in `1.10`. If the minor version is omitted, it defaults to zero. If the entire version field is empty, the major version is derived from the package name, as outlined below. If the field is not empty, the version in the package name will be verified to be consistent with what is provided here. 此 API 的版本字符串。如果指定，必须采用 `major-version.minor-version` 的形式，如 `1.10` 。如果省略次要版本，则默认为零。如果整个版本字段为空，则主版本将从包名派生，如下所述。如果该字段不为空，则会验证包名中的版本与此处提供的内容是否一致。The versioning schema uses [semantic versioning](http://semver.org/) where the major version number indicates a breaking change and the minor version an additive, non-breaking change. Both version numbers are signals to users what to expect from different versions, and should be carefully chosen based on the product plan. 版本控制方案使用语义版本控制，其中主版本号表示重大更改，次要版本表示累加的非重大更改。两个版本号都向用户发出信号，告知他们可以从不同版本中期待什么，并且应该根据产品计划仔细选择。The major version is also reflected in the package name of the API, which must end in `v<major-version>`, as in `google.feature.v1`. For major versions 0 and 1, the suffix can be omitted. Zero major versions must only be used for experimental, none-GA apis. 主版本还反映在 API 的包名中，该包名必须以 `v<major-version>` 结尾，如 `google.feature.v1` 。对于主版本 0 和 1，可以省略后缀。零主版本只能用于实验性的非 GA API。 |
| `source_context`    | `SourceContext` | Source context for the protocol buffer service represented by this message. 此消息表示的协议缓冲区服务的源上下文。 |
| `mixins`            | `Mixin`         | Included APIs. See `Mixin`. 包含的 API。请参阅 `Mixin` 。    |
| `syntax`            | `Syntax`        | The source syntax of the service. 服务的源语法。             |

## BoolValue

Wrapper message for `bool`.
用于 `bool` 的包装消息。

The JSON representation for `BoolValue` is JSON `true` and `false`.
`BoolValue` 的 JSON 表示形式是 JSON `true` 和 `false` 。

| Field name 字段名称 | Type 类型 | Description 说明         |
| ------------------- | --------- | ------------------------ |
| `value`             | `bool`    | The bool value. 布尔值。 |

## BytesValue

Wrapper message for `bytes`.
用于 `bytes` 的包装器消息。

The JSON representation for `BytesValue` is JSON string.
用于 `BytesValue` 的 JSON 表示形式是 JSON 字符串。

| Field name 字段名称 | Type 类型 | Description 说明          |
| ------------------- | --------- | ------------------------- |
| `value`             | `bytes`   | The bytes value. 字节值。 |

## DoubleValue 双精度值

Wrapper message for `double`.
用于 `double` 的包装消息。

The JSON representation for `DoubleValue` is JSON number.
对于 `DoubleValue` 的 JSON 表示形式是 JSON 数字。

| Field name 字段名称 | Type 类型 | Description 说明             |
| ------------------- | --------- | ---------------------------- |
| `value`             | `double`  | The double value. 双精度值。 |

## Duration

A Duration represents a signed, fixed-length span of time represented as a count of seconds and fractions of seconds at nanosecond resolution. It is independent of any calendar and concepts like "day" or "month". It is related to Timestamp in that the difference between two Timestamp values is a Duration and it can be added or subtracted from a Timestamp. Range is approximately +-10,000 years.
Duration 表示一个有符号的固定长度时间跨度，表示为以纳秒分辨率计算的秒数和小数秒。它独立于任何日历和“天”或“月”等概念。它与 Timestamp 相关，因为两个 Timestamp 值之间的差值是一个 Duration，并且可以将其添加到 Timestamp 中或从中减去。范围大约为 +-10,000 年。

Example 1: Compute Duration from two Timestamps in pseudo code.
示例 1：在伪代码中根据两个 Timestamp 计算 Duration。

```c
Timestamp start = ...;
Timestamp end = ...;
Duration duration = ...;

duration.seconds = end.seconds - start.seconds;
duration.nanos = end.nanos - start.nanos;

if (duration.seconds < 0 && duration.nanos > 0) {
  duration.seconds += 1;
  duration.nanos -= 1000000000;
} else if (duration.seconds > 0 && duration.nanos < 0) {
  duration.seconds -= 1;
  duration.nanos += 1000000000;
}
```

Example 2: Compute Timestamp from Timestamp + Duration in pseudo code.
示例 2：在伪代码中根据 Timestamp + Duration 计算 Timestamp。

```c
Timestamp start = ...;
Duration duration = ...;
Timestamp end = ...;

end.seconds = start.seconds + duration.seconds;
end.nanos = start.nanos + duration.nanos;

if (end.nanos < 0) {
  end.seconds -= 1;
  end.nanos += 1000000000;
} else if (end.nanos >= 1000000000) {
  end.seconds += 1;
  end.nanos -= 1000000000;
}
```

The JSON representation for `Duration` is a `String` that ends in `s` to indicate seconds and is preceded by the number of seconds, with nanoseconds expressed as fractional seconds.
对于 `Duration` 的 JSON 表示形式是 `String` ，以 `s` 结尾以指示秒数，并以秒数为前缀，纳秒表示为小数秒。

| Field name 字段名称 | Type 类型 | Description 说明                                             |
| ------------------- | --------- | ------------------------------------------------------------ |
| `seconds`           | `int64`   | Signed seconds of the span of time. Must be from -315,576,000,000 to +315,576,000,000 inclusive. 时间跨度的有符号秒数。必须介于 -315,576,000,000 到 +315,576,000,000（含）之间。 |
| `nanos`             | `int32`   | Signed fractions of a second at nanosecond resolution of the span of time. Durations less than one second are represented with a 0 `seconds` field and a positive or negative `nanos` field. For durations of one second or more, a non-zero value for the `nanos` field must be of the same sign as the `seconds` field. Must be from -999,999,999 to +999,999,999 inclusive. 以纳秒分辨率表示的时间跨度的带符号小数秒。小于一秒的持续时间用 0 `seconds` 字段和正或负 `nanos` 字段表示。对于持续时间为一秒或更长时间， `nanos` 字段的非零值必须与 `seconds` 字段同号。必须介于 -999,999,999 到 +999,999,999（含）之间。 |

## Empty

A generic empty message that you can re-use to avoid defining duplicated empty messages in your APIs. A typical example is to use it as the request or the response type of an API method. For instance:
一个通用的空消息，您可以重复使用它来避免在 API 中定义重复的空消息。一个典型的示例是将其用作 API 方法的请求或响应类型。例如：

```proto
service Foo {
  rpc Bar(google.protobuf.Empty) returns (google.protobuf.Empty);
}
```

The JSON representation for `Empty` is empty JSON object `{}`.
`Empty` 的 JSON 表示形式是空 JSON 对象 `{}` 。

## Enum

Enum type definition
枚举类型定义

| Field name 字段名称 | Type 类型       | Description 说明                          |
| ------------------- | --------------- | ----------------------------------------- |
| `name`              | `string`        | Enum type name. 枚举类型名称。            |
| `enumvalue`         | `EnumValue`     | Enum value definitions. 枚举值定义。      |
| `options`           | `Option`        | Protocol buffer options. 协议缓冲区选项。 |
| `source_context`    | `SourceContext` | The source context. 源上下文。            |
| `syntax`            | `Syntax`        | The source syntax. 源语法。               |

## EnumValue

Enum value definition.
枚举值定义。

| Field name 字段名称 | Type 类型 | Description 说明                          |
| ------------------- | --------- | ----------------------------------------- |
| `name`              | `string`  | Enum value name. 枚举值名称。             |
| `number`            | `int32`   | Enum value number. 枚举值编号。           |
| `options`           | `Option`  | Protocol buffer options. 协议缓冲区选项。 |

## Field

A single field of a message type.
消息类型的单个字段。

| Field name 字段名称 | Type 类型     | Description 说明                                             |
| ------------------- | ------------- | ------------------------------------------------------------ |
| `kind`              | `Kind`        | The field type. 字段类型。                                   |
| `cardinality`       | `Cardinality` | The field cardinality. 字段基数。                            |
| `number`            | `int32`       | The field number. 字段编号。                                 |
| `name`              | `string`      | The field name. 字段名称。                                   |
| `type_url`          | `string`      | The field type URL, without the scheme, for message or enumeration types. Example: `"type.googleapis.com/google.protobuf.Timestamp"`. 字段类型 URL，不含方案，适用于消息或枚举类型。示例： `"type.googleapis.com/google.protobuf.Timestamp"` 。 |
| `oneof_index`       | `int32`       | The index of the field type in `Type.oneofs`, for message or enumeration types. The first type has index 1; zero means the type is not in the list. 字段类型在 `Type.oneofs` 中的索引，适用于消息或枚举类型。第一个类型的索引为 1；零表示该类型不在列表中。 |
| `packed`            | `bool`        | Whether to use alternative packed wire representation. 是否使用备用打包的线制表示形式。 |
| `options`           | `Option`      | The protocol buffer options. 协议缓冲区选项。                |
| `json_name`         | `string`      | The field JSON name. 字段的 JSON 名称。                      |
| `default_value`     | `string`      | The string value of the default value of this field. Proto2 syntax only. 此字段的默认值的字符串值。仅限于 Proto2 语法。 |

## Cardinality 基数

Whether a field is optional, required, or repeated.
字段是可选的、必需的还是重复的。

| Enum value 枚举值      | Description 说明                                             |
| ---------------------- | ------------------------------------------------------------ |
| `CARDINALITY_UNKNOWN`  | For fields with unknown cardinality. 对于具有未知基数的字段。 |
| `CARDINALITY_OPTIONAL` | For optional fields. 对于可选字段。                          |
| `CARDINALITY_REQUIRED` | For required fields. Proto2 syntax only. 对于必填字段。仅限 Proto2 语法。 |
| `CARDINALITY_REPEATED` | For repeated fields. 对于重复字段。                          |

## Kind 类型

Basic field types.
基本字段类型。

| Enum value 枚举值 | Description 说明                                             |
| ----------------- | ------------------------------------------------------------ |
| `TYPE_UNKNOWN`    | Field type unknown. 字段类型未知。                           |
| `TYPE_DOUBLE`     | Field type double. 字段类型双精度。                          |
| `TYPE_FLOAT`      | Field type float. 字段类型 float.                            |
| `TYPE_INT64`      | Field type int64. 字段类型 int64.                            |
| `TYPE_UINT64`     | Field type uint64. 字段类型 uint64.                          |
| `TYPE_INT32`      | Field type int32. 字段类型 int32.                            |
| `TYPE_FIXED64`    | Field type fixed64. 字段类型 fixed64.                        |
| `TYPE_FIXED32`    | Field type fixed32. 字段类型 fixed32.                        |
| `TYPE_BOOL`       | Field type bool. 字段类型 bool.                              |
| `TYPE_STRING`     | Field type string. 字段类型 string.                          |
| `TYPE_GROUP`      | Field type group. Proto2 syntax only, and deprecated. 字段类型 group。仅限 Proto2 语法，且已弃用。 |
| `TYPE_MESSAGE`    | Field type message. 字段类型 message.                        |
| `TYPE_BYTES`      | Field type bytes. 字段类型字节。                             |
| `TYPE_UINT32`     | Field type uint32. 字段类型 uint32。                         |
| `TYPE_ENUM`       | Field type enum. 字段类型枚举。                              |
| `TYPE_SFIXED32`   | Field type sfixed32. 字段类型 sfixed32。                     |
| `TYPE_SFIXED64`   | Field type sfixed64. 字段类型 sfixed64。                     |
| `TYPE_SINT32`     | Field type sint32. 字段类型 sint32。                         |
| `TYPE_SINT64`     | Field type sint64. 字段类型 sint64。                         |

## FieldMask

`FieldMask` represents a set of symbolic field paths, for example:
`FieldMask` 表示一组符号字段路径，例如：

```proto
paths: "f.a"
paths: "f.b.d"
```

Here `f` represents a field in some root message, `a` and `b` fields in the message found in `f`, and `d` a field found in the message in `f.b`.
此处 `f` 表示某个根消息中的字段， `a` 和 `b` 表示在 `f` 中找到的消息中的字段， `d` 表示在 `f.b` 中找到的消息中的字段。

Field masks are used to specify a subset of fields that should be returned by a get operation (a *projection*), or modified by an update operation. Field masks also have a custom JSON encoding (see below).
字段掩码用于指定应由获取操作（投影）返回或由更新操作修改的字段子集。字段掩码还具有自定义 JSON 编码（请参见下文）。

#### Field Masks in Projections 投影中的字段掩码

When a `FieldMask` specifies a *projection*, the API will filter the response message (or sub-message) to contain only those fields specified in the mask. For example, consider this "pre-masking" response message:
当 `FieldMask` 指定投影时，API 将过滤响应消息（或子消息）以仅包含掩码中指定的部分字段。例如，考虑以下“预掩码”响应消息：

```proto
f {
  a : 22
  b {
    d : 1
    x : 2
  }
  y : 13
}
z: 8
```

After applying the mask in the previous example, the API response will not contain specific values for fields x, y, or z (their value will be set to the default, and omitted in proto text output):
应用上一个示例中的掩码后，API 响应将不包含字段 x、y 或 z 的具体值（它们的值将设置为默认值，并在 proto 文本输出中省略）：

```proto
f {
  a : 22
  b {
    d : 1
  }
}
```

A repeated field is not allowed except at the last position of a field mask.
重复字段仅允许位于字段掩码的最后位置。

If a `FieldMask` object is not present in a get operation, the operation applies to all fields (as if a FieldMask of all fields had been specified).
如果 `FieldMask` 对象不存在于获取操作中，则该操作将应用于所有字段（就像指定了所有字段的 FieldMask 一样）。

Note that a field mask does not necessarily apply to the top-level response message. In case of a REST get operation, the field mask applies directly to the response, but in case of a REST list operation, the mask instead applies to each individual message in the returned resource list. In case of a REST custom method, other definitions may be used. Where the mask applies will be clearly documented together with its declaration in the API. In any case, the effect on the returned resource/resources is required behavior for APIs.
请注意，字段掩码不一定适用于顶级响应消息。在 REST 获取操作的情况下，字段掩码直接应用于响应，但在 REST 列表操作的情况下，掩码改为应用于返回的资源列表中的每个单独消息。在 REST 自定义方法的情况下，可以使用其他定义。掩码的应用位置将与其在 API 中的声明一起明确记录。在任何情况下，对返回的资源/资源的影响都是 API 所需的行为。

#### Field Masks in Update Operations 更新操作中的字段掩码

A field mask in update operations specifies which fields of the targeted resource are going to be updated. The API is required to only change the values of the fields as specified in the mask and leave the others untouched. If a resource is passed in to describe the updated values, the API ignores the values of all fields not covered by the mask.
更新操作中的字段掩码指定要更新的目标资源的哪些字段。API 仅需更改掩码中指定字段的值，而不对其他字段进行更改。如果传入一个资源来描述更新后的值，则 API 会忽略掩码未涵盖的所有字段的值。

In order to reset a field’s value to the default, the field must be in the mask and set to the default value in the provided resource. Hence, in order to reset all fields of a resource, provide a default instance of the resource and set all fields in the mask, or do not provide a mask as described below.
要将字段的值重置为默认值，该字段必须在掩码中，并设置为所提供资源中的默认值。因此，要重置资源的所有字段，请提供资源的默认实例并在掩码中设置所有字段，或者按照以下说明不提供掩码。

If a field mask is not present on update, the operation applies to all fields (as if a field mask of all fields has been specified). Note that in the presence of schema evolution, this may mean that fields the client does not know and has therefore not filled into the request will be reset to their default. If this is unwanted behavior, a specific service may require a client to always specify a field mask, producing an error if not.
如果在更新时不存在字段掩码，则该操作适用于所有字段（就像已指定所有字段的字段掩码一样）。请注意，在存在架构演变的情况下，这可能意味着客户端不知道且因此未填入请求的字段将重置为其默认值。如果这是不希望的行为，则特定服务可能要求客户端始终指定字段掩码，如果未指定，则会产生错误。

As with get operations, the location of the resource which describes the updated values in the request message depends on the operation kind. In any case, the effect of the field mask is required to be honored by the API.
与获取操作一样，请求消息中描述更新值的资源的位置取决于操作类型。在任何情况下，API 都必须遵守字段掩码的效果。

##### Considerations for HTTP REST HTTP REST 的注意事项

The HTTP kind of an update operation which uses a field mask must be set to PATCH instead of PUT in order to satisfy HTTP semantics (PUT must only be used for full updates).
为了满足 HTTP 语义，使用字段掩码的更新操作的 HTTP 类型必须设置为 PATCH，而不是 PUT（PUT 只能用于完全更新）。

#### JSON Encoding of Field Masks 字段掩码的 JSON 编码

In JSON, a field mask is encoded as a single string where paths are separated by a comma. Fields name in each path are converted to/from lower-camel naming conventions.
在 JSON 中，字段掩码编码为单个字符串，其中路径以逗号分隔。每个路径中的字段名称转换为/从小驼峰命名约定。

As an example, consider the following message declarations:
例如，考虑以下消息声明：

```proto
message Profile {
  User user = 1;
  Photo photo = 2;
}
message User {
  string display_name = 1;
  string address = 2;
}
```

In proto a field mask for `Profile` may look as such:
在 proto 中， `Profile` 的字段掩码可能如下所示：

```proto
mask {
  paths: "user.display_name"
  paths: "photo"
}
```

In JSON, the same mask is represented as below:
在 JSON 中，相同的掩码表示如下：

```json
{
  mask: "user.displayName,photo"
}
```

| Field name 字段名称 | Type 类型 | Description 说明                                  |
| ------------------- | --------- | ------------------------------------------------- |
| `paths`             | `string`  | The set of field mask paths. 字段掩码路径的集合。 |

## FloatValue

Wrapper message for `float`.
用于 `float` 的包装消息。

The JSON representation for `FloatValue` is JSON number.
对于 `FloatValue` 的 JSON 表示形式是 JSON 数字。

| Field name 字段名称 | Type 类型 | Description 说明          |
| ------------------- | --------- | ------------------------- |
| `value`             | `float`   | The float value. 浮点值。 |

## Int32Value

Wrapper message for `int32`.
用于 `int32` 的包装消息。

The JSON representation for `Int32Value` is JSON number.
`Int32Value` 的 JSON 表示是 JSON 数字。

| Field name 字段名称 | Type 类型 | Description 说明            |
| ------------------- | --------- | --------------------------- |
| `value`             | `int32`   | The int32 value. int32 值。 |

## Int64Value

Wrapper message for `int64`.
用于 `int64` 的包装消息。

The JSON representation for `Int64Value` is JSON string.
用于 `Int64Value` 的 JSON 表示形式是 JSON 字符串。

| Field name 字段名称 | Type 类型 | Description 说明            |
| ------------------- | --------- | --------------------------- |
| `value`             | `int64`   | The int64 value. int64 值。 |

## ListValue

`ListValue` is a wrapper around a repeated field of values.
`ListValue` 是一个围绕重复字段值的包装器。

The JSON representation for `ListValue` is JSON array.
`ListValue` 的 JSON 表示形式是 JSON 数组。

| Field name 字段名称 | Type 类型 | Description 说明                                             |
| ------------------- | --------- | ------------------------------------------------------------ |
| `values`            | `Value`   | Repeated field of dynamically typed values. 动态类型值的重复字段。 |

## Method

Method represents a method of an api.
表示一个 api 的方法。

| Field name 字段名称  | Type 类型 | Description 说明                                             |
| -------------------- | --------- | ------------------------------------------------------------ |
| `name`               | `string`  | The simple name of this method. 此方法的简单名称。           |
| `request_type_url`   | `string`  | A URL of the input message type. 输入消息类型的 URL。        |
| `request_streaming`  | `bool`    | If true, the request is streamed. 如果为 true，则请求将被流式传输。 |
| `response_type_url`  | `string`  | The URL of the output message type. 输出消息类型的 URL。     |
| `response_streaming` | `bool`    | If true, the response is streamed. 如果为 true，则响应将被流式传输。 |
| `options`            | `Option`  | Any metadata attached to the method. 附加到该方法的任何元数据。 |
| `syntax`             | `Syntax`  | The source syntax of this method. 此方法的源语法。           |

## Mixin

Declares an API to be included in this API. The including API must redeclare all the methods from the included API, but documentation and options are inherited as follows:
声明要包含在此 API 中的 API。包含的 API 必须重新声明包含的 API 中的所有方法，但文档和选项将按如下方式继承：

- If after comment and whitespace stripping, the documentation string of the redeclared method is empty, it will be inherited from the original method.
  如果在去除注释和空格后，重新声明的方法的文档字符串为空，则它将从原始方法继承。
- Each annotation belonging to the service config (http, visibility) which is not set in the redeclared method will be inherited.
  属于服务配置（http、可见性）且未在重新声明的方法中设置的每个注释都将被继承。
- If an http annotation is inherited, the path pattern will be modified as follows. Any version prefix will be replaced by the version of the including API plus the `root` path if specified.
  如果继承了 http 注释，则路径模式将按如下方式修改。任何版本前缀都将替换为包含 API 的版本加上指定的 `root` 路径（如果指定）。

Example of a simple mixin:
简单混合示例：

```proto
package google.acl.v1;
service AccessControl {
  // Get the underlying ACL object.
  rpc GetAcl(GetAclRequest) returns (Acl) {
    option (google.api.http).get = "/v1/{resource=**}:getAcl";
  }
}

package google.storage.v2;
service Storage {
  //       rpc GetAcl(GetAclRequest) returns (Acl);

  // Get a data record.
  rpc GetData(GetDataRequest) returns (Data) {
    option (google.api.http).get = "/v2/{resource=**}";
  }
}
```

Example of a mixin configuration:
混合配置示例：

```fallback
apis:
- name: google.storage.v2.Storage
  mixins:
  - name: google.acl.v1.AccessControl
```

The mixin construct implies that all methods in `AccessControl` are also declared with same name and request/response types in `Storage`. A documentation generator or annotation processor will see the effective `Storage.GetAcl` method after inherting documentation and annotations as follows:
mixin 构造意味着 `AccessControl` 中的所有方法在 `Storage` 中也声明了相同名称和请求/响应类型。文档生成器或注释处理器将在继承文档和注释后看到有效的 `Storage.GetAcl` 方法，如下所示：

```proto
service Storage {
  // Get the underlying ACL object.
  rpc GetAcl(GetAclRequest) returns (Acl) {
    option (google.api.http).get = "/v2/{resource=**}:getAcl";
  }
  ...
}
```

Note how the version in the path pattern changed from `v1` to `v2`.
注意路径模式中的版本如何从 `v1` 更改为 `v2` 。

If the `root` field in the mixin is specified, it should be a relative path under which inherited HTTP paths are placed. Example:
如果在 mixin 中指定了 `root` 字段，它应为放置继承的 HTTP 路径的相对路径。示例：

```fallback
apis:
- name: google.storage.v2.Storage
  mixins:
  - name: google.acl.v1.AccessControl
    root: acls
```

This implies the following inherited HTTP annotation:
这意味着以下继承的 HTTP 注释：

```proto
service Storage {
  // Get the underlying ACL object.
  rpc GetAcl(GetAclRequest) returns (Acl) {
    option (google.api.http).get = "/v2/acls/{resource=**}:getAcl";
  }
  ...
}
```

| Field name 字段名称 | Type 类型 | Description 说明                                             |
| ------------------- | --------- | ------------------------------------------------------------ |
| `name`              | `string`  | The fully qualified name of the API which is included. 包含的 API 的完全限定名称。 |
| `root`              | `string`  | If non-empty specifies a path under which inherited HTTP paths are rooted. 如果非空，则指定继承的 HTTP 路径的根路径。 |

## NullValue

`NullValue` is a singleton enumeration to represent the null value for the `Value` type union.
`NullValue` 是一个单例枚举，用于表示 `Value` 类型联合的空值。

The JSON representation for `NullValue` is JSON `null`.
`NullValue` 的 JSON 表示形式是 JSON `null` 。

| Enum value 枚举值 | Description 说明   |
| ----------------- | ------------------ |
| `NULL_VALUE`      | Null value. 空值。 |

## Option 选项

A protocol buffer option, which can be attached to a message, field, enumeration, etc.
协议缓冲区选项，可以附加到消息、字段、枚举等。

| Field name 字段名称 | Type 类型 | Description 说明                                             |
| ------------------- | --------- | ------------------------------------------------------------ |
| `name`              | `string`  | The option's name. For example, `"java_package"`. 选项的名称。例如， `"java_package"` . |
| `value`             | `Any`     | The option's value. For example, `"com.google.protobuf"`. 选项的值。例如， `"com.google.protobuf"` . |

## SourceContext

`SourceContext` represents information about the source of a protobuf element, like the file in which it is defined.
`SourceContext` 表示有关 protobuf 元素的来源的信息，例如定义它的文件。

| Field name 字段名称 | Type 类型 | Description 说明                                             |
| ------------------- | --------- | ------------------------------------------------------------ |
| `file_name`         | `string`  | The path-qualified name of the .proto file that contained the associated protobuf element. For example: `"google/protobuf/source.proto"`. 包含关联的 protobuf 元素的 .proto 文件的路径限定名称。例如： `"google/protobuf/source.proto"` . |

## StringValue

Wrapper message for `string`.
用于 `string` 的包装消息。

The JSON representation for `StringValue` is JSON string.
用于 `StringValue` 的 JSON 表示形式是 JSON 字符串。

| Field name 字段名称 | Type 类型 | Description 说明             |
| ------------------- | --------- | ---------------------------- |
| `value`             | `string`  | The string value. 字符串值。 |

## Struct

`Struct` represents a structured data value, consisting of fields which map to dynamically typed values. In some languages, `Struct` might be supported by a native representation. For example, in scripting languages like JS a struct is represented as an object. The details of that representation are described together with the proto support for the language.
`Struct` 表示结构化数据值，由映射到动态类型值的字段组成。在某些语言中， `Struct` 可能受本机表示支持。例如，在 JS 等脚本语言中，结构表示为对象。该表示的详细信息与该语言的 proto 支持一起描述。

The JSON representation for `Struct` is JSON object.
`Struct` 的 JSON 表示是 JSON 对象。

| Field name 字段名称 | Type 类型            | Description 说明                                    |
| ------------------- | -------------------- | --------------------------------------------------- |
| `fields`            | `map<string, Value>` | Map of dynamically typed values. 动态类型值的映射。 |

## Syntax 语法

The syntax in which a protocol buffer element is defined.
定义协议缓冲区元素的语法。

| Enum value 枚举值 | Description 说明                  |
| ----------------- | --------------------------------- |
| `SYNTAX_PROTO2`   | Syntax `proto2`. 语法 `proto2` 。 |
| `SYNTAX_PROTO3`   | Syntax `proto3`. 语法 `proto3` 。 |

## Timestamp 时间戳

A Timestamp represents a point in time independent of any time zone or calendar, represented as seconds and fractions of seconds at nanosecond resolution in UTC Epoch time. It is encoded using the Proleptic Gregorian Calendar which extends the Gregorian calendar backwards to year one. It is encoded assuming all minutes are 60 seconds long, i.e. leap seconds are "smeared" so that no leap second table is needed for interpretation. Range is from 0001-01-01T00:00:00Z to 9999-12-31T23:59:59.999999999Z. By restricting to that range, we ensure that we can convert to and from RFC 3339 date strings. See https://www.ietf.org/rfc/rfc3339.txt.
时间戳表示独立于任何时区或日历的时间点，表示为 UTC 纪元时间以纳秒分辨率表示的秒数和秒数分数。它使用前儒略历编码，该历法将格里高利历向后延伸至公元 1 年。它假定所有分钟都是 60 秒，即闰秒被“抹去”，因此不需要闰秒表进行解释。范围从 0001-01-01T00:00:00Z 到 9999-12-31T23:59:59.999999999Z。通过限制该范围，我们确保可以与 RFC 3339 日期字符串进行转换。请参阅 https://www.ietf.org/rfc/rfc3339.txt。

Example 1: Compute Timestamp from POSIX `time()`.
示例 1：从 POSIX `time()` 计算时间戳。

```cpp
Timestamp timestamp;
timestamp.set_seconds(time(NULL));
timestamp.set_nanos(0);
```

Example 2: Compute Timestamp from POSIX `gettimeofday()`.
示例 2：从 POSIX `gettimeofday()` 计算时间戳。

```cpp
struct timeval tv;
gettimeofday(&tv, NULL);

Timestamp timestamp;
timestamp.set_seconds(tv.tv_sec);
timestamp.set_nanos(tv.tv_usec * 1000);
```

Example 3: Compute Timestamp from Win32 `GetSystemTimeAsFileTime()`.
示例 3：从 Win32 `GetSystemTimeAsFileTime()` 计算时间戳。

```cpp
FILETIME ft;
GetSystemTimeAsFileTime(&ft);
UINT64 ticks = (((UINT64)ft.dwHighDateTime) << 32) | ft.dwLowDateTime;

// A Windows tick is 100 nanoseconds. Windows epoch 1601-01-01T00:00:00Z
// is 11644473600 seconds before Unix epoch 1970-01-01T00:00:00Z.
Timestamp timestamp;
timestamp.set_seconds((INT64) ((ticks / 10000000) - 11644473600LL));
timestamp.set_nanos((INT32) ((ticks % 10000000) * 100));
```

Example 4: Compute Timestamp from Java `System.currentTimeMillis()`.
示例 4：从 Java `System.currentTimeMillis()` 计算时间戳。

```java
long millis = System.currentTimeMillis();

Timestamp timestamp = Timestamp.newBuilder().setSeconds(millis / 1000)
    .setNanos((int) ((millis % 1000) * 1000000)).build();
```

Example 5: Compute Timestamp from current time in Python.
示例 5：从 Python 中的当前时间计算时间戳。

```py
now = time.time()
seconds = int(now)
nanos = int((now - seconds) * 10**9)
timestamp = Timestamp(seconds=seconds, nanos=nanos)
```

| Field name 字段名称 | Type 类型 | Description 说明                                             |
| ------------------- | --------- | ------------------------------------------------------------ |
| `seconds`           | `int64`   | Represents seconds of UTC time since Unix epoch 1970-01-01T00:00:00Z. Must be from 0001-01-01T00:00:00Z to 9999-12-31T23:59:59Z inclusive. 表示自 Unix 纪元 1970-01-01T00:00:00Z 起的 UTC 时间的秒数。必须介于 0001-01-01T00:00:00Z 到 9999-12-31T23:59:59Z（含）之间。 |
| `nanos`             | `int32`   | Non-negative fractions of a second at nanosecond resolution. Negative second values with fractions must still have non-negative nanos values that count forward in time. Must be from 0 to 999,999,999 inclusive. 以纳秒分辨率表示的非负秒数分数。带分数的负秒值仍必须具有非负纳秒值，这些值按时间顺序向前计数。必须介于 0 到 999,999,999（含）之间。 |

## Type 类型

A protocol buffer message type.
协议缓冲区消息类型。

| Field name 字段名称 | Type 类型       | Description 说明                                             |
| ------------------- | --------------- | ------------------------------------------------------------ |
| `name`              | `string`        | The fully qualified message name. 完全限定的消息名称。       |
| `fields`            | `Field`         | The list of fields. 字段列表。                               |
| `oneofs`            | `string`        | The list of types appearing in `oneof` definitions in this type. 此类型中出现的类型列表 `oneof` 定义。 |
| `options`           | `Option`        | The protocol buffer options. 协议缓冲区选项。                |
| `source_context`    | `SourceContext` | The source context. 源上下文。                               |
| `syntax`            | `Syntax`        | The source syntax. 源语法。                                  |

## UInt32Value

Wrapper message for `uint32`.
用于 `uint32` 的包装消息。

The JSON representation for `UInt32Value` is JSON number.
`UInt32Value` 的 JSON 表示形式是 JSON 数字。

| Field name 字段名称 | Type 类型 | Description 说明              |
| ------------------- | --------- | ----------------------------- |
| `value`             | `uint32`  | The uint32 value. uint32 值。 |

## UInt64Value

Wrapper message for `uint64`.
用于 `uint64` 的包装消息。

The JSON representation for `UInt64Value` is JSON string.
用于 `UInt64Value` 的 JSON 表示形式是 JSON 字符串。

| Field name 字段名称 | Type 类型 | Description 说明              |
| ------------------- | --------- | ----------------------------- |
| `value`             | `uint64`  | The uint64 value. uint64 值。 |

## Value 值

`Value` represents a dynamically typed value which can be either null, a number, a string, a boolean, a recursive struct value, or a list of values. A producer of value is expected to set one of that variants, absence of any variant indicates an error.
`Value` 表示可以为 null、数字、字符串、布尔值、递归结构值或值列表的动态类型值。值生成器应设置其中一个变量，任何变量的缺失都表示错误。

The JSON representation for `Value` is JSON value.
JSON 表示形式为 `Value` 是 JSON 值。

| Field name 字段名称                                          | Type 类型   | Description 说明                                             |
| ------------------------------------------------------------ | ----------- | ------------------------------------------------------------ |
| Union field, only one of the following: 联合字段，以下字段之一： |             |                                                              |
| `null_value`                                                 | `NullValue` | Represents a null value. 表示空值。                          |
| `number_value`                                               | `double`    | Represents a double value. Note that attempting to serialize NaN or Infinity results in error. (We can't serialize these as string "NaN" or "Infinity" values like we do for regular fields, because they would parse as string_value, not number_value). 表示双精度值。请注意，尝试序列化 NaN 或 Infinity 会导致错误。（我们无法将它们序列化为字符串“NaN”或“Infinity”值，就像我们对常规字段所做的那样，因为它们将解析为 string_value，而不是 number_value）。 |
| `string_value`                                               | `string`    | Represents a string value. 表示字符串值。                    |
| `bool_value`                                                 | `bool`      | Represents a boolean value. 表示布尔值。                     |
| `struct_value`                                               | `Struct`    | Represents a structured value. 表示结构化值。                |
| `list_value`                                                 | `ListValue` | Represents a repeated `Value`. 表示重复的 `Value` 。         |
