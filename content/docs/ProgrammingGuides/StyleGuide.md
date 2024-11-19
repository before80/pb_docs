+++
title = "样式指南"
date = 2024-11-17T09:35:36+08:00
weight = 40
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

> 原文：[https://protobuf.dev/programming-guides/style/](https://protobuf.dev/programming-guides/style/)
>
> 收录该文档的时间：`2024-11-17T09:35:36+08:00`

# Style Guide - 样式指南

Provides direction for how best to structure your proto definitions.

​	提供有关如何最佳地组织您的 proto 定义的指导。

This document provides a style guide for `.proto` files. By following these conventions, you’ll make your protocol buffer message definitions and their corresponding classes consistent and easy to read.

​	本文档为 `.proto` 文件提供了样式指南。通过遵循这些约定，您可以使协议缓冲区消息定义及其对应的类保持一致并易于阅读。

Note that protocol buffer style has evolved over time, so it is likely that you will see `.proto` files written in different conventions or styles. **Respect the existing style** when you modify these files. **Consistency is key**. However, it is best to adopt the current best style when you are creating a new `.proto` file.

​	请注意，协议缓冲区的样式随着时间的发展而演变，因此您可能会看到以不同约定或样式编写的 `.proto` 文件。在修改这些文件时，**尊重现有样式**。**一致性是关键**。然而，在创建新的 `.proto` 文件时，最好采用当前的最佳样式。

## 标准文件格式 Standard File Formatting

- Keep the line length to 80 characters.
  - 将行长度限制为 80 个字符。

- Use an indent of 2 spaces.
  - 使用 2 个空格缩进。

- Prefer the use of double quotes for strings.
  - 优先使用双引号表示字符串。

## 文件结构 File Structure

Files should be named `lower_snake_case.proto`.

​	文件应命名为 `lower_snake_case.proto`。

All files should be ordered in the following manner:

​	所有文件应按以下顺序排列：

1. License header (if applicable) 许可证头（如适用）
2. File overview 文件概述
3. Syntax 语法
4. Package
5. Imports (sorted) 导入（按字母顺序排序）
6. File options 文件选项
7. Everything else 其他内容

## Packages

Package names should be in lowercase. Package names should have unique names based on the project name, and possibly based on the path of the file containing the protocol buffer type definitions.

​	包名称应为小写。包名称应基于项目名称唯一命名，并可能基于包含协议缓冲区类型定义的文件路径。

## 消息和字段名称 Message and Field Names

Use PascalCase (with an initial capital) for message names: `SongServerRequest`. Prefer to capitalize abbreviations as single words: `GetDnsRequest` rather than `GetDNSRequest`. Use lower_snake_case for field names, including oneof field and extension names: `song_name`.

​	使用 PascalCase（首字母大写）表示消息名称：`SongServerRequest`。建议将缩写作为单词处理：`GetDnsRequest` 而不是 `GetDNSRequest`。字段名称应使用 lower_snake_case，包括 oneof 字段和扩展名称：`song_name`。

```proto
message SongServerRequest {
  optional string song_name = 1;
}
```

Using this naming convention for field names gives you accessors like those shown in the following two code samples.

​	使用这种命名约定为字段名称生成的访问器如以下代码示例所示。

C++:

```cpp
const string& song_name() { ... }
void set_song_name(const string& x) { ... }
```

Java:

```java
public String getSongName() { ... }
public Builder setSongName(String v) { ... }
```

If your field name contains a number, the number should appear after the letter instead of after the underscore. For example, use `song_name1` instead of `song_name_1`

​	如果字段名称中包含数字，数字应出现在字母后面，而不是下划线之后。例如，应使用 `song_name1` 而不是 `song_name_1`。

## 重复字段 Repeated Fields

Use pluralized names for repeated fields.

​	对于重复字段，使用复数名称。

```proto
repeated string keys = 1;
  ...
  repeated MyMessage accounts = 17;
```

## 枚举 Enums

Use PascalCase (with an initial capital) for enum type names and CAPITALS_WITH_UNDERSCORES for value names:

​	使用 PascalCase（首字母大写）表示枚举类型名称，使用全大写加下划线表示值名称：

```proto
enum FooBar {
  FOO_BAR_UNSPECIFIED = 0;
  FOO_BAR_FIRST_VALUE = 1;
  FOO_BAR_SECOND_VALUE = 2;
}
```

Each enum value should end with a semicolon, not a comma. Prefer prefixing enum values instead of surrounding them in an enclosing message. Since some languages don’t support an enum being defined inside a “struct” type, this ensures a consistent approach across binding languages.

​	每个枚举值应以分号结尾，而不是逗号。优先为枚举值添加前缀，而不是将其包裹在封闭消息中。因为某些语言不支持在“结构”类型中定义枚举，这确保了跨绑定语言的一致性。

The zero value enum should have the suffix `UNSPECIFIED`, because a server or application that gets an unexpected enum value will mark the field as unset in the proto instance. The field accessor will then return the default value, which for enum fields is the first enum value. For more information on the unspecified enum value, see [the Proto Best Practices page]({{< ref "/docs/ProgrammingGuides/ProtoBestPractices#unspecified-enum" >}}).

​	零值枚举应以 `UNSPECIFIED` 为后缀，因为服务器或应用程序获取到意外的枚举值时，会将该字段标记为 proto 实例中的未设置状态。字段访问器随后将返回默认值，对于枚举字段，默认值是第一个枚举值。有关未指定枚举值的更多信息，请参见 [Proto Best Practices 页面]({{< ref "/docs/ProgrammingGuides/ProtoBestPractices#unspecified-enum" >}})。

## 服务 Services

If your `.proto` defines an RPC service, you should use PascalCase (with an initial capital) for both the service name and any RPC method names:

​	如果 `.proto` 定义了一个 RPC 服务，您应使用 PascalCase（首字母大写）表示服务名称以及任何 RPC 方法名称：

```proto
service FooService {
  rpc GetSomething(GetSomethingRequest) returns (GetSomethingResponse);
  rpc ListSomething(ListSomethingRequest) returns (ListSomethingResponse);
}
```

For more service-related guidance, see [Create Unique Protos per Method]({{< ref "/docs/ProgrammingGuides/APIBestPractices#unique-protos" >}}) and [Don’t Include Primitive Types in a Top-level Request or Response Proto]({{< ref "/docs/ProgrammingGuides/APIBestPractices#dont-include-primitive-types" >}}) in the API Best Practices topic, and [Define Messages in Separate Files](https://protobuf.dev/programming-guides/dos-donts.md#separate-files) in Proto Best Practices.

​	有关更多与服务相关的指导，请参见 [Create Unique Protos per Method]({{< ref "/docs/ProgrammingGuides/APIBestPractices#unique-protos" >}}) 和 [Don’t Include Primitive Types in a Top-level Request or Response Proto]({{< ref "/docs/ProgrammingGuides/APIBestPractices#dont-include-primitive-types" >}}) 中的 API 最佳实践主题，以及 [Define Messages in Separate Files](https://protobuf.dev/programming-guides/dos-donts.md#separate-files) 中的 Proto 最佳实践。

## 应避免的事项 Things to Avoid

- Required fields (only for proto2)
  - 必须字段（仅适用于 proto2）
- Groups (only for proto2)
  - 组（仅适用于 proto2）
