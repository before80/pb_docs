+++
title = "PHP Generated Code Guide"
weight = 730
linkTitle = "Generated Code Guide"
description = "Describes the PHP code that the protocol buffer compiler generates for any given protocol definition."
type = "docs"
+++

# PHP Generated Code Guide PHP 生成代码指南

Describes the PHP code that the protocol buffer compiler generates for any given protocol definition.
描述协议缓冲区编译器为任何给定的协议定义生成的 PHP 代码。



You should read the [proto3 language guide](https://protobuf.dev/programming-guides/proto3) before reading this document. Note that the protocol buffer compiler currently only supports proto3 code generation for PHP.
在阅读本文档之前，您应该阅读 proto3 语言指南。请注意，协议缓冲区编译器目前仅支持 PHP 的 proto3 代码生成。

## Compiler Invocation 编译器调用

The protocol buffer compiler produces PHP output when invoked with the `--php_out=` command-line flag. The parameter to the `--php_out=` option is the directory where you want the compiler to write your PHP output. In order to conform to PSR-4, the compiler creates a sub-directory corresponding to the package defined in the proto file. In addition, for each message in the proto file input the compiler creates a separate file in the package’s subdirectory. The names of the output files of messages are composed of three parts:
使用 `--php_out=` 命令行标志调用协议缓冲区编译器时，它会生成 PHP 输出。 `--php_out=` 选项的参数是要让编译器写入 PHP 输出的目录。为了符合 PSR-4，编译器会创建一个与 proto 文件中定义的包对应的子目录。此外，对于 proto 文件输入中的每条消息，编译器都会在包的子目录中创建一个单独的文件。消息的输出文件的名称由三部分组成：

- Base directory: The proto path (specified with the `--proto_path=` or `-I` command-line flag) is replaced with the output path (specified with the `--php_out=` flag).
  基本目录：用输出路径（使用 `--php_out=` 标志指定）替换 proto 路径（使用 `--proto_path=` 或 `-I` 命令行标志指定）。
- Sub-directory: `.` in the package name are replaced by the operating system directory separator. Each package name component is capitalized.
  子目录：包名称中的 `.` 用操作系统目录分隔符替换。每个包名称组件都采用大写字母。
- File: The message name is appended by `.php`.
  文件：消息名称后追加 `.php` 。

So, for example, let’s say you invoke the compiler as follows:
因此，例如，假设您按如下方式调用编译器：

```shell
protoc --proto_path=src --php_out=build/gen src/example.proto
```

And `src/example.proto` is defined as:
并且 `src/example.proto` 定义为：

```proto
package foo.bar;
message MyMessage {}
```

The compiler will read the files `src/foo.proto` and produce the output file: `build/gen/Foo/Bar/MyMessage.php`. The compiler will automatically create the directory `build/gen/Foo/Bar` if necessary, but it will *not* create `build` or `build/gen`; they must already exist.
编译器将读取文件 `src/foo.proto` 并生成输出文件： `build/gen/Foo/Bar/MyMessage.php` 。编译器将在必要时自动创建目录 `build/gen/Foo/Bar` ，但不会创建 `build` 或 `build/gen` ；它们必须已存在。

## Packages 包裹

The package name defined in the `.proto` file is used by default to generate a module structure for the generated PHP classes. Given a file like:
在 `.proto` 文件中定义的包名称默认用于为生成的 PHP 类生成模块结构。给定如下文件：

```proto
package foo.bar;

message MyMessage {}
```

The protocol compiler generates an output class with the name `Foo\Bar\MyMessage`.
协议编译器生成一个名为 `Foo\Bar\MyMessage` 的输出类。

### Namespace Options 命名空间选项

The compiler supports additional options to define the PHP and metadata namespace. When defined, these are used to generate the module structure and the namespace. Given options like:
编译器支持其他选项来定义 PHP 和元数据命名空间。定义后，这些选项用于生成模块结构和命名空间。给定如下选项：

```proto
package foo.bar;
option php_namespace = "baz\\qux";
option php_metadata_namespace = "Foo";
message MyMessage {}
```

The protocol compiler generates an output class with the name `baz\qux\MyMessage`. The class will have the namespace `namespace baz\qux`.
协议编译器生成一个名为 `baz\qux\MyMessage` 的输出类。该类将具有命名空间 `namespace baz\qux` 。

The protocol compiler generates a metadata class with the name `Foo\Metadata`. The class will have the namespace `namespace Foo`.
协议编译器生成一个名为 `Foo\Metadata` 的元数据类。该类将具有命名空间 `namespace Foo` 。

*The options generated are case-sensitive. By default, the package is converted to Pascal case.
生成的选项区分大小写。默认情况下，包转换为帕斯卡命名法。*

## Messages 消息

Given a simple message declaration:
给定一个简单的消息声明：

```proto
message Foo {}
```

The protocol buffer compiler generates a PHP class called `Foo`. This class inherits from a common base class, `Google\Protobuf\Internal\Message`, which provides methods for encoding and decoding your message types, as shown in the following example:
协议缓冲区编译器生成一个名为 `Foo` 的 PHP 类。此类继承自一个公共基类 `Google\Protobuf\Internal\Message` ，该类提供用于对消息类型进行编码和解码的方法，如下例所示：

```php
$from = new Foo();
$from->setInt32(1);
$from->setString('a');
$from->getRepeatedInt32()[] = 1;
$from->getMapInt32Int32()[1] = 1;
$data = $from->serializeToString();
try {
  $to->mergeFromString($data);
} catch (Exception $e) {
  // Handle parsing error from invalid data.
  ...
}
```

You should *not* create your own `Foo` subclasses. Generated classes are not designed for subclassing and may lead to "fragile base class" problems.
您不应创建自己的 `Foo` 子类。生成的类不适用于子类化，并且可能导致“脆弱的基类”问题。

Nested messages result in a PHP class of the same name prefixed by their containing message and separated by underscores, as PHP doesn’t support nested classes. So, for example, if you have this in your `.proto`:
嵌套消息导致一个 PHP 类，其名称与包含它的消息相同，并用下划线分隔，因为 PHP 不支持嵌套类。因此，例如，如果您在 `.proto` 中有以下内容：

```proto
message TestMessage {
  optional int32 a = 1;
  message NestedMessage {...}
}
```

The compiler will generate the following classes:
编译器将生成以下类：

```php
class TestMessage {
  public a;
}

// PHP doesn’t support nested classes.
class TestMessage_NestedMessage {...}
```

If the message class name is reserved (for example, `Empty`), the prefix `PB` is prepended to the class name:
如果消息类名称是保留的（例如 `Empty` ），则前缀 `PB` 会被预先添加到类名称：

```php
class PBEmpty {...}
```

We have also provided the file level option `php_class_prefix`. If this is specified, it is prepended to all generated message classes.
我们还提供了文件级别选项 `php_class_prefix` 。如果指定了此选项，则会将其预先添加到所有生成的 message 类。

## Fields 字段

For each field in a message type, there are accessor methods to set and get the field. So given a field `x` you can write:
对于消息类型中的每个字段，都有访问器方法来设置和获取该字段。因此，给定一个字段 `x` ，您可以编写：

```php
$m = new MyMessage();
$m->setX(1);
$val = $m->getX();

$a = 1;
$m->setX($a);
```

Whenever you set a field, the value is type-checked against the declared type of that field. If the value is of the wrong type (or out of range), an exception will be raised. By default type conversions (for example, when assigning a value to a field or adding an element to a repeated field) are permitted to and from integer, float, and numeric strings. Conversions that are not permitted include all conversions to/from arrays or objects. Float to integer overflow conversions are undefined.
无论何时设置字段，都会根据该字段的声明类型对值进行类型检查。如果值类型错误（或超出范围），将引发异常。默认情况下，允许类型转换（例如，将值分配给字段或将元素添加到重复字段时）在整数、浮点数和数字字符串之间进行。不允许的转换包括所有到/从数组或对象的转换。浮点数到整数溢出转换是未定义的。

You can see the corresponding PHP type for each scalar protocol buffers type in the [scalar value types table](https://protobuf.dev/programming-guides/proto3#scalar).
您可以在标量值类型表中看到每个标量协议缓冲区类型的相应 PHP 类型。

### Singular Message Fields 单一消息字段

A field with a message type defaults to nil, and is not automatically created when the field is accessed. Thus you need to explicitly create sub messages, as in the following:
具有消息类型的字段默认为 nil，并且在访问该字段时不会自动创建。因此，您需要显式创建子消息，如下所示：

```php
$m = new MyMessage();
$m->setZ(new SubMessage());
$m->getZ()->setFoo(42);

$m2 = new MyMessage();
$m2->getZ()->setFoo(42);  // FAILS with an exception
```

You can assign any instance to a message field, even if the instance is also held elsewhere (for example, as a field value on another message).
您可以将任何实例分配给消息字段，即使该实例也保存在其他地方（例如，作为另一个消息的字段值）。

### Repeated Fields 重复字段

The protocol buffer compiler generates a special `RepeatedField` for each repeated field. So, for example, given the following field:
协议缓冲区编译器为每个重复字段生成一个特殊的 `RepeatedField` 。因此，例如，给定以下字段：

```proto
repeated int32 foo = 1;
```

The generated code lets you do this:
生成的代码允许您执行此操作：

```php
$m->getFoo()[] =1;
$m->setFoo($array);
```

### Map Fields 映射字段

The protocol buffer compiler generates a `MapField` for each map field. So given this field:
协议缓冲区编译器为每个映射字段生成一个 `MapField` 。因此，给定此字段：

```proto
map<int32, int32> weight = 1;
```

You can do the following with the generated code:
使用生成的代码，您可以执行以下操作：

```php
$m->getWeight()[1] = 1;
```

## Enumerations 枚举

PHP doesn’t have native enums, so instead the protocol buffer compiler generates a PHP class for each enum type in your `.proto` file, just like for [messages](https://protobuf.dev/reference/php/php-generated/#message), with constants defined for each value. So, given this enum:
PHP 没有本机枚举，因此协议缓冲区编译器会为 `.proto` 文件中的每个枚举类型生成一个 PHP 类，就像消息一样，为每个值定义常量。因此，给定此枚举：

```proto
enum TestEnum {
  Default = 0;
  A = 1;
}
```

The compiler generates the following class:
编译器会生成以下类：

```php
class TestEnum {
  const DEFAULT = 0;
  const A = 1;
}
```

Also as with messages, a nested enum will result in a PHP class of the same name prefixed by its containing message(s) and separated with underscores, as PHP does not support nested classes.
此外，与消息一样，嵌套枚举将导致一个 PHP 类，其名称与包含它的消息相同，并以下划线分隔，因为 PHP 不支持嵌套类。

```php
class TestMessage_NestedEnum {...}
```

If an enum class or value name is reserved (for example, `Empty`), the prefix `PB` is prepended to the class or value name:
如果枚举类或值名称是保留的（例如， `Empty` ），则前缀 `PB` 会被添加至类或值名称：

```php
class PBEmpty {
  const PBECHO = 0;
}
```

We have also provided the file level option `php_class_prefix`. If this is specified, it is prepended to all generated enum classes.
我们还提供了文件级别选项 `php_class_prefix` 。如果指定此选项，它将被添加至所有生成的枚举类。

## Oneof

For a [oneof](https://protobuf.dev/programming-guides/proto3#oneof)s, the protocol buffer compiler generates the same code as it would for regular singular fields, but also adds a special accessor method that lets you find out which oneof field (if any) is set. So, given this message:
对于 oneof，协议缓冲区编译器会生成与常规单数字段相同的代码，但还会添加一个特殊的访问器方法，该方法可让您找出已设置的 oneof 字段（如果有）。因此，给定此消息：

```proto
message TestMessage {
  oneof test_oneof {
    int32 oneof_int32 = 1;
    int64 oneof_int64 = 2;
  }
}
```

The compiler generates the following fields and special method:
编译器会生成以下字段和特殊方法：

```php
class TestMessage {
  private oneof_int32;
  private oneof_int64;
  public function getOneofInt32();
  public function setOneofInt32($var);
  public function getOneofInt64();
  public function setOneofInt64($var);
  public function getTestOneof();  // Return field name
}
```

The accessor method’s name is based on the oneof’s name, and returns an enum value representing the field in the oneof that is currently set.
访问器方法的名称基于 oneof 的名称，并返回一个枚举值，表示当前设置的 oneof 中的字段。
