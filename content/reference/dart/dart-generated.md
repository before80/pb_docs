+++
title = "Dart Generated Code"
weight = 580
linkTitle = "Generated Code"
description = "Describes what Dart code the protocol buffer compiler generates for any given protocol definition."
type = "docs"
+++

# Dart Generated Code Dart 生成代码

Describes what Dart code the protocol buffer compiler generates for any given protocol definition.
描述协议缓冲区编译器为任何给定协议定义生成的 Dart 代码。



Any differences between proto2 and proto3 generated code are highlighted - note that these differences are in the generated code as described in this document, not the base API, which are the same in both versions. You should read the [proto2 language guide](https://protobuf.dev/programming-guides/proto) and/or the [proto3 language guide](https://protobuf.dev/programming-guides/proto3) before reading this document.
突出显示 proto2 和 proto3 生成代码之间的任何差异 - 请注意，这些差异在于本文档中描述的生成代码中，而不是基本 API 中，基本 API 在两个版本中都是相同的。在阅读本文档之前，您应该阅读 proto2 语言指南和/或 proto3 语言指南。

## Compiler Invocation 编译器调用

The protocol buffer compiler requires a [plugin to generate Dart](https://github.com/dart-lang/dart-protoc-plugin) code. Installing it following the [instructions](https://github.com/dart-lang/dart-protoc-plugin#how-to-build-and-use) provides a `protoc-gen-dart` binary which `protoc` uses when invoked with the `--dart_out` command-line flag. The `--dart_out` flag tells the compiler where to write the Dart source files. For a `.proto` file input, the compiler produces among others a `.pb.dart` file.
协议缓冲区编译器需要一个插件来生成 Dart 代码。按照说明进行安装可提供一个 `protoc-gen-dart` 二进制文件，当使用 `--dart_out` 命令行标志调用时， `protoc` 会使用该文件。 `--dart_out` 标志告诉编译器在何处写入 Dart 源文件。对于 `.proto` 文件输入，编译器会生成 `.pb.dart` 文件等。

The name of the `.pb.dart` file is computed by taking the name of the `.proto` file and making two changes:
`.pb.dart` 文件的名称是通过获取 `.proto` 文件的名称并进行两次更改来计算的：

- The extension (`.proto`) is replaced with `.pb.dart`. For example, a file called `foo.proto` results in an output file called `foo.pb.dart`.
  扩展名 ( `.proto` ) 被替换为 `.pb.dart` 。例如，名为 `foo.proto` 的文件会生成名为 `foo.pb.dart` 的输出文件。
- The proto path (specified with the `--proto_path` or `-I` command-line flag) is replaced with the output path (specified with the `--dart_out` flag).
  使用 `--proto_path` 或 `-I` 命令行标志指定的 proto 路径将替换为使用 `--dart_out` 标志指定的输出路径。

For example, when you invoke the compiler as follows:
例如，当您按如下方式调用编译器时：

```shell
protoc --proto_path=src --dart_out=build/gen src/foo.proto src/bar/baz.proto
```

the compiler will read the files `src/foo.proto` and `src/bar/baz.proto`. It produces: `build/gen/foo.pb.dart` and `build/gen/bar/baz.pb.dart`. The compiler automatically creates the directory `build/gen/bar` if necessary, but it will *not* create `build` or `build/gen`; they must already exist.
编译器将读取文件 `src/foo.proto` 和 `src/bar/baz.proto` 。它生成： `build/gen/foo.pb.dart` 和 `build/gen/bar/baz.pb.dart` 。编译器会在必要时自动创建目录 `build/gen/bar` ，但不会创建 `build` 或 `build/gen` ；它们必须已存在。

## Messages 消息

Given a simple message declaration:
给定一个简单的消息声明：

```proto
message Foo {}
```

The protocol buffer compiler generates a class called `Foo`, which extends the class `GeneratedMessage`.
协议缓冲区编译器会生成一个名为 `Foo` 的类，该类扩展了类 `GeneratedMessage` 。

The class `GeneratedMessage` defines methods that let you check, manipulate, read, or write the entire message. In addition to these methods, the `Foo` class defines the following methods and constructors:
类 `GeneratedMessage` 定义了允许您检查、操作、读取或写入整个消息的方法。除了这些方法之外， `Foo` 类还定义了以下方法和构造函数：

- `Foo()`: Default constructor. Creates an instance where all singular fields are unset and repeated fields are empty.
  `Foo()` ：默认构造函数。创建一个实例，其中所有单数字段都未设置，重复字段为空。
- `Foo.fromBuffer(...)`: Creates a `Foo` from serialized protocol buffer data representing the message.
  `Foo.fromBuffer(...)` ：从表示消息的序列化协议缓冲区数据创建 `Foo` 。
- `Foo.fromJson(...)`: Creates a `Foo` from a JSON string encoding the message.
  `Foo.fromJson(...)` ：从对消息进行编码的 JSON 字符串创建 `Foo` 。
- `Foo clone()`: Creates a deep clone of the fields in the message.
  `Foo clone()` ：创建消息中字段的深度克隆。
- `Foo copyWith(void Function(Foo) updates)`: Makes a writable copy of this message, applies the `updates` to it, and marks the copy read-only before returning it.
  `Foo copyWith(void Function(Foo) updates)` ：创建此消息的可写副本，将 `updates` 应用于该副本，并在返回之前将该副本标记为只读。
- `static Foo create()`: Factory function to create a single `Foo`.
  `static Foo create()` ：创建单个 `Foo` 的工厂函数。
- `static PbList<Foo> createRepeated()`: Factory function to create a List implementing a mutable repeated field of `Foo` elements.
  `static PbList<Foo> createRepeated()` ：创建实现 `Foo` 元素的可变重复字段的 List 的工厂函数。
- `static Foo getDefault()`: Returns a singleton instance of `Foo`, which is identical to a newly-constructed instance of Foo (so all singular fields are unset and all repeated fields are empty).
  `static Foo getDefault()` ：返回 `Foo` 的单例实例，该实例与新构造的 Foo 实例相同（因此所有单数字段都未设置，所有重复字段都为空）。

### Nested Types 嵌套类型

A message can be declared inside another message. For example:
可以在另一个消息中声明消息。例如：

```proto
message Foo {
  message Bar {
  }
}
```

In this case, the compiler generates two classes: `Foo` and `Foo_Bar`.
在这种情况下，编译器会生成两个类： `Foo` 和 `Foo_Bar` 。

## Fields 字段

In addition to the methods described in the previous section, the protocol buffer compiler generates accessor methods for each field defined within the message in the `.proto` file.
除了上一节中描述的方法外，协议缓冲区编译器还会为 `.proto` 文件中消息中定义的每个字段生成访问器方法。

Note that the generated names always use camel-case naming, even if the field name in the `.proto` file uses lower-case with underscores ([as it should](https://protobuf.dev/programming-guides/style)). The case-conversion works as follows:
请注意，生成的名称始终使用驼峰命名法，即使 `.proto` 文件中的字段名称使用下划线的小写字母（应该如此）。大小写转换的工作方式如下：

1. For each underscore in the name, the underscore is removed, and the following letter is capitalized.
   名称中的每个下划线都会被删除，后面的字母会大写。
2. If the name will have a prefix attached (e.g. "has"), the first letter is capitalized. Otherwise, it is lower-cased.
   如果名称将附加前缀（例如“has”），则第一个字母大写。否则，它将变为小写。

Thus, for the field `foo_bar_baz`, the getter becomes `get fooBarBaz` and a method prefixed with `has` would be `hasFooBarBaz`.
因此，对于字段 `foo_bar_baz` ，getter 变为 `get fooBarBaz` ，而带有前缀 `has` 的方法将为 `hasFooBarBaz` 。

### Singular Primitive Fields (proto2) 单数基本字段（proto2）

For any of these field definitions:
对于以下任何字段定义：

```proto
optional int32 foo = 1;
required int32 foo = 1;
```

The compiler will generate the following accessor methods in the message class:
编译器将在消息类中生成以下访问器方法：

- `int get foo`: Returns the current value of the field. If the field is not set, returns the default value.
  `int get foo` ：返回字段的当前值。如果未设置该字段，则返回默认值。
- `bool hasFoo()`: Returns `true` if the field is set.
  `bool hasFoo()` ：如果设置了字段，则返回 `true` 。
- `set foo(int value)`: Sets the value of the field. After calling this, `hasFoo()` will return `true` and `get foo` will return `value`.
  `set foo(int value)` ：设置字段的值。调用此方法后， `hasFoo()` 将返回 `true` ， `get foo` 将返回 `value` 。
- `void clearFoo()`: Clears the value of the field. After calling this, `hasFoo()` will return `false` and `get foo` will return the default value.
  `void clearFoo()` ：清除字段的值。调用此方法后， `hasFoo()` 将返回 `false` ， `get foo` 将返回默认值。

For other simple field types, the corresponding Dart type is chosen according to the [scalar value types table](https://protobuf.dev/programming-guides/proto#scalar). For message and enum types, the value type is replaced with the message or enum class.
对于其他简单字段类型，将根据标量值类型表选择相应的 Dart 类型。对于消息和枚举类型，值类型将替换为消息或枚举类。

### Singular Primitive Fields (proto3) 单数基本字段 (proto3)

For this field definition:
对于此字段定义：

```proto
int32 foo = 1;
```

The compiler will generate the following accessor methods in the message class:
编译器将在消息类中生成以下访问器方法：

- `int get foo`: Returns the current value of the field. If the field is not set, returns the default value.
  `int get foo` ：返回字段的当前值。如果未设置字段，则返回默认值。
- `set foo(int value)`: Sets the value of the field. After calling this, `get foo` will return `value`.
  `set foo(int value)` ：设置字段的值。调用此方法后， `get foo` 将返回 `value` 。
- `void clearFoo()`: Clears the value of the field. After calling this,`get foo` will return the default value.
  `void clearFoo()` ：清除字段的值。调用此方法后， `get foo` 将返回默认值。

**NOTE:** Due to a quirk in the Dart proto3 implementation, the following methods are generated even if the `optional` modifier, used to request [presence semantics](https://protobuf.dev/programming-guides/field_presence#presence-in-proto3-apis), isn’t in the proto definition.
注意：由于 Dart proto3 实现中的一个怪癖，即使 proto 定义中没有用于请求存在语义的修饰符 `optional` ，也会生成以下方法。

- `bool hasFoo()`: Returns `true` if the field is set.
  `bool hasFoo()` ：如果设置了字段，则返回 `true` 。
- `void clearFoo()`: Clears the value of the field. After calling this, `hasFoo()` will return `false` and `get foo` will return the default value.
  `void clearFoo()` ：清除字段的值。调用此方法后， `hasFoo()` 将返回 `false` ， `get foo` 将返回默认值。

### Singular Message Fields 单数消息字段

Given the message type:
给定消息类型：

```proto
message Bar {}
```

For a message with a `Bar` field:
对于具有 `Bar` 字段的消息：

```proto
// proto2
message Baz {
  optional Bar bar = 1;
  // The generated code is the same result if required instead of optional.
}

// proto3
message Baz {
  Bar bar = 1;
}
```

The compiler will generate the following accessor methods in the message class:
编译器将在消息类中生成以下访问器方法：

- `Bar get bar`: Returns the current value of the field. If the field is not set, returns the default value.
  `Bar get bar` ：返回字段的当前值。如果未设置该字段，则返回默认值。
- `set bar(Bar value)`: Sets the value of the field. After calling this, `hasBar()` will return `true` and `get bar` will return `value`.
  `set bar(Bar value)` ：设置字段的值。调用此方法后， `hasBar()` 将返回 `true` ， `get bar` 将返回 `value` 。
- `bool hasBar()`: Returns `true` if the field is set.
  `bool hasBar()` ：如果设置了字段，则返回 `true` 。
- `void clearBar()`: Clears the value of the field. After calling this, `hasBar()` will return `false` and `get bar` will return the default value.
  `void clearBar()` ：清除字段的值。调用此方法后， `hasBar()` 将返回 `false` ， `get bar` 将返回默认值。
- `Bar ensureBar()`: Sets `bar` to the empty instance if `hasBar()` returns `false`, and then returns the value of `bar`. After calling this, `hasBar()` will return `true`.
  `Bar ensureBar()` ：如果 `hasBar()` 返回 `false` ，则将 `bar` 设置为空实例，然后返回 `bar` 的值。调用此方法后， `hasBar()` 将返回 `true` 。

### Repeated Fields 重复字段

For this field definition:
对于此字段定义：

```proto
repeated int32 foo = 1;
```

The compiler will generate:
编译器将生成：

- `List<int> get foo`: Returns the list backing the field. If the field is not set, returns an empty list. Modifications to the list are reflected in the field.
  `List<int> get foo` ：返回支持该字段的列表。如果未设置该字段，则返回一个空列表。对列表的修改会反映在该字段中。

### Int64 Fields Int64 字段

For this field definition:
对于此字段定义：

```proto
// proto2
optional int64 bar = 1;

// proto3
int64 bar = 1;
```

The compiler will generate:
编译器将生成：

- `Int64 get bar`: Returns an `Int64` object containing the field value.
  `Int64 get bar` ：返回包含字段值的一个 `Int64` 对象。

Note that `Int64` is not built into the Dart core libraries. To work with these objects, you may need to import the Dart `fixnum` library:
请注意， `Int64` 未内置到 Dart 核心库中。要使用这些对象，您可能需要导入 Dart `fixnum` 库：

```dart
import 'package:fixnum/fixnum.dart';
```

### Map Fields Map 字段

Given a [`map`](https://protobuf.dev/programming-guides/proto3#maps) field definition like this:
给定类似这样的 `map` 字段定义：

```proto
map<int32, int32> map_field = 1;
```

The compiler will generate the following getter:
编译器将生成以下 getter：

- `Map<int, int> get mapField`: Returns the Dart map backing the field. If the field is not set, returns an empty map. Modifications to the map are reflected in the field.
  `Map<int, int> get mapField` ：返回支持该字段的 Dart map。如果未设置该字段，则返回一个空 map。对 map 的修改会反映在该字段中。

## Any

Given an [`Any`](https://protobuf.dev/programming-guides/proto3#any) field like this:
给定类似这样的 `Any` 字段：

```proto
import "google/protobuf/any.proto";

message ErrorStatus {
  string message = 1;
  google.protobuf.Any details = 2;
}
```

In our generated code, the getter for the `details` field returns an instance of `com.google.protobuf.Any`. This provides the following special methods to pack and unpack the `Any`’s values:
在我们的生成代码中， `details` 字段的 getter 返回 `com.google.protobuf.Any` 的一个实例。这提供了以下特殊方法来打包和解包 `Any` 的值：

```dart
    /// Unpacks the message in [value] into [instance].
    ///
    /// Throws a [InvalidProtocolBufferException] if [typeUrl] does not correspond
    /// to the type of [instance].
    ///
    /// A typical usage would be `any.unpackInto(new Message())`.
    ///
    /// Returns [instance].
    T unpackInto<T extends GeneratedMessage>(T instance,
        {ExtensionRegistry extensionRegistry = ExtensionRegistry.EMPTY});

    /// Returns `true` if the encoded message matches the type of [instance].
    ///
    /// Can be used with a default instance:
    /// `any.canUnpackInto(Message.getDefault())`
    bool canUnpackInto(GeneratedMessage instance);

    /// Creates a new [Any] encoding [message].
    ///
    /// The [typeUrl] will be [typeUrlPrefix]/`fullName` where `fullName` is
    /// the fully qualified name of the type of [message].
    static Any pack(GeneratedMessage message,
        {String typeUrlPrefix = 'type.googleapis.com'});
```

## Oneof

Given a [`oneof`](https://protobuf.dev/programming-guides/proto3#oneof) definition like this:
给定类似这样的 `oneof` 定义：

```proto
message Foo {
  oneof test {
    string name = 1;
    SubMessage sub_message = 2;
  }
}
```

The compiler will generate the following Dart enum type:
编译器将生成以下 Dart 枚举类型：

```proto
 enum Foo_Test { name, subMessage, notSet }
```

In addition, it will generate these methods:
此外，它还将生成这些方法：

- `Foo_Test whichTest()`: Returns the enum indicating which field is set. Returns `Foo_Test.notSet` if none of them is set.
  `Foo_Test whichTest()` ：返回指示设置了哪个字段的枚举。如果未设置任何字段，则返回 `Foo_Test.notSet` 。
- `void clearTest()`: Clears the value of the oneof field which is currently set (if any), and sets the oneof case to `Foo_Test.notSet`.
  `void clearTest()` ：清除当前设置的 oneof 字段的值（如果有），并将 oneof 案例设置为 `Foo_Test.notSet` 。

For each field inside the oneof definition the regular field accessor methods are generated. For instance for `name`:
为 oneof 定义中的每个字段生成常规字段访问器方法。例如，对于 `name` ：

- `String get name`: Returns the current value of the field if the oneof case is `Foo_Test.name`. Otherwise, returns the default value.
  `String get name` ：如果 oneof 案例为 `Foo_Test.name` ，则返回字段的当前值。否则，返回默认值。
- `set name(String value)`: Sets the value of the field and sets the oneof case to `Foo_Test.name`. After calling this, `get name` will return `value` and `whichTest()` will return `Foo_Test.name`.
  `set name(String value)` ：设置字段的值并将 oneof 案例设置为 `Foo_Test.name` 。调用此方法后， `get name` 将返回 `value` ， `whichTest()` 将返回 `Foo_Test.name` 。
- `void clearName()`: Nothing will be changed if the oneof case is not `Foo_Test.name`. Otherwise, clears the value of the field. After calling this, `get name` will return the default value and `whichTest()` will return `Foo_Test.notSet`.
  `void clearName()` ：如果 oneof 案例不是 `Foo_Test.name` ，则不会更改任何内容。否则，清除字段的值。调用此方法后， `get name` 将返回默认值， `whichTest()` 将返回 `Foo_Test.notSet` 。

## Enumerations 枚举

Given an enum definition like:
给定枚举定义，例如：

```proto
enum Color {
  COLOR_UNSPECIFIED = 0;
  COLOR_RED = 1;
  COLOR_GREEN = 2;
  COLOR_BLUE = 3;
}
```

The protocol buffer compiler will generate a class called `Color`, which extends the `ProtobufEnum` class. The class will include a `static const Color` for each of the three values defined as well as a `static const List<Color>` containing all the three non-unspecified values. It will also include the following method:
协议缓冲区编译器将生成一个名为 `Color` 的类，它扩展了 `ProtobufEnum` 类。该类将包含一个 `static const Color` ，用于定义的所有三个值，以及一个 `static const List<Color>` ，其中包含所有三个未指定的非值。它还将包含以下方法：

- `static Color? valueOf(int value)`: Returns the `Color` corresponding to the given numeric value.
  `static Color? valueOf(int value)` ：返回与给定数值对应的 `Color` 。

Each value will have the following properties:
每个值都将具有以下属性：

- `name`: The enum’s name, as specified in the .proto file.
  `name` ：枚举的名称，如 .proto 文件中指定。
- `value`: The enum’s integer value, as specified in the .proto file.
  `value` ：枚举的整数值，如 .proto 文件中指定。

Note that the `.proto` language allows multiple enum symbols to have the same numeric value. Symbols with the same numeric value are synonyms. For example:
请注意， `.proto` 语言允许多个枚举符号具有相同的数值。具有相同数值的符号是同义词。例如：

```proto
enum Foo {
  BAR = 0;
  BAZ = 0;
}
```

In this case, `BAZ` is a synonym for `BAR` and will be defined like so:
在这种情况下， `BAZ` 是 `BAR` 的同义词，将定义如下：

```dart
static const Foo BAZ = BAR;
```

An enum can be defined nested within a message type. For instance, given an enum definition like:
枚举可以定义在消息类型中。例如，给定枚举定义如下：

```proto
message Bar {
  enum Color {
    COLOR_UNSPECIFIED = 0;
    COLOR_RED = 1;
    COLOR_GREEN = 2;
    COLOR_BLUE = 3;
  }
}
```

The protocol buffer compiler will generate a class called `Bar`, which extends `GeneratedMessage`, and a class called `Bar_Color`, which extends `ProtobufEnum`.
协议缓冲区编译器将生成一个名为 `Bar` 的类，它扩展 `GeneratedMessage` ，以及一个名为 `Bar_Color` 的类，它扩展 `ProtobufEnum` 。

## Extensions (proto2 only) 扩展（仅限 proto2）

Given a file `foo_test.proto` including a message with an [extension range](https://protobuf.dev/programming-guides/proto#extensions) and a top-level extension definition:
给定一个文件 `foo_test.proto` ，其中包含一条带有扩展范围的消息和一个顶级扩展定义：

```proto
message Foo {
  extensions 100 to 199;
}

extend Foo {
  optional int32 bar = 101;
}
```

The protocol buffer compiler will generate, in addition to the `Foo` class, a class `Foo_test` which will contain a `static Extension` for each extension field in the file along with a method for registering all the exensions in an `ExtensionRegistry` :
除了 `Foo` 类之外，协议缓冲区编译器还会生成一个 `Foo_test` 类，其中包含文件中每个扩展字段的 `static Extension` ，以及用于在 `ExtensionRegistry` 中注册所有扩展的方法：

- `static final Extension bar`
- `static void registerAllExtensions(ExtensionRegistry registry)` : Registers all the defined extensions in the given registry.
  `static void registerAllExtensions(ExtensionRegistry registry)` ：在给定的注册表中注册所有已定义的扩展。

The extension accessors of `Foo` can be used as follows:
`Foo` 的扩展访问器可以使用如下方式：

```dart
Foo foo = Foo();
foo.setExtension(Foo_test.bar, 1);
assert(foo.hasExtension(Foo_test.bar));
assert(foo.getExtension(Foo_test.bar)) == 1);
```

Extensions can also be declared nested inside of another message:
扩展也可以声明为嵌套在另一个消息内部：

```proto
message Baz {
  extend Foo {
    optional int32 bar = 124;
  }
}
```

In this case, the extension `bar` is instead declared as a static member of the `Baz` class.
在这种情况下，扩展 `bar` 反而被声明为 `Baz` 类的静态成员。

When parsing a message that might have extensions, you must provide an `ExtensionRegistry` in which you have registered any extensions that you want to be able to parse. Otherwise, those extensions will just be treated like unknown fields. For example:
在解析可能具有扩展的消息时，您必须提供一个 `ExtensionRegistry` ，在其中您已注册要能够解析的任何扩展。否则，这些扩展将被视为未知字段。例如：

```dart
ExtensionRegistry registry = ExtensionRegistry();
registry.add(Baz.bar);
Foo foo = Foo.fromBuffer(input, registry);
```

If you already have a parsed message with unknown fields, you can use `reparseMessage` on an `ExtensionRegistry` to reparse the message. If the set of unknown fields contains extensions that are present in the registry, these extensions are parsed and removed from the unknown field set. Extensions already present in the message are preserved.
如果您已经有一个具有未知字段的已解析消息，则可以在 `ExtensionRegistry` 上使用 `reparseMessage` 来重新解析消息。如果未知字段集包含注册表中存在的扩展，则会解析这些扩展并将其从未知字段集中删除。消息中已存在的扩展将被保留。

```dart
Foo foo = Foo.fromBuffer(input);
ExtensionRegistry registry = ExtensionRegistry();
registry.add(Baz.bar);
Foo reparsed = registry.reparseMessage(foo);
```

Be aware that this method to retrieve extensions is more expensive overall. Where possible we recommend using `ExtensionRegistry` with all the needed extensions when doing `GeneratedMessage.fromBuffer`.
请注意，这种检索扩展的方法总体上开销更大。在可能的情况下，我们建议在执行 `GeneratedMessage.fromBuffer` 时使用 `ExtensionRegistry` 和所有必需的扩展。

## Services 服务

Given a service definition:
给定服务定义：

```proto
service Foo {
  rpc Bar(FooRequest) returns(FooResponse);
}
```

The protocol buffer compiler can be invoked with the `grpc` option (e.g. `--dart_out=grpc:output_folder`), in which case it will generate code to support [gRPC](https://www.grpc.io/). See the [gRPC Dart Quickstart guide](https://grpc.io/docs/quickstart/dart.html) for more details.
可以使用 `grpc` 选项（例如 `--dart_out=grpc:output_folder` ）调用协议缓冲区编译器，在这种情况下，它将生成支持 gRPC 的代码。有关更多详细信息，请参阅 gRPC Dart 快速入门指南。
