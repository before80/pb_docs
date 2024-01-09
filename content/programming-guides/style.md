+++
title = "风格指南"
weight = 50
description = "提供有关如何最好地构建您的 proto 定义的指导。"
type = "docs"

+++

## Style Guide 风格指南

Provides direction for how best to structure your proto definitions.

​	提供有关如何最好地构建您的 proto 定义的指导。



This document provides a style guide for `.proto` files. By following these conventions, you’ll make your protocol buffer message definitions and their corresponding classes consistent and easy to read.

​	此文档为 `.proto` 文件提供样式指南。通过遵循这些约定，您将使您的协议缓冲区消息定义及其对应的类保持一致且易于阅读。

Note that protocol buffer style has evolved over time, so it is likely that you will see `.proto` files written in different conventions or styles. **Respect the existing style** when you modify these files. **Consistency is key**. However, it is best to adopt the current best style when you are creating a new `.proto` file.

​	请注意，协议缓冲区样式随着时间的推移而演变，因此您可能会看到以不同约定或样式编写的 `.proto` 文件。修改这些文件时，请尊重现有样式。一致性是关键。但是，在创建新的 `.proto` 文件时，最好采用当前最佳样式。

## 标准文件格式 Standard File Formatting 

- Keep the line length to 80 characters.
- 将行长保持在 80 个字符内。
- Use an indent of 2 spaces.
- 使用 2 个空格的缩进。
- Prefer the use of double quotes for strings.
- 字符串首选使用双引号。

## 文件结构 File Structure 

Files should be named `lower_snake_case.proto`.

​	文件应命名为 `lower_snake_case.proto` 。

All files should be ordered in the following manner:

​	所有文件应按以下方式排序：

1. License header (if applicable)
2. 许可证页眉（如果适用）
3. File overview
4. 文件概述
5. Syntax
6. 语法
7. Package
8. 包
9. Imports (sorted)
10. 导入（已排序）
11. File options
12. 文件选项
13. Everything else
14. 其他所有内容

## 包 Packages 

Package names should be in lowercase. Package names should have unique names based on the project name, and possibly based on the path of the file containing the protocol buffer type definitions.

​	包名称应为小写。包名称应具有基于项目名称的唯一名称，并且可能基于包含协议缓冲区类型定义的文件的路径。

## 消息和字段名称 Message and Field Names 

Use PascalCase (with an initial capital) for message names – for example, `SongServerRequest`. Use lower_snake_case for field names (including oneof field and extension names) – for example, `song_name`.

​	对消息名称使用 PascalCase（以大写字母开头）——例如， `SongServerRequest` 。对字段名称（包括 oneof 字段和扩展名）使用 lower_snake_case——例如， `song_name` 。

```proto
message SongServerRequest {
  optional string song_name = 1;
}
```

Using this naming convention for field names gives you accessors like those shown in the following two code samples.

​	对字段名称使用此命名约定可让您访问如下两个代码示例中所示的访问器。

C++:

```cpp
const string& song_name() { ... }
void set_song_name(const string& x) { ... }
```

Java：

```java
public String getSongName() { ... }
public Builder setSongName(String v) { ... }
```

If your field name contains a number, the number should appear after the letter instead of after the underscore. For example, use `song_name1` instead of `song_name_1`

​	如果字段名称包含数字，则数字应出现在字母之后，而不是下划线之后。例如，使用 `song_name1` 而不是 `song_name_1`

## 重复字段 Repeated Fields 

Use pluralized names for repeated fields.

​	对重复字段使用复数名称。

```proto
repeated string keys = 1;
  ...
  repeated MyMessage accounts = 17;
```

## 枚举 Enums 

Use PascalCase (with an initial capital) for enum type names and CAPITALS_WITH_UNDERSCORES for value names:

​	对枚举类型名称使用 PascalCase（首字母大写），对值名称使用 CAPITALS_WITH_UNDERSCORES：

```proto
enum FooBar {
  FOO_BAR_UNSPECIFIED = 0;
  FOO_BAR_FIRST_VALUE = 1;
  FOO_BAR_SECOND_VALUE = 2;
}
```

Each enum value should end with a semicolon, not a comma. Prefer prefixing enum values instead of surrounding them in an enclosing message. Since some languages don’t support an enum being defined inside a “struct” type, this ensures a consistent approach across binding languages.

​	每个枚举值应以分号结尾，而不是逗号。优先使用枚举值前缀，而不是将它们包围在封闭消息中。由于某些语言不支持在“struct”类型中定义枚举，因此这可确保跨绑定语言的一致方法。

The zero value enum should have the suffix `UNSPECIFIED`, because a server or application that gets an unexpected enum value will mark the field as unset in the proto instance. The field accessor will then return the default value, which for enum fields is the first enum value. For more information on the unspecified enum value, see [the Proto Best Practices page](https://protobuf.dev/programming-guides/dos-donts#unspecified-enum).

​	零值枚举应具有后缀 `UNSPECIFIED` ，因为获取意外枚举值的服务器或应用程序将在 proto 实例中将该字段标记为未设置。然后，字段访问器将返回默认值，对于枚举字段，默认值是第一个枚举值。有关未指定枚举值的更多信息，请参阅 Proto 最佳做法页面。

## 服务 Services 

If your `.proto` defines an RPC service, you should use PascalCase (with an initial capital) for both the service name and any RPC method names:

​	如果您的 `.proto` 定义了 RPC 服务，则应为服务名称和任何 RPC 方法名称使用 PascalCase（首字母大写）：

```proto
service FooService {
  rpc GetSomething(GetSomethingRequest) returns (GetSomethingResponse);
  rpc ListSomething(ListSomethingRequest) returns (ListSomethingResponse);
}
```

## 要避免的事项 Things to Avoid 

- Required fields (only for proto2)
- 必需字段（仅适用于 proto2）
- Groups (only for proto2)
- 组（仅适用于 proto2）
