+++
title = "Version 2 Language Specification"
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

# Protocol Buffers Version 2 Language Specification

Language specification reference for version 2 of the Protocol Buffers language (proto2).



The syntax is specified using [Extended Backus-Naur Form (EBNF)](https://en.wikipedia.org/wiki/Extended_Backus–Naur_Form):

```fallback
|   alternation
()  grouping
[]  option (zero or one time)
{}  repetition (any number of times)
```

For more information about using proto2, see the [language guide](https://protobuf.dev/programming-guides/proto2).

## Lexical Elements

### Letters and Digits

```fallback
letter = "A" ... "Z" | "a" ... "z"
capitalLetter =  "A" ... "Z"
decimalDigit = "0" ... "9"
octalDigit   = "0" ... "7"
hexDigit     = "0" ... "9" | "A" ... "F" | "a" ... "f"
```

### Identifiers

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

### Integer Literals

```fallback
intLit     = decimalLit | octalLit | hexLit
decimalLit = [-] ( "1" ... "9" ) { decimalDigit }
octalLit   = [-] "0" { octalDigit }
hexLit     = [-] "0" ( "x" | "X" ) hexDigit { hexDigit }
```

### Floating-point Literals

```fallback
floatLit = [-] ( decimals "." [ decimals ] [ exponent ] | decimals exponent | "."decimals [ exponent ] ) | "inf" | "nan"
decimals  = [-] decimalDigit { decimalDigit }
exponent  = ( "e" | "E" ) [ "+" | "-" ] decimals
```

### Boolean

```fallback
boolLit = "true" | "false"
```

### String Literals

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

### EmptyStatement

```fallback
emptyStatement = ";"
```

### Constant

```gdscript3
constant = fullIdent | ( [ "-" | "+" ] intLit ) | ( [ "-" | "+" ] floatLit ) |
                strLit | boolLit | MessageValue
```

`MessageValue` is defined in the [Text Format Language Specification](https://protobuf.dev/reference/protobuf/textformat-spec#fields).

## Syntax

The syntax statement is used to define the protobuf version.

```fallback
syntax = "syntax" "=" ("'" "proto2" "'" | '"' "proto2" '"') ";"
```

## Import Statement

The import statement is used to import another .proto’s definitions.

```fallback
import = "import" [ "weak" | "public" ] strLit ";"
```

Example:

```proto
import public "other.proto";
```

## Package

The package specifier can be used to prevent name clashes between protocol message types.

```go
package = "package" fullIdent ";"
```

Example:

```proto
package foo.bar;
```

## Option

Options can be used in proto files, messages, enums and services. An option can be a protobuf defined option or a custom option. For more information, see [Options](https://protobuf.dev/programming-guides/proto2#options) in the language guide.

```gdscript3
option = "option" optionName  "=" constant ";"
optionName = ( ident | bracedFullIdent ) { "." ( ident | bracedFullIdent ) }
bracedFullIdent = "(" ["."] fullIdent ")"
```

For examples:

```proto
option java_package = "com.example.foo";
```

## Fields

Fields are the basic elements of a protocol buffer message. Fields can be normal fields, group fields, oneof fields, or map fields. A field has a label, type and field number.

```gdscript3
label = "required" | "optional" | "repeated"
type = "double" | "float" | "int32" | "int64" | "uint32" | "uint64"
      | "sint32" | "sint64" | "fixed32" | "fixed64" | "sfixed32" | "sfixed64"
      | "bool" | "string" | "bytes" | messageType | enumType
fieldNumber = intLit;
```

### Normal field

Each field has label, type, name and field number. It may have field options.

```gdscript3
field = label type fieldName "=" fieldNumber [ "[" fieldOptions "]" ] ";"
fieldOptions = fieldOption { ","  fieldOption }
fieldOption = optionName "=" constant
```

Examples:

```proto
optional foo.bar nested_message = 2;
repeated int32 samples = 4 [packed=true];
```

### Group field

**Note that this feature is deprecated and should not be used when creating new message types – use nested message types instead.**

Groups are one way to nest information in message definitions. The group name must begin with capital letter.

```fallback
group = label "group" groupName "=" fieldNumber messageBody
```

Example:

```proto
repeated group Result = 1 {
    required string url = 1;
    optional string title = 2;
    repeated string snippets = 3;
}
```

### Oneof and oneof field

A oneof consists of oneof fields and a oneof name. Oneof fields do not have labels.

```fallback
oneof = "oneof" oneofName "{" { option | oneofField } "}"
oneofField = type fieldName "=" fieldNumber [ "[" fieldOptions "]" ] ";"
```

Example:

```proto
oneof foo {
    string name = 4;
    SubMessage sub_message = 9;
}
```

### Map field

A map field has a key type, value type, name, and field number. The key type can be any integral or string type. Note, the key type may not be an enum.

```fallback
mapField = "map" "<" keyType "," type ">" mapName "=" fieldNumber [ "[" fieldOptions "]" ] ";"
keyType = "int32" | "int64" | "uint32" | "uint64" | "sint32" | "sint64" |
          "fixed32" | "fixed64" | "sfixed32" | "sfixed64" | "bool" | "string"
```

Example:

```proto
map<string, Project> projects = 3;
```

## Extensions and Reserved

Extensions and reserved are message elements that declare a range of field numbers or field names.

### Extensions

Extensions declare that a range of field numbers in a message are available for third-party extensions. Other people can declare new fields for your message type with those numeric tags in their own .proto files without having to edit the original file.

```fallback
extensions = "extensions" ranges ";"
ranges = range { "," range }
range =  intLit [ "to" ( intLit | "max" ) ]
```

Examples:

```proto
extensions 100 to 199;
extensions 4, 20 to max;
```

### Reserved

Reserved declares a range of field numbers or field names in a message that can not be used.

```fallback
reserved = "reserved" ( ranges | strFieldNames ) ";"
strFieldNames = strFieldName { "," strFieldName }
strFieldName = "'" fieldName "'" | '"' fieldName '"'
```

Examples:

```proto
reserved 2, 15, 9 to 11;
reserved "foo", "bar";
```

## Top Level definitions

### Enum definition

The enum definition consists of a name and an enum body. The enum body can have options, enum fields, and reserved statements.

```gdscript3
enum = "enum" enumName enumBody
enumBody = "{" { option | enumField | emptyStatement | reserved } "}"
enumField = ident "=" [ "-" ] intLit [ "[" enumValueOption { ","  enumValueOption } "]" ]";"
enumValueOption = optionName "=" constant
```

Example:

```proto
enum EnumAllowingAlias {
  option allow_alias = true;
  EAA_UNSPECIFIED = 0;
  EAA_STARTED = 1;
  EAA_RUNNING = 2 [(custom_option) = "hello world"];
}
```

### Message definition

A message consists of a message name and a message body. The message body can have fields, nested enum definitions, nested message definitions, extend statements, extensions, groups, options, oneofs, map fields, and reserved statements. A message cannot contain two fields with the same name in the same message schema.

```gdscript3
message = "message" messageName messageBody
messageBody = "{" { field | enum | message | extend | extensions | group |
option | oneof | mapField | reserved | emptyStatement } "}"
```

Example:

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

### Extend

If a message in the same or imported .proto file has reserved a range for extensions, the message can be extended.

```fallback
extend = "extend" messageType "{" {field | group} "}"
```

Example:

```proto
extend Foo {
  optional int32 bar = 126;
}
```

### Service definition

```fallback
service = "service" serviceName "{" { option | rpc | emptyStatement } "}"
rpc = "rpc" rpcName "(" [ "stream" ] messageType ")" "returns" "(" [ "stream" ]
messageType ")" (( "{" { option | emptyStatement } "}" ) | ";" )
```

Example:

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
