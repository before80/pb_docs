+++
title = "C# API"
date = 2024-11-17T12:07:24+08:00
weight = 10
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文：[https://protobuf.dev/reference/csharp/api-docs](https://protobuf.dev/reference/csharp/api-docs)
>
> 收录该文档的时间：`2024-11-17T12:07:24+08:00`

# Protocol Buffers .NET Runtime Library API Reference

Protocol Buffers .NET Runtime Library

## [Google.Protobuf](https://protobuf.dev/reference/csharp/api-docs/namespace/google/protobuf.html)

| Classes                                                      |                                                              |
| :----------------------------------------------------------- | ------------------------------------------------------------ |
| [ByteString](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/byte-string.html) | Immutable array of bytes.                                    |
| [CodedInputStream](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/coded-input-stream.html) | Reads and decodes protocol message fields.                   |
| [CodedOutputStream](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/coded-output-stream.html) | Encodes and writes protocol message fields.                  |
| [CodedOutputStream.OutOfSpaceException](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/coded-output-stream/out-of-space-exception.html) | Indicates that a [CodedOutputStream](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/coded-output-stream.html#class_google_1_1_protobuf_1_1_coded_output_stream) wrapping a flat byte array ran out of space. |
| [FieldCodec](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/field-codec.html) | Factory methods for FieldCodec{T}.                           |
| [FieldCodec< T >](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/field-codec-t-.html) |                                                              |
| [InvalidJsonException](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/invalid-json-exception.html) | Thrown when an attempt is made to parse invalid JSON, e.g.   |
| [InvalidProtocolBufferException](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/invalid-protocol-buffer-exception.html) | Thrown when a protocol message being parsed is invalid in some way, e.g. |
| [JsonFormatter](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/json-formatter.html) | Reflection-based converter from messages to JSON.            |
| [JsonFormatter.Settings](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/json-formatter/settings.html) | [Settings](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/json-formatter/settings.html#class_google_1_1_protobuf_1_1_json_formatter_1_1_settings) controlling JSON formatting. |
| [JsonParser](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/json-parser.html) | Reflection-based converter from JSON to messages.            |
| [JsonParser.Settings](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/json-parser/settings.html) | [Settings](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/json-parser/settings.html#class_google_1_1_protobuf_1_1_json_parser_1_1_settings) controlling JSON parsing. |
| [MessageExtensions](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/message-extensions.html) | Extension methods on [IMessage](https://protobuf.dev/reference/csharp/api-docs/interface/google/protobuf/i-message.html#interface_google_1_1_protobuf_1_1_i_message) and IMessage{T}. |
| [MessageParser](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/message-parser.html) | A general message parser, typically used by reflection-based code as all the methods return simple [IMessage](https://protobuf.dev/reference/csharp/api-docs/interface/google/protobuf/i-message.html#interface_google_1_1_protobuf_1_1_i_message). |
| [MessageParser< T >](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/message-parser-t-.html) | A parser for a specific message type.                        |
| [ProtoPreconditions](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/proto-preconditions.html) | Helper methods for throwing exceptions when preconditions are not met. |
| [WireFormat](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/wire-format.html) | This class is used internally by the Protocol Buffer Library and generated message implementations. |

| Interfaces                                                   |                                                              |
| :----------------------------------------------------------- | ------------------------------------------------------------ |
| [ICustomDiagnosticMessage](https://protobuf.dev/reference/csharp/api-docs/interface/google/protobuf/i-custom-diagnostic-message.html) | A message type that has a custom string format for diagnostic purposes. |
| [IDeepCloneable< T >](https://protobuf.dev/reference/csharp/api-docs/interface/google/protobuf/i-deep-cloneable-t-.html) | Generic interface for a deeply cloneable type.               |
| [IMessage](https://protobuf.dev/reference/csharp/api-docs/interface/google/protobuf/i-message.html) | Interface for a Protocol Buffers message, supporting basic operations required for serialization. |
| [IMessage< T >](https://protobuf.dev/reference/csharp/api-docs/interface/google/protobuf/i-message-t-.html) | Generic interface for a Protocol Buffers message, where the type parameter is expected to be the same type as the implementation class. |

## [Google.Protobuf.Collections](https://protobuf.dev/reference/csharp/api-docs/namespace/google/protobuf/collections.html)

| Classes                                                      |                                                              |
| :----------------------------------------------------------- | ------------------------------------------------------------ |
| [MapField< TKey, TValue >](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/collections/map-field-t-key-t-value-.html) | Representation of a map field in a Protocol Buffer message.  |
| [MapField< TKey, TValue >.Codec](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/collections/map-field-t-key-t-value-/codec.html) | A codec for a specific map field.                            |
| [RepeatedField< T >](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/collections/repeated-field-t-.html) | The contents of a repeated field: essentially, a collection with some extra restrictions (no null values) and capabilities (deep cloning). |

## [Google.Protobuf.Reflection](https://protobuf.dev/reference/csharp/api-docs/namespace/google/protobuf/reflection.html)

| Classes                                                      |                                                              |
| :----------------------------------------------------------- | ------------------------------------------------------------ |
| [DescriptorBase](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/reflection/descriptor-base.html) | Base class for nearly all descriptors, providing common functionality. |
| [DescriptorValidationException](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/reflection/descriptor-validation-exception.html) | Thrown when building descriptors fails because the source DescriptorProtos are not valid. |
| [EnumDescriptor](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/reflection/enum-descriptor.html) | Descriptor for an enum type in a .proto file.                |
| [EnumValueDescriptor](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/reflection/enum-value-descriptor.html) | Descriptor for a single enum value within an enum in a .proto file. |
| [FieldDescriptor](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/reflection/field-descriptor.html) | Descriptor for a field or extension within a message in a .proto file. |
| [FileDescriptor](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/reflection/file-descriptor.html) | Describes a .proto file, including everything defined within. |
| [GeneratedClrTypeInfo](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/reflection/generated-clr-type-info.html) | Extra information provided by generated code when initializing a message or file descriptor. |
| [MessageDescriptor](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/reflection/message-descriptor.html) | Describes a message type.                                    |
| [MessageDescriptor.FieldCollection](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/reflection/message-descriptor/field-collection.html) | A collection to simplify retrieving the field accessor for a particular field. |
| [MethodDescriptor](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/reflection/method-descriptor.html) | Describes a single method in a service.                      |
| [OneofAccessor](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/reflection/oneof-accessor.html) | [Reflection](https://protobuf.dev/reference/csharp/api-docs/namespace/google/protobuf/reflection.html#namespace_google_1_1_protobuf_1_1_reflection) access for a oneof, allowing clear and "get case" actions. |
| [OneofDescriptor](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/reflection/oneof-descriptor.html) | Describes a "oneof" field collection in a message type: a set of fields of which at most one can be set in any particular message. |
| [OriginalNameAttribute](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/reflection/original-name-attribute.html) | Specifies the original name (in the .proto file) of a named element, such as an enum value. |
| [ServiceDescriptor](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/reflection/service-descriptor.html) | Describes a service type.                                    |
| [TypeRegistry](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/reflection/type-registry.html) | An immutable registry of types which can be looked up by their full name. |

| Interfaces                                                   |                                                |
| :----------------------------------------------------------- | ---------------------------------------------- |
| [IDescriptor](https://protobuf.dev/reference/csharp/api-docs/interface/google/protobuf/reflection/i-descriptor.html) | Interface implemented by all descriptor types. |
| [IFieldAccessor](https://protobuf.dev/reference/csharp/api-docs/interface/google/protobuf/reflection/i-field-accessor.html) | Allows fields to be reflectively accessed.     |

## [Google.Protobuf.WellKnownTypes](https://protobuf.dev/reference/csharp/api-docs/namespace/google/protobuf/well-known-types.html)

| Classes                                                      |                                                              |
| :----------------------------------------------------------- | ------------------------------------------------------------ |
| [Any](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/any.html) | `Any` contains an arbitrary serialized protocol buffer message along with a URL that describes the type of the serialized message. |
| [AnyReflection](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/any-reflection.html) | Holder for reflection information generated from google/protobuf/any.proto |
| [Api](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/api.html) | [Api](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/api.html#class_google_1_1_protobuf_1_1_well_known_types_1_1_api) is a light-weight descriptor for a protocol buffer service. |
| [ApiReflection](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/api-reflection.html) | Holder for reflection information generated from google/protobuf/api.proto |
| [BoolValue](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/bool-value.html) | Wrapper message for `bool`.                                  |
| [BytesValue](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/bytes-value.html) | Wrapper message for `bytes`.                                 |
| [DoubleValue](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/double-value.html) | Wrapper message for `double`.                                |
| [Duration](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/duration.html) | A [Duration](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/duration.html#class_google_1_1_protobuf_1_1_well_known_types_1_1_duration) represents a signed, fixed-length span of time represented as a count of seconds and fractions of seconds at nanosecond resolution. |
| [DurationReflection](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/duration-reflection.html) | Holder for reflection information generated from google/protobuf/duration.proto |
| [Empty](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/empty.html) | A generic empty message that you can re-use to avoid defining duplicated empty messages in your APIs. |
| [EmptyReflection](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/empty-reflection.html) | Holder for reflection information generated from google/protobuf/empty.proto |
| [Enum](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/enum.html) | [Enum](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/enum.html#class_google_1_1_protobuf_1_1_well_known_types_1_1_enum) type definition. |
| [EnumValue](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/enum-value.html) | [Enum](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/enum.html#class_google_1_1_protobuf_1_1_well_known_types_1_1_enum) value definition. |
| [Field](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/field.html) | A single field of a message type.                            |
| [Field.Types](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/field/types.html) | Container for nested types declared in the [Field](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/field.html#class_google_1_1_protobuf_1_1_well_known_types_1_1_field) message type. |
| [FieldMask](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/field-mask.html) | `FieldMask` represents a set of symbolic field paths, for example: |
| [FieldMaskReflection](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/field-mask-reflection.html) | Holder for reflection information generated from google/protobuf/field_mask.proto |
| [FloatValue](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/float-value.html) | Wrapper message for `float`.                                 |
| [Int32Value](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/int32-value.html) | Wrapper message for `int32`.                                 |
| [Int64Value](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/int64-value.html) | Wrapper message for `int64`.                                 |
| [ListValue](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/list-value.html) | `ListValue` is a wrapper around a repeated field of values.  |
| [Method](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/method.html) | [Method](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/method.html#class_google_1_1_protobuf_1_1_well_known_types_1_1_method) represents a method of an api. |
| [Mixin](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/mixin.html) | Declares an API to be included in this API.                  |
| [Option](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/option.html) | A protocol buffer option, which can be attached to a message, field, enumeration, etc. |
| [SourceContext](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/source-context.html) | `SourceContext` represents information about the source of a protobuf element, like the file in which it is defined. |
| [SourceContextReflection](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/source-context-reflection.html) | Holder for reflection information generated from google/protobuf/source_context.proto |
| [StringValue](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/string-value.html) | Wrapper message for `string`.                                |
| [Struct](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/struct.html) | `Struct` represents a structured data value, consisting of fields which map to dynamically typed values. |
| [StructReflection](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/struct-reflection.html) | Holder for reflection information generated from google/protobuf/struct.proto |
| [TimeExtensions](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/time-extensions.html) | Extension methods on BCL time-related types, converting to protobuf types. |
| [Timestamp](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/timestamp.html) | A [Timestamp](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/timestamp.html#class_google_1_1_protobuf_1_1_well_known_types_1_1_timestamp) represents a point in time independent of any time zone or calendar, represented as seconds and fractions of seconds at nanosecond resolution in UTC Epoch time. |
| [TimestampReflection](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/timestamp-reflection.html) | Holder for reflection information generated from google/protobuf/timestamp.proto |
| [Type](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/type.html) | A protocol buffer message type.                              |
| [TypeReflection](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/type-reflection.html) | Holder for reflection information generated from google/protobuf/type.proto |
| [UInt32Value](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/u-int32-value.html) | Wrapper message for `uint32`.                                |
| [UInt64Value](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/u-int64-value.html) | Wrapper message for `uint64`.                                |
| [Value](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/value.html) | `Value` represents a dynamically typed value which can be either null, a number, a string, a boolean, a recursive struct value, or a list of values. |
| [WrappersReflection](https://protobuf.dev/reference/csharp/api-docs/class/google/protobuf/well-known-types/wrappers-reflection.html) | Holder for reflection information generated from google/protobuf/wrappers.proto |
