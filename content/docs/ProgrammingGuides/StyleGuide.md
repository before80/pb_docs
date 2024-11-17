+++
title = "Style Guide"
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

# Style Guide

Provides direction for how best to structure your proto definitions.



This document provides a style guide for `.proto` files. By following these conventions, you’ll make your protocol buffer message definitions and their corresponding classes consistent and easy to read.

Note that protocol buffer style has evolved over time, so it is likely that you will see `.proto` files written in different conventions or styles. **Respect the existing style** when you modify these files. **Consistency is key**. However, it is best to adopt the current best style when you are creating a new `.proto` file.

## Standard File Formatting

- Keep the line length to 80 characters.
- Use an indent of 2 spaces.
- Prefer the use of double quotes for strings.

## File Structure

Files should be named `lower_snake_case.proto`.

All files should be ordered in the following manner:

1. License header (if applicable)
2. File overview
3. Syntax
4. Package
5. Imports (sorted)
6. File options
7. Everything else

## Packages

Package names should be in lowercase. Package names should have unique names based on the project name, and possibly based on the path of the file containing the protocol buffer type definitions.

## Message and Field Names

Use PascalCase (with an initial capital) for message names: `SongServerRequest`. Prefer to capitalize abbreviations as single words: `GetDnsRequest` rather than `GetDNSRequest`. Use lower_snake_case for field names, including oneof field and extension names: `song_name`.

```proto
message SongServerRequest {
  optional string song_name = 1;
}
```

Using this naming convention for field names gives you accessors like those shown in the following two code samples.

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

## Repeated Fields

Use pluralized names for repeated fields.

```proto
repeated string keys = 1;
  ...
  repeated MyMessage accounts = 17;
```

## Enums

Use PascalCase (with an initial capital) for enum type names and CAPITALS_WITH_UNDERSCORES for value names:

```proto
enum FooBar {
  FOO_BAR_UNSPECIFIED = 0;
  FOO_BAR_FIRST_VALUE = 1;
  FOO_BAR_SECOND_VALUE = 2;
}
```

Each enum value should end with a semicolon, not a comma. Prefer prefixing enum values instead of surrounding them in an enclosing message. Since some languages don’t support an enum being defined inside a “struct” type, this ensures a consistent approach across binding languages.

The zero value enum should have the suffix `UNSPECIFIED`, because a server or application that gets an unexpected enum value will mark the field as unset in the proto instance. The field accessor will then return the default value, which for enum fields is the first enum value. For more information on the unspecified enum value, see [the Proto Best Practices page](https://protobuf.dev/programming-guides/dos-donts#unspecified-enum).

## Services

If your `.proto` defines an RPC service, you should use PascalCase (with an initial capital) for both the service name and any RPC method names:

```proto
service FooService {
  rpc GetSomething(GetSomethingRequest) returns (GetSomethingResponse);
  rpc ListSomething(ListSomethingRequest) returns (ListSomethingResponse);
}
```

For more service-related guidance, see [Create Unique Protos per Method](https://protobuf.dev/programming-guides/api#unique-protos) and [Don’t Include Primitive Types in a Top-level Request or Response Proto](https://protobuf.dev/programming-guides/api#dont-include-primitive-types) in the API Best Practices topic, and [Define Messages in Separate Files](https://protobuf.dev/programming-guides/dos-donts.md#separate-files) in Proto Best Practices.

## Things to Avoid

- Required fields (only for proto2)
- Groups (only for proto2)
