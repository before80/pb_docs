+++
title = "第 2 版语言规范"
date = 2024-11-17T12:07:24+08:00
weight = 10
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

> 原文：[https://protobuf.dev/reference/protobuf/proto2-spec/](https://protobuf.dev/reference/protobuf/proto2-spec/)
>
> 收录该文档的时间：`2024-11-17T12:07:24+08:00`

# Protocol Buffers Version 2 Language Specification - Protocol Buffers 第 2 版语言规范

Language specification reference for version 2 of the Protocol Buffers language (proto2).

​	版本 2 的 Protocol Buffers 语言（proto2）的语言规范参考。

The syntax is specified using [Extended Backus-Naur Form (EBNF)](https://en.wikipedia.org/wiki/Extended_Backus–Naur_Form):

​	语法使用 [扩展巴科斯-瑙尔范式 (EBNF)](https://en.wikipedia.org/wiki/Extended_Backus–Naur_Form) 指定：

```fallback
|   alternation 交替选择
()  grouping 分组
[]  option (zero or one time) 可选（零次或一次）
{}  repetition (any number of times) 重复（任意次数）
```

For more information about using proto2, see the [language guide]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2" >}}).

​	有关使用 proto2 的更多信息，请参阅 [语言指南]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2" >}})。

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

`MessageValue` is defined in the [Text Format Language Specification]({{< ref "/docs/ReferenceGuides/ProtocolBuffers/TextFormatLanguageSpecification#fields" >}}).

​	`MessageValue` 定义在 [文本格式语言规范]({{< ref "/docs/ReferenceGuides/ProtocolBuffers/TextFormatLanguageSpecification#fields" >}}) 中。

## 语法 Syntax

The syntax statement is used to define the protobuf version.

​	语法语句用于定义 protobuf 的版本。

```fallback
syntax = "syntax" "=" ("'" "proto2" "'" | '"' "proto2" '"') ";"
```

## 导入语句 Import Statement

The import statement is used to import another .proto’s definitions.

​	导入语句用于引入另一个 .proto 文件的定义。

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

​	包声明可以用于防止协议消息类型之间的命名冲突。

```go
package = "package" fullIdent ";"
```

Example:

​	示例：

```proto
package foo.bar;
```

## 选项 Option

Options can be used in proto files, messages, enums and services. An option can be a protobuf defined option or a custom option. For more information, see [Options]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2#options" >}}) in the language guide.

​	选项可用于 .proto 文件、消息、枚举和服务中。选项可以是 protobuf 定义的选项，也可以是自定义选项。有关更多信息，请参阅语言指南中的 [选项]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2#options" >}})。

```gdscript3
option = "option" optionName  "=" constant ";"
optionName = ( ident | bracedFullIdent ) { "." ( ident | bracedFullIdent ) }
bracedFullIdent = "(" ["."] fullIdent ")"
```

For examples:

​	示例：

```proto
option java_package = "com.example.foo";
```

## 字段 Fields

Fields are the basic elements of a protocol buffer message. Fields can be normal fields, group fields, oneof fields, or map fields. A field has a label, type and field number.

​	字段是协议缓冲区消息的基本元素。字段可以是普通字段、组字段、oneof 字段或映射字段。一个字段具有标签、类型和字段编号。

```gdscript3
label = "required" | "optional" | "repeated"
type = "double" | "float" | "int32" | "int64" | "uint32" | "uint64"
      | "sint32" | "sint64" | "fixed32" | "fixed64" | "sfixed32" | "sfixed64"
      | "bool" | "string" | "bytes" | messageType | enumType
fieldNumber = intLit;
```

### 普通字段 Normal field

Each field has label, type, name and field number. It may have field options.

​	每个字段具有标签、类型、名称和字段编号。它可能有字段选项。

```gdscript3
field = label type fieldName "=" fieldNumber [ "[" fieldOptions "]" ] ";"
fieldOptions = fieldOption { ","  fieldOption }
fieldOption = optionName "=" constant
```

Examples:

​	示例：

```proto
optional foo.bar nested_message = 2;
repeated int32 samples = 4 [packed=true];
```

### 组字段 Group field

**Note that this feature is deprecated and should not be used when creating new message types – use nested message types instead.**

​	注意：此功能已弃用，在创建新的消息类型时不应使用此功能——请使用嵌套消息类型代替。

Groups are one way to nest information in message definitions. The group name must begin with capital letter.

​	组是用于在消息定义中嵌套信息的一种方式。组名称必须以大写字母开头。

```fallback
group = label "group" groupName "=" fieldNumber messageBody
```

Example:

​	示例：

```proto
repeated group Result = 1 {
    required string url = 1;
    optional string title = 2;
    repeated string snippets = 3;
}
```

### Oneof 和 Oneof 字段 Oneof and oneof field

A oneof consists of oneof fields and a oneof name. Oneof fields do not have labels.

​	一个 oneof 由 oneof 字段和 oneof 名称组成。oneof 字段没有标签。

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

​	映射字段具有键类型、值类型、名称和字段编号。键类型可以是任何整数类型或字符串类型。注意，键类型不能是枚举类型。

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

​	扩展和保留是消息元素，用于声明一组字段编号或字段名称。

### 扩展 Extensions

Extensions declare that a range of field numbers in a message are available for third-party extensions. Other people can declare new fields for your message type with those numeric tags in their own .proto files without having to edit the original file.

​	扩展声明在消息中可用于第三方扩展的一组字段编号。其他人可以在他们自己的 .proto 文件中为您的消息类型声明这些编号的新字段，而无需编辑原始文件。

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

Reserved declares a range of field numbers or field names in a message that can not be used.

​	保留声明一组字段编号或字段名称，这些编号或名称在消息中不能使用。

```fallback
reserved = "reserved" ( ranges | strFieldNames ) ";"
strFieldNames = strFieldName { "," strFieldName }
strFieldName = "'" fieldName "'" | '"' fieldName '"'
```

Examples:

​	示例：

```proto
reserved 2, 15, 9 to 11;
reserved "foo", "bar";
```

## 顶层定义 Top Level definitions

### 枚举定义 Enum definition

The enum definition consists of a name and an enum body. The enum body can have options, enum fields, and reserved statements.

​	枚举定义由一个名称和一个枚举体组成。枚举体可以包含选项、枚举字段和保留声明。

```gdscript3
enum = "enum" enumName enumBody
enumBody = "{" { option | enumField | emptyStatement | reserved } "}"
enumField = ident "=" [ "-" ] intLit [ "[" enumValueOption { ","  enumValueOption } "]" ]";"
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

​	一个消息由消息名称和消息体组成。消息体可以包含字段、嵌套的枚举定义、嵌套的消息定义、扩展语句、扩展范围、组、选项、oneofs、映射字段和保留声明。消息不能包含在同一消息架构中具有相同名称的两个字段。

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

​	消息中声明的实体名称不得冲突。以下情况均被禁止：

```gdscript3
message MyMessage {
  optional string foo = 1;
  message foo {}
}

message MyMessage {
  optional string foo = 1;
  oneof foo {
    string bar = 2;
  }
}

message MyMessage {
  optional string foo = 1;
  extend Extendable {
    optional string foo = 2;
  }
}

message MyMessage {
  optional string foo = 1;
  enum E {
    foo = 0;
  }
}
```

### 扩展 Extend

If a message in the same or imported .proto file has reserved a range for extensions, the message can be extended.

​	如果同一文件或导入的 .proto 文件中的消息保留了一组扩展范围，则该消息可以被扩展。

```fallback
extend = "extend" messageType "{" {field | group} "}"
```

Example:

​	示例：

```proto
extend Foo {
  optional int32 bar = 126;
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

## Proto file

```gdscript3
proto = [syntax] { import | package | option | topLevelDef | emptyStatement }
topLevelDef = message | enum | extend | service
```

An example .proto file:

​	一个示例 .proto 文件：

```proto
syntax = "proto2";
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
    required int64 ival = 1;
  }
  repeated Inner inner_message = 2;
  optional EnumAllowingAlias enum_field = 3;
  map<int32, string> my_map = 4;
  extensions 20 to 30;
}
message Foo {
  optional group GroupMessage = 1 {
    optional bool a = 1;
  }
}
```
