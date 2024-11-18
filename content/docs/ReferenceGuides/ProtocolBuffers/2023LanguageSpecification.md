+++
title = "2023 语言规范"
date = 2024-11-17T12:07:24+08:00
weight = 1
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

> 原文：[https://protobuf.dev/reference/protobuf/edition-2023-spec/](https://protobuf.dev/reference/protobuf/edition-2023-spec/)
>
> 收录该文档的时间：`2024-11-17T12:07:24+08:00`

# Protocol Buffers Edition 2023 Language Specification - 协议缓冲区 Edition 2023 语言规范

Language specification reference for edition 2023 of the Protocol Buffers language.

​	Edition 2023 的协议缓冲区语言规范参考文档。

The syntax is specified using [Extended Backus-Naur Form (EBNF)](https://en.wikipedia.org/wiki/Extended_Backus–Naur_Form):

​	该语法使用 [扩展巴科斯范式 (EBNF)](https://en.wikipedia.org/wiki/Extended_Backus–Naur_Form) 指定：

```fallback
|   alternation 选择
()  grouping 分组
[]  option (zero or one time) 可选（零次或一次）
{}  repetition (any number of times) 重复（任意次数）
```

## 词法元素 Lexical Elements

### 字母和数字 Letters and Digits

```fallback
letter = "A" ... "Z" | "a" ... "z"
capitalLetter =  "A" ... "Z"
decimalDigit = "0" ... "9"
octalDigit   = "0" ... "7"
hexDigit     = "0" ... "9" | "A" ... "F" | "a" ... "f"
```

### 标识符 Identifiers

```gdscript3
ident = letter { letter | decimalDigit | "_" }
fullIdent = ident { "." ident }
messageName = ident
enumName = ident
fieldName = ident
oneofName = ident
mapName = ident
serviceName = ident
rpcName = ident
streamName = ident
messageType = [ "." ] { ident "." } messageName
enumType = [ "." ] { ident "." } enumName
groupName = capitalLetter { letter | decimalDigit | "_" }
```

### 整数字面量 Integer Literals

```fallback
intLit     = decimalLit | octalLit | hexLit
decimalLit = [-] ( "1" ... "9" ) { decimalDigit }
octalLit   = [-] "0" { octalDigit }
hexLit     = [-] "0" ( "x" | "X" ) hexDigit { hexDigit }
```

### 浮点数字面量 Floating-point Literals

```fallback
floatLit = [-] ( decimals "." [ decimals ] [ exponent ] | decimals exponent | "."decimals [ exponent ] ) | "inf" | "nan"
decimals  = [-] decimalDigit { decimalDigit }
exponent  = ( "e" | "E" ) [ "+" | "-" ] decimals
```

### 布尔值 Boolean

```fallback
boolLit = "true" | "false"
```

### 字符串字面量 String Literals

```fallback
strLit = strLitSingle { strLitSingle }
strLitSingle = ( "'" { charValue } "'" ) | ( '"' { charValue } '"' )
charValue = hexEscape | octEscape | charEscape | unicodeEscape | unicodeLongEscape | /[^\0\n\\]/
hexEscape = '\' ( "x" | "X" ) hexDigit [ hexDigit ]
octEscape = '\' octalDigit [ octalDigit [ octalDigit ] ]
charEscape = '\' ( "a" | "b" | "f" | "n" | "r" | "t" | "v" | '\' | "'" | '"' )
unicodeEscape = '\' "u" hexDigit hexDigit hexDigit hexDigit
unicodeLongEscape = '\' "U" ( "000" hexDigit hexDigit hexDigit hexDigit hexDigit |
                              "0010" hexDigit hexDigit hexDigit hexDigit
```

### 空语句 EmptyStatement

```fallback
emptyStatement = ";"
```

### 常量 Constant

```gdscript3
constant = fullIdent | ( [ "-" | "+" ] intLit ) | ( [ "-" | "+" ] floatLit ) |
                strLit | boolLit | MessageValue
```

`MessageValue` is defined in the [Text Format Language Specification](https://protobuf.dev/reference/protobuf/textformat-spec#fields).

​	`MessageValue` 定义在 [文本格式语言规范](https://protobuf.dev/reference/protobuf/textformat-spec#fields)中。

## Edition

The edition statement replaces the legacy `syntax` keyword, and is used to define the edition that this file is using.

​	`edition` 声明替代了传统的 `syntax` 关键字，用于定义文件使用的 Edition。

```fallback
edition = "edition" "=" [ ( "'" decimalLit "'" ) | ( '"' decimalLit '"' ) ] ";"
```

## 导入语句 Import Statement

The import statement is used to import another .proto’s definitions.

​	导入语句用于导入其他 .proto 文件的定义。

```fallback
import = "import" [ "weak" | "public" ] strLit ";"
```

Example:

​	示例：

```proto
import public "other.proto";
```

## Package

The package specifier can be used to prevent name clashes between protocol message types.

​	包规范可以用于防止协议消息类型之间的名称冲突。

```go
package = "package" fullIdent ";"
```

Example:

​	示例：

```proto
package foo.bar;
```

## 选项 Option

Options can be used in proto files, messages, enums and services. An option can be a protobuf defined option or a custom option. For more information, see [Options](https://protobuf.dev/programming-guides/proto2#options) in the language guide. Options are also be used to control [Feature Settings](https://protobuf.dev/editions/features).

​	选项可以用于 proto 文件、消息、枚举和服务。选项可以是协议缓冲区定义的选项或自定义选项。有关更多信息，请参阅语言指南中的 [选项](https://protobuf.dev/programming-guides/proto2#options)。选项也用于控制 [功能设置](https://protobuf.dev/editions/features)。

```gdscript3
option = "option" optionName  "=" constant ";"
optionName = ( ident | "(" ["."] fullIdent ")" )
```

For examples:

​	示例：

```proto
option java_package = "com.example.foo";
option features.enum_type = CLOSED;
```

## 字段 Fields

Fields are the basic elements of a protocol buffer message. Fields can be normal fields, group fields, oneof fields, or map fields. A field has a label, type and field number.

​	字段是协议缓冲区消息的基本元素。字段可以是普通字段、组字段、`oneof` 字段或映射字段。一个字段包含标签、类型和字段编号。

```gdscript3
label = [ "repeated" ]
type = "double" | "float" | "int32" | "int64" | "uint32" | "uint64"
      | "sint32" | "sint64" | "fixed32" | "fixed64" | "sfixed32" | "sfixed64"
      | "bool" | "string" | "bytes" | messageType | enumType
fieldNumber = intLit;
```

### 普通字段 Normal field

Each field has a label, type, name, and field number. It may have field options.

​	每个字段都有一个标签、类型、名称和字段编号。它可能还有字段选项。

```gdscript3
field = [label] type fieldName "=" fieldNumber [ "[" fieldOptions "]" ] ";"
fieldOptions = fieldOption { ","  fieldOption }
fieldOption = optionName "=" constant
```

Examples:

​	示例：

```proto
foo.bar nested_message = 2;
repeated int32 samples = 4 [packed=true];
```

### `oneof` 和 `oneof` 字段 Oneof and oneof field

A oneof consists of oneof fields and a oneof name. Oneof fields do not have labels.

​	一个 `oneof` 由多个 `oneof` 字段和一个 `oneof` 名称组成。`oneof` 字段没有标签。

```fallback
oneof = "oneof" oneofName "{" { option | oneofField } "}"
oneofField = type fieldName "=" fieldNumber [ "[" fieldOptions "]" ] ";"
```

Example:

​	示例：

```proto
oneof foo {
    string name = 4;
    SubMessage sub_message = 9;
}
```

### Map field

A map field has a key type, value type, name, and field number. The key type can be any integral or string type. Note, the key type may not be an enum.

​	一个映射字段包含键类型、值类型、名称和字段编号。键类型可以是任意整型或字符串类型，但不能是枚举类型。

```fallback
mapField = "map" "<" keyType "," type ">" mapName "=" fieldNumber [ "[" fieldOptions "]" ] ";"
keyType = "int32" | "int64" | "uint32" | "uint64" | "sint32" | "sint64" |
          "fixed32" | "fixed64" | "sfixed32" | "sfixed64" | "bool" | "string"
```

Example:

​	示例：

```proto
map<string, Project> projects = 3;
```

## 扩展和保留 Extensions and Reserved

Extensions and reserved are message elements that declare a range of field numbers or field names.

​	扩展和保留是消息元素，用于声明字段编号或字段名称的范围。

### 扩展 Extensions

Extensions declare that a range of field numbers in a message are available for third-party extensions. Other people can declare new fields for your message type with those numeric tags in their own .proto files without having to edit the original file.

​	扩展声明了一个消息中的字段编号范围，这些范围可供第三方扩展使用。其他人可以在自己的 `.proto` 文件中声明使用这些编号的新字段，而无需编辑原始文件。

```fallback
extensions = "extensions" ranges ";"
ranges = range { "," range }
range =  intLit [ "to" ( intLit | "max" ) ]
```

Examples:

​	示例：

```proto
extensions 100 to 199;
extensions 4, 20 to max;
```

### 保留 Reserved

Reserved declares a range of field numbers or names in a message or enum that can’t be used.

​	保留用于声明一个消息或枚举中不能使用的字段编号或名称范围。

```fallback
reserved = "reserved" ( ranges | reservedIdent ) ";"
fieldNames = fieldName { "," fieldName }
```

Examples:

​	示例：

```proto
reserved 2, 15, 9 to 11;
reserved foo, bar;
```

## 顶级定义 Top Level definitions

### 枚举定义 Enum definition

The enum definition consists of a name and an enum body. The enum body can have options, enum fields, and reserved statements.

​	枚举定义包含一个名称和一个枚举主体。枚举主体可以包含选项、枚举字段和保留声明。

```gdscript3
enum = "enum" enumName enumBody
enumBody = "{" { option | enumField | emptyStatement | reserved } "}"
enumField = fieldName "=" [ "-" ] intLit [ "[" enumValueOption { ","  enumValueOption } "]" ]";"
enumValueOption = optionName "=" constant
```

Example:

​	示例：

```proto
enum EnumAllowingAlias {
  option allow_alias = true;
  EAA_UNSPECIFIED = 0;
  EAA_STARTED = 1;
  EAA_RUNNING = 2 [(custom_option) = "hello world"];
}
```

### 消息定义 Message definition

A message consists of a message name and a message body. The message body can have fields, nested enum definitions, nested message definitions, extend statements, extensions, groups, options, oneofs, map fields, and reserved statements. A message cannot contain two fields with the same name in the same message schema.

​	一个消息由消息名称和消息主体组成。消息主体可以包含字段、嵌套枚举定义、嵌套消息定义、扩展声明、扩展范围、组、选项、`oneof`、映射字段以及保留声明。一个消息不能在同一模式中包含具有相同名称的两个字段。

```gdscript3
message = "message" messageName messageBody
messageBody = "{" { field | enum | message | extend | extensions | group |
option | oneof | mapField | reserved | emptyStatement } "}"
```

Example:

​	示例：

```proto
message Outer {
  option (my_option).a = true;
  message Inner {   // Level 2
    required int64 ival = 1;
  }
  map<int32, string> my_map = 2;
  extensions 20 to 30;
}
```

None of the entities declared inside a message may have conflicting names. All of the following are prohibited:

​	消息中声明的所有实体不得有冲突的名称。以下示例均不允许：

```gdscript3
message MyMessage {
  string foo = 1;
  message foo {}
}

message MyMessage {
  string foo = 1;
  oneof foo {
    string bar = 2;
  }
}

message MyMessage {
  string foo = 1;
  extend Extendable {
    string foo = 2;
  }
}

message MyMessage {
  string foo = 1;
  enum E {
    foo = 0;
  }
}
```

### 扩展 Extend

If a message in the same or imported .proto file has reserved a range for extensions, the message can be extended.

​	如果同一或导入的 `.proto` 文件中的某个消息保留了扩展的范围，则可以扩展该消息。

```fallback
extend = "extend" messageType "{" {field | group} "}"
```

Example:

​	示例：

```proto
extend Foo {
  int32 bar = 126;
}
```

### 服务定义 Service definition

```fallback
service = "service" serviceName "{" { option | rpc | emptyStatement } "}"
rpc = "rpc" rpcName "(" [ "stream" ] messageType ")" "returns" "(" [ "stream" ]
messageType ")" (( "{" { option | emptyStatement } "}" ) | ";" )
```

Example:

​	示例：

```proto
service SearchService {
  rpc Search (SearchRequest) returns (SearchResponse);
}
```

## Proto 文件 Proto file

```gdscript3
proto = [syntax] { import | package | option | topLevelDef | emptyStatement }
topLevelDef = message | enum | extend | service
```

An example .proto file:

​	一个`.proto` 文件示例：

```proto
edition = "2023";
import public "other.proto";
option java_package = "com.example.foo";
enum EnumAllowingAlias {
  option allow_alias = true;
  EAA_UNSPECIFIED = 0;
  EAA_STARTED = 1;
  EAA_RUNNING = 1;
  EAA_FINISHED = 2 [(custom_option) = "hello world"];
}
message Outer {
  option (my_option).a = true;
  message Inner {   // Level 2
    int64 ival = 1 [features.field_presence = LEGACY_REQUIRED];
  }
  repeated Inner inner_message = 2;
  EnumAllowingAlias enum_field = 3;
  map<int32, string> my_map = 4;
  extensions 20 to 30;
  reserved reserved_field;
}
message Foo {
  message GroupMessage {
    bool a = 1;
  }
  GroupMessage groupmessage = [features.message_encoding = DELIMITED];
}
```
