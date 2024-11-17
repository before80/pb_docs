+++
title = "C++ API"
date = 2024-11-17T12:07:24+08:00
weight = 30
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文：[https://protobuf.dev/reference/cpp/api-docs/](https://protobuf.dev/reference/cpp/api-docs/)
>
> 收录该文档的时间：`2024-11-17T12:07:24+08:00`

# C++ Reference

This section contains reference documentation for working with protocol buffer classes in C++.



| Packages                                                     |
| ------------------------------------------------------------ |
| `google::protobuf`                                           |
| `google::protobuf::io`Auxiliary classes used for I/O.        |
| `google::protobuf::util`Utility classes.                     |
| `google::protobuf::compiler`Implementation of the Protocol Buffer compiler. |

## google::protobuf



| Files                                                        |
| ------------------------------------------------------------ |
| `google/protobuf/arena.h`This file defines an [Arena](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.arena#Arena) allocator for better allocation performance. |
| `google/protobuf/descriptor.h`This file contains classes which describe a type of protocol message. |
| `google/protobuf/descriptor.pb.h`Protocol buffer representations of descriptors. |
| `google/protobuf/descriptor_database.h`Interface for manipulating databases of descriptors. |
| `google/protobuf/dynamic_message.h`Defines an implementation of [Message](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.message#Message) which can emulate types which are not known at compile-time. |
| `google/protobuf/map.h`This file defines the map container and its helpers to support protobuf maps. |
| `google/protobuf/message.h`Defines [Message](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.message#Message), the abstract interface implemented by non-lite protocol message objects. |
| `google/protobuf/message_lite.h`Defines [MessageLite](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.message_lite#MessageLite), the abstract interface implemented by all (lite and non-lite) protocol message objects. |
| `google/protobuf/repeated_field.h`[RepeatedField](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.repeated_field#RepeatedField) and [RepeatedPtrField](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.repeated_field#RepeatedPtrField) are used by generated protocol message classes to manipulate repeated fields. |
| `google/protobuf/service.h`DEPRECATED: This module declares the abstract interfaces underlying proto2 RPC services. |
| `google/protobuf/text_format.h`Utilities for printing and parsing protocol messages in a human-readable, text-based format. |
| `google/protobuf/unknown_field_set.h`Contains classes used to keep track of unrecognized fields seen while parsing a protocol message. |

## google::protobuf::io

Auxiliary classes used for I/O.

The Protocol Buffer library uses the classes in this package to deal with I/O and encoding/decoding raw bytes. Most users will not need to deal with this package. However, users who want to adapt the system to work with their own I/O abstractions – e.g., to allow Protocol Buffers to be read from a different kind of input stream without the need for a temporary buffer – should take a closer look.

| Files                                                        |
| ------------------------------------------------------------ |
| `google/protobuf/io/coded_stream.h`This file contains the [CodedInputStream](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.io.coded_stream#CodedInputStream) and [CodedOutputStream](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.io.coded_stream#CodedOutputStream) classes, which wrap a [ZeroCopyInputStream](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.io.zero_copy_stream#ZeroCopyInputStream) or [ZeroCopyOutputStream](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.io.zero_copy_stream#ZeroCopyOutputStream), respectively, and allow you to read or write individual pieces of data in various formats. |
| `google/protobuf/io/printer.h`Utility class for writing text to a [ZeroCopyOutputStream](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.io.zero_copy_stream#ZeroCopyOutputStream). |
| `google/protobuf/io/tokenizer.h`Class for parsing tokenized text from a [ZeroCopyInputStream](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.io.zero_copy_stream#ZeroCopyInputStream). |
| `google/protobuf/io/zero_copy_stream.h`This file contains the [ZeroCopyInputStream](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.io.zero_copy_stream#ZeroCopyInputStream) and [ZeroCopyOutputStream](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.io.zero_copy_stream#ZeroCopyOutputStream) interfaces, which represent abstract I/O streams to and from which protocol buffers can be read and written. |
| `google/protobuf/io/zero_copy_stream_impl.h`This file contains common implementations of the interfaces defined in [zero_copy_stream.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.io.zero_copy_stream) which are only included in the full (non-lite) protobuf library. |
| `google/protobuf/io/zero_copy_stream_impl_lite.h`This file contains common implementations of the interfaces defined in [zero_copy_stream.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.io.zero_copy_stream) which are included in the "lite" protobuf library. |

## google::protobuf::util

Utility classes.

This package contains various utilities for message comparison, JSON conversion, well known types, etc.

| Files                                                        |
| ------------------------------------------------------------ |
| `google/protobuf/util/field_comparator.h`Defines classes for field comparison. |
| `google/protobuf/util/field_mask_util.h`Defines utilities for the FieldMask well known type. |
| `google/protobuf/util/json_util.h`Utility functions to convert between protobuf binary format and proto3 JSON format. |
| `google/protobuf/util/message_differencer.h`This file defines static methods and classes for comparing Protocol Messages. |
| `google/protobuf/util/time_util.h`Defines utilities for the Timestamp and Duration well known types. |
| `google/protobuf/util/type_resolver.h`Defines a TypeResolver for the Any message. |
| `google/protobuf/util/type_resolver_util.h`Defines utilities for the TypeResolver. |

## google::protobuf::compiler

Implementation of the Protocol Buffer compiler.

This package contains code for parsing .proto files and generating code based on them. There are two reasons you might be interested in this package:

- You want to parse .proto files at runtime. In this case, you should look at [importer.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.compiler.importer). Since this functionality is widely useful, it is included in the libprotobuf base library; you do not have to link against libprotoc.
- You want to write a custom protocol compiler which generates different kinds of code, e.g. code in a different language which is not supported by the official compiler. For this purpose, [command_line_interface.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.compiler.command_line_interface) provides you with a complete compiler front-end, so all you need to do is write a custom implementation of [CodeGenerator](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.compiler.code_generator#CodeGenerator) and a trivial main() function. You can even make your compiler support the official languages in addition to your own. Since this functionality is only useful to those writing custom compilers, it is in a separate library called "libprotoc" which you will have to link against.

| Files                                                        |
| ------------------------------------------------------------ |
| `google/protobuf/compiler/code_generator.h`Defines the abstract interface implemented by each of the language-specific code generators. |
| `google/protobuf/compiler/command_line_interface.h`Implements the Protocol Compiler front-end such that it may be reused by custom compilers written to support other languages. |
| `google/protobuf/compiler/importer.h`This file is the public interface to the .proto file parser. |
| `google/protobuf/compiler/parser.h`Implements parsing of .proto files to FileDescriptorProtos. |
| `google/protobuf/compiler/plugin.h`Front-end for protoc code generator plugins written in C++. |
| `google/protobuf/compiler/plugin.pb.h`API for protoc plugins. |
| `google/protobuf/compiler/cpp/cpp_generator.h`Generates C++ code for a given .proto file. |
| `google/protobuf/compiler/csharp/csharp_generator.h`Generates C# code for a given .proto file. |
| `google/protobuf/compiler/csharp/csharp_names.h`Provides a mechanism for mapping a descriptor to the fully-qualified name of the corresponding C# class. |
| `google/protobuf/compiler/java/java_generator.h`Generates Java code for a given .proto file. |
| `google/protobuf/compiler/java/java_names.h`Provides a mechanism for mapping a descriptor to the fully-qualified name of the corresponding Java class. |
| `google/protobuf/compiler/js/js_generator.h`Generates JavaScript code for a given .proto file. |
| `google/protobuf/compiler/objectivec/objectivec_generator.h`Generates ObjectiveC code for a given .proto file. |
| `google/protobuf/compiler/objectivec/objectivec_helpers.h`Helper functions for generating ObjectiveC code. |
| `google/protobuf/compiler/python/python_generator.h`Generates Python code for a given .proto file. |
| `google/protobuf/compiler/ruby/ruby_generator.h`Generates Ruby code for a given .proto file. |

------

##### [arena.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.arena/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [common.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.common/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [code_generator.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.compiler.code_generator/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [command_line_interface.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.compiler.command_line_interface/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [cpp_generator.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.compiler.cpp_generator/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [csharp_generator.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.compiler.csharp_generator/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [csharp_names.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.compiler.csharp_names/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [importer.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.compiler.importer/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [java_generator.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.compiler.java_generator/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [java_names.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.compiler.java_names/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [javanano_generator.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.compiler.javanano_generator/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [js_generator.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.compiler.js_generator/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [objectivec_generator.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.compiler.objectivec_generator/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [objectivec_helpers.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.compiler.objectivec_helpers/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [parser.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.compiler.parser/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [plugin.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.compiler.plugin/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [plugin.pb.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.compiler.plugin.pb/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [python_generator.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.compiler.python_generator/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [ruby_generator.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.compiler.ruby_generator/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [descriptor.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.descriptor/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [descriptor.pb.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.descriptor.pb/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [descriptor_database.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.descriptor_database/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [dynamic_message.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.dynamic_message/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [coded_stream.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.io.coded_stream/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [gzip_stream.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.io.gzip_stream/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [printer.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.io.printer/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [tokenizer.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.io.tokenizer/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [zero_copy_stream.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.io.zero_copy_stream/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [zero_copy_stream_impl.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.io.zero_copy_stream_impl/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [zero_copy_stream_iml_lite.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.io.zero_copy_stream_impl_lite/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [map.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.map/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [message.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.message/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [message_lite.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.message_lite/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [repeated_field.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.repeated_field/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [service.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.service/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [text_format.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.text_format/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [unknown_field_set.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.unknown_field_set/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [field_comparator.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.util.field_comparator/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [field_mask_util.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.util.field_mask_util/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [json_util.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.util.json_util/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [message_differencer.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.util.message_differencer/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [time_util.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.util.time_util/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [type_resolver.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.util.type_resolver/)

This section contains reference documentation for working with protocol buffer classes in C++.

##### [type_resolver_util.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.util.type_resolver_util/)

This section contains reference documentation for working with protocol buffer classes in C++.
