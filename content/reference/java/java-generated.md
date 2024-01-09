+++
title = "Java Generated Code Guide"
weight = 650
linkTitle = "Generated Code Guide"
description = "Describes exactly what Java code the protocol buffer compiler generates for any given protocol definition."
type = "docs"
+++

> 原文网址：  https://protobuf.dev/reference/java/java-generated/

## Java Generated Code Guide Java 生成代码指南

Describes exactly what Java code the protocol buffer compiler generates for any given protocol definition.
准确描述协议缓冲区编译器为任何给定协议定义生成的 Java 代码。



Any differences between proto2 and proto3 generated code are highlighted—note that these differences are in the generated code as described in this document, not the base message classes/interfaces, which are the same in both versions. You should read the [proto2 language guide](https://protobuf.dev/programming-guides/proto2) and/or [proto3 language guide](https://protobuf.dev/programming-guides/proto3) before reading this document.
突出显示了 proto2 和 proto3 生成的代码之间的任何差异——请注意，这些差异在生成的代码中，如本文档中所述，而不是基本消息类/接口，它们在两个版本中都是相同的。在阅读本文档之前，您应该阅读 proto2 语言指南和/或 proto3 语言指南。

Note that no Java protocol buffer methods accept or return nulls unless otherwise specified.
请注意，除非另有说明，否则没有任何 Java 协议缓冲区方法接受或返回空值。

## Compiler Invocation 编译器调用

The protocol buffer compiler produces Java output when invoked with the `--java_out=` command-line flag. The parameter to the `--java_out=` option is the directory where you want the compiler to write your Java output. For each `.proto` file input, the compiler creates a wrapper `.java` file containing a Java class which represents the `.proto` file itself.
使用 `--java_out=` 命令行标志调用协议缓冲区编译器时，它会生成 Java 输出。 `--java_out=` 选项的参数是您希望编译器写入 Java 输出的目录。对于每个 `.proto` 文件输入，编译器都会创建一个包装器 `.java` 文件，其中包含一个表示 `.proto` 文件本身的 Java 类。

If the `.proto` file contains a line like the following:
如果 `.proto` 文件包含如下行：

```proto
option java_multiple_files = true;
```

Then the compiler will also create separate `.java` files for each of the classes/enums which it will generate for each top-level message, enumeration, and service declared in the `.proto` file.
那么编译器还将为每个顶级消息、枚举和在 `.proto` 文件中声明的服务生成单独的 `.java` 文件。

Otherwise (i.e. when the `java_multiple_files` option is disabled; which is the default), the aforementioned wrapper class will also be used as an outer class, and the generated classes/enums for each top-level message, enumeration, and service declared in the `.proto` file will all be nested within the outer wrapper class. Thus the compiler will only generate a single `.java` file for the entire `.proto` file.
否则（即当 `java_multiple_files` 选项被禁用时；这是默认设置），上述包装器类还将用作外部类，并且在 `.proto` 文件中声明的每个顶级消息、枚举和服务的生成类/枚举都将嵌套在外部包装器类中。因此，编译器将仅为整个 `.proto` 文件生成一个 `.java` 文件。

The wrapper class’s name is chosen as follows: If the `.proto` file contains a line like the following:
包装器类的名称选择如下：如果 `.proto` 文件包含如下所示的行：

```proto
option java_outer_classname = "Foo";
```

Then the wrapper class name will be `Foo`. Otherwise, the wrapper class name is determined by converting the `.proto` file base name to camel case. For example, `foo_bar.proto` will generate a class name of `FooBar`. If there’s a service, enum, or message (including nested types) in the file with the same name, “OuterClass” will be appended to the wrapper class’s name. Examples:
那么包装器类名称将是 `Foo` 。否则，包装器类名称由将 `.proto` 文件基本名称转换为驼峰式大小写来确定。例如， `foo_bar.proto` 将生成类名称 `FooBar` 。如果文件中存在具有相同名称的服务、枚举或消息（包括嵌套类型），则“OuterClass”将附加到包装器类的名称。示例：

- If `foo_bar.proto` contains a message called `FooBar`, the wrapper class will generate a class name of `FooBarOuterClass`.
  如果 `foo_bar.proto` 包含一个名为 `FooBar` 的消息，则包装器类将生成类名称 `FooBarOuterClass` 。
- If `foo_bar.proto` contains a service called `FooService`, and `java_outer_classname` is also set to the string `FooService`, then the wrapper class will generate a class name of `FooServiceOuterClass`.
  如果 `foo_bar.proto` 包含一个名为 `FooService` 的服务，并且 `java_outer_classname` 也设置为字符串 `FooService` ，则包装器类将生成类名称 `FooServiceOuterClass` 。

**Note:** If you are using the deprecated v1 of the protobuf API, `OuterClass` is added regardless of any collisions with message names.
注意：如果您使用的是 protobuf API 的已弃用 v1，则无论与消息名称发生任何冲突，都会添加 `OuterClass` 。

In addition to any nested classes, the wrapper class itself will have the following API (assuming the wrapper class is named `Foo` and was generated from `foo.proto`):
除了任何嵌套类之外，包装器类本身还将具有以下 API（假设包装器类名为 `Foo` 且由 `foo.proto` 生成）：

```java
public final class Foo {
  private Foo() {}  // Not instantiable.

  /** Returns a FileDescriptor message describing the contents of {@code foo.proto}. */
  public static com.google.protobuf.Descriptors.FileDescriptor getDescriptor();
  /** Adds all extensions defined in {@code foo.proto} to the given registry. */
  public static void registerAllExtensions(com.google.protobuf.ExtensionRegistry registry);
  public static void registerAllExtensions(com.google.protobuf.ExtensionRegistryLite registry);

  // (Nested classes omitted)
}
```

The Java package name is chosen as described under [Packages](https://protobuf.dev/reference/java/java-generated/#package), below.
Java 包名称的选择如下文“包”中所述。

The output file is chosen by concatenating the parameter to `--java_out=`, the package name (with `.`s replaced with `/`s), and the `.java` file name.
输出文件是通过连接 `--java_out=` 的参数、包名称（将 `.` 替换为 `/` ）和 `.java` 文件名来选择的。

So, for example, let’s say you invoke the compiler as follows:

```shell
protoc --proto_path=src --java_out=build/gen src/foo.proto
```

If `foo.proto`’s Java package is `com.example` and it doesn’t enable `java_multiple_files` and its outer classname is `FooProtos`, then the protocol buffer compiler will generate the file `build/gen/com/example/FooProtos.java`. The protocol buffer compiler will automatically create the `build/gen/com` and `build/gen/com/example` directories if needed. However, it will not create `build/gen` or `build`; they must already exist. You can specify multiple `.proto` files in a single invocation; all output files will be generated at once.
如果 `foo.proto` 的 Java 包是 `com.example` 且它未启用 `java_multiple_files` 且其外部类名为 `FooProtos` ，则协议缓冲区编译器将生成文件 `build/gen/com/example/FooProtos.java` 。协议缓冲区编译器将根据需要自动创建 `build/gen/com` 和 `build/gen/com/example` 目录。但是，它不会创建 `build/gen` 或 `build` ；它们必须已经存在。您可以在一次调用中指定多个 `.proto` 文件；所有输出文件都将一次生成。

When outputting Java code, the protocol buffer compiler’s ability to output directly to JAR archives is particularly convenient, as many Java tools are able to read source code directly from JAR files. To output to a JAR file, simply provide an output location ending in `.jar`. Note that only the Java source code is placed in the archive; you must still compile it separately to produce Java class files.
在输出 Java 代码时，协议缓冲区编译器能够直接输出到 JAR 存档，这非常方便，因为许多 Java 工具能够直接从 JAR 文件读取源代码。要输出到 JAR 文件，只需提供一个以 `.jar` 结尾的输出位置。请注意，只有 Java 源代码会放入存档中；您仍必须单独编译它才能生成 Java 类文件。

For example, if the `.proto` file contains:
例如，如果 `.proto` 文件包含：

```proto
package foo.bar;
```

Then the resulting Java class will be placed in Java package `foo.bar`. However, if the `.proto` file also contains a `java_package` option, like so:
然后生成的 Java 类将被放置在 Java 包 `foo.bar` 中。但是，如果 `.proto` 文件还包含一个 `java_package` 选项，如下所示：

```proto
package foo.bar;
option java_package = "com.example.foo.bar";
```

Then the class is placed in the `com.example.foo.bar` package instead. The `java_package` option is provided because normal `.proto` `package` declarations are not expected to start with a backwards domain name.
那么类将被放置在 `com.example.foo.bar` 包中。提供 `java_package` 选项是因为不希望正常的 `.proto` `package` 声明以反向域名开头。

## Messages 消息

Given a simple message declaration:
给定一个简单的消息声明：

```proto
message Foo {}
```

The protocol buffer compiler generates a class called `Foo`, which implements the `Message` interface. The class is declared `final`; no further subclassing is allowed. `Foo` extends `GeneratedMessage`, but this should be considered an implementation detail. By default, `Foo` overrides many methods of `GeneratedMessage` with specialized versions for maximum speed. However, if the `.proto` file contains the line:
协议缓冲区编译器生成一个名为 `Foo` 的类，该类实现 `Message` 接口。该类被声明为 `final` ；不允许进一步的子类化。 `Foo` 扩展 `GeneratedMessage` ，但这应该被视为实现细节。默认情况下， `Foo` 用专门版本覆盖 `GeneratedMessage` 的许多方法，以实现最高速度。但是，如果 `.proto` 文件包含以下行：

```proto
option optimize_for = CODE_SIZE;
```

then `Foo` will override only the minimum set of methods necessary to function and rely on `GeneratedMessage`’s reflection-based implementations of the rest. This significantly reduces the size of the generated code, but also reduces performance. Alternatively, if the `.proto` file contains:
那么 `Foo` 将仅覆盖运行和依赖于 `GeneratedMessage` 基于反射的其余实现所需的最小方法集。这会显著减小生成代码的大小，但也降低了性能。或者，如果 `.proto` 文件包含：

```proto
option optimize_for = LITE_RUNTIME;
```

then `Foo` will include fast implementations of all methods, but will implement the `MessageLite` interface, which contains a subset of the methods of `Message`. In particular, it does not support descriptors, nested builders, or reflection. However, in this mode, the generated code only needs to link against `libprotobuf-lite.jar` instead of `libprotobuf.jar`. The “lite” library is much smaller than the full library, and is more appropriate for resource-constrained systems such as mobile phones.
那么 `Foo` 将包含所有方法的快速实现，但将实现 `MessageLite` 接口，其中包含 `Message` 方法的子集。特别是，它不支持描述符、嵌套生成器或反射。但是，在此模式下，生成的代码只需要链接到 `libprotobuf-lite.jar` 而不是 `libprotobuf.jar` 。“精简”库比完整库小得多，更适合资源受限的系统，例如手机。

The `Message` interface defines methods that let you check, manipulate, read, or write the entire message. In addition to these methods, the `Foo` class defines the following static methods:
`Message` 接口定义了允许您检查、操作、读取或写入整个消息的方法。除了这些方法之外， `Foo` 类还定义了以下静态方法：

- `static Foo getDefaultInstance()`: Returns the *singleton* instance of `Foo`. This instance’s contents are identical to what you’d get if you called `Foo.newBuilder().build()` (so all singular fields are unset and all repeated fields are empty). Note that the default instance of a message can be used as a factory by calling its `newBuilderForType()` method.
  `static Foo getDefaultInstance()` ：返回 `Foo` 的单例实例。此实例的内容与调用 `Foo.newBuilder().build()` 时获得的内容相同（因此所有单数字段都未设置，所有重复字段都为空）。请注意，可以通过调用消息的 `newBuilderForType()` 方法将消息的默认实例用作工厂。
- `static Descriptor getDescriptor()`: Returns the type’s descriptor. This contains information about the type, including what fields it has and what their types are. This can be used with the reflection methods of the `Message`, such as `getField()`.
  `static Descriptor getDescriptor()` ：返回类型的描述符。其中包含有关类型的信息，包括它具有哪些字段以及它们的类型是什么。这可与 `Message` 的反射方法（例如 `getField()` ）配合使用。
- `static Foo parseFrom(...)`: Parses a message of type `Foo` from the given source and returns it. There is one `parseFrom` method corresponding to each variant of `mergeFrom()` in the `Message.Builder` interface. Note that `parseFrom()` never throws `UninitializedMessageException`; it throws `InvalidProtocolBufferException` if the parsed message is missing required fields. This makes it subtly different from calling `Foo.newBuilder().mergeFrom(...).build()`.
  `static Foo parseFrom(...)` ：从给定源解析 `Foo` 类型的消息并返回它。在 `Message.Builder` 接口中，每个 `mergeFrom()` 变体对应一个 `parseFrom` 方法。请注意， `parseFrom()` 绝不会引发 `UninitializedMessageException` ；如果解析的消息缺少必需的字段，它会引发 `InvalidProtocolBufferException` 。这使得它与调用 `Foo.newBuilder().mergeFrom(...).build()` 略有不同。
- `static Parser parser()`: Returns an instance of the `Parser`, which implements various `parseFrom()` methods.
  `static Parser parser()` ：返回 `Parser` 的实例，该实例实现了各种 `parseFrom()` 方法。
- `Foo.Builder newBuilder()`: Creates a new builder (described below).
  `Foo.Builder newBuilder()` ：创建一个新的构建器（如下所述）。
- `Foo.Builder newBuilder(Foo prototype)`: Creates a new builder with all fields initialized to the same values that they have in `prototype`. Since embedded message and string objects are immutable, they are shared between the original and the copy.
  `Foo.Builder newBuilder(Foo prototype)` ：创建一个新的构建器，其中所有字段都初始化为它们在 `prototype` 中具有的相同值。由于嵌入式消息和字符串对象是不可变的，因此它们在原始对象和副本之间共享。

### Builders 构建器

Message objects—such as instances of the `Foo` class described above—are immutable, just like a Java `String`. To construct a message object, you need to use a *builder*. Each message class has its own builder class—so in our `Foo` example, the protocol buffer compiler generates a nested class `Foo.Builder` which can be used to build a `Foo`. `Foo.Builder` implements the `Message.Builder` interface. It extends the `GeneratedMessage.Builder` class, but, again, this should be considered an implementation detail. Like `Foo`, `Foo.Builder` may rely on generic method implementations in `GeneratedMessage.Builder` or, when the `optimize_for` option is used, generated custom code that is much faster.
消息对象（例如上面描述的 `Foo` 类的实例）是不可变的，就像 Java `String` 一样。要构造消息对象，您需要使用构建器。每个消息类都有自己的构建器类，因此在我们的 `Foo` 示例中，协议缓冲区编译器会生成一个嵌套类 `Foo.Builder` ，该类可用于构建 `Foo` 。 `Foo.Builder` 实现 `Message.Builder` 接口。它扩展了 `GeneratedMessage.Builder` 类，但同样，这应该被视为实现细节。与 `Foo` 一样， `Foo.Builder` 可能依赖于 `GeneratedMessage.Builder` 中的通用方法实现，或者在使用 `optimize_for` 选项时，依赖于生成的自定义代码，该代码的速度要快得多。

`Foo.Builder` does not define any static methods. Its interface is exactly as defined by the `Message.Builder` interface, with the exception that return types are more specific: methods of `Foo.Builder` that modify the builder return type `Foo.Builder`, and `build()` returns type `Foo`.
`Foo.Builder` 不定义任何静态方法。它的接口与 `Message.Builder` 接口定义的完全相同，不同之处在于返回类型更具体：修改构建器返回类型的 `Foo.Builder` 方法 `Foo.Builder` ，而 `build()` 返回类型 `Foo` 。

Methods that modify the contents of a builder—including field setters—always return a reference to the builder (i.e. they “`return this;`”). This allows multiple method calls to be chained together in one line. For example: `builder.mergeFrom(obj).setFoo(1).setBar("abc").clearBaz();`
修改构建器内容的方法（包括字段设置器）始终返回对构建器的引用（即它们“ `return this;` ”）。这允许在一行中将多个方法调用链接在一起。例如： `builder.mergeFrom(obj).setFoo(1).setBar("abc").clearBaz();`

Note that builders are not thread-safe, so Java synchronization should be used whenever it is necessary for multiple different threads to be modifying the contents of a single builder.
请注意，构建器不是线程安全的，因此每当多个不同线程需要修改单个构建器的内容时，都应使用 Java 同步。

### Sub-Builders 子构建器

For messages containing sub-messages, the compiler also generates sub-builders. This allows you to repeatedly modify deep-nested sub-messages without rebuilding them. For example:
对于包含子消息的消息，编译器还会生成子构建器。这允许您重复修改深层嵌套的子消息，而无需重建它们。例如：

```proto
message Foo {
  optional int32 val = 1;
  // some other fields.
}

message Bar {
  optional Foo foo = 1;
  // some other fields.
}

message Baz {
  optional Bar bar = 1;
  // some other fields.
}
```

If you have a `Baz` message already, and want to change the deeply nested `val` in `Foo`. Instead of:
如果您已经有一个 `Baz` 消息，并且想要更改 `Foo` 中深层嵌套的 `val` 。而不是：

```java
baz = baz.toBuilder().setBar(
    baz.getBar().toBuilder().setFoo(
        baz.getBar().getFoo().toBuilder().setVal(10).build()
    ).build()).build();
```

You can write:
您可以编写：

```java
Baz.Builder builder = baz.toBuilder();
builder.getBarBuilder().getFooBuilder().setVal(10);
baz = builder.build();
```

### Nested Types 嵌套类型

A message can be declared inside another message. For example:
可以在另一个消息中声明消息。例如：

```proto
message Foo {
  message Bar { }
}
```

In this case, the compiler simply generates `Bar` as an inner class nested inside `Foo`.
在这种情况下，编译器只是将 `Bar` 生成为嵌套在 `Foo` 内部的一个内部类。

## Fields 字段

In addition to the methods described in the previous section, the protocol buffer compiler generates a set of accessor methods for each field defined within the message in the `.proto` file. The methods that read the field value are defined both in the message class and its corresponding builder; the methods that modify the value are defined in the builder only.
除了上一节中描述的方法外，协议缓冲区编译器还会为 `.proto` 文件中定义的每个字段生成一组访问器方法。读取字段值的方法在消息类及其相应的构建器中均已定义；修改值的方法仅在构建器中定义。

Note that method names always use camel-case naming, even if the field name in the `.proto` file uses lower-case with underscores ([as it should](https://protobuf.dev/programming-guides/style)). The case-conversion works as follows:
请注意，即使 `.proto` 文件中的字段名称使用下划线分隔的小写字母（应该如此），方法名称始终使用驼峰式命名。大小写转换的工作方式如下：

- For each underscore in the name, the underscore is removed, and the following letter is capitalized.
  名称中的每个下划线都会被删除，后面的字母会大写。
- If the name will have a prefix attached (e.g. “get”), the first letter is capitalized. Otherwise, it is lower-cased.
  如果名称将附加前缀（例如“get”），则第一个字母大写。否则，它为小写。
- The letter following the last digit in each number in a method name is capitalized.
  方法名称中每个数字后的字母大写。

Thus, the field `foo_bar_baz` becomes `fooBarBaz`. If prefixed with `get`, it would be `getFooBarBaz`. And `foo_ba23r_baz` becomes `fooBa23RBaz`.
因此，字段 `foo_bar_baz` 变为 `fooBarBaz` 。如果加上前缀 `get` ，则为 `getFooBarBaz` 。 `foo_ba23r_baz` 变为 `fooBa23RBaz` 。

As well as accessor methods, the compiler generates an integer constant for each field containing its field number. The constant name is the field name converted to upper-case followed by `_FIELD_NUMBER`. For example, given the field `optional int32 foo_bar = 5;`, the compiler will generate the constant `public static final int FOO_BAR_FIELD_NUMBER = 5;`.
除了访问器方法外，编译器还会为每个包含其字段编号的字段生成一个整数常量。常量名称是字段名称转换为大写，后跟 `_FIELD_NUMBER` 。例如，给定字段 `optional int32 foo_bar = 5;` ，编译器将生成常量 `public static final int FOO_BAR_FIELD_NUMBER = 5;` 。

### Singular Fields (proto2) 单数字段 (proto2)

For any of these field definitions:
对于以下任何字段定义：

```proto
optional int32 foo = 1;
required int32 foo = 1;
```

The compiler will generate the following accessor methods in both the message class and its builder:
编译器将在消息类及其构建器中生成以下访问器方法：

- `boolean hasFoo()`: Returns `true` if the field is set.
  `boolean hasFoo()` ：如果设置了字段，则返回 `true` 。
- `int getFoo()`: Returns the current value of the field. If the field is not set, returns the default value.
  `int getFoo()` ：返回字段的当前值。如果未设置该字段，则返回默认值。

The compiler will generate the following methods only in the message’s builder:
编译器仅在消息的构建器中生成以下方法：

- `Builder setFoo(int value)`: Sets the value of the field. After calling this, `hasFoo()` will return `true` and `getFoo()` will return `value`.
  `Builder setFoo(int value)` ：设置字段的值。调用此方法后， `hasFoo()` 将返回 `true` ， `getFoo()` 将返回 `value` 。
- `Builder clearFoo()`: Clears the value of the field. After calling this, `hasFoo()` will return `false` and `getFoo()` will return the default value.
  `Builder clearFoo()` ：清除字段的值。调用此方法后， `hasFoo()` 将返回 `false` ， `getFoo()` 将返回默认值。

For other simple field types, the corresponding Java type is chosen according to the [scalar value types table](https://protobuf.dev/programming-guides/proto2#scalar). For message and enum types, the value type is replaced with the message or enum class.
对于其他简单字段类型，将根据标量值类型表选择相应的 Java 类型。对于消息和枚举类型，值类型将替换为消息或枚举类。

#### Embedded Message Fields 嵌入式消息字段

For message types, `setFoo()` also accepts an instance of the message’s builder type as the parameter. This is just a shortcut which is equivalent to calling `.build()` on the builder and passing the result to the method.
对于消息类型， `setFoo()` 还接受消息的生成器类型的实例作为参数。这只是一个快捷方式，相当于在生成器上调用 `.build()` 并将结果传递给方法。

If the field is not set, `getFoo()` will return a Foo instance with none of its fields set (possibly the instance returned by `Foo.getDefaultInstance()`).
如果未设置该字段， `getFoo()` 将返回一个未设置任何字段的 Foo 实例（可能是 `Foo.getDefaultInstance()` 返回的实例）。

In addition, the compiler generates two accessor methods that allow you to access the relevant sub-builders for message types. The following method is generated in both the message class and its builder:
此外，编译器还会生成两个访问器方法，允许您访问消息类型的相关子生成器。以下方法在消息类及其生成器中均生成：

- `FooOrBuilder getFooOrBuilder()`: Returns the builder for the field, if it already exists, or the message if not. Calling this method on builders will not create a sub-builder for the field.
  `FooOrBuilder getFooOrBuilder()` ：如果字段已存在，则返回该字段的生成器，否则返回消息。在生成器上调用此方法不会为该字段创建子生成器。

The compiler generates the following method only in the message’s builder.
编译器仅在消息的生成器中生成以下方法。

- `Builder getFooBuilder()`: Returns the builder for the field.
  `Builder getFooBuilder()` ：返回该字段的生成器。

### Singular Fields (proto3) 单数字段 (proto3)

For this field definition:
对于此字段定义：

```proto
int32 foo = 1;
```

The compiler will generate the following accessor method in both the message class and its builder:
编译器将在消息类及其生成器中生成以下访问器方法：

- `int getFoo()`: Returns the current value of the field. If the field is not set, returns the default value for the field’s type.
  `int getFoo()` ：返回字段的当前值。如果未设置该字段，则返回该字段类型的默认值。

The compiler will generate the following methods only in the message’s builder:
编译器仅在消息的生成器中生成以下方法：

- `Builder setFoo(int value)`: Sets the value of the field. After calling this, `getFoo()` will return `value`.
  `Builder setFoo(int value)` ：设置字段的值。调用此方法后， `getFoo()` 将返回 `value` 。
- `Builder clearFoo()`: Clears the value of the field. After calling this, `getFoo()` will return the default value for the field’s type.
  `Builder clearFoo()` ：清除字段的值。调用此方法后， `getFoo()` 将返回字段类型的默认值。

For other simple field types, the corresponding Java type is chosen according to the [scalar value types table](https://protobuf.dev/programming-guides/proto2#scalar). For message and enum types, the value type is replaced with the message or enum class.
对于其他简单字段类型，将根据标量值类型表选择相应的 Java 类型。对于消息和枚举类型，值类型将替换为消息或枚举类。

#### Embedded Message Fields 嵌入式消息字段

For message field types, an additional accessor method is generated in both the message class and its builder:
对于消息字段类型，将在消息类及其构建器中生成一个附加的访问器方法：

- `boolean hasFoo()`: Returns `true` if the field has been set.
  `boolean hasFoo()` ：如果已设置字段，则返回 `true` 。

`setFoo()` also accepts an instance of the message’s builder type as the parameter. This is just a shortcut which is equivalent to calling `.build()` on the builder and passing the result to the method.
`setFoo()` 还接受消息构建器类型的实例作为参数。这只是一个快捷方式，相当于在构建器上调用 `.build()` ，并将结果传递给该方法。

If the field is not set, `getFoo()` will return a Foo instance with none of its fields set (possibly the instance returned by `Foo.getDefaultInstance()`).
如果未设置字段， `getFoo()` 将返回一个 Foo 实例，其中没有任何字段被设置（可能是 `Foo.getDefaultInstance()` 返回的实例）。

In addition, the compiler generates two accessor methods that allow you to access the relevant sub-builders for message types. The following method is generated in both the message class and its builder:
此外，编译器还会生成两个访问器方法，允许您访问消息类型的相关子构建器。以下方法在消息类及其构建器中生成：

- `FooOrBuilder getFooOrBuilder()`: Returns the builder for the field, if it already exists, or the message if not. Calling this method on builders will not create a sub-builder for the field.
  `FooOrBuilder getFooOrBuilder()` ：如果字段已存在，则返回该字段的构建器，否则返回消息。在构建器上调用此方法不会为该字段创建子构建器。

The compiler generates the following method only in the message’s builder.
编译器仅在消息的构建器中生成以下方法。

- `Builder getFooBuilder()`: Returns the builder for the field.
  `Builder getFooBuilder()` ：返回该字段的构建器。

#### Enum Fields 枚举字段

For enum field types, an additional accessor method is generated in both the message class and its builder:
对于枚举字段类型，在消息类及其构建器中都会生成一个额外的访问器方法：

- `int getFooValue()`: Returns the integer value of the enum.
  `int getFooValue()` ：返回枚举的整数值。

The compiler will generate the following additional method only in the message’s builder:
编译器仅在消息的构建器中生成以下附加方法：

- `Builder setFooValue(int value)`: Sets the integer value of the enum.
  `Builder setFooValue(int value)` ：设置枚举的整数值。

In addition, `getFoo()` will return `UNRECOGNIZED` if the enum value is unknown—this is a special additional value added by the proto3 compiler to the generated [enum type](https://protobuf.dev/reference/java/java-generated/#enum).
此外，如果枚举值未知， `getFoo()` 将返回 `UNRECOGNIZED` ，这是 proto3 编译器添加到生成的枚举类型中的一个特殊的附加值。

### Repeated Fields 重复字段

For this field definition:
对于此字段定义：

```proto
repeated string foos = 1;
```

The compiler will generate the following accessor methods in both the message class and its builder:
编译器将在消息类及其构建器中生成以下访问器方法：

- `int getFoosCount()`: Returns the number of elements currently in the field.
  `int getFoosCount()` ：返回字段中当前的元素数。
- `String getFoos(int index)`: Returns the element at the given zero-based index.
  `String getFoos(int index)` ：返回给定基于零的索引处的元素。
- `ProtocolStringList getFoosList()`: Returns the entire field as a `ProtocolStringList`. If the field is not set, returns an empty list.
  `ProtocolStringList getFoosList()` ：将整个字段作为 `ProtocolStringList` 返回。如果未设置字段，则返回一个空列表。

The compiler will generate the following methods only in the message’s builder:
编译器仅在消息的构建器中生成以下方法：

- `Builder setFoos(int index, String value)`: Sets the value of the element at the given zero-based index.
  `Builder setFoos(int index, String value)` ：设置给定基于零的索引处的元素的值。
- `Builder addFoos(String value)`: Appends a new element to the field with the given value.
  `Builder addFoos(String value)` ：将具有给定值的新元素追加到字段。
- `Builder addAllFoos(Iterable<? extends String> value)`: Appends all elements in the given `Iterable` to the field.
  `Builder addAllFoos(Iterable<? extends String> value)` ：将给定 `Iterable` 中的所有元素追加到字段。
- `Builder clearFoos()`: Removes all elements from the field. After calling this, `getFoosCount()` will return zero.
  `Builder clearFoos()` ：从字段中删除所有元素。调用此方法后， `getFoosCount()` 将返回零。

For other simple field types, the corresponding Java type is chosen according to the [scalar value types table](https://protobuf.dev/programming-guides/proto2#scalar). For message and enum types, the type is the message or enum class.
对于其他简单字段类型，将根据标量值类型表选择相应的 Java 类型。对于消息和枚举类型，该类型是消息或枚举类。

#### Repeated Embedded Message Fields 重复嵌入的消息字段

For message types, `setFoos()` and `addFoos()` also accept an instance of the message’s builder type as the parameter. This is just a shortcut which is equivalent to calling `.build()` on the builder and passing the result to the method. There is also an additional generated method:
对于消息类型， `setFoos()` 和 `addFoos()` 还接受消息构建器类型的实例作为参数。这只是一个快捷方式，相当于在构建器上调用 `.build()` 并将结果传递给该方法。还有一个额外生成的方法：

- `Builder addFoos(int index, Field value)`: Inserts a new element at the given zero-based index. Shifts the element currently at that position (if any) and any subsequent elements to the right (adds one to their indices).
  `Builder addFoos(int index, Field value)` ：在给定的基于零的索引处插入一个新元素。将当前位于该位置的元素（如果有）和任何后续元素向右移动（将它们的索引加一）。

In addition, the compiler generates the following additional accessor methods in both the message class and its builder for message types, allowing you to access the relevant sub-builders:
此外，编译器在消息类及其构建器中为消息类型生成以下附加访问器方法，允许您访问相关的子构建器：

- `FooOrBuilder getFoosOrBuilder(int index)`: Returns the builder for the specified element, if it already exists, or throws `IndexOutOfBoundsException` if not. If this is called from a message class, it will always return a message (or throw an exception) rather than a builder. Calling this method on builders will not create a sub-builder for the field.
  `FooOrBuilder getFoosOrBuilder(int index)` ：返回指定元素的构建器（如果已存在），否则抛出 `IndexOutOfBoundsException` 。如果这是从消息类调用的，它将始终返回消息（或抛出异常），而不是构建器。在构建器上调用此方法不会为该字段创建子构建器。
- `List<FooOrBuilder> getFoosOrBuilderList()`: Returns the entire field as an unmodifiable list of builders (if available) or messages if not. If this is called from a message class, it will always return an immutable list of messages rather than an unmodifiable list of builders.
  `List<FooOrBuilder> getFoosOrBuilderList()` ：将整个字段作为不可修改的构建器列表（如果可用）或消息（如果不可用）返回。如果这是从消息类调用的，它将始终返回不可变的消息列表，而不是不可修改的构建器列表。

The compiler will generate the following methods only in the message’s builder:
编译器仅在消息的构建器中生成以下方法：

- `Builder getFoosBuilder(int index)`: Returns the builder for the element at the specified index, or throws `IndexOutOfBoundsException` if the index is out of bounds.
  `Builder getFoosBuilder(int index)` ：返回指定索引处的元素的构建器，或者如果索引超出范围，则抛出 `IndexOutOfBoundsException` 。
- `Builder addFoosBuilder(int index)`: Inserts and returns a builder for a default message instance of the repeated message at the specified index. The existing entries are shifted to higher indices to make room for the inserted builder.
  `Builder addFoosBuilder(int index)` ：插入并返回指定索引处重复消息的默认消息实例的构建器。将现有条目移至较高的索引，为插入的构建器腾出空间。
- `Builder addFoosBuilder()`: Appends and returns a builder for a default message instance of the repeated message.
  `Builder addFoosBuilder()` ：附加并返回重复消息的默认消息实例的构建器。
- `Builder removeFoos(int index)`: Removes the element at the given zero-based index.
  `Builder removeFoos(int index)` ：移除给定基于零的索引处的元素。
- `List<Builder> getFoosBuilderList()`: Returns the entire field as an unmodifiable list of builders.
  `List<Builder> getFoosBuilderList()` ：将整个字段作为不可修改的构建器列表返回。

#### Repeated Enum Fields (proto3 only) 重复枚举字段（仅限 proto3）

The compiler will generate the following additional methods in both the message class and its builder:
编译器将在消息类及其构建器中生成以下附加方法：

- `int getFoosValue(int index)`: Returns the integer value of the enum at the specified index.
  `int getFoosValue(int index)` ：返回指定索引处枚举的整数值。
- `List<java.lang.Integer> getFoosValueList()`: Returns the entire field as a list of Integers.
  `List<java.lang.Integer> getFoosValueList()` ：将整个字段作为整数列表返回。

The compiler will generate the following additional method only in the message’s builder:
编译器仅在消息的构建器中生成以下附加方法：

- `Builder setFoosValue(int index, int value)`: Sets the integer value of the enum at the specified index.
  `Builder setFoosValue(int index, int value)` ：设置指定索引处枚举的整数值。

#### Name Conflicts 名称冲突

If another non-repeated field has a name that conflicts with one of the repeated field’s generated methods, then both field names will have their protobuf field number appended to the end.
如果另一个非重复字段的名称与重复字段生成的方法之一冲突，那么这两个字段名称都会在其末尾附加 protobuf 字段编号。

For these field definitions:
对于这些字段定义：

```proto
int32 foos_count = 1;
repeated string foos = 2;
```

The compiler will first rename them to the following:
编译器首先会将它们重命名为以下内容：

```proto
int32 foos_count_1 = 1;
repeated string foos_2 = 2;
```

The accessor methods will subsequently be generated as described above.
访问器方法随后将按上述方式生成。



### Oneof Fields Oneof 字段

For this oneof field definition:
对于此 oneof 字段定义：

```proto
oneof choice {
    int32 foo_int = 4;
    string foo_string = 9;
    ...
}
```

All the fields in the `choice` oneof will use a single private field for their value. In addition, the protocol buffer compiler will generate a Java enum type for the oneof case, as follows:
`choice` oneof 中的所有字段都将使用一个私有字段作为其值。此外，协议缓冲区编译器将为 oneof 案例生成一个 Java 枚举类型，如下所示：

```java
public enum ChoiceCase
        implements com.google.protobuf.Internal.EnumLite {
      FOO_INT(4),
      FOO_STRING(9),
      ...
      CHOICE_NOT_SET(0);
      ...
    };
```

The values of this enum type have the following special methods:
此枚举类型的值具有以下特殊方法：

- `int getNumber()`: Returns the object’s numeric value as defined in the .proto file.
  `int getNumber()` ：返回对象在 .proto 文件中定义的数字值。
- `static ChoiceCase forNumber(int value)`: Returns the enum object corresponding to the given numeric value (or `null` for other numeric values).
  `static ChoiceCase forNumber(int value)` ：返回对应于给定数字值（或其他数字值的 `null` ）的枚举对象。

The compiler will generate the following accessor methods in both the message class and its builder:
编译器将在消息类及其构建器中生成以下访问器方法：

- `boolean hasFooInt()`: Returns `true` if the oneof case is `FOO`.
  `boolean hasFooInt()` ：如果 oneof 案例是 `FOO` ，则返回 `true` 。
- `int getFooInt()`: Returns the current value of `foo` if the oneof case is `FOO`. Otherwise, returns the default value of this field.
  `int getFooInt()` ：如果 oneof 案例是 `FOO` ，则返回 `foo` 的当前值。否则，返回此字段的默认值。
- `ChoiceCase getChoiceCase()`: Returns the enum indicating which field is set. Returns `CHOICE_NOT_SET` if none of them is set.
  `ChoiceCase getChoiceCase()` ：返回指示设置了哪个字段的枚举。如果未设置任何字段，则返回 `CHOICE_NOT_SET` 。

The compiler will generate the following methods only in the message’s builder:
编译器仅在消息的构建器中生成以下方法：

- `Builder setFooInt(int value)`: Sets `Foo` to this value and sets the oneof case to `FOO`. After calling this, `hasFooInt()` will return `true`, `getFooInt()` will return `value` and `getChoiceCase()` will return `FOO`.
  `Builder setFooInt(int value)` ：将 `Foo` 设置为此值，并将 oneof 案例设置为 `FOO` 。调用此方法后， `hasFooInt()` 将返回 `true` ， `getFooInt()` 将返回 `value` ， `getChoiceCase()` 将返回 `FOO` 。

- ```
  Builder clearFooInt()
  ```

  :

  - Nothing will be changed if the oneof case is not `FOO`.
    如果 oneof 案例不是 `FOO` ，则不会更改任何内容。
  - If the oneof case is `FOO`, sets `Foo` to null and the oneof case to `FOO_NOT_SET`. After calling this, `hasFooInt()` will return `false`, `getFooInt()` will return the default value and `getChoiceCase()` will return `FOO_NOT_SET`.
    如果 oneof 案例是 `FOO` ，将 `Foo` 设置为 null，并将 oneof 案例设置为 `FOO_NOT_SET` 。调用此方法后， `hasFooInt()` 将返回 `false` ， `getFooInt()` 将返回默认值， `getChoiceCase()` 将返回 `FOO_NOT_SET` 。

- `Builder.clearChoice()`: Resets the value for `choice`, returning the builder.
  `Builder.clearChoice()` ：重置 `choice` 的值，返回构建器。

For other simple field types, the corresponding Java type is chosen according to the [scalar value types table](https://protobuf.dev/programming-guides/proto2#scalar). For message and enum types, the value type is replaced with the message or enum class.
对于其他简单字段类型，将根据标量值类型表选择相应的 Java 类型。对于消息和枚举类型，值类型将替换为消息或枚举类。

### Map Fields Map 字段

For this map field definition:
对于此映射字段定义：

```proto
map<int32, int32> weight = 1;
```

The compiler will generate the following accessor methods in both the message class and its builder:
编译器将在消息类及其构建器中生成以下访问器方法：

- `Map<Integer, Integer> getWeightMap();`: Returns an unmodifiable `Map`.
  `Map<Integer, Integer> getWeightMap();` ：返回不可修改的 `Map` 。
- `int getWeightOrDefault(int key, int default);`: Returns the value for key or the default value if not present.
  `int getWeightOrDefault(int key, int default);` ：返回键的值，如果不存在，则返回默认值。
- `int getWeightOrThrow(int key);`: Returns the value for key or throws IllegalArgumentException if not present.
  `int getWeightOrThrow(int key);` ：返回键的值，如果不存在，则抛出 IllegalArgumentException。
- `boolean containsWeight(int key);`: Indicates if the key is present in this field.
  `boolean containsWeight(int key);` ：指示此字段中是否存在键。
- `int getWeightCount();`: Returns the number of elements in the map.
  `int getWeightCount();` ：返回映射中的元素数。

The compiler will generate the following methods only in the message’s builder:
编译器仅在消息的构建器中生成以下方法：

- `Builder putWeight(int key, int value);`: Add the weight to this field.
  `Builder putWeight(int key, int value);` ：向此字段添加权重。
- `Builder putAllWeight(Map<Integer, Integer> value);`: Adds all entries in the given map to this field.
  `Builder putAllWeight(Map<Integer, Integer> value);` ：将给定映射中的所有条目添加到此字段。
- `Builder removeWeight(int key);`: Removes the weight from this field.
  `Builder removeWeight(int key);` ：从此字段中移除权重。
- `Builder clearWeight();`: Removes all weights from this field.
  `Builder clearWeight();` ：从此字段中移除所有权重。
- `@Deprecated Map<Integer, Integer> getMutableWeight();`: Returns a mutable `Map`. Note that multiple calls to this method may return different map instances. The returned map reference may be invalidated by any subsequent method calls to the Builder.
  `@Deprecated Map<Integer, Integer> getMutableWeight();` ：返回一个可变的 `Map` 。请注意，对该方法的多次调用可能会返回不同的映射实例。对 Builder 的任何后续方法调用都可能会使返回的映射引用无效。

#### Message Value Map Fields 消息值映射字段

For maps with message types as values, the compiler will generate an additional method in the message’s builder:
对于将消息类型用作值的映射，编译器将在消息的构建器中生成一个附加方法：

- `Foo.Builder putFooBuilderIfAbsent(int key);`: Ensures that `key` is present in the mapping, and inserts a new `Foo.Builder` if one does not already exist. Changes to the returned `Foo.Builder` will be reflected in the final message.
  `Foo.Builder putFooBuilderIfAbsent(int key);` ：确保 `key` 存在于映射中，如果尚不存在，则插入一个新的 `Foo.Builder` 。对返回的 `Foo.Builder` 所做的更改将反映在最终消息中。

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

```java
class Any {
  // Packs the given message into an Any using the default type URL
  // prefix “type.googleapis.com”.
  public static Any pack(Message message);
  // Packs the given message into an Any using the given type URL
  // prefix.
  public static Any pack(Message message,
                         String typeUrlPrefix);

  // Checks whether this Any message’s payload is the given type.
  public <T extends Message> boolean is(class<T> clazz);

  // Unpacks Any into the given message type. Throws exception if
  // the type doesn’t match or parsing the payload has failed.
  public <T extends Message> T unpack(class<T> clazz)
      throws InvalidProtocolBufferException;
}
```

## Enumerations 枚举

Given an enum definition like:
给定枚举定义，例如：

```proto
enum Foo {
  VALUE_A = 0;
  VALUE_B = 5;
  VALUE_C = 1234;
}
```

The protocol buffer compiler will generate a Java enum type called `Foo` with the same set of values. If you are using proto3, it also adds the special value `UNRECOGNIZED` to the enum type. The values of the generated enum type have the following special methods:
协议缓冲区编译器将生成一个名为 `Foo` 的 Java 枚举类型，其中包含相同的值集。如果您使用的是 proto3，它还会将特殊值 `UNRECOGNIZED` 添加到枚举类型。生成的枚举类型的值具有以下特殊方法：

- `int getNumber()`: Returns the object’s numeric value as defined in the `.proto` file.
  `int getNumber()` ：返回对象在 `.proto` 文件中定义的数字值。
- `EnumValueDescriptor getValueDescriptor()`: Returns the value’s descriptor, which contains information about the value’s name, number, and type.
  `EnumValueDescriptor getValueDescriptor()` ：返回该值的描述符，其中包含有关该值名称、编号和类型的信息。
- `EnumDescriptor getDescriptorForType()`: Returns the enum type’s descriptor, which contains e.g. information about each defined value.
  `EnumDescriptor getDescriptorForType()` ：返回枚举类型的描述符，其中包含有关每个已定义值的信息。

Additionally, the `Foo` enum type contains the following static methods:
此外， `Foo` 枚举类型包含以下静态方法：

- `static Foo forNumber(int value)`: Returns the enum object corresponding to the given numeric value. Returns null when there is no corresponding enum object.
  `static Foo forNumber(int value)` ：返回与给定数值对应的枚举对象。当没有相应的枚举对象时，返回 null。
- `static Foo valueOf(int value)`: Returns the enum object corresponding to the given numeric value. This method is deprecated in favor of `forNumber(int value)` and will be removed in an upcoming release.
  `static Foo valueOf(int value)` ：返回与给定数值对应的枚举对象。此方法已弃用，建议使用 `forNumber(int value)` ，且将在即将发布的版本中将其移除。
- `static Foo valueOf(EnumValueDescriptor descriptor)`: Returns the enum object corresponding to the given value descriptor. May be faster than `valueOf(int)`. In proto3 returns `UNRECOGNIZED` if passed an unknown value descriptor.
  `static Foo valueOf(EnumValueDescriptor descriptor)` ：返回与给定值描述符对应的枚举对象。可能比 `valueOf(int)` 更快。在 proto3 中，如果传递未知的值描述符，则返回 `UNRECOGNIZED` 。
- `EnumDescriptor getDescriptor()`: Returns the enum type’s descriptor, which contains e.g. information about each defined value. (This differs from `getDescriptorForType()` only in that it is a static method.)
  `EnumDescriptor getDescriptor()` ：返回枚举类型的描述符，其中包含有关每个已定义值的信息。（这与 `getDescriptorForType()` 的唯一区别在于它是一个静态方法。）

An integer constant is also generated with the suffix _VALUE for each enum value.
还会为每个枚举值生成一个带有后缀_VALUE的整数常量。

Note that the `.proto` language allows multiple enum symbols to have the same numeric value. Symbols with the same numeric value are synonyms. For example:
请注意， `.proto` 语言允许多个枚举符号具有相同的数值。具有相同数值的符号是同义词。例如：

```proto
enum Foo {
  BAR = 0;
  BAZ = 0;
}
```

In this case, `BAZ` is a synonym for `BAR`. In Java, `BAZ` will be defined as a static final field like so:
在这种情况下， `BAZ` 是 `BAR` 的同义词。在 Java 中， `BAZ` 将定义为一个静态 final 字段，如下所示：

```java
static final Foo BAZ = BAR;
```

Thus, `BAR` and `BAZ` compare equal, and `BAZ` should never appear in switch statements. The compiler always chooses the first symbol defined with a given numeric value to be the “canonical” version of that symbol; all subsequent symbols with the same number are just aliases.
因此， `BAR` 和 `BAZ` 比较相等，并且 `BAZ` 永远不应出现在 switch 语句中。编译器始终选择使用给定数字值定义的第一个符号作为该符号的“规范”版本；所有具有相同数字的后续符号都只是别名。

An enum can be defined nested within a message type. The compiler generates the Java enum definition nested within that message type’s class.
可以在消息类型中嵌套定义枚举。编译器在该消息类型的类中生成嵌套的 Java 枚举定义。

**Caution: when generating Java code, the maximum number of values in a protobuf enum may be surprisingly low**—in the worst case, the maximum is slightly over 1,700 values. This limit is due to per-method size limits for Java bytecode, and it varies across Java implementations, different versions of the protobuf suite, and any options set on the enum in the `.proto` file.
注意：在生成 Java 代码时，protobuf 枚举中的最大值数量可能出乎意料地低——在最坏的情况下，最大值略高于 1,700 个值。此限制是由于 Java 字节码的每种方法大小限制，并且它因不同的 Java 实现、protobuf 套件的不同版本以及在 `.proto` 文件中对枚举设置的任何选项而异。

## Extensions (proto2 only) 扩展（仅限 proto2）

Given a message with an extension range:
给定具有扩展范围的消息：

```proto
message Foo {
  extensions 100 to 199;
}
```

The protocol buffer compiler will make `Foo` extend `GeneratedMessage.ExtendableMessage` instead of the usual `GeneratedMessage`. Similarly, `Foo`’s builder will extend `GeneratedMessage.ExtendableBuilder`. You should never refer to these base types by name (`GeneratedMessage` is considered an implementation detail). However, these superclasses define a number of additional methods that you can use to manipulate extensions.
协议缓冲区编译器将使 `Foo` 扩展 `GeneratedMessage.ExtendableMessage` ，而不是通常的 `GeneratedMessage` 。同样， `Foo` 的构建器将扩展 `GeneratedMessage.ExtendableBuilder` 。您永远不应按名称引用这些基本类型（ `GeneratedMessage` 被视为实现细节）。但是，这些超类定义了许多其他方法，您可以使用这些方法来操作扩展。

In particular `Foo` and `Foo.Builder` will inherit the methods `hasExtension()`, `getExtension()`, and `getExtensionCount()`. Additionally, `Foo.Builder` will inherit methods `setExtension()` and `clearExtension()`. Each of these methods takes, as its first parameter, an extension identifier (described below), which identifies an extension field. The remaining parameters and the return value are exactly the same as those for the corresponding accessor methods that would be generated for a normal (non-extension) field of the same type as the extension identifier.
特别是 `Foo` 和 `Foo.Builder` 将继承方法 `hasExtension()` 、 `getExtension()` 和 `getExtensionCount()` 。此外， `Foo.Builder` 将继承方法 `setExtension()` 和 `clearExtension()` 。这些方法中的每一个都将扩展标识符（如下所述）作为其第一个参数，该标识符标识扩展字段。其余参数和返回值与为与扩展标识符类型相同的普通（非扩展）字段生成的相应访问器方法的参数和返回值完全相同。

Given an extension definition:
给定扩展定义：

```proto
extend Foo {
  optional int32 bar = 123;
}
```

The protocol buffer compiler generates an “extension identifier” called `bar`, which you can use with `Foo`’s extension accessors to access this extension, like so:
协议缓冲区编译器生成一个名为 `bar` 的“扩展标识符”，您可以将其与 `Foo` 的扩展访问器一起使用来访问此扩展，如下所示：

```java
Foo foo =
  Foo.newBuilder()
     .setExtension(bar, 1)
     .build();
assert foo.hasExtension(bar);
assert foo.getExtension(bar) == 1;
```

(The exact implementation of extension identifiers is complicated and involves magical use of generics—however, you don’t need to worry about how extension identifiers work to use them.)
（扩展标识符的确切实现很复杂，涉及泛型的魔术用法——但是，您无需担心扩展标识符的工作原理即可使用它们。）

Note that `bar` would be declared as a static field of the wrapper class for the `.proto` file, as described above; we have omitted the wrapper class name in the example.
请注意， `bar` 将被声明为 `.proto` 文件的包装器类的静态字段，如上所述；我们在示例中省略了包装器类名称。

Extensions can be declared inside the scope of another type to prefix their generated symbol names. For example, a common pattern is to extend a message by a field *inside* the declaration of the field’s type:
可以在另一个类型的范围内声明扩展名，以给生成的符号名称添加前缀。例如，一种常见模式是通过字段类型声明中的字段来扩展消息：

```proto
message Baz {
  extend Foo {
    optional Baz foo_ext = 124;
  }
}
```

In this case, an extension with identifier `foo_ext` and type `Baz` is declared inside the declaration of `Baz`, and referring to `foo_ext` requires the addition of a `Baz.` prefix:
在这种情况下，扩展名（标识符为 `foo_ext` ，类型为 `Baz` ）在 `Baz` 的声明中声明，并且引用 `foo_ext` 需要添加 `Baz.` 前缀：

```java
Baz baz = createMyBaz();
Foo foo =
  Foo.newBuilder()
     .setExtension(Baz.fooExt, baz)
     .build();
assert foo.hasExtension(Baz.fooExt);
assert foo.getExtension(Baz.fooExt) == baz;
```

When parsing a message that might have extensions, you must provide an [`ExtensionRegistry`](https://protobuf.dev/reference/java/api-docs/com/google/protobuf/ExtensionRegistry.html) in which you have registered any extensions that you want to be able to parse. Otherwise, those extensions will be treated like unknown fields and the methods observing extensions will behave as if they don’t exist.
解析可能具有扩展名的消息时，您必须提供一个 `ExtensionRegistry` ，其中注册了您想要解析的任何扩展名。否则，这些扩展名将被视为未知字段，并且观察扩展名的函数将表现得好像它们不存在一样。

```java
ExtensionRegistry registry = ExtensionRegistry.newInstance();
registry.add(Baz.fooExt);
Foo foo = Foo.parseFrom(input, registry);
assert foo.hasExtension(Baz.fooExt);
ExtensionRegistry registry = ExtensionRegistry.newInstance();
Foo foo = Foo.parseFrom(input, registry);
assert foo.hasExtension(Baz.fooExt) == false;
```

## Services 服务

If the `.proto` file contains the following line:
如果 `.proto` 文件包含以下行：

```proto
option java_generic_services = true;
```

Then the protocol buffer compiler will generate code based on the service definitions found in the file as described in this section. However, the generated code may be undesirable as it is not tied to any particular RPC system, and thus requires more levels of indirection than code tailored to one system. If you do NOT want this code to be generated, add this line to the file:
然后，协议缓冲区编译器将根据本节所述的文件中找到的服务定义生成代码。但是，生成的代码可能不理想，因为它与任何特定的 RPC 系统无关，因此需要比针对一个系统定制的代码更多级别的间接。如果您不希望生成此代码，请将此行添加到文件中：

```proto
option java_generic_services = false;
```

If neither of the above lines are given, the option defaults to `false`, as generic services are deprecated. (Note that prior to 2.4.0, the option defaults to `true`)
如果未给出上述任何一行，则该选项默认为 `false` ，因为通用服务已弃用。（请注意，在 2.4.0 之前，该选项默认为 `true` ）

RPC systems based on `.proto`-language service definitions should provide [plugins](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.compiler.plugin.pb) to generate code appropriate for the system. These plugins are likely to require that abstract services are disabled, so that they can generate their own classes of the same names. Plugins are new in version 2.3.0 (January 2010).
基于 `.proto` 语言服务定义的 RPC 系统应提供用于生成适用于该系统的代码的插件。这些插件可能需要禁用抽象服务，以便它们能够生成具有相同名称的自己的类。插件是 2.3.0 版（2010 年 1 月）中的新增功能。

The remainder of this section describes what the protocol buffer compiler generates when abstract services are enabled.
本节的其余部分描述了在启用抽象服务时协议缓冲区编译器生成的内容。

### Interface 接口

Given a service definition:
给定服务定义：

```proto
service Foo {
  rpc Bar(FooRequest) returns(FooResponse);
}
```

The protocol buffer compiler will generate an abstract class `Foo` to represent this service. `Foo` will have an abstract method for each method defined in the service definition. In this case, the method `Bar` is defined as:
协议缓冲区编译器将生成一个抽象类 `Foo` 来表示此服务。 `Foo` 将为服务定义中定义的每个方法提供一个抽象方法。在这种情况下，方法 `Bar` 定义为：

```java
abstract void bar(RpcController controller, FooRequest request,
                  RpcCallback<FooResponse> done);
```

The parameters are equivalent to the parameters of `Service.CallMethod()`, except that the `method` argument is implied and `request` and `done` specify their exact type.
这些参数等同于 `Service.CallMethod()` 的参数，但隐含了 `method` 参数，而 `request` 和 `done` 指定了它们的确切类型。

`Foo` subclasses the `Service` interface. The protocol buffer compiler automatically generates implementations of the methods of `Service` as follows:
`Foo` 对 `Service` 接口进行子类化。协议缓冲区编译器自动生成 `Service` 的方法的实现，如下所示：

- `getDescriptorForType`: Returns the service’s `ServiceDescriptor`.
  `getDescriptorForType` ：返回服务的 `ServiceDescriptor` 。
- `callMethod`: Determines which method is being called based on the provided method descriptor and calls it directly, down-casting the request message and callback to the correct types.
  `callMethod` ：根据提供的 method descriptor 确定正在调用哪个方法，并直接调用它，将请求消息和回调向下转换为正确的类型。
- `getRequestPrototype` and `getResponsePrototype`: Returns the default instance of the request or response of the correct type for the given method.
  `getRequestPrototype` 和 `getResponsePrototype` ：返回给定方法的正确类型的请求或响应的默认实例。

The following static method is also generated:
还会生成以下静态方法：

- `static ServiceDescriptor getServiceDescriptor()`: Returns the type’s descriptor, which contains information about what methods this service has and what their input and output types are.
  `static ServiceDescriptor getServiceDescriptor()` ：返回类型的描述符，其中包含有关此服务具有哪些方法以及它们的输入和输出类型是什么的信息。

`Foo` will also contain a nested interface `Foo.Interface`. This is a pure interface that again contains methods corresponding to each method in your service definition. However, this interface does not extend the `Service` interface. This is a problem because RPC server implementations are usually written to use abstract `Service` objects, not your particular service. To solve this problem, if you have an object `impl` implementing `Foo.Interface`, you can call `Foo.newReflectiveService(impl)` to construct an instance of `Foo` that simply delegates to `impl`, and implements `Service`.
`Foo` 还将包含一个嵌套接口 `Foo.Interface` 。这是一个纯接口，它再次包含与服务定义中的每个方法相对应的方法。但是，此接口不会扩展 `Service` 接口。这是一个问题，因为 RPC 服务器实现通常被编写为使用抽象 `Service` 对象，而不是您的特定服务。要解决此问题，如果您有一个实现 `Foo.Interface` 的对象 `impl` ，则可以调用 `Foo.newReflectiveService(impl)` 来构造一个 `Foo` 实例，该实例仅委托给 `impl` ，并实现 `Service` 。

To recap, when implementing your own service, you have two options:
回顾一下，在实现您自己的服务时，您有两个选择：

- Subclass `Foo` and implement its methods as appropriate, then hand instances of your subclass directly to the RPC server implementation. This is usually easiest, but some consider it less “pure”.
  子类化 `Foo` 并根据需要实现其方法，然后将子类的实例直接交给 RPC 服务器实现。这通常最简单，但有些人认为它不太“纯”。
- Implement `Foo.Interface` and use `Foo.newReflectiveService(Foo.Interface)` to construct a `Service` wrapping it, then pass the wrapper to your RPC implementation.
  实现 `Foo.Interface` 并使用 `Foo.newReflectiveService(Foo.Interface)` 构造一个包装它的 `Service` ，然后将包装器传递给您的 RPC 实现。

### Stub 存根

The protocol buffer compiler also generates a “stub” implementation of every service interface, which is used by clients wishing to send requests to servers implementing the service. For the `Foo` service (above), the stub implementation `Foo.Stub` will be defined as a nested class.
协议缓冲区编译器还会生成每个服务接口的“存根”实现，供希望向实现该服务的服务器发送请求的客户端使用。对于 `Foo` 服务（以上），存根实现 `Foo.Stub` 将被定义为嵌套类。

`Foo.Stub` is a subclass of `Foo` which also implements the following methods:
`Foo.Stub` 是 `Foo` 的子类，它还实现了以下方法：

- `Foo.Stub(RpcChannel channel)`: Constructs a new stub which sends requests on the given channel.
  `Foo.Stub(RpcChannel channel)` ：构造一个在给定通道上发送请求的新存根。
- `RpcChannel getChannel()`: Returns this stub’s channel, as passed to the constructor.
  `RpcChannel getChannel()` ：返回此存根的通道，如传递给构造函数。

The stub additionally implements each of the service’s methods as a wrapper around the channel. Calling one of the methods simply calls `channel.callMethod()`.
此外，存根还将服务的每个方法实现为通道周围的包装器。调用其中一个方法只是调用 `channel.callMethod()` 。

The Protocol Buffer library does not include an RPC implementation. However, it includes all of the tools you need to hook up a generated service class to any arbitrary RPC implementation of your choice. You need only provide implementations of `RpcChannel` and `RpcController`.
协议缓冲区库不包括 RPC 实现。但是，它包含将生成的 service 类连接到您选择的任何任意 RPC 实现所需的所有工具。您只需提供 `RpcChannel` 和 `RpcController` 的实现。

### Blocking Interfaces 阻塞接口

The RPC classes described above all have non-blocking semantics: when you call a method, you provide a callback object which will be invoked once the method completes. Often it is easier (though possibly less scalable) to write code using blocking semantics, where the method simply doesn’t return until it is done. To accommodate this, the protocol buffer compiler also generates blocking versions of your service class. `Foo.BlockingInterface` is equivalent to `Foo.Interface` except that each method simply returns the result rather than call a callback. So, for example, `bar` is defined as:
上面描述的所有 RPC 类都具有非阻塞语义：当您调用方法时，您提供一个回调对象，该对象将在方法完成后被调用。通常，使用阻塞语义编写代码更容易（尽管可能扩展性较差），其中方法在完成之前根本不会返回。为了适应这一点，协议缓冲区编译器还会生成服务类的阻塞版本。 `Foo.BlockingInterface` 等同于 `Foo.Interface` ，只是每个方法只是返回结果，而不是调用回调。因此，例如， `bar` 被定义为：

```java
abstract FooResponse bar(RpcController controller, FooRequest request)
                         throws ServiceException;
```

Analogous to non-blocking services, `Foo.newReflectiveBlockingService(Foo.BlockingInterface)` returns a `BlockingService` wrapping some `Foo.BlockingInterface`. Finally, `Foo.BlockingStub` returns a stub implementation of `Foo.BlockingInterface` that sends requests to a particular `BlockingRpcChannel`.
类似于非阻塞服务， `Foo.newReflectiveBlockingService(Foo.BlockingInterface)` 返回一个包装某些 `Foo.BlockingInterface` 的 `BlockingService` 。最后， `Foo.BlockingStub` 返回一个 `Foo.BlockingInterface` 的存根实现，该实现将请求发送到特定的 `BlockingRpcChannel` 。

## Plugin Insertion Points 插件插入点

[Code generator plugins](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.compiler.plugin.pb) that want to extend the output of the Java code generator may insert code of the following types using the given insertion point names.
希望扩展 Java 代码生成器输出的代码生成器插件可以使用给定的插入点名称插入以下类型的代码。

- `outer_class_scope`: Member declarations that belong in the file’s wrapper class.
  `outer_class_scope` ：属于文件包装器类的成员声明。
- `class_scope:TYPENAME`: Member declarations that belong in a message class. `TYPENAME` is the full proto name, e.g. `package.MessageType`.
  `class_scope:TYPENAME` ：属于消息类的成员声明。 `TYPENAME` 是完整协议名称，例如 `package.MessageType` 。
- `builder_scope:TYPENAME`: Member declarations that belong in a message’s builder class. `TYPENAME` is the full proto name, e.g. `package.MessageType`.
  `builder_scope:TYPENAME` ：属于消息构建器类的成员声明。 `TYPENAME` 是完整协议名称，例如 `package.MessageType` 。
- `enum_scope:TYPENAME`: Member declarations that belong in an enum class. `TYPENAME` is the full proto enum name, e.g. `package.EnumType`.
  `enum_scope:TYPENAME` ：属于枚举类的成员声明。 `TYPENAME` 是完整协议枚举名称，例如 `package.EnumType` 。
- `message_implements:TYPENAME`: Class implementation declarations for a message class. `TYPENAME` is the full proto name, e.g. `package.MessageType`.
  `message_implements:TYPENAME` ：消息类的类实现声明。 `TYPENAME` 是完整协议名称，例如 `package.MessageType` 。
- `builder_implements:TYPENAME`: Class implementation declarations for a builder class. `TYPENAME` is the full proto name, e.g. `package.MessageType`.
  `builder_implements:TYPENAME` ：构建器类的类实现声明。 `TYPENAME` 是完整协议名称，例如 `package.MessageType` 。

Generated code cannot contain import statements, as these are prone to conflict with type names defined within the generated code itself. Instead, when referring to an external class, you must always use its fully-qualified name.
生成的代码不能包含 import 语句，因为这些语句容易与生成的代码本身中定义的类型名称发生冲突。相反，在引用外部类时，您必须始终使用其完全限定名称。

The logic for determining output file names in the Java code generator is fairly complicated. You should probably look at the `protoc` source code, particularly `java_headers.cc`, to make sure you have covered all cases.
Java 代码生成器中确定输出文件名逻辑非常复杂。您可能应该查看 `protoc` 源代码，特别是 `java_headers.cc` ，以确保您已涵盖所有情况。

Do not generate code which relies on private class members declared by the standard code generator, as these implementation details may change in future versions of Protocol Buffers.
不要生成依赖于标准代码生成器声明的私有类成员的代码，因为这些实现细节可能会在 Protocol Buffers 的未来版本中发生变化。

## Utility Classes 实用程序类

Protocol buffer provides [utility classes](https://protobuf.dev/reference/java/api-docs/com/google/protobuf/util/package-summary.html) for message comparison, JSON conversion and working with [well-known types (predefined protocol buffer messages for common use-cases).](https://protobuf.dev/reference/protobuf/google.protobuf)
Protocol buffer 提供实用程序类，用于消息比较、JSON 转换以及处理众所周知的类型（用于常见用例的预定义 protocol buffer 消息）。
