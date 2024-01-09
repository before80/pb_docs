+++
title = "Protocol Buffers 3 版语言规范"
weight = 810
linkTitle = "Version 3 Language Specification"
description = "Protocol Buffers 语言 (proto3) 3 版的语言规范参考。"
type = "docs"

+++



## Protocol Buffers Version 3 Language Specification - Protocol Buffers 3 版语言规范

Language specification reference for version 3 of the Protocol Buffers language (proto3).

​	Protocol Buffers 语言 (proto3) 3 版的语言规范参考。



The syntax is specified using [Extended Backus-Naur Form (EBNF)](https://en.wikipedia.org/wiki/Extended_Backus–Naur_Form):

​	语法使用扩展巴科斯范式 (EBNF) 指定：

```fallback
|   alternation
()  grouping
[]  option (zero or one time)
{}  repetition (any number of times)
```

For more information about using proto3, see the [language guide](https://protobuf.dev/programming-guides/proto3).

​	有关使用 proto3 的更多信息，请参阅语言指南。

## 词法元素 Lexical Elements 

### 字母和数字 Letters and Digits 

```fallback
letter = "A" ... "Z" | "a" ... "z"
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
messageType = [ "." ] { ident "." } messageName
enumType = [ "." ] { ident "." } enumName
```

### 整数字面量 Integer Literals 

```fallback
intLit     = decimalLit | octalLit | hexLit
decimalLit = ( "1" ... "9" ) { decimalDigit }
octalLit   = "0" { octalDigit }
hexLit     = "0" ( "x" | "X" ) hexDigit { hexDigit }
```

### 浮点字面量 Floating-point Literals 

```fallback
floatLit = ( decimals "." [ decimals ] [ exponent ] | decimals exponent | "."decimals [ exponent ] ) | "inf" | "nan"
decimals  = decimalDigit { decimalDigit }
exponent  = ( "e" | "E" ) [ "+" | "-" ] decimals
```

### 布尔值 Boolean 

```fallback
boolLit = "true" | "false"
```

### 字符串字面量 String Literals 

```fallback
strLit = strLitSingle { strLitSingle }
strLitSingle = ( "'" { charValue } "'" ) |  ( '"' { charValue } '"' )
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

​	`MessageValue` 在文本格式语言规范中定义。

## 语法 Syntax 

The syntax statement is used to define the protobuf version.

​	语法语句用于定义 protobuf 版本。

```fallback
syntax = "syntax" "=" ("'" "proto3" "'" | '"' "proto3" '"') ";"
```

Example:

​	示例：

```proto
syntax = "proto3";
```

## 导入语句 Import Statement 

The import statement is used to import another .proto’s definitions.

​	导入语句用于导入另一个 .proto 的定义。

```fallback
import = "import" [ "weak" | "public" ] strLit ";"
```

Example:

​	示例：

```proto
import public "other.proto";
```

## 包 Package 

The package specifier can be used to prevent name clashes between protocol message types.

​	包说明符可用于防止协议消息类型之间的名称冲突。

```go
package = "package" fullIdent ";"
```

Example:

​	示例：

```proto
package foo.bar;
```

## 选项 Option 

Options can be used in proto files, messages, enums and services. An option can be a protobuf defined option or a custom option. For more information, see [Options](https://protobuf.dev/programming-guides/proto3#options) in the language guide.

​	选项可用于 proto 文件、消息、枚举和服务。选项可以是 protobuf 定义的选项或自定义选项。有关更多信息，请参阅语言指南中的选项。

```gdscript3
option = "option" optionName  "=" constant ";"
optionName = optionNamePart { "." optionNamePart }
optionNamePart = { ident | "(" ["."] fullIdent ")" }
```

Example:
示例：

```proto
option java_package = "com.example.foo";
```

## 字段 Fields 

Fields are the basic elements of a protocol buffer message. Fields can be normal fields, oneof fields, or map fields. A field has a type and field number.

​	字段是协议缓冲区消息的基本元素。字段可以是普通字段、oneof 字段或 map 字段。字段具有类型和字段编号。

```gdscript3
type = "double" | "float" | "int32" | "int64" | "uint32" | "uint64"
      | "sint32" | "sint64" | "fixed32" | "fixed64" | "sfixed32" | "sfixed64"
      | "bool" | "string" | "bytes" | messageType | enumType
fieldNumber = intLit;
```

### 普通字段 Normal Field 

Each field has type, name and field number. It may have field options.

​	每个字段都有类型、名称和字段编号。它可能具有字段选项。

```gdscript3
field = [ "repeated" ] type fieldName "=" fieldNumber [ "[" fieldOptions "]" ] ";"
fieldOptions = fieldOption { ","  fieldOption }
fieldOption = optionName "=" constant
```

Examples:

​	示例：

```proto
foo.Bar nested_message = 2;
repeated int32 samples = 4 [packed=true];
```

### Oneof 和 Oneof 字段 Oneof and Oneof Field 

A oneof consists of oneof fields and a oneof name.

​	Oneof 由 oneof 字段和 oneof 名称组成。

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

### Map 字段 Map Field 

A map field has a key type, value type, name, and field number. The key type can be any integral or string type.

​	Map 字段具有键类型、值类型、名称和字段编号。键类型可以是任何整数或字符串类型。

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

## 保留 Reserved 

Reserved statements declare a range of field numbers or field names that cannot be used in this message.

​	保留语句声明一系列不能在此消息中使用的字段编号或字段名称。

```fallback
reserved = "reserved" ( ranges | strFieldNames ) ";"
ranges = range { "," range }
range =  intLit [ "to" ( intLit | "max" ) ]
strFieldNames = strFieldName { "," strFieldName }
strFieldName = "'" fieldName "'" | '"' fieldName '"'
```

Examples:

​	示例：

```proto
reserved 2, 15, 9 to 11;
reserved "foo", "bar";
```

## 顶级定义 Top Level Definitions 

### 枚举定义 Enum Definition 

The enum definition consists of a name and an enum body. The enum body can have options, enum fields, and reserved statements.

​	枚举定义由名称和枚举主体组成。枚举主体可以包含选项、枚举字段和保留语句。

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

### 消息定义 Message Definition 

A message consists of a message name and a message body. The message body can have fields, nested enum definitions, nested message definitions, options, oneofs, map fields, and reserved statements. A message cannot contain two fields with the same name in the same message schema.

​	消息由消息名称和消息主体组成。消息主体可以包含字段、嵌套枚举定义、嵌套消息定义、选项、oneof、映射字段和保留语句。消息不能在同一个消息模式中包含两个同名的字段。

```gdscript3
message = "message" messageName messageBody
messageBody = "{" { field | enum | message | option | oneof | mapField |
reserved | emptyStatement } "}"
```

Example:

​	示例：

```proto
message Outer {
  option (my_option).a = true;
  message Inner {   // Level 2
    int64 ival = 1;
  }
  map<int32, string> my_map = 2;
}
```

None of the entities declared inside a message may have conflicting names. All of the following are prohibited:

​	在消息中声明的任何实体都不能有冲突的名称。以下所有情况均被禁止：

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
  enum E {
    foo = 0;
  }
}
```

### 服务定义 Service Definition 

```fallback
service = "service" serviceName "{" { option | rpc | emptyStatement } "}"
rpc = "rpc" rpcName "(" [ "stream" ] messageType ")" "returns" "(" [ "stream" ]
messageType ")" (( "{" {option | emptyStatement } "}" ) | ";")
```

Example:

​	示例：

```proto
service SearchService {
  rpc Search (SearchRequest) returns (SearchResponse);
}
```

## Proto 文件 Proto File 

```gdscript3
proto = syntax { import | package | option | topLevelDef | emptyStatement }
topLevelDef = message | enum | service
```

An example .proto file:

​	一个示例 .proto 文件：

```proto
syntax = "proto3";
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
    int64 ival = 1;
  }
  repeated Inner inner_message = 2;
  EnumAllowingAlias enum_field = 3;
  map<int32, string> my_map = 4;
}
```
