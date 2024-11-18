+++
title = "常用类型"
date = 2024-11-17T12:07:24+08:00
weight = 40
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

> 原文：[https://protobuf.dev/reference/protobuf/google.protobuf/](https://protobuf.dev/reference/protobuf/google.protobuf/)
>
> 收录该文档的时间：`2024-11-17T12:07:24+08:00`

# Protocol Buffers Well-Known Types - Protocol Buffers 常用类型

API documentation for the google.protobuf package.

​	`google.protobuf` 包的 API 文档。

Well-Known Types that end in “`Value`” are wrapper messages for other types, such as `BoolValue` and `EnumValue`. These are now obsolete. The only reasons to use wrappers today would be:

​	以 "`Value`" 结尾的常用类型是其他类型的包装消息，例如 `BoolValue` 和 `EnumValue`。这些类型现已过时。使用包装类型的唯一原因是：

- Wire compatibility with messages that already use them.
  - 与已使用这些类型的消息的线缆兼容性。

- If you want to put a scalar value into an `Any` message.
  - 如果需要将标量值放入 `Any` 消息中。


In most cases, there are better options:

​	在大多数情况下，有更好的选择：

- For new messages, it’s better to use regular explicit-presence fields (`optional` in proto2/proto3, regular field in edition >= 2023).
  - 对于新消息，最好使用常规的显式存在字段（`proto2/proto3` 中的 `optional` 字段，或 2023 版及以上的普通字段）。

- Extensions are generally a better option than `Any` fields.
  - 扩展字段通常是比 `Any` 字段更好的选择。


## Any

`Any` contains an arbitrary serialized message along with a URL that describes the type of the serialized message.

​	`Any` 包含一个任意的序列化消息及描述该序列化消息类型的 URL。

#### JSON

The JSON representation of an `Any` value uses the regular representation of the deserialized, embedded message, with an additional field `@type` which contains the type URL. Example:

​	`Any` 值的 JSON 表示使用反序列化的嵌入消息的常规表示，外加一个包含类型 URL 的附加字段 `@type`。示例：

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

​	如果嵌入的消息类型是常用类型并具有自定义 JSON 表示，则该表示将嵌入，添加一个字段 `value` 以包含自定义 JSON，以及字段 `@type`。示例（针对消息 `google.protobuf.Duration`）：

```json
{
  "@type": "type.googleapis.com/google.protobuf.Duration",
  "value": "1.212s"
}
```

| Field name | Type     | Description                                                  |
| ---------- | -------- | ------------------------------------------------------------ |
| `type_url` | `string` | 一个 URL/资源名称，其内容描述了序列化消息的类型。有关使用 `http`、`https` 或无架构的 URL 的限制和解释如下：如果未提供架构，则假定为 `https`。URL 路径的最后一段必须表示完全限定的类型名称（如 `path/google.protobuf.Duration`）。对 URL 执行 HTTP GET 应返回以二进制格式表示的 `google.protobuf.Type` 值，或产生错误。应用程序可以基于 URL 缓存查找结果，或将其预编译到二进制中以避免任何查找。因此，需要在类型更改时维护二进制兼容性（使用版本化的类型名称来管理破坏性更改）。除了 `http`、`https`（或空架构）外的架构可能以实现特定语义使用。 A URL/resource name whose content describes the type of the serialized message.For URLs which use the schema `http`, `https`, or no schema, the following restrictions and interpretations apply:If no schema is provided, `https` is assumed.The last segment of the URL's path must represent the fully qualified name of the type (as in `path/google.protobuf.Duration`).An HTTP GET on the URL must yield a `google.protobuf.Type` value in binary format, or produce an error.Applications are allowed to cache lookup results based on the URL, or have them precompiled into a binary to avoid any lookup. Therefore, binary compatibility needs to be preserved on changes to types. (Use versioned type names to manage breaking changes.)Schemas other than `http`, `https` (or the empty schema) might be used with implementation specific semantics. |
| `value`    | `bytes`  | 必须是上述指定类型的有效序列化数据。 Must be valid serialized data of the above specified type. |

## Api

Api is a light-weight descriptor for a protocol buffer service.

​	`Api` 是协议缓冲服务的轻量描述符。

| Field name       | Type            | Description                                                  |
| ---------------- | --------------- | ------------------------------------------------------------ |
| `name`           | `string`        | 此 API 的完全限定名称，包括包名和 API 的简单名称。 The fully qualified name of this api, including package name followed by the api's simple name. |
| `methods`        | `Method`        | 此 API 的方法，顺序未指定。 The methods of this api, in unspecified order. |
| `options`        | `Option`        | 附加到 API 的任何元数据。 Any metadata attached to the API.  |
| `version`        | `string`        | 此 API 的版本字符串。如果指定，必须具有 `major-version.minor-version` 的形式，如 `1.10`。次版本号若省略，默认为 0。如果整个版本字段为空，则从包名中推导主要版本号，如下所述。如果字段非空，则将验证包名中的版本与此处提供的版本一致。版本控制方案使用 [语义版本控制](http://semver.org/)，其中主版本号表示重大更改，次版本号表示附加的非破坏性更改。这两个版本号是向用户表明不同版本预期的重要信号，应根据产品计划谨慎选择。主版本号还反映在 API 的包名中，包名必须以 `v<major-version>` 结尾，例如 `google.feature.v1`。对于主版本号 0 和 1，可省略后缀。主版本号为 0 仅应用于实验性、非 GA 的 API。 A version string for this api. If specified, must have the form `major-version.minor-version`, as in `1.10`. If the minor version is omitted, it defaults to zero. If the entire version field is empty, the major version is derived from the package name, as outlined below. If the field is not empty, the version in the package name will be verified to be consistent with what is provided here.The versioning schema uses [semantic versioning](http://semver.org/) where the major version number indicates a breaking change and the minor version an additive, non-breaking change. Both version numbers are signals to users what to expect from different versions, and should be carefully chosen based on the product plan.The major version is also reflected in the package name of the API, which must end in `v<major-version>`, as in `google.feature.v1`. For major versions 0 and 1, the suffix can be omitted. Zero major versions must only be used for experimental, none-GA apis. |
| `source_context` | `SourceContext` | 此消息表示的协议缓冲服务的源上下文。 Source context for the protocol buffer service represented by this message. |
| `mixins`         | `Mixin`         | 包含的 API。参见 `Mixin`。 Included APIs. See `Mixin`.       |
| `syntax`         | `Syntax`        | 服务的源语法。 The source syntax of the service.             |

## BoolValue

Wrapper message for `bool`.

​	`bool` 的包装消息。

The JSON representation for `BoolValue` is JSON `true` and `false`.

​	`BoolValue` 的 JSON 表示为 JSON `true` 和 `false`。

| Field name | Type   | Description     |
| ---------- | ------ | --------------- |
| `value`    | `bool` | The bool value. |

## BytesValue

Wrapper message for `bytes`.

​	`bytes` 的包装消息。

The JSON representation for `BytesValue` is JSON string.

​	`BytesValue` 的 JSON 表示为 JSON 字符串。

| Field name | Type    | Description              |
| ---------- | ------- | ------------------------ |
| `value`    | `bytes` | 字节值。The bytes value. |

## DoubleValue

Wrapper message for `double`.

​	`double` 的包装消息。

The JSON representation for `DoubleValue` is JSON number.

​	`DoubleValue` 的 JSON 表示为 JSON 数字。

| Field name | Type     | Description                  |
| ---------- | -------- | ---------------------------- |
| `value`    | `double` | 双精度值。 The double value. |

## Duration

A Duration represents a signed, fixed-length span of time represented as a count of seconds and fractions of seconds at nanosecond resolution. It is independent of any calendar and concepts like "day" or "month". It is related to Timestamp in that the difference between two Timestamp values is a Duration and it can be added or subtracted from a Timestamp. Range is approximately +-10,000 years.

​	`Duration` 表示一个有符号的固定长度时间跨度，该跨度表示为以纳秒分辨率的秒和秒的小数部分。它独立于任何日历以及“天”或“月”等概念。它与 `Timestamp` 相关，两者的差值为 `Duration`，且可以将 `Duration` 添加到或从 `Timestamp` 中减去。范围大约为 ±10,000 年。

Example 1: Compute Duration from two Timestamps in pseudo code.

​	示例 1：用伪代码计算两个 `Timestamp` 的差值以获取 `Duration`。

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

​	示例 2：用伪代码计算 `Timestamp` 加上 `Duration` 的结果。

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

​	`Duration` 的 JSON 表示为一个以 `s` 结尾的 `String`，表示秒数，秒的小数部分表示为纳秒。

| Field name | Type    | Description                                                  |
| ---------- | ------- | ------------------------------------------------------------ |
| `seconds`  | `int64` | 表示时间跨度的有符号秒数。必须在 -315,576,000,000 到 +315,576,000,000（含）之间。Signed seconds of the span of time. Must be from -315,576,000,000 to +315,576,000,000 inclusive. |
| `nanos`    | `int32` | 表示时间跨度的纳秒分辨率的有符号秒数的小数部分。小于一秒的时间跨度用 `seconds` 字段为 0 和一个正或负的 `nanos` 字段表示。对于一秒或更长时间的时间跨度，`nanos` 字段的非零值必须与 `seconds` 字段的符号相同。必须在 -999,999,999 到 +999,999,999（含）之间。 Signed fractions of a second at nanosecond resolution of the span of time. Durations less than one second are represented with a 0 `seconds` field and a positive or negative `nanos` field. For durations of one second or more, a non-zero value for the `nanos` field must be of the same sign as the `seconds` field. Must be from -999,999,999 to +999,999,999 inclusive. |

## Empty

A generic empty message that you can re-use to avoid defining duplicated empty messages in your APIs. A typical example is to use it as the request or the response type of an API method. For instance:

​	一个通用的空消息，可以重用以避免在 API 中定义重复的空消息。典型示例是将其用作 API 方法的请求或响应类型。例如：



```proto
service Foo {
  rpc Bar(google.protobuf.Empty) returns (google.protobuf.Empty);
}
```

The JSON representation for `Empty` is empty JSON object `{}`.

​	`Empty` 的 JSON 表示为空 JSON 对象 `{}`。

## Enum

Enum type definition

​	枚举类型定义。

| Field name       | Type            | Description                               |
| ---------------- | --------------- | ----------------------------------------- |
| `name`           | `string`        | 枚举类型名称。 Enum type name.            |
| `enumvalue`      | `EnumValue`     | 枚举值定义。 Enum value definitions.      |
| `options`        | `Option`        | 协议缓冲区选项。 Protocol buffer options. |
| `source_context` | `SourceContext` | 源上下文。 The source context.            |
| `syntax`         | `Syntax`        | 源语法。 The source syntax.               |

## EnumValue

Enum value definition.

​	枚举值定义。

| Field name | Type     | Description                               |
| ---------- | -------- | ----------------------------------------- |
| `name`     | `string` | 枚举值名称。 Enum value name.             |
| `number`   | `int32`  | 枚举值编号。 Enum value number.           |
| `options`  | `Option` | 协议缓冲区选项。 Protocol buffer options. |

## Field

A single field of a message type.

​	消息类型的单个字段。

| Field name      | Type          | Description                                                  |
| --------------- | ------------- | ------------------------------------------------------------ |
| `kind`          | `Kind`        | 字段类型。 The field type.                                   |
| `cardinality`   | `Cardinality` | 字段的基数。 The field cardinality.                          |
| `number`        | `int32`       | 字段编号。The field number.                                  |
| `name`          | `string`      | 字段名称。The field name.                                    |
| `type_url`      | `string`      | 消息或枚举类型的字段类型 URL，不包括模式。例如：`"type.googleapis.com/google.protobuf.Timestamp"`。 The field type URL, without the scheme, for message or enumeration types. Example: `"type.googleapis.com/google.protobuf.Timestamp"`. |
| `oneof_index`   | `int32`       | `Type.oneofs` 中字段类型的索引，用于消息或枚举类型。第一个类型索引为 1；零表示类型不在列表中。 The index of the field type in `Type.oneofs`, for message or enumeration types. The first type has index 1; zero means the type is not in the list. |
| `packed`        | `bool`        | 是否使用替代的打包线格式表示。 Whether to use alternative packed wire representation. |
| `options`       | `Option`      | 协议缓冲区选项。 The protocol buffer options.                |
| `json_name`     | `string`      | 字段的 JSON 名称。 The field JSON name.                      |
| `default_value` | `string`      | 此字段的默认值的字符串值。仅适用于 Proto2 语法。 The string value of the default value of this field. Proto2 syntax only. |

## Cardinality

Whether a field is optional, required, or repeated.

​	字段是可选、必填还是重复的。

| Enum value             | Description                                                  |
| ---------------------- | ------------------------------------------------------------ |
| `CARDINALITY_UNKNOWN`  | 未知基数的字段。 For fields with unknown cardinality.        |
| `CARDINALITY_OPTIONAL` | 可选字段。 For optional fields.                              |
| `CARDINALITY_REQUIRED` | 必填字段，仅适用于 Proto2 语法。 For required fields. Proto2 syntax only. |
| `CARDINALITY_REPEATED` | 重复字段。 For repeated fields.                              |

## Kind

Basic field types.

​	基础字段类型。

| Enum value      | Description                                                  |
| --------------- | ------------------------------------------------------------ |
| `TYPE_UNKNOWN`  | 未知字段类型。 Field type unknown.                           |
| `TYPE_DOUBLE`   | 字段类型为 double。 Field type double.                       |
| `TYPE_FLOAT`    | 字段类型为 float。 Field type float.                         |
| `TYPE_INT64`    | 字段类型为 int64。 Field type int64.                         |
| `TYPE_UINT64`   | 字段类型为 uint64。 Field type uint64.                       |
| `TYPE_INT32`    | 字段类型为 int32。 Field type int32.                         |
| `TYPE_FIXED64`  | 字段类型为 fixed64。 Field type fixed64.                     |
| `TYPE_FIXED32`  | 字段类型为 fixed32。Field type fixed32.                      |
| `TYPE_BOOL`     | 字段类型为 bool。Field type bool.                            |
| `TYPE_STRING`   | 字段类型为 string。Field type string.                        |
| `TYPE_GROUP`    | 字段类型为 group。仅适用于 Proto2 语法，且已弃用。Field type group. Proto2 syntax only, and deprecated. |
| `TYPE_MESSAGE`  | 字段类型为消息。Field type message.                          |
| `TYPE_BYTES`    | 字段类型为字节。Field type bytes.                            |
| `TYPE_UINT32`   | 字段类型为 uint32。Field type uint32.                        |
| `TYPE_ENUM`     | 字段类型为枚举。Field type enum.                             |
| `TYPE_SFIXED32` | 字段类型为 sfixed32。Field type sfixed32.                    |
| `TYPE_SFIXED64` | 字段类型为 sfixed64。Field type sfixed64.                    |
| `TYPE_SINT32`   | 字段类型为 sint32。Field type sint32.                        |
| `TYPE_SINT64`   | 字段类型为 sint64。Field type sint64.                        |

## FieldMask

`FieldMask` represents a set of symbolic field paths, for example:

​	`FieldMask` 表示一组符号字段路径，例如：

```proto
paths: "f.a"
paths: "f.b.d"
```

Here `f` represents a field in some root message, `a` and `b` fields in the message found in `f`, and `d` a field found in the message in `f.b`.

​	其中 `f` 表示某个根消息中的字段，`a` 和 `b` 表示 `f` 中消息的字段，`d` 表示 `f.b` 中消息的字段。

Field masks are used to specify a subset of fields that should be returned by a get operation (a *projection*), or modified by an update operation. Field masks also have a custom JSON encoding (see below).

​	字段掩码用于指定应由 `get` 操作返回（投影）或由 `update` 操作修改的字段子集。字段掩码还具有自定义 JSON 编码（见下文）。

### 投影中的字段掩码 Field Masks in Projections

When a `FieldMask` specifies a *projection*, the API will filter the response message (or sub-message) to contain only those fields specified in the mask. For example, consider this "pre-masking" response message:

​	当 `FieldMask` 指定 *投影* 时，API 将过滤响应消息（或子消息），仅包含掩码中指定的字段。例如，考虑以下“预掩码”响应消息：

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

​	在应用上述掩码后，API 响应将不包含字段 x、y 或 z 的特定值（它们的值将设置为默认值，并在 proto 文本输出中省略）：

```proto
f {
  a : 22
  b {
    d : 1
  }
}
```

A repeated field is not allowed except at the last position of a field mask.

​	字段掩码中除了最后一个位置之外，不允许使用重复字段。

If a `FieldMask` object is not present in a get operation, the operation applies to all fields (as if a FieldMask of all fields had been specified).

​	如果 `FieldMask` 对象不存在于 `get` 操作中，则操作将应用于所有字段（相当于指定了包含所有字段的字段掩码）。

Note that a field mask does not necessarily apply to the top-level response message. In case of a REST get operation, the field mask applies directly to the response, but in case of a REST list operation, the mask instead applies to each individual message in the returned resource list. In case of a REST custom method, other definitions may be used. Where the mask applies will be clearly documented together with its declaration in the API. In any case, the effect on the returned resource/resources is required behavior for APIs.

​	请注意，字段掩码不一定适用于顶级响应消息。在 REST `get` 操作中，字段掩码直接应用于响应；但在 REST `list` 操作中，掩码则应用于返回资源列表中的每个单独消息。在 REST 自定义方法的情况下，可以使用其他定义。字段掩码的适用位置将在 API 的声明中明确记录。在任何情况下，对返回资源的影响都是 API 的必需行为。

#### 更新操作中的字段掩码 Field Masks in Update Operations

A field mask in update operations specifies which fields of the targeted resource are going to be updated. The API is required to only change the values of the fields as specified in the mask and leave the others untouched. If a resource is passed in to describe the updated values, the API ignores the values of all fields not covered by the mask.

​	在更新操作中，字段掩码指定要更新的目标资源字段。API 必须仅更改掩码中指定字段的值，而不触及其他字段。如果传递一个资源来描述更新的值，API 将忽略掩码未覆盖字段的所有值。

In order to reset a field’s value to the default, the field must be in the mask and set to the default value in the provided resource. Hence, in order to reset all fields of a resource, provide a default instance of the resource and set all fields in the mask, or do not provide a mask as described below.

​	为了将字段的值重置为默认值，必须在掩码中包含该字段，并在提供的资源中将其设置为默认值。因此，为了重置资源的所有字段，可以提供该资源的默认实例，并在掩码中设置所有字段，或者不提供掩码。

If a field mask is not present on update, the operation applies to all fields (as if a field mask of all fields has been specified). Note that in the presence of schema evolution, this may mean that fields the client does not know and has therefore not filled into the request will be reset to their default. If this is unwanted behavior, a specific service may require a client to always specify a field mask, producing an error if not.

​	如果更新操作中未提供字段掩码，则操作适用于所有字段（相当于指定了包含所有字段的字段掩码）。请注意，在存在模式演化的情况下，这可能意味着客户端不知道且因此未填入请求中的字段将被重置为默认值。如果这是不期望的行为，特定服务可能会要求客户端始终指定字段掩码，若未指定则产生错误。

As with get operations, the location of the resource which describes the updated values in the request message depends on the operation kind. In any case, the effect of the field mask is required to be honored by the API.

​	与 `get` 操作类似，请求消息中描述更新值的资源的位置取决于操作的类型。在任何情况下，字段掩码的效果都需要被 API 确保。

#### 关于 HTTP REST 的注意事项 Considerations for HTTP REST

The HTTP kind of an update operation which uses a field mask must be set to PATCH instead of PUT in order to satisfy HTTP semantics (PUT must only be used for full updates).

​	使用字段掩码的更新操作的 HTTP 类型必须设置为 `PATCH` 而不是 `PUT`，以满足 HTTP 语义（`PUT` 仅可用于完整更新）。

### 字段掩码的 JSON 编码 JSON Encoding of Field Masks

In JSON, a field mask is encoded as a single string where paths are separated by a comma. Fields name in each path are converted to/from lower-camel naming conventions.

​	在 JSON 中，字段掩码编码为单个字符串，路径之间用逗号分隔。路径中的字段名称根据 lower-camel 命名约定进行转换。

As an example, consider the following message declarations:

​	例如，考虑以下消息声明：

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

​	在 proto 中，`Profile` 的字段掩码可能如下所示：

```proto
mask {
  paths: "user.display_name"
  paths: "photo"
}
```

In JSON, the same mask is represented as below:

​	在 JSON 中，相同的掩码表示如下：

```json
{
  mask: "user.displayName,photo"
}
```

| Field name | Type     | Description                                       |
| ---------- | -------- | ------------------------------------------------- |
| `paths`    | `string` | 字段掩码路径的集合。 The set of field mask paths. |

## FloatValue

Wrapper message for `float`.

​	`float` 的包装消息。

The JSON representation for `FloatValue` is JSON number.

​	`FloatValue` 的 JSON 表示为 JSON 数字。

| Field name | Type    | Description              |
| ---------- | ------- | ------------------------ |
| `value`    | `float` | 浮点值。The float value. |

## Int32Value

Wrapper message for `int32`.

​	`int32` 的包装消息。

The JSON representation for `Int32Value` is JSON number.

​	`Int32Value` 的 JSON 表示为 JSON 数字。

| Field name | Type    | Description                   |
| ---------- | ------- | ----------------------------- |
| `value`    | `int32` | 32 位整数值。The int32 value. |

## Int64Value

Wrapper message for `int64`.

​	`int64` 的包装消息。

The JSON representation for `Int64Value` is JSON string.

​	`Int64Value` 的 JSON 表示为 JSON 字符串。

| Field name | Type    | Description                   |
| ---------- | ------- | ----------------------------- |
| `value`    | `int64` | 64 位整数值。The int64 value. |

## ListValue

`ListValue` is a wrapper around a repeated field of values.

​	`ListValue` 是一个包装器，包含一个值的重复字段。

The JSON representation for `ListValue` is JSON array.

​	`ListValue` 的 JSON 表示为 JSON 数组。

| Field name | Type    | Description                                                  |
| ---------- | ------- | ------------------------------------------------------------ |
| `values`   | `Value` | 动态类型值的重复字段。Repeated field of dynamically typed values. |

## Method

Method represents a method of an api.

​	表示 API 的一个方法。

| Field name           | Type     | Description                                                  |
| -------------------- | -------- | ------------------------------------------------------------ |
| `name`               | `string` | 此方法的简单名称。The simple name of this method.            |
| `request_type_url`   | `string` | 输入消息类型的 URL。A URL of the input message type.         |
| `request_streaming`  | `bool`   | 如果为 true，则请求为流式传输。If true, the request is streamed. |
| `response_type_url`  | `string` | 输出消息类型的 URL。The URL of the output message type.      |
| `response_streaming` | `bool`   | 如果为 true，则响应为流式传输。If true, the response is streamed. |
| `options`            | `Option` | 附加到方法的任何元数据。Any metadata attached to the method. |
| `syntax`             | `Syntax` | 方法的源语法。The source syntax of this method.              |

## Mixin

Declares an API to be included in this API. The including API must redeclare all the methods from the included API, but documentation and options are inherited as follows:

​	声明将某个 API 包含在当前 API 中。包含的 API 必须重新声明被包含 API 的所有方法，但文档和选项会继承，具体如下：

- If after comment and whitespace stripping, the documentation string of the redeclared method is empty, it will be inherited from the original method.
  - 如果重新声明的方法在去掉注释和空格后，其文档字符串为空，则将继承原始方法的文档。

- Each annotation belonging to the service config (http, visibility) which is not set in the redeclared method will be inherited.
  - 属于服务配置的注解（如 HTTP、可见性）如果未在重新声明的方法中设置，将继承原始方法的注解。

- If an http annotation is inherited, the path pattern will be modified as follows. Any version prefix will be replaced by the version of the including API plus the `root` path if specified.
  - 如果继承了 HTTP 注解，路径模式将被修改。任何版本前缀将替换为包含 API 的版本，并加上 `root` 路径（如果指定）。


Example of a simple mixin:

​	简单的 Mixin 示例

```proto
package google.acl.v1;
service AccessControl {
  // Get the underlying ACL object.
  // 获取底层的 ACL 对象。
  rpc GetAcl(GetAclRequest) returns (Acl) {
    option (google.api.http).get = "/v1/{resource=**}:getAcl";
  }
}

package google.storage.v2;
service Storage {
  //       rpc GetAcl(GetAclRequest) returns (Acl);

  // Get a data record.
  // 获取数据记录。
  rpc GetData(GetDataRequest) returns (Data) {
    option (google.api.http).get = "/v2/{resource=**}";
  }
}
```

Example of a mixin configuration:

​	Mixin 配置示例

```fallback
apis:
- name: google.storage.v2.Storage
  mixins:
  - name: google.acl.v1.AccessControl
```

The mixin construct implies that all methods in `AccessControl` are also declared with same name and request/response types in `Storage`. A documentation generator or annotation processor will see the effective `Storage.GetAcl` method after inheriting documentation and annotations as follows:

​	Mixin 结构暗示了 `AccessControl` 中的所有方法都在 `Storage` 中用相同的名称和请求/响应类型进行了重新声明。文档生成器或注解处理器在继承文档和注解后将看到如下的 `Storage.GetAcl` 方法：

```proto
service Storage {
  // Get the underlying ACL object.
  // 获取底层的 ACL 对象。
  rpc GetAcl(GetAclRequest) returns (Acl) {
    option (google.api.http).get = "/v2/{resource=**}:getAcl";
  }
  ...
}
```

Note how the version in the path pattern changed from `v1` to `v2`.

​	注意路径模式中的版本如何从 `v1` 更改为 `v2`。

If the `root` field in the mixin is specified, it should be a relative path under which inherited HTTP paths are placed. Example:

​	如果在 Mixin 中指定了 `root` 字段，它应是一个相对路径，表示继承的 HTTP 路径所在的根目录。例如：

```fallback
apis:
- name: google.storage.v2.Storage
  mixins:
  - name: google.acl.v1.AccessControl
    root: acls
```

This implies the following inherited HTTP annotation:

​	这意味着以下继承的 HTTP 注解：

```proto
service Storage {
  // Get the underlying ACL object.
  // 获取底层的 ACL 对象。
  rpc GetAcl(GetAclRequest) returns (Acl) {
    option (google.api.http).get = "/v2/acls/{resource=**}:getAcl";
  }
  ...
}
```

| Field name | Type     | Description                                                  |
| ---------- | -------- | ------------------------------------------------------------ |
| `name`     | `string` | 要包含的 API 的完全限定名称。The fully qualified name of the API which is included. |
| `root`     | `string` | 如果非空，指定一个路径，该路径是继承的 HTTP 路径的根。If non-empty specifies a path under which inherited HTTP paths are rooted. |

## NullValue

`NullValue` is a singleton enumeration to represent the null value for the `Value` type union.

​	`NullValue` 是一个单例枚举，用于表示 `Value` 类型联合的空值。

The JSON representation for `NullValue` is JSON `null`.

​	`NullValue` 的 JSON 表示为 JSON `null`。

| Enum value   | Description       |
| ------------ | ----------------- |
| `NULL_VALUE` | 空值。Null value. |

## Option

A protocol buffer option, which can be attached to a message, field, enumeration, etc.

​	协议缓冲区选项，可附加到消息、字段、枚举等。

| Field name | Type     | Description                                                  |
| ---------- | -------- | ------------------------------------------------------------ |
| `name`     | `string` | 选项名称，例如 `"java_package"`。The option's name. For example, `"java_package"`. |
| `value`    | `Any`    | 选项值，例如 `"com.google.protobuf"`。The option's value. For example, `"com.google.protobuf"`. |

## SourceContext

`SourceContext` represents information about the source of a protobuf element, like the file in which it is defined.

​	`SourceContext` 表示与 protobuf 元素相关的源信息，例如定义该元素的文件。

| Field name  | Type     | Description                                                  |
| ----------- | -------- | ------------------------------------------------------------ |
| `file_name` | `string` | `SourceContext` 表示与 protobuf 元素相关的源信息，例如定义该元素的文件。The path-qualified name of the .proto file that contained the associated protobuf element. For example: `"google/protobuf/source.proto"`. |

## StringValue

Wrapper message for `string`.

​	`string` 的包装消息。

The JSON representation for `StringValue` is JSON string.

​	`StringValue` 的 JSON 表示为 JSON 字符串。

| Field name | Type     | Description                 |
| ---------- | -------- | --------------------------- |
| `value`    | `string` | 字符串值。The string value. |

## Struct

`Struct` represents a structured data value, consisting of fields which map to dynamically typed values. In some languages, `Struct` might be supported by a native representation. For example, in scripting languages like JS a struct is represented as an object. The details of that representation are described together with the proto support for the language.

​	`Struct` 表示一个结构化数据值，由映射到动态类型值的字段组成。在某些语言中，`Struct` 可能由本地表示支持。例如，在 JavaScript 等脚本语言中，结构可以表示为对象。这种表示的细节与语言支持的 proto 一起描述。

The JSON representation for `Struct` is JSON object.

​	`Struct` 的 JSON 表示为 JSON 对象。

| Field name | Type                 | Description                                        |
| ---------- | -------------------- | -------------------------------------------------- |
| `fields`   | `map<string, Value>` | 动态类型值的映射。Map of dynamically typed values. |

## Syntax

The syntax in which a protocol buffer element is defined.

​	定义协议缓冲区元素的语法。

| Enum value      | Description      |
| --------------- | ---------------- |
| `SYNTAX_PROTO2` | Syntax `proto2`. |
| `SYNTAX_PROTO3` | Syntax `proto3`. |

## Timestamp

A Timestamp represents a point in time independent of any time zone or calendar, represented as seconds and fractions of seconds at nanosecond resolution in UTC Epoch time. It is encoded using the Proleptic Gregorian Calendar which extends the Gregorian calendar backwards to year one. It is encoded assuming all minutes are 60 seconds long, i.e. leap seconds are "smeared" so that no leap second table is needed for interpretation. Range is from 0001-01-01T00:00:00Z to 9999-12-31T23:59:59.999999999Z. By restricting to that range, we ensure that we can convert to and from RFC 3339 date strings. See https://www.ietf.org/rfc/rfc3339.txt.

​	`Timestamp` 表示与任何时区或日历无关的时间点，以纳秒分辨率的秒和秒的小数部分表示为 UTC 时间。它使用公历（Proleptic Gregorian Calendar）编码，该日历向前扩展到公元 1 年，并假定所有分钟均为 60 秒长（即，闰秒被“平滑”处理，无需使用闰秒表）。范围从 0001-01-01T00:00:00Z 到 9999-12-31T23:59:59.999999999Z。通过限制该范围，我们确保可以与 RFC 3339 日期字符串互相转换。详见 [RFC 3339](https://www.ietf.org/rfc/rfc3339.txt)。

Example 1: Compute Timestamp from POSIX `time()`.

​	示例 1：从 POSIX `time()` 计算 `Timestamp`

```cpp
Timestamp timestamp;
timestamp.set_seconds(time(NULL));
timestamp.set_nanos(0);
```

Example 2: Compute Timestamp from POSIX `gettimeofday()`.

​	示例 2：从 POSIX `gettimeofday()` 计算 `Timestamp`

```cpp
struct timeval tv;
gettimeofday(&tv, NULL);

Timestamp timestamp;
timestamp.set_seconds(tv.tv_sec);
timestamp.set_nanos(tv.tv_usec * 1000);
```

Example 3: Compute Timestamp from Win32 `GetSystemTimeAsFileTime()`.

​	示例 3：从 Win32 `GetSystemTimeAsFileTime()` 计算 `Timestamp`

```cpp
FILETIME ft;
GetSystemTimeAsFileTime(&ft);
UINT64 ticks = (((UINT64)ft.dwHighDateTime) << 32) | ft.dwLowDateTime;

// A Windows tick is 100 nanoseconds. Windows epoch 1601-01-01T00:00:00Z
// is 11644473600 seconds before Unix epoch 1970-01-01T00:00:00Z.
// Windows tick 为 100 纳秒。Windows 纪元 1601-01-01T00:00:00Z
// 比 Unix 纪元 1970-01-01T00:00:00Z 提前 11644473600 秒。
Timestamp timestamp;
timestamp.set_seconds((INT64) ((ticks / 10000000) - 11644473600LL));
timestamp.set_nanos((INT32) ((ticks % 10000000) * 100));
```

Example 4: Compute Timestamp from Java `System.currentTimeMillis()`.

​	示例 4：从 Java `System.currentTimeMillis()` 计算 `Timestamp`

```java
long millis = System.currentTimeMillis();

Timestamp timestamp = Timestamp.newBuilder().setSeconds(millis / 1000)
    .setNanos((int) ((millis % 1000) * 1000000)).build();
```

Example 5: Compute Timestamp from current time in Python.

​	示例 5：从当前时间用 Python 计算 `Timestamp`

```py
now = time.time()
seconds = int(now)
nanos = int((now - seconds) * 10**9)
timestamp = Timestamp(seconds=seconds, nanos=nanos)
```

| Field name | Type    | Description                                                  |
| ---------- | ------- | ------------------------------------------------------------ |
| `seconds`  | `int64` | 自 Unix 纪元 1970-01-01T00:00:00Z 以来的 UTC 秒数。范围：0001-01-01T00:00:00Z 到 9999-12-31T23:59:59Z。 Represents seconds of UTC time since Unix epoch 1970-01-01T00:00:00Z. Must be from 0001-01-01T00:00:00Z to 9999-12-31T23:59:59Z inclusive. |
| `nanos`    | `int32` | 纳秒分辨率的非负秒数的小数部分。对于负秒值，纳秒值必须为非负数。范围：0 到 999,999,999。 Non-negative fractions of a second at nanosecond resolution. Negative second values with fractions must still have non-negative nanos values that count forward in time. Must be from 0 to 999,999,999 inclusive. |

## Type

A protocol buffer message type.

​	协议缓冲区消息类型。

| Field name       | Type            | Description                                                  |
| ---------------- | --------------- | ------------------------------------------------------------ |
| `name`           | `string`        | 完全限定消息名称。 The fully qualified message name.         |
| `fields`         | `Field`         | 字段列表。The list of fields.                                |
| `oneofs`         | `string`        | 此类型中 `oneof` 定义中出现的类型列表。The list of types appearing in `oneof` definitions in this type. |
| `options`        | `Option`        | 协议缓冲区选项。The protocol buffer options.                 |
| `source_context` | `SourceContext` | 源上下文。The source context.                                |
| `syntax`         | `Syntax`        | 源语法。The source syntax.                                   |

## UInt32Value

Wrapper message for `uint32`.

​	`uint32` 的包装消息。

The JSON representation for `UInt32Value` is JSON number.

​	`UInt32Value` 的 JSON 表示为 JSON 数字。

| Field name | Type     | Description                          |
| ---------- | -------- | ------------------------------------ |
| `value`    | `uint32` | 32 位无符号整数值。The uint32 value. |

## UInt64Value

Wrapper message for `uint64`.

​	`uint64` 的包装消息。

The JSON representation for `UInt64Value` is JSON string.

​	`UInt64Value` 的 JSON 表示为 JSON 字符串。

| Field name | Type     | Description                          |
| ---------- | -------- | ------------------------------------ |
| `value`    | `uint64` | 64 位无符号整数值。The uint64 value. |

## Value

`Value` represents a dynamically typed value which can be either null, a number, a string, a boolean, a recursive struct value, or a list of values. A producer of value is expected to set one of that variants, absence of any variant indicates an error.

​	`Value` 表示一个动态类型值，可以是 `null`、数字、字符串、布尔值、递归结构值或值列表。生成值的生产者应设置这些变体之一，未设置任何变体表示错误。

The JSON representation for `Value` is JSON value.

​	`Value` 的 JSON 表示为 JSON 值。

| Field name                                                   | Type        | Description                                                  |
| ------------------------------------------------------------ | ----------- | ------------------------------------------------------------ |
| 联合字段（以下仅选一项）：Union field, only one of the following: |             |                                                              |
| `null_value`                                                 | `NullValue` | 表示空值。Represents a null value.                           |
| `number_value`                                               | `double`    | 表示一个双精度值。注意，尝试序列化 NaN 或无穷大会导致错误。（我们无法像处理常规字段那样将这些值序列化为字符串 "NaN" 或 "Infinity"，因为它们会被解析为 `string_value`，而不是 `number_value`）。Represents a double value. Note that attempting to serialize NaN or Infinity results in error. (We can't serialize these as string "NaN" or "Infinity" values like we do for regular fields, because they would parse as string_value, not number_value). |
| `string_value`                                               | `string`    | 表示字符串值。Represents a string value.                     |
| `bool_value`                                                 | `bool`      | 表示布尔值。Represents a boolean value.                      |
| `struct_value`                                               | `Struct`    | 表示结构化值。Represents a structured value.                 |
| `list_value`                                                 | `ListValue` | 表示重复 `Value` 值的列表。Represents a repeated `Value`.    |
