+++
title = "C++ Generated Code Guide"
weight = 510
linkTitle = "Generated Code Guide"
description = "Describes exactly what C++ code the protocol buffer compiler generates for any given protocol definition. "
type = "docs"
+++

> 原文网址：https://protobuf.dev/reference/cpp/cpp-generated/

# C++ Generated Code Guide C++ 生成代码指南

Describes exactly what C++ code the protocol buffer compiler generates for any given protocol definition.
准确描述协议缓冲区编译器为任何给定协议定义生成的 C++ 代码。突出显示 proto2 和 proto3 生成代码之间的任何差异 - 请注意，这些差异在于本文档中描述的生成代码中，而不是基本消息类/接口，它们在两个版本中都是相同的。在阅读本文档之前，您应该阅读 proto2 语言指南和/或 proto3 语言指南。



Any differences between proto2 and proto3 generated code are highlighted - note that these differences are in the generated code as described in this document, not the base message classes/interfaces, which are the same in both versions. You should read the [proto2 language guide](https://protobuf.dev/programming-guides/proto2) and/or [proto3 language guide](https://protobuf.dev/programming-guides/proto3) before reading this document.
使用 命令行标志调用协议缓冲区编译器时，它会生成 C++ 输出。 选项的参数是要让编译器写入 C++ 输出的目录。编译器为每个 文件输入创建一个头文件和一个实现文件。输出文件的名称是通过获取 文件的名称并进行两次更改来计算的：

## Compiler Invocation 编译器调用

The protocol buffer compiler produces C++ output when invoked with the `--cpp_out=` command-line flag. The parameter to the `--cpp_out=` option is the directory where you want the compiler to write your C++ output. The compiler creates a header file and an implementation file for each `.proto` file input. The names of the output files are computed by taking the name of the `.proto` file and making two changes:
扩展名 ( `--cpp_out=` ) 分别替换为 `--cpp_out=` 或 `.proto` ，用于头文件或实现文件。

- The extension (`.proto`) is replaced with either `.pb.h` or `.pb.cc` for the header or implementation file, respectively.
  因此，例如，假设您按如下方式调用编译器：
- The proto path (specified with the `--proto_path=` or `-I` command-line flag) is replaced with the output path (specified with the `--cpp_out=` flag).
  使用 `--proto_path=` 或 `-I` 命令行标志指定的 proto 路径将替换为使用 `--cpp_out=` 标志指定的输出路径。

So, for example, let’s say you invoke the compiler as follows:

```shell
protoc --proto_path=src --cpp_out=build/gen src/foo.proto src/bar/baz.proto
```

The compiler will read the files `src/foo.proto` and `src/bar/baz.proto` and produce four output files: `build/gen/foo.pb.h`, `build/gen/foo.pb.cc`, `build/gen/bar/baz.pb.h`, `build/gen/bar/baz.pb.cc`. The compiler will automatically create the directory `build/gen/bar` if necessary, but it will *not* create `build` or `build/gen`; they must already exist.
编译器将读取文件 `src/foo.proto` 和 `src/bar/baz.proto` 并生成四个输出文件： `build/gen/foo.pb.h` 、 `build/gen/foo.pb.cc` 、 `build/gen/bar/baz.pb.h` 、 `build/gen/bar/baz.pb.cc` 。编译器将自动创建目录 `build/gen/bar` （如果需要），但不会创建 `build` 或 `build/gen` ；它们必须已经存在。

## Packages 包

If a `.proto` file contains a `package` declaration, the entire contents of the file will be placed in a corresponding C++ namespace. For example, given the `package` declaration:
如果 `.proto` 文件包含 `package` 声明，则该文件的全部内容将被置于相应的 C++ 命名空间中。例如，给定 `package` 声明：

```proto
package foo.bar;
```

All declarations in the file will reside in the `foo::bar` namespace.
文件中的所有声明都将驻留在 `foo::bar` 命名空间中。

## Messages 消息

Given a simple message declaration:
给定一个简单的消息声明：

```proto
message Foo {}
```

The protocol buffer compiler generates a class called `Foo`, which publicly derives from [`google::protobuf::Message`](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.message). The class is a concrete class; no pure-virtual methods are left unimplemented. Methods that are virtual in `Message` but not pure-virtual may or may not be overridden by `Foo`, depending on the optimization mode. By default, `Foo` implements specialized versions of all methods for maximum speed. However, if the `.proto` file contains the line:
协议缓冲区编译器生成一个名为 `Foo` 的类，该类公开派生自 `google::protobuf::Message` 。该类是一个具体类；没有纯虚方法未实现。在 `Message` 中为虚方法但不是纯虚方法可能会被 `Foo` 覆盖，具体取决于优化模式。默认情况下， `Foo` 为所有方法实现专门版本，以实现最高速度。但是，如果 `.proto` 文件包含以下行：

```proto
option optimize_for = CODE_SIZE;
```

then `Foo` will override only the minimum set of methods necessary to function and rely on reflection-based implementations of the rest. This significantly reduces the size of the generated code, but also reduces performance. Alternatively, if the `.proto` file contains:
那么 `Foo` 将仅覆盖运行所需的最小方法集，并依赖于其余方法的基于反射的实现。这显著减小了生成代码的大小，但也降低了性能。或者，如果 `.proto` 文件包含：

```proto
option optimize_for = LITE_RUNTIME;
```

then `Foo` will include fast implementations of all methods, but will implement the [`google::protobuf::MessageLite`](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.message_lite) interface, which only contains a subset of the methods of `Message`. In particular, it does not support descriptors or reflection. However, in this mode, the generated code only needs to link against `libprotobuf-lite.so` (`libprotobuf-lite.lib` on Windows) instead of `libprotobuf.so` (`libprotobuf.lib`). The “lite” library is much smaller than the full library, and is more appropriate for resource-constrained systems such as mobile phones.
那么 `Foo` 将包含所有方法的快速实现，但将实现 `google::protobuf::MessageLite` 接口，该接口仅包含 `Message` 方法的子集。特别是，它不支持描述符或反射。但是，在此模式下，生成的代码只需链接到 `libprotobuf-lite.so` （Windows 上的 `libprotobuf-lite.lib` ），而不是 `libprotobuf.so` （ `libprotobuf.lib` ）。“精简”库比完整库小得多，更适合资源受限的系统，例如手机。

You should *not* create your own `Foo` subclasses. If you subclass this class and override a virtual method, the override may be ignored, as many generated method calls are de-virtualized to improve performance.
您不应创建自己的 `Foo` 子类。如果您对该类进行子类化并覆盖一个虚拟方法，则可能会忽略该覆盖，因为许多生成的调用方法被取消虚拟化以提高性能。

The `Message` interface defines methods that let you check, manipulate, read, or write the entire message, including parsing from and serializing to binary strings.
接口 `Message` 定义了允许您检查、操作、读取或写入整个消息的方法，包括从二进制字符串进行解析和序列化到二进制字符串。

- `bool ParseFromString(const string& data)`: Parse the message from the given serialized binary string (also known as wire format).
  `bool ParseFromString(const string& data)` ：解析给定的序列化二进制字符串（也称为线格式）中的消息。
- `bool SerializeToString(string* output) const`: Serialize the given message to a binary string.
  `bool SerializeToString(string* output) const` ：将给定的消息序列化为二进制字符串。
- `string DebugString()`: Return a string giving the `text_format` representation of the proto (should only be used for debugging).
  `string DebugString()` ：返回一个字符串，提供 proto 的 `text_format` 表示形式（仅应用于调试）。

In addition to these methods, the `Foo` class defines the following methods:
除了这些方法之外， `Foo` 类还定义了以下方法：

- `Foo()`: Default constructor.
  `Foo()` ：默认构造函数。
- `~Foo()`: Default destructor.
  `~Foo()` ：默认析构函数。
- `Foo(const Foo& other)`: Copy constructor.
  `Foo(const Foo& other)` ：复制构造函数。
- `Foo(Foo&& other)`: Move constructor.
  `Foo(Foo&& other)` ：移动构造函数。
- `Foo& operator=(const Foo& other)`: Assignment operator.
  `Foo& operator=(const Foo& other)` ：赋值运算符。
- `Foo& operator=(Foo&& other)`: Move-assignment operator.
  `Foo& operator=(Foo&& other)` ：移动赋值运算符。
- `void Swap(Foo* other)`: Swap content with another message.
  `void Swap(Foo* other)` ：与另一条消息交换内容。
- `const UnknownFieldSet& unknown_fields() const`: Returns the set of unknown fields encountered while parsing this message. If `option optimize_for = LITE_RUNTIME` is specified in the `.proto` file, then the return type changes to `std::string&`.
  `const UnknownFieldSet& unknown_fields() const` ：返回解析此消息时遇到的未知字段集。如果在 `.proto` 文件中指定了 `option optimize_for = LITE_RUNTIME` ，则返回类型将更改为 `std::string&` 。
- `UnknownFieldSet* mutable_unknown_fields()`: Returns a pointer to the mutable set of unknown fields encountered while parsing this message. If `option optimize_for = LITE_RUNTIME` is specified in the `.proto` file, then the return type changes to `std::string*`.
  `UnknownFieldSet* mutable_unknown_fields()` ：返回解析此消息时遇到的未知字段的可变集的指针。如果在 `.proto` 文件中指定了 `option optimize_for = LITE_RUNTIME` ，则返回类型将更改为 `std::string*` 。

The class also defines the following static methods:
该类还定义了以下静态方法：

- `static const Descriptor* descriptor()`: Returns the type’s descriptor. This contains information about the type, including what fields it has and what their types are. This can be used with [reflection](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.message#Reflection) to inspect fields programmatically.
  `static const Descriptor* descriptor()` ：返回类型的描述符。其中包含有关该类型的信息，包括它拥有的字段及其类型。这可与反射一起使用，以通过编程方式检查字段。
- `static const Foo& default_instance()`: Returns a const singleton instance of `Foo` which is identical to a newly-constructed instance of `Foo` (so all singular fields are unset and all repeated fields are empty). Note that the default instance of a message can be used as a factory by calling its `New()` method.
  `static const Foo& default_instance()` ：返回 `Foo` 的 const 单例实例，该实例与新构造的 `Foo` 实例相同（因此所有奇数字段都未设置，所有重复字段都为空）。请注意，可以通过调用消息的 `New()` 方法将消息的默认实例用作工厂。

### Generated Filenames 生成的文件名

[Reserved keywords](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/compiler/cpp/helpers.cc#L4) are appended with an underscore in the generated output.
在生成的输出中，保留关键字后跟下划线。

For example, the following proto3 definition syntax:
例如，以下 proto3 定义语法：

```proto
message MyMessage {
  string false = 1;
  string myFalse = 2;
}
```

generates the following partial output:
生成以下部分输出：

```cpp
  void clear_false_() ;
  const std::string& false_() const;
  void set_false_(Arg_&& arg, Args_... args);
  std::string* mutable_false_();
  PROTOBUF_NODISCARD std::string* release_false_();
  void set_allocated_false_(std::string* ptr);

  void clear_myfalse() ;
  const std::string& myfalse() const;
  void set_myfalse(Arg_&& arg, Args_... args);
  std::string* mutable_myfalse();
  PROTOBUF_NODISCARD std::string* release_myfalse();
  void set_allocated_myfalse(std::string* ptr);
```

### Nested Types 嵌套类型

A message can be declared inside another message. For example:
可以在另一个消息中声明消息。例如：

```proto
message Foo {
  message Bar {}
}
```

In this case, the compiler generates two classes: `Foo` and `Foo_Bar`. In addition, the compiler generates a typedef inside `Foo` as follows:
在这种情况下，编译器会生成两个类： `Foo` 和 `Foo_Bar` 。此外，编译器还会在 `Foo` 中生成一个 typedef，如下所示：

```cpp
typedef Foo_Bar Bar;
```

This means that you can use the nested type’s class as if it was the nested class `Foo::Bar`. However, note that C++ does not allow nested types to be forward-declared. If you want to forward-declare `Bar` in another file and use that declaration, you must identify it as `Foo_Bar`.
这意味着您可以使用嵌套类型的类，就好像它是嵌套类 `Foo::Bar` 一样。但是，请注意，C++ 不允许嵌套类型被前向声明。如果您想在另一个文件中前向声明 `Bar` 并使用该声明，则必须将其标识为 `Foo_Bar` 。

## Fields 字段

In addition to the methods described in the previous section, the protocol buffer compiler generates a set of accessor methods for each field defined within the message in the `.proto` file. These methods are in lower-case/snake-case, such as `has_foo()` and `clear_foo()`.
除了上一节中描述的方法外，协议缓冲区编译器还会为 `.proto` 文件中定义的每个字段生成一组访问器方法。这些方法采用小写/蛇形命名法，例如 `has_foo()` 和 `clear_foo()` 。

As well as accessor methods, the compiler generates an integer constant for each field containing its field number. The constant name is the letter `k`, followed by the field name converted to camel-case, followed by `FieldNumber`. For example, given the field `optional int32 foo_bar = 5;`, the compiler will generate the constant `static const int kFooBarFieldNumber = 5;`.
除了访问器方法外，编译器还会为包含其字段编号的每个字段生成一个整数常量。常量名称是字母 `k` ，后跟转换为驼峰式大小写的字段名称，后跟 `FieldNumber` 。例如，给定字段 `optional int32 foo_bar = 5;` ，编译器将生成常量 `static const int kFooBarFieldNumber = 5;` 。

For field accessors returning a `const` reference, that reference may be invalidated when the next modifying access is made to the message. This includes calling any non-`const` accessor of any field, calling any non-`const` method inherited from `Message` or modifying the message through other ways (for example, by using the message as the argument of `Swap()`). Correspondingly, the address of the returned reference is only guaranteed to be the same across different invocations of the accessor if no modifying access was made to the message in the meantime.
对于返回 `const` 引用类型的字段访问器，当对消息进行下一次修改访问时，该引用可能会失效。这包括调用任何字段的非 `const` 访问器，调用从 `Message` 继承的任何非 `const` 方法，或通过其他方式修改消息（例如，通过将消息用作 `Swap()` 的参数）。相应地，只有在在此期间未对消息进行修改访问的情况下，才能保证返回的引用的地址在访问器的不同调用之间相同。

For field accessors returning a pointer, that pointer may be invalidated when the next modifying or non-modifying access is made to the message. This includes, regardless of constness, calling any accessor of any field, calling any method inherited from `Message` or accessing the message through other ways (for example, by copying the message using the copy constructor). Correspondingly, the value of the returned pointer is never guaranteed to be the same across two different invocations of the accessor.
对于返回指针的字段访问器，当对消息进行下一次修改或非修改访问时，该指针可能会失效。这包括，无论是否为 const，调用任何字段的任何访问器，调用从 `Message` 继承的任何方法或通过其他方式访问消息（例如，使用复制构造函数复制消息）。相应地，返回指针的值绝不会保证在访问器的两次不同调用中相同。

### Optional Numeric Fields (proto2 and proto3) 可选数字字段（proto2 和 proto3）

For either of these field definitions:
对于以下任一字段定义：

```proto
optional int32 foo = 1;
required int32 foo = 1;
```

The compiler will generate the following accessor methods:
编译器将生成以下访问器方法：

- `bool has_foo() const`: Returns `true` if the field is set.
  `bool has_foo() const` ：如果设置了字段，则返回 `true` 。
- `int32 foo() const`: Returns the current value of the field. If the field is not set, returns the default value.
  `int32 foo() const` ：返回字段的当前值。如果未设置该字段，则返回默认值。
- `void set_foo(int32 value)`: Sets the value of the field. After calling this, `has_foo()` will return `true` and `foo()` will return `value`.
  `void set_foo(int32 value)` ：设置字段的值。调用此方法后， `has_foo()` 将返回 `true` ， `foo()` 将返回 `value` 。
- `void clear_foo()`: Clears the value of the field. After calling this, `has_foo()` will return `false` and `foo()` will return the default value.
  `void clear_foo()` ：清除字段的值。调用此方法后， `has_foo()` 将返回 `false` ， `foo()` 将返回默认值。

For other numeric field types (including `bool`), `int32` is replaced with the corresponding C++ type according to the [scalar value types table](https://protobuf.dev/programming-guides/proto3#scalar).
对于其他数字字段类型（包括 `bool` ）， `int32` 会根据标量值类型表替换为相应的 C++ 类型。

### Implicit Presence Numeric Fields (proto3) 隐式存在数字字段（proto3）

For these field definitions:
对于以下字段定义：

```proto
optional int32 foo = 1;
int32 foo = 1;  // no field label specified, defaults to implicit presence.
```

The compiler will generate the following accessor methods:
编译器将生成以下访问器方法：

- `int32 foo() const`: Returns the current value of the field. If the field is not set, returns 0.
  `int32 foo() const` ：返回字段的当前值。如果未设置字段，则返回 0。
- `void set_foo(int32 value)`: Sets the value of the field. After calling this, `foo()` will return `value`.
  `void set_foo(int32 value)` ：设置字段的值。调用此方法后， `foo()` 将返回 `value` 。
- `void clear_foo()`: Clears the value of the field. After calling this, `foo()` will return 0.
  `void clear_foo()` ：清除字段的值。调用此方法后， `foo()` 将返回 0。

For other numeric field types (including `bool`), `int32` is replaced with the corresponding C++ type according to the [scalar value types table](https://protobuf.dev/programming-guides/proto3#scalar).
对于其他数字字段类型（包括 `bool` ）， `int32` 会根据标量值类型表替换为相应的 C++ 类型。

### Optional String/Bytes Fields (proto2 and proto3) 可选字符串/字节字段（proto2 和 proto3）

For any of these field definitions:
对于以下任何字段定义：

```proto
optional string foo = 1;
required string foo = 1;
optional bytes foo = 1;
required bytes foo = 1;
```

The compiler will generate the following accessor methods:
编译器将生成以下访问器方法：

- `bool has_foo() const`: Returns `true` if the field is set.
  `bool has_foo() const` ：如果设置了字段，则返回 `true` 。
- `const string& foo() const`: Returns the current value of the field. If the field is not set, returns the default value.
  `const string& foo() const` ：返回字段的当前值。如果未设置该字段，则返回默认值。
- `void set_foo(const string& value)`: Sets the value of the field. After calling this, `has_foo()` will return `true` and `foo()` will return a copy of `value`.
  `void set_foo(const string& value)` ：设置字段的值。调用此方法后， `has_foo()` 将返回 `true` ， `foo()` 将返回 `value` 的副本。
- `void set_foo(string&& value)` (C++11 and beyond): Sets the value of the field, moving from the passed string. After calling this, `has_foo()` will return `true` and `foo()` will return a copy of `value`.
  `void set_foo(string&& value)` （C++11 及更高版本）：设置字段的值，从传递的字符串移动。调用此方法后， `has_foo()` 将返回 `true` ， `foo()` 将返回 `value` 的副本。
- `void set_foo(const char* value)`: Sets the value of the field using a C-style null-terminated string. After calling this, `has_foo()` will return `true` and `foo()` will return a copy of `value`.
  `void set_foo(const char* value)` ：使用以 C 风格结尾的空字符串设置字段的值。调用此方法后， `has_foo()` 将返回 `true` ， `foo()` 将返回 `value` 的副本。
- `void set_foo(const char* value, int size)`: Like above, but the string size is given explicitly rather than determined by looking for a null-terminator byte.
  `void set_foo(const char* value, int size)` ：与上面类似，但字符串大小是明确给出的，而不是通过查找空终止符字节来确定。
- `string* mutable_foo()`: Returns a pointer to the mutable `string` object that stores the field’s value. If the field was not set prior to the call, then the returned string will be empty (*not* the default value). After calling this, `has_foo()` will return `true` and `foo()` will return whatever value is written into the given string.
  `string* mutable_foo()` ：返回一个指向存储字段值的 `string` 可变对象的指针。如果在调用之前未设置字段，则返回的字符串将为空（不是默认值）。调用此方法后， `has_foo()` 将返回 `true` ， `foo()` 将返回写入给定字符串的任何值。
- `void clear_foo()`: Clears the value of the field. After calling this, `has_foo()` will return `false` and `foo()` will return the default value.
  `void clear_foo()` ：清除字段的值。调用此方法后， `has_foo()` 将返回 `false` ， `foo()` 将返回默认值。
- `void set_allocated_foo(string* value)`: Sets the `string` object to the field and frees the previous field value if it exists. If the `string` pointer is not `NULL`, the message takes ownership of the allocated `string` object and `has_foo()` will return `true`. The message is free to delete the allocated `string` object at any time, so references to the object may be invalidated. Otherwise, if the `value` is `NULL`, the behavior is the same as calling `clear_foo()`.
  `void set_allocated_foo(string* value)` ：将 `string` 对象设置为字段，并在存在时释放以前的字段值。如果 `string` 指针不是 `NULL` ，则消息将获取已分配的 `string` 对象的所有权， `has_foo()` 将返回 `true` 。消息可以随时删除已分配的 `string` 对象，因此对该对象的引用可能会失效。否则，如果 `value` 为 `NULL` ，则行为与调用 `clear_foo()` 相同。
- `string* release_foo()`: Releases the ownership of the field and returns the pointer of the `string` object. After calling this, caller takes the ownership of the allocated `string` object, `has_foo()` will return `false`, and `foo()` will return the default value.
  `string* release_foo()` ：释放字段的所有权并返回 `string` 对象的指针。调用此方法后，调用者获取已分配的 `string` 对象的所有权， `has_foo()` 将返回 `false` ， `foo()` 将返回默认值。



### Implicit Presence String/Bytes Fields (proto3) 隐式存在字符串/字节字段 (proto3)

For any of these field definitions:
对于以下任何字段定义：

```proto
optional string foo = 1;
string foo = 1;  // no field label specified, defaults to implicit presence.
optional bytes foo = 1;
bytes foo = 1;
```

The compiler will generate the following accessor methods:
编译器将生成以下访问器方法：

- `const string& foo() const`: Returns the current value of the field. If the field is not set, returns the empty string/empty bytes.
  `const string& foo() const` ：返回字段的当前值。如果未设置字段，则返回空字符串/空字节。
- `void set_foo(const string& value)`: Sets the value of the field. After calling this, `foo()` will return a copy of `value`.
  `void set_foo(const string& value)` ：设置字段的值。调用此方法后， `foo()` 将返回 `value` 的副本。
- `void set_foo(string&& value)` (C++11 and beyond): Sets the value of the field, moving from the passed string. After calling this, `foo()` will return a copy of `value`.
  `void set_foo(string&& value)` （C++11 及更高版本）：设置字段的值，从传递的字符串移动。调用此方法后， `foo()` 将返回 `value` 的副本。
- `void set_foo(const char* value)`: Sets the value of the field using a C-style null-terminated string. After calling this, `foo()` will return a copy of `value`.
  `void set_foo(const char* value)` ：使用以 C 样式结尾的空字符串设置字段的值。调用此方法后， `foo()` 将返回 `value` 的副本。
- `void set_foo(const char* value, int size)`: Like above, but the string size is given explicitly rather than determined by looking for a null-terminator byte.
  `void set_foo(const char* value, int size)` ：与上面类似，但字符串大小是明确给出的，而不是通过查找空终止符字节来确定。
- `string* mutable_foo()`: Returns a pointer to the mutable `string` object that stores the field’s value. If the field was not set prior to the call, then the returned string will be empty. After calling this, `foo()` will return whatever value is written into the given string.
  `string* mutable_foo()` ：返回指向存储字段值的 `string` 可变对象的指针。如果在调用之前未设置字段，则返回的字符串将为空。调用此方法后， `foo()` 将返回写入给定字符串的任何值。
- `void clear_foo()`: Clears the value of the field. After calling this, `foo()` will return the empty string/empty bytes.
  `void clear_foo()` ：清除字段的值。调用此方法后， `foo()` 将返回空字符串/空字节。
- `void set_allocated_foo(string* value)`: Sets the `string` object to the field and frees the previous field value if it exists. If the `string` pointer is not `NULL`, the message takes ownership of the allocated `string` object. The message is free to delete the allocated `string` object at any time, so references to the object may be invalidated. Otherwise, if the `value` is `NULL`, the behavior is the same as calling `clear_foo()`.
  `void set_allocated_foo(string* value)` ：将 `string` 对象设置为该字段，并在存在时释放先前的字段值。如果 `string` 指针不是 `NULL` ，则消息将获取已分配的 `string` 对象的拥有权。消息可以随时删除已分配的 `string` 对象，因此对该对象的引用可能会失效。否则，如果 `value` 是 `NULL` ，则行为与调用 `clear_foo()` 相同。
- `string* release_foo()`: Releases the ownership of the field and returns the pointer of the `string` object. After calling this, caller takes the ownership of the allocated `string` object and `foo()` will return the empty string/empty bytes.
  `string* release_foo()` ：释放字段的所有权并返回 `string` 对象的指针。调用此方法后，调用者将获取已分配的 `string` 对象的所有权， `foo()` 将返回空字符串/空字节。

### Singular Bytes Fields with Cord Support 具有 Cord 支持的单数字节字段

v23.0 added support for [`absl::Cord`](https://github.com/abseil/abseil-cpp/blob/master/absl/strings/cord.h) for singular `bytes` fields (including [`oneof` fields](https://protobuf.dev/reference/cpp/cpp-generated/#oneof-numeric)). Singular `string`, `repeated string`, and `repeated bytes` fields do not support using `Cord`s.
v23.0 为单数 `bytes` 字段（包括 `oneof` 字段）添加了对 `absl::Cord` 的支持。单数 `string` 、 `repeated string` 和 `repeated bytes` 字段不支持使用 `Cord` 。

To set a singular `bytes` field to store data using `absl::Cord`, use the following syntax:
要使用 `absl::Cord` 设置存储数据的单一 `bytes` 字段，请使用以下语法：

```proto
optional bytes foo = 25 [ctype=CORD];
bytes bar = 26 [ctype=CORD];
```

Using `cord` is not available for `repeated bytes` fields. Protoc ignores `[ctype=CORD]` settings on those fields.
对于 `repeated bytes` 字段，无法使用 `cord` 。Protoc 会忽略这些字段上的 `[ctype=CORD]` 设置。

The compiler will generate the following accessor methods:
编译器将生成以下访问器方法：

- `const ::absl::Cord& foo() const`: Returns the current value of the field. If the field is not set, returns an empty `Cord` (proto3) or the default value (proto2).
  `const ::absl::Cord& foo() const` ：返回字段的当前值。如果未设置该字段，则返回一个空 `Cord` （proto3）或默认值（proto2）。
- `void set_foo(const ::absl::Cord& value)`: Sets the value of the field. After calling this, `foo()` will return `value`.
  `void set_foo(const ::absl::Cord& value)` ：设置字段的值。调用此方法后， `foo()` 将返回 `value` 。
- `void set_foo(::absl::string_view value)`: Sets the value of the field. After calling this, `foo()` will return `value` as an `absl::Cord`.
  `void set_foo(::absl::string_view value)` ：设置字段的值。调用此方法后， `foo()` 将返回 `value` 作为 `absl::Cord` 。
- `void clear_foo()`: Clears the value of the field. After calling this, `foo()` will return an empty `Cord` (proto3) or the default value (proto2).
  `void clear_foo()` ：清除字段的值。调用此方法后， `foo()` 将返回一个空 `Cord` （proto3）或默认值（proto2）。
- `bool has_foo()`: Returns `true` if the field is set.
  `bool has_foo()` ：如果设置了字段，则返回 `true` 。

### Optional Enum Fields (proto2 and proto3) 可选枚举字段（proto2 和 proto3）

Given the enum type:
给定枚举类型：

```proto
enum Bar {
  BAR_UNSPECIFIED = 0;
  BAR_VALUE = 1;
  BAR_OTHER_VALUE = 2;
}
```

For either of these field definitions:
对于以下任一字段定义：

```proto
optional Bar foo = 1;
required Bar foo = 1;
```

The compiler will generate the following accessor methods:
编译器将生成以下访问器方法：

- `bool has_foo() const`: Returns `true` if the field is set.
  `bool has_foo() const` ：如果设置了字段，则返回 `true` 。
- `Bar foo() const`: Returns the current value of the field. If the field is not set, returns the default value.
  `Bar foo() const` ：返回字段的当前值。如果未设置该字段，则返回默认值。
- `void set_foo(Bar value)`: Sets the value of the field. After calling this, `has_foo()` will return `true` and `foo()` will return `value`. In debug mode (i.e. NDEBUG is not defined), if `value` does not match any of the values defined for `Bar`, this method will abort the process.
  `void set_foo(Bar value)` ：设置字段的值。调用此方法后， `has_foo()` 将返回 `true` ， `foo()` 将返回 `value` 。在调试模式下（即未定义 NDEBUG），如果 `value` 与为 `Bar` 定义的任何值不匹配，此方法将中止进程。
- `void clear_foo()`: Clears the value of the field. After calling this, `has_foo()` will return `false` and `foo()` will return the default value.
  `void clear_foo()` ：清除字段的值。调用此方法后， `has_foo()` 将返回 `false` ， `foo()` 将返回默认值。

### Implicit Presence Enum Fields (proto3) 隐式存在枚举字段（proto3）

Given the enum type:
给定枚举类型：

```proto
enum Bar {
  BAR_UNSPECIFIED = 0;
  BAR_VALUE = 1;
  BAR_OTHER_VALUE = 2;
}
```

For these field definitions:
对于这些字段定义：

```proto
optional Bar foo = 1;
Bar foo = 1;  // no field label specified, defaults to implicit presence.
```

The compiler will generate the following accessor methods:
编译器将生成以下访问器方法：

- `Bar foo() const`: Returns the current value of the field. If the field is not set, returns the default value (0).
  `Bar foo() const` ：返回字段的当前值。如果未设置字段，则返回默认值 (0)。
- `void set_foo(Bar value)`: Sets the value of the field. After calling this, `foo()` will return `value`.
  `void set_foo(Bar value)` ：设置字段的值。调用此方法后， `foo()` 将返回 `value` 。
- `void clear_foo()`: Clears the value of the field. After calling this, `foo()` will return the default value.
  `void clear_foo()` ：清除字段的值。调用此方法后， `foo()` 将返回默认值。

### Optional Embedded Message Fields (proto2 and proto3) 可选嵌入式消息字段（proto2 和 proto3）

Given the message type:
给定消息类型：

```proto
message Bar {}
```

For any of these field definitions:
对于以下任何字段定义：

```proto
//proto2
optional Bar foo = 1;
required Bar foo = 1;

//proto3
Bar foo = 1;
```

The compiler will generate the following accessor methods:
编译器将生成以下访问器方法：

- `bool has_foo() const`: Returns `true` if the field is set.
  `bool has_foo() const` ：如果设置了字段，则返回 `true` 。
- `const Bar& foo() const`: Returns the current value of the field. If the field is not set, returns a `Bar` with none of its fields set (possibly `Bar::default_instance()`).
  `const Bar& foo() const` ：返回字段的当前值。如果未设置字段，则返回一个未设置任何字段（可能是 `Bar::default_instance()` ）的 `Bar` 。
- `Bar* mutable_foo()`: Returns a pointer to the mutable `Bar` object that stores the field’s value. If the field was not set prior to the call, then the returned `Bar` will have none of its fields set (i.e. it will be identical to a newly-allocated `Bar`). After calling this, `has_foo()` will return `true` and `foo()` will return a reference to the same instance of `Bar`.
  `Bar* mutable_foo()` ：返回一个指向存储字段值的 `Bar` 可变对象的指针。如果在调用之前未设置字段，则返回的 `Bar` 将不会设置任何字段（即它将与新分配的 `Bar` 相同）。调用此方法后， `has_foo()` 将返回 `true` ， `foo()` 将返回对 `Bar` 的同一实例的引用。
- `void clear_foo()`: Clears the value of the field. After calling this, `has_foo()` will return `false` and `foo()` will return the default value.
  `void clear_foo()` ：清除字段的值。调用此方法后， `has_foo()` 将返回 `false` ， `foo()` 将返回默认值。
- `void set_allocated_foo(Bar* bar)`: Sets the `Bar` object to the field and frees the previous field value if it exists. If the `Bar` pointer is not `NULL`, the message takes ownership of the allocated `Bar` object and `has_foo()` will return `true`. Otherwise, if the `Bar` is `NULL`, the behavior is the same as calling `clear_foo()`.
  `void set_allocated_foo(Bar* bar)` ：将 `Bar` 对象设置为字段，并在存在时释放以前的字段值。如果 `Bar` 指针不是 `NULL` ，则消息将获取已分配的 `Bar` 对象的所有权， `has_foo()` 将返回 `true` 。否则，如果 `Bar` 为 `NULL` ，则行为与调用 `clear_foo()` 相同。
- `Bar* release_foo()`: Releases the ownership of the field and returns the pointer of the `Bar` object. After calling this, caller takes the ownership of the allocated `Bar` object, `has_foo()` will return `false`, and `foo()` will return the default value.
  `Bar* release_foo()` ：释放字段的所有权并返回 `Bar` 对象的指针。调用此方法后，调用者获取已分配的 `Bar` 对象的所有权， `has_foo()` 将返回 `false` ， `foo()` 将返回默认值。

### Repeated Numeric Fields 重复数字字段

For this field definition:
对于此字段定义：

```proto
repeated int32 foo = 1;
```

The compiler will generate the following accessor methods:
编译器将生成以下访问器方法：

- `int foo_size() const`: Returns the number of elements currently in the field. To check for an empty set, consider using the [`empty()`](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.repeated_field#RepeatedPtrField) method in the underlying `RepeatedField` instead of this method.
  `int foo_size() const` ：返回字段中当前的元素数。要检查空集，请考虑使用底层 `RepeatedField` 中的 `empty()` 方法，而不是此方法。
- `int32 foo(int index) const`: Returns the element at the given zero-based index. Calling this method with index outside of [0, foo_size()) yields undefined behavior.
  `int32 foo(int index) const` ：返回给定从零开始的索引处的元素。使用 [0, foo_size()) 范围之外的索引调用此方法会产生未定义的行为。
- `void set_foo(int index, int32 value)`: Sets the value of the element at the given zero-based index.
  `void set_foo(int index, int32 value)` ：设置给定从零开始的索引处的元素的值。
- `void add_foo(int32 value)`: Appends a new element to the end of the field with the given value.
  `void add_foo(int32 value)` ：将具有给定值的新元素追加到字段的末尾。
- `void clear_foo()`: Removes all elements from the field. After calling this, `foo_size()` will return zero.
  `void clear_foo()` ：从字段中删除所有元素。调用此方法后， `foo_size()` 将返回零。
- `const RepeatedField<int32>& foo() const`: Returns the underlying [`RepeatedField`](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.repeated_field#RepeatedField) that stores the field’s elements. This container class provides STL-like iterators and other methods.
  `const RepeatedField<int32>& foo() const` ：返回存储字段元素的底层 `RepeatedField` 。此容器类提供类似 STL 的迭代器和其他方法。
- `RepeatedField<int32>* mutable_foo()`: Returns a pointer to the underlying mutable `RepeatedField` that stores the field’s elements. This container class provides STL-like iterators and other methods.
  `RepeatedField<int32>* mutable_foo()` ：返回指向存储字段元素的底层可变 `RepeatedField` 的指针。此容器类提供类似 STL 的迭代器和其他方法。

For other numeric field types (including `bool`), `int32` is replaced with the corresponding C++ type according to the [scalar value types table](https://protobuf.dev/programming-guides/proto2#scalar).
对于其他数字字段类型（包括 `bool` ）， `int32` 会根据标量值类型表替换为相应的 C++ 类型。

### Repeated String Fields 重复字符串字段

For either of these field definitions:
对于这些字段定义中的任何一个：

```proto
repeated string foo = 1;
repeated bytes foo = 1;
```

The compiler will generate the following accessor methods:
编译器将生成以下访问器方法：

- `int foo_size() const`: Returns the number of elements currently in the field. To check for an empty set, consider using the [`empty()`](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.repeated_field#RepeatedPtrField) method in the underlying `RepeatedField` instead of this method.
  `int foo_size() const` ：返回字段中当前的元素数。要检查空集，请考虑使用基础 `RepeatedField` 中的 `empty()` 方法，而不是此方法。
- `const string& foo(int index) const`: Returns the element at the given zero-based index. Calling this method with index outside of [0, foo_size()-1] yields undefined behavior.
  `const string& foo(int index) const` ：返回给定基于零的索引处的元素。使用 [0, foo_size()-1] 之外的索引调用此方法会产生未定义的行为。
- `void set_foo(int index, const string& value)`: Sets the value of the element at the given zero-based index.
  `void set_foo(int index, const string& value)` ：设置给定基于零的索引处的元素的值。
- `void set_foo(int index, const char* value)`: Sets the value of the element at the given zero-based index using a C-style null-terminated string.
  `void set_foo(int index, const char* value)` ：使用 C 风格的以空结尾的字符串设置给定基于零的索引处的元素的值。
- `void set_foo(int index, const char* value, int size)`: Like above, but the string size is given explicitly rather than determined by looking for a null-terminator byte.
  `void set_foo(int index, const char* value, int size)` ：与上面类似，但字符串大小是明确给出的，而不是通过查找空终止符字节来确定的。
- `string* mutable_foo(int index)`: Returns a pointer to the mutable `string` object that stores the value of the element at the given zero-based index. Calling this method with index outside of [0, foo_size()) yields undefined behavior.
  `string* mutable_foo(int index)` ：返回指向可变 `string` 对象的指针，该对象存储给定基于零的索引处的元素的值。使用 [0, foo_size()) 之外的索引调用此方法会产生未定义的行为。
- `void add_foo(const string& value)`: Appends a new element to the end of the field with the given value.
  `void add_foo(const string& value)` ：使用给定值将新元素追加到字段的末尾。
- `void add_foo(const char* value)`: Appends a new element to the end of the field using a C-style null-terminated string.
  `void add_foo(const char* value)` ：使用 C 风格的以空结尾的字符串将新元素追加到字段的末尾。
- `void add_foo(const char* value, int size)`: Like above, but the string size is given explicitly rather than determined by looking for a null-terminator byte.
  `void add_foo(const char* value, int size)` ：与上面类似，但字符串大小是明确给出的，而不是通过查找空终止符字节来确定的。
- `string* add_foo()`: Adds a new empty string element to the end of the field and returns a pointer to it.
  `string* add_foo()` ：在字段末尾添加一个新的空字符串元素，并返回指向它的指针。
- `void clear_foo()`: Removes all elements from the field. After calling this, `foo_size()` will return zero.
  `void clear_foo()` ：从字段中删除所有元素。调用此方法后， `foo_size()` 将返回零。
- `const RepeatedPtrField<string>& foo() const`: Returns the underlying [`RepeatedPtrField`](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.repeated_field#RepeatedPtrField) that stores the field’s elements. This container class provides STL-like iterators and other methods.
  `const RepeatedPtrField<string>& foo() const` ：返回存储字段元素的基础 `RepeatedPtrField` 。此容器类提供类似 STL 的迭代器和其他方法。
- `RepeatedPtrField<string>* mutable_foo()`: Returns a pointer to the underlying mutable `RepeatedPtrField` that stores the field’s elements. This container class provides STL-like iterators and other methods.
  `RepeatedPtrField<string>* mutable_foo()` ：返回指向存储字段元素的基础可变 `RepeatedPtrField` 的指针。此容器类提供类似 STL 的迭代器和其他方法。

### Repeated Enum Fields 重复枚举字段

Given the enum type:
给定枚举类型：

```proto
enum Bar {
  BAR_UNSPECIFIED = 0;
  BAR_VALUE = 1;
  BAR_OTHER_VALUE = 2;
}
```

For this field definition:
对于此字段定义：

```proto
repeated Bar foo = 1;
```

The compiler will generate the following accessor methods:
编译器将生成以下访问器方法：

- `int foo_size() const`: Returns the number of elements currently in the field. To check for an empty set, consider using the [`empty()`](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.repeated_field#RepeatedPtrField) method in the underlying `RepeatedField` instead of this method.
  `int foo_size() const` ：返回字段中当前的元素数。要检查空集，请考虑使用基础 `RepeatedField` 中的 `empty()` 方法，而不是此方法。
- `Bar foo(int index) const`: Returns the element at the given zero-based index. Calling this method with index outside of [0, foo_size()) yields undefined behavior.
  `Bar foo(int index) const` ：返回给定从零开始的索引处的元素。使用 [0, foo_size()) 范围之外的索引调用此方法会导致未定义的行为。
- `void set_foo(int index, Bar value)`: Sets the value of the element at the given zero-based index. In debug mode (i.e. NDEBUG is not defined), if `value` does not match any of the values defined for `Bar`, this method will abort the process.
  `void set_foo(int index, Bar value)` ：设置给定基于零的索引处的元素值。在调试模式下（即未定义 NDEBUG），如果 `value` 与为 `Bar` 定义的任何值不匹配，此方法将中止进程。
- `void add_foo(Bar value)`: Appends a new element to the end of the field with the given value. In debug mode (i.e. NDEBUG is not defined), if `value` does not match any of the values defined for `Bar`, this method will abort the process.
  `void add_foo(Bar value)` ：使用给定值将新元素追加到字段末尾。在调试模式下（即未定义 NDEBUG），如果 `value` 与为 `Bar` 定义的任何值不匹配，此方法将中止进程。
- `void clear_foo()`: Removes all elements from the field. After calling this, `foo_size()` will return zero.
  `void clear_foo()` ：从字段中删除所有元素。调用此方法后， `foo_size()` 将返回零。
- `const RepeatedField<int>& foo() const`: Returns the underlying [`RepeatedField`](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.repeated_field#RepeatedField) that stores the field’s elements. This container class provides STL-like iterators and other methods.
  `const RepeatedField<int>& foo() const` ：返回存储字段元素的基础 `RepeatedField` 。此容器类提供类似 STL 的迭代器和其他方法。
- `RepeatedField<int>* mutable_foo()`: Returns a pointer to the underlying mutable `RepeatedField` that stores the field’s elements. This container class provides STL-like iterators and other methods.
  `RepeatedField<int>* mutable_foo()` ：返回指向存储字段元素的基础可变 `RepeatedField` 的指针。此容器类提供类似 STL 的迭代器和其他方法。

### Repeated Embedded Message Fields 重复嵌入的消息字段

Given the message type:
给定消息类型：

```proto
message Bar {}
```

For this field definitions:
对于此字段定义：

```proto
repeated Bar foo = 1;
```

The compiler will generate the following accessor methods:
编译器将生成以下访问器方法：

- `int foo_size() const`: Returns the number of elements currently in the field. To check for an empty set, consider using the [`empty()`](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.repeated_field#RepeatedPtrField) method in the underlying `RepeatedField` instead of this method.
  `int foo_size() const` ：返回字段中当前的元素数。要检查空集，请考虑使用基础 `RepeatedField` 中的 `empty()` 方法，而不是此方法。
- `const Bar& foo(int index) const`: Returns the element at the given zero-based index. Calling this method with index outside of [0, foo_size()) yields undefined behavior.
  `const Bar& foo(int index) const` ：返回给定基于零的索引处的元素。使用 [0, foo_size()) 范围之外的索引调用此方法会产生未定义的行为。
- `Bar* mutable_foo(int index)`: Returns a pointer to the mutable `Bar` object that stores the value of the element at the given zero-based index. Calling this method with index outside of [0, foo_size()) yields undefined behavior.
  `Bar* mutable_foo(int index)` ：返回一个指向可变 `Bar` 对象的指针，该对象存储给定基于零的索引处的元素的值。使用 [0, foo_size()) 范围之外的索引调用此方法会产生未定义的行为。
- `Bar* add_foo()`: Adds a new element to the end of the field and returns a pointer to it. The returned `Bar` is mutable and will have none of its fields set (i.e. it will be identical to a newly-allocated `Bar`).
  `Bar* add_foo()` ：向字段末尾添加一个新元素并返回指向它的指针。返回的 `Bar` 是可变的，并且不会设置任何字段（即它将与新分配的 `Bar` 相同）。
- `void clear_foo()`: Removes all elements from the field. After calling this, `foo_size()` will return zero.
  `void clear_foo()` ：从字段中删除所有元素。调用此方法后， `foo_size()` 将返回零。
- `const RepeatedPtrField<Bar>& foo() const`: Returns the underlying [`RepeatedPtrField`](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.repeated_field#RepeatedPtrField) that stores the field’s elements. This container class provides STL-like iterators and other methods.
  `const RepeatedPtrField<Bar>& foo() const` ：返回存储字段元素的基础 `RepeatedPtrField` 。此容器类提供类似 STL 的迭代器和其他方法。
- `RepeatedPtrField<Bar>* mutable_foo()`: Returns a pointer to the underlying mutable `RepeatedPtrField` that stores the field’s elements. This container class provides STL-like iterators and other methods.
  `RepeatedPtrField<Bar>* mutable_foo()` ：返回一个指向存储字段元素的基础可变 `RepeatedPtrField` 的指针。此容器类提供类似 STL 的迭代器和其他方法。

### Oneof Numeric Fields 其中一个数字字段

For this [oneof](https://protobuf.dev/reference/cpp/cpp-generated/#oneof) field definition:
对于此 oneof 字段定义：

```proto
oneof example_name {
    int32 foo = 1;
    ...
}
```

The compiler will generate the following accessor methods:
编译器将生成以下访问器方法：

- `bool has_foo() const` (proto2 only): Returns `true` if oneof case is `kFoo`.
  `bool has_foo() const` （仅限 proto2）：如果 oneof 案例是 `kFoo` ，则返回 `true` 。

- `int32 foo() const`: Returns the current value of the field if oneof case is `kFoo`. Otherwise, returns the default value.
  `int32 foo() const` ：如果 oneof 案例是 `kFoo` ，则返回字段的当前值。否则，返回默认值。

- ```
  void set_foo(int32 value)
  ```

  :

  - If any other oneof field in the same oneof is set, calls `clear_example_name()`.
    如果设置了同一 oneof 中的任何其他 oneof 字段，则调用 `clear_example_name()` 。
  - Sets the value of this field and sets the oneof case to `kFoo`.
    设置此字段的值并将 oneof 案例设置为 `kFoo` 。
  - `has_foo()` (proto2 only) will return true, `foo()` will return `value`, and `example_name_case()` will return `kFoo`.
    `has_foo()` （仅限 proto2）将返回 true， `foo()` 将返回 `value` ， `example_name_case()` 将返回 `kFoo` 。

- ```
  void clear_foo()
  ```

  :

  - Nothing will be changed if oneof case is not `kFoo`.
    如果其中一个 case 不是 `kFoo` ，则不会更改任何内容。
  - If oneof case is `kFoo`, clears the value of the field and oneof case. `has_foo()` (proto2 only) will return `false`, `foo()` will return the default value and `example_name_case()` will return `EXAMPLE_NAME_NOT_SET`.
    如果其中一个 case 是 `kFoo` ，则清除字段的值和其中一个 case。 `has_foo()` （仅限 proto2）将返回 `false` ， `foo()` 将返回默认值， `example_name_case()` 将返回 `EXAMPLE_NAME_NOT_SET` 。

For other numeric field types (including `bool`),`int32` is replaced with the corresponding C++ type according to the [scalar value types table](https://protobuf.dev/programming-guides/proto3#scalar).
对于其他数字字段类型（包括 `bool` ）， `int32` 会根据标量值类型表替换为相应的 C++ 类型。

### Oneof String Fields 字符串字段之一

For any of these [oneof](https://protobuf.dev/reference/cpp/cpp-generated/#oneof) field definitions:
对于这些 oneof 字段定义中的任何一个：

```proto
oneof example_name {
    string foo = 1;
    ...
}
oneof example_name {
    bytes foo = 1;
    ...
}
```

The compiler will generate the following accessor methods:
编译器将生成以下访问器方法：

- `bool has_foo() const`: Returns `true` if the oneof case is `kFoo`.
  `bool has_foo() const` ：如果 oneof 案例是 `kFoo` ，则返回 `true` 。

- `const string& foo() const`: Returns the current value of the field if the oneof case is `kFoo`. Otherwise, returns the default value.
  `const string& foo() const` ：如果 oneof 案例为 `kFoo` ，则返回字段的当前值。否则，返回默认值。

- ```
  void set_foo(const string& value)
  ```

  :

  - If any other oneof field in the same oneof is set, calls `clear_example_name()`.
    如果设置了同一 oneof 中的任何其他 oneof 字段，则调用 `clear_example_name()` 。
  - Sets the value of this field and sets the oneof case to `kFoo`.
    设置此字段的值并将 oneof 案例设置为 `kFoo` 。
  - `has_foo()` will return `true`, `foo()` will return a copy of `value` and `example_name_case()` will return `kFoo`.
    `has_foo()` 将返回 `true` ， `foo()` 将返回 `value` 的副本， `example_name_case()` 将返回 `kFoo` 。

- ```
  void set_foo(const char* value)
  ```

  :

  - If any other oneof field in the same oneof is set, calls `clear_example_name()`.
    如果设置了同一 oneof 中的任何其他 oneof 字段，则调用 `clear_example_name()` 。
  - Sets the value of the field using a C-style null-terminated string and set the oneof case to `kFoo`.
    使用 C 风格的以空字符结尾的字符串设置字段的值，并将 oneof 案例设置为 `kFoo` 。
  - `has_foo()` will return `true`, `foo()` will return a copy of `value` and `example_name_case()` will return `kFoo`.
    `has_foo()` 将返回 `true` ， `foo()` 将返回 `value` 的副本， `example_name_case()` 将返回 `kFoo` 。

- `void set_foo(const char* value, int size)`: Like above, but the string size is given explicitly rather than determined by looking for a null-terminator byte.
  `void set_foo(const char* value, int size)` ：与上面类似，但字符串大小是明确给出的，而不是通过查找空终止符字节来确定。

- ```
  string* mutable_foo()
  ```

  :

  - If any other oneof field in the same oneof is set, calls `clear_example_name()`.
    如果设置了同一 oneof 中的任何其他 oneof 字段，则调用 `clear_example_name()` 。
  - Sets the oneof case to `kFoo` and returns a pointer to the mutable string object that stores the field’s value. If the oneof case was not `kFoo` prior to the call, then the returned string will be empty (not the default value).
    将 oneof 案例设置为 `kFoo` ，并返回指向存储字段值的字符串对象的可变指针。如果在调用之前 oneof 案例不是 `kFoo` ，则返回的字符串将为空（不是默认值）。
  - `has_foo()` will return `true`, `foo()` will return whatever value is written into the given string and `example_name_case()` will return `kFoo`.
    `has_foo()` 将返回 `true` ， `foo()` 将返回写入给定字符串的任何值， `example_name_case()` 将返回 `kFoo` 。

- ```
  void clear_foo()
  ```

  :

  - If the oneof case is not `kFoo`, nothing will be changed .
    如果 oneof 案例不是 `kFoo` ，则不会更改任何内容。
  - If the oneof case is `kFoo`, frees the field and clears the oneof case . `has_foo()` will return `false`, `foo()` will return the default value, and `example_name_case()` will return `EXAMPLE_NAME_NOT_SET`.
    如果 oneof 案例是 `kFoo` ，则释放字段并清除 oneof 案例。 `has_foo()` 将返回 `false` ， `foo()` 将返回默认值， `example_name_case()` 将返回 `EXAMPLE_NAME_NOT_SET` 。

- ```
  void set_allocated_foo(string* value)
  ```

  :

  - Calls `clear_example_name()`.
    调用 `clear_example_name()` 。
  - If the string pointer is not `NULL`: Sets the string object to the field and sets the oneof case to `kFoo`. The message takes ownership of the allocated string object, `has_foo()` will return `true` and `example_name_case()` will return `kFoo`.
    如果字符串指针不是 `NULL` ：将字符串对象设置为字段并将 oneof 案例设置为 `kFoo` 。消息获取已分配字符串对象的所有权， `has_foo()` 将返回 `true` ， `example_name_case()` 将返回 `kFoo` 。
  - If the string pointer is `NULL`, `has_foo()` will return `false` and `example_name_case()` will return `EXAMPLE_NAME_NOT_SET`.
    如果字符串指针是 `NULL` ， `has_foo()` 将返回 `false` ， `example_name_case()` 将返回 `EXAMPLE_NAME_NOT_SET` 。

- ```
  string* release_foo()
  ```

  :

  - Returns `NULL` if oneof case is not `kFoo`.
    如果 oneof 案例不是 `kFoo` ，则返回 `NULL` 。
  - Clears the oneof case, releases the ownership of the field and returns the pointer of the string object. After calling this, caller takes the ownership of the allocated string object, `has_foo()` will return false, `foo()` will return the default value, and `example_name_case()` will return `EXAMPLE_NAME_NOT_SET`.
    清除 oneof 案例，释放字段的所有权并返回字符串对象的指针。调用此方法后，调用者获取已分配字符串对象的所有权， `has_foo()` 将返回 false， `foo()` 将返回默认值， `example_name_case()` 将返回 `EXAMPLE_NAME_NOT_SET` 。

### Oneof Enum Fields Oneof 枚举字段

Given the enum type:
给定枚举类型：

```proto
enum Bar {
  BAR_UNSPECIFIED = 0;
  BAR_VALUE = 1;
  BAR_OTHER_VALUE = 2;
}
```

For the [oneof](https://protobuf.dev/reference/cpp/cpp-generated/#oneof) field definition:
对于 oneof 字段定义：

```proto
oneof example_name {
    Bar foo = 1;
    ...
}
```

The compiler will generate the following accessor methods:
编译器将生成以下访问器方法：

- `bool has_foo() const` (proto2 only): Returns `true` if oneof case is `kFoo`.
  `bool has_foo() const` （仅限 proto2）：如果 oneof 案例是 `kFoo` ，则返回 `true` 。

- `Bar foo() const`: Returns the current value of the field if oneof case is `kFoo`. Otherwise, returns the default value.
  `Bar foo() const` ：如果 oneof 案例是 `kFoo` ，则返回字段的当前值。否则，返回默认值。

- ```
  void set_foo(Bar value)
  ```

  :

  - If any other oneof field in the same oneof is set, calls `clear_example_name()`.
    如果设置了同一 oneof 中的任何其他 oneof 字段，则调用 `clear_example_name()` 。
  - Sets the value of this field and sets the oneof case to `kFoo`.
    设置此字段的值并将 oneof 案例设置为 `kFoo` 。
  - `has_foo()` (proto2 only) will return `true`, `foo()` will return `value` and `example_name_case()` will return `kFoo`.
    `has_foo()` （仅限 proto2）将返回 `true` ， `foo()` 将返回 `value` ， `example_name_case()` 将返回 `kFoo` 。
  - In debug mode (i.e. NDEBUG is not defined), if `value` does not match any of the values defined for `Bar`, this method will abort the process.
    在调试模式（即未定义 NDEBUG）下，如果 `value` 与为 `Bar` 定义的任何值都不匹配，此方法将中止进程。

- ```
  void clear_foo()
  ```

  :

  - Nothing will be changed if the oneof case is not `kFoo`.
    如果 oneof 案例不是 `kFoo` ，则不会更改任何内容。
  - If the oneof case is `kFoo`, clears the value of the field and the oneof case. `has_foo()` (proto2 only) will return `false`, `foo()` will return the default value and `example_name_case()` will return `EXAMPLE_NAME_NOT_SET`.
    如果 oneof 案例是 `kFoo` ，则清除字段值和 oneof 案例。 `has_foo()` （仅限 proto2）将返回 `false` ， `foo()` 将返回默认值， `example_name_case()` 将返回 `EXAMPLE_NAME_NOT_SET` 。

### Oneof Embedded Message Fields Oneof 嵌入式消息字段

Given the message type:
给定消息类型：

```proto
message Bar {}
```

For the [oneof](https://protobuf.dev/reference/cpp/cpp-generated/#oneof) field definition:
对于 oneof 字段定义：

```proto
oneof example_name {
    Bar foo = 1;
    ...
}
```

The compiler will generate the following accessor methods:
编译器将生成以下访问器方法：

- `bool has_foo() const`: Returns true if oneof case is `kFoo`.
  `bool has_foo() const` ：如果 oneof 案例是 `kFoo` ，则返回 true。

- `const Bar& foo() const`: Returns the current value of the field if oneof case is `kFoo`. Otherwise, returns a Bar with none of its fields set (possibly `Bar::default_instance()`).
  `const Bar& foo() const` ：如果 oneof 案例是 `kFoo` ，则返回字段的当前值。否则，返回一个未设置任何字段的 Bar（可能是 `Bar::default_instance()` ）。

- ```
  Bar* mutable_foo()
  ```

  :

  - If any other oneof field in the same oneof is set, calls `clear_example_name()`.
    如果设置了同一 oneof 中的任何其他 oneof 字段，则调用 `clear_example_name()` 。
  - Sets the oneof case to `kFoo` and returns a pointer to the mutable Bar object that stores the field’s value. If the oneof case was not `kFoo` prior to the call, then the returned Bar will have none of its fields set (i.e. it will be identical to a newly-allocated Bar).
    将 oneof 案例设置为 `kFoo` ，并返回指向存储字段值的 Bar 可变对象的指针。如果在调用之前 oneof 案例不是 `kFoo` ，则返回的 Bar 将未设置任何字段（即它将与新分配的 Bar 相同）。
  - After calling this, `has_foo()` will return `true`, `foo()` will return a reference to the same instance of `Bar` and `example_name_case()` will return `kFoo`.
    调用此方法后， `has_foo()` 将返回 `true` ， `foo()` 将返回对 `Bar` 的同一实例的引用， `example_name_case()` 将返回 `kFoo` 。

- ```
  void clear_foo()
  ```

  :

  - Nothing will be changed if the oneof case is not `kFoo`.
    如果 oneof 案例不是 `kFoo` ，则不会更改任何内容。
  - If the oneof case equals `kFoo`, frees the field and clears the oneof case. `has_foo()` will return `false`, `foo()` will return the default value and `example_name_case()` will return `EXAMPLE_NAME_NOT_SET`.
    如果 oneof 案例等于 `kFoo` ，则释放该字段并清除 oneof 案例。 `has_foo()` 将返回 `false` ， `foo()` 将返回默认值， `example_name_case()` 将返回 `EXAMPLE_NAME_NOT_SET` 。

- ```
  void set_allocated_foo(Bar* bar)
  ```

  :

  - Calls `clear_example_name()`.
    调用 `clear_example_name()` 。
  - If the `Bar` pointer is not `NULL`: Sets the `Bar` object to the field and sets the oneof case to `kFoo`. The message takes ownership of the allocated `Bar` object, has_foo() will return true and `example_name_case()` will return `kFoo`.
    如果 `Bar` 指针不是 `NULL` ：将 `Bar` 对象设置为该字段，并将 oneof 案例设置为 `kFoo` 。该消息获取已分配的 `Bar` 对象的所有权，has_foo() 将返回 true， `example_name_case()` 将返回 `kFoo` 。
  - If the pointer is `NULL`, `has_foo()` will return `false` and `example_name_case()` will return `EXAMPLE_NAME_NOT_SET`. (The behavior is like calling `clear_example_name()`)
    如果指针是 `NULL` ， `has_foo()` 将返回 `false` ， `example_name_case()` 将返回 `EXAMPLE_NAME_NOT_SET` 。（行为类似于调用 `clear_example_name()` ）

- ```
  Bar* release_foo()
  ```

  :

  - Returns `NULL` if oneof case is not `kFoo`.
    如果 oneof 案例不是 `kFoo` ，则返回 `NULL` 。
  - If the oneof case is `kFoo`, clears the oneof case, releases the ownership of the field and returns the pointer of the `Bar` object. After calling this, caller takes the ownership of the allocated `Bar` object, `has_foo()` will return `false`, `foo()` will return the default value and `example_name_case()` will return `EXAMPLE_NAME_NOT_SET`.
    如果 oneof 案例是 `kFoo` ，则清除 oneof 案例，释放字段的所有权并返回 `Bar` 对象的指针。调用此方法后，调用方获取已分配 `Bar` 对象的所有权， `has_foo()` 将返回 `false` ， `foo()` 将返回默认值， `example_name_case()` 将返回 `EXAMPLE_NAME_NOT_SET` 。

### Map Fields Map 字段

For this map field definition:
对于此映射字段定义：

```proto
map<int32, int32> weight = 1;
```

The compiler will generate the following accessor methods:
编译器将生成以下访问器方法：

- `const google::protobuf::Map<int32, int32>& weight();`: Returns an immutable `Map`.
  `const google::protobuf::Map<int32, int32>& weight();` ：返回一个不可变的 `Map` 。
- `google::protobuf::Map<int32, int32>* mutable_weight();`: Returns a mutable `Map`.
  `google::protobuf::Map<int32, int32>* mutable_weight();` ：返回一个可变的 `Map` 。

A [`google::protobuf::Map`](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.map) is a special container type used in protocol buffers to store map fields. As you can see from its interface below, it uses a commonly-used subset of `std::map` and `std::unordered_map` methods.
在协议缓冲区中， `google::protobuf::Map` 是一种特殊容器类型，用于存储映射字段。从其下面的接口可以看到，它使用 `std::map` 和 `std::unordered_map` 方法的常用子集。

**NOTE:** These maps are unordered.
注意：这些映射是无序的。

```cpp
template<typename Key, typename T> {
class Map {
  // Member types
  typedef Key key_type;
  typedef T mapped_type;
  typedef MapPair< Key, T > value_type;

  // Iterators
  iterator begin();
  const_iterator begin() const;
  const_iterator cbegin() const;
  iterator end();
  const_iterator end() const;
  const_iterator cend() const;
  // Capacity
  int size() const;
  bool empty() const;

  // Element access
  T& operator[](const Key& key);
  const T& at(const Key& key) const;
  T& at(const Key& key);

  // Lookup
  bool contains(const Key& key) const;
  int count(const Key& key) const;
  const_iterator find(const Key& key) const;
  iterator find(const Key& key);

  // Modifiers
  pair<iterator, bool> insert(const value_type& value);
  template<class InputIt>
  void insert(InputIt first, InputIt last);
  size_type erase(const Key& Key);
  iterator erase(const_iterator pos);
  iterator erase(const_iterator first, const_iterator last);
  void clear();

  // Copy
  Map(const Map& other);
  Map& operator=(const Map& other);
}
```

The easiest way to add data is to use normal map syntax, for example:
添加数据最简单的方法是使用常规映射语法，例如：

```cpp
std::unique_ptr<ProtoName> my_enclosing_proto(new ProtoName);
(*my_enclosing_proto->mutable_weight())[my_key] = my_value;
```

`pair<iterator, bool> insert(const value_type& value)` will implicitly cause a deep copy of the `value_type` instance. The most efficient way to insert a new value into a `google::protobuf::Map` is as follows:
`pair<iterator, bool> insert(const value_type& value)` 将隐式导致 `value_type` 实例的深度复制。将新值插入 `google::protobuf::Map` 的最有效方法如下：

```cpp
T& operator[](const Key& key): map[new_key] = new_mapped;
```

#### Using `google::protobuf::Map` with standard maps 使用 `google::protobuf::Map` 与标准映射

`google::protobuf::Map` supports the same iterator API as `std::map` and `std::unordered_map`. If you don’t want to use `google::protobuf::Map` directly, you can convert a `google::protobuf::Map` to a standard map by doing the following:
`google::protobuf::Map` 支持与 `std::map` 和 `std::unordered_map` 相同的迭代器 API。如果您不想直接使用 `google::protobuf::Map` ，可以通过执行以下操作将 `google::protobuf::Map` 转换为标准映射：

```cpp
std::map<int32, int32> standard_map(message.weight().begin(),
                                    message.weight().end());
```

Note that this will make a deep copy of the entire map.
请注意，这将对整个映射进行深度复制。

You can also construct a `google::protobuf::Map` from a standard map as follows:
您还可以按照如下方式从标准映射构建 `google::protobuf::Map` ：

```cpp
google::protobuf::Map<int32, int32> weight(standard_map.begin(), standard_map.end());
```

#### Parsing unknown values 解析未知值

On the wire, a .proto map is equivalent to a map entry message for each key/value pair, while the map itself is a repeated field of map entries. Like ordinary message types, it’s possible for a parsed map entry message to have unknown fields: for example a field of type `int64` in a map defined as `map<int32, string>`.
在传输过程中，.proto 映射等效于每个键/值对的映射条目消息，而映射本身是映射条目的重复字段。与普通消息类型一样，已解析的映射条目消息可能具有未知字段：例如，在定义为 `map<int32, string>` 的映射中，类型为 `int64` 的字段。

If there are unknown fields in the wire format of a map entry message, they will be discarded.
如果映射条目消息的传输格式中存在未知字段，则会将其丢弃。

If there is an unknown enum value in the wire format of a map entry message, it’s handled differently in proto2 and proto3. In proto2, the whole map entry message is put into the unknown field set of the containing message. In proto3, it is put into a map field as if it is a known enum value.
如果映射条目消息的传输格式中存在未知枚举值，则在 proto2 和 proto3 中的处理方式不同。在 proto2 中，整个映射条目消息将放入包含消息的未知字段集中。在 proto3 中，它将放入映射字段中，就像它是已知枚举值一样。

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

In our generated code, the getter for the `details` field returns an instance of `google::protobuf::Any`. This provides the following special methods to pack and unpack the `Any`’s values:
在我们的生成代码中， `details` 字段的 getter 返回 `google::protobuf::Any` 的一个实例。这提供了以下特殊方法来打包和解包 `Any` 的值：

```cpp
class Any {
 public:
  // Packs the given message into this Any using the default type URL
  // prefix “type.googleapis.com”. Returns false if serializing the message failed.
  bool PackFrom(const google::protobuf::Message& message);

  // Packs the given message into this Any using the given type URL
  // prefix. Returns false if serializing the message failed.
  bool PackFrom(const google::protobuf::Message& message,
                const string& type_url_prefix);

  // Unpacks this Any to a Message. Returns false if this Any
  // represents a different protobuf type or parsing fails.
  bool UnpackTo(google::protobuf::Message* message) const;

  // Returns true if this Any represents the given protobuf type.
  template<typename T> bool Is() const;
}
```

## Oneof

Given a oneof definition like this:
假设存在如下 oneof 定义：

```proto
oneof example_name {
    int32 foo_int = 4;
    string foo_string = 9;
    ...
}
```

The compiler will generate the following C++ enum type:
编译器将生成以下 C++ 枚举类型：

```cpp
enum ExampleNameCase {
  kFooInt = 4,
  kFooString = 9,
  EXAMPLE_NAME_NOT_SET = 0
}
```

In addition, it will generate these methods:
此外，它还将生成这些方法：

- `ExampleNameCase example_name_case() const`: Returns the enum indicating which field is set. Returns `EXAMPLE_NAME_NOT_SET` if none of them is set.
  `ExampleNameCase example_name_case() const` ：返回指示设置了哪个字段的枚举。如果未设置任何字段，则返回 `EXAMPLE_NAME_NOT_SET` 。
- `void clear_example_name()`: Frees the object if the oneof field set uses a pointer (Message or String), and sets the oneof case to `EXAMPLE_NAME_NOT_SET`.
  `void clear_example_name()` ：如果 oneof 字段集使用指针（消息或字符串），则释放对象，并将 oneof 案例设置为 `EXAMPLE_NAME_NOT_SET` 。

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

The protocol buffer compiler will generate a C++ enum type called `Foo` with the same set of values. In addition, the compiler will generate the following functions:
协议缓冲区编译器将生成一个名为 `Foo` 的 C++ 枚举类型，其中包含相同的值集。此外，编译器还将生成以下函数：

- `const EnumDescriptor* Foo_descriptor()`: Returns the type’s descriptor, which contains information about what values this enum type defines.
  `const EnumDescriptor* Foo_descriptor()` ：返回类型的描述符，其中包含有关此枚举类型定义的值的信息。
- `bool Foo_IsValid(int value)`: Returns `true` if the given numeric value matches one of `Foo`’s defined values. In the above example, it would return `true` if the input were 0, 5, or 1234.
  `bool Foo_IsValid(int value)` ：如果给定的数值与 `Foo` 的定义值之一匹配，则返回 `true` 。在上面的示例中，如果输入为 0、5 或 1234，它将返回 `true` 。
- `const string& Foo_Name(int value)`: Returns the name for given numeric value. Returns an empty string if no such value exists. If multiple values have this number, the first one defined is returned. In the above example, `Foo_Name(5)` would return `"VALUE_B"`.
  `const string& Foo_Name(int value)` ：返回给定数值的名称。如果不存在此类值，则返回一个空字符串。如果多个值具有此数字，则返回定义的第一个值。在上面的示例中， `Foo_Name(5)` 将返回 `"VALUE_B"` 。
- `bool Foo_Parse(const string& name, Foo* value)`: If `name` is a valid value name for this enum, assigns that value into `value` and returns true. Otherwise returns false. In the above example, `Foo_Parse("VALUE_C", &some_foo)` would return true and set `some_foo` to 1234.
  `bool Foo_Parse(const string& name, Foo* value)` ：如果 `name` 是此枚举的有效值名称，则将该值分配给 `value` 并返回 true。否则返回 false。在上面的示例中， `Foo_Parse("VALUE_C", &some_foo)` 将返回 true 并将 `some_foo` 设置为 1234。
- `const Foo Foo_MIN`: the smallest valid value of the enum (VALUE_A in the example).
  `const Foo Foo_MIN` ：枚举的最小有效值（示例中的 VALUE_A）。
- `const Foo Foo_MAX`: the largest valid value of the enum (VALUE_C in the example).
  `const Foo Foo_MAX` ：枚举的最大有效值（示例中的 VALUE_C）。
- `const int Foo_ARRAYSIZE`: always defined as `Foo_MAX + 1`.
  `const int Foo_ARRAYSIZE` ：始终定义为 `Foo_MAX + 1` 。

**Be careful when casting integers to proto2 enums.** If an integer is cast to a proto2 enum value, the integer *must* be one of the valid values for that enum, or the results may be undefined. If in doubt, use the generated `Foo_IsValid()` function to test if the cast is valid. Setting an enum-typed field of a proto2 message to an invalid value may cause an assertion failure. If an invalid enum value is read when parsing a proto2 message, it will be treated as an [unknown field](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.unknown_field_set). These semantics have been changed in proto3. It’s safe to cast any integer to a proto3 enum value as long as it fits into int32. Invalid enum values will also be kept when parsing a proto3 message and returned by enum field accessors.
将整数强制转换为 proto2 枚举时要小心。如果将整数强制转换为 proto2 枚举值，则该整数必须是该枚举的有效值之一，否则结果可能未定义。如有疑问，请使用生成的 `Foo_IsValid()` 函数来测试强制转换是否有效。将 proto2 消息的枚举类型字段设置为无效值可能会导致断言失败。如果在解析 proto2 消息时读取了无效枚举值，则该值将被视为未知字段。这些语义在 proto3 中已更改。只要整数适合 int32，就可以安全地将任何整数强制转换为 proto3 枚举值。解析 proto3 消息时也会保留无效枚举值，并由枚举字段访问器返回。

**Be careful when using proto3 enums in switch statements.** Proto3 enums are open enum types with possible values outside the range of specified symbols. Unrecognized enum values will be kept when parsing a proto3 message and returned by the enum field accessors. A switch statement on a proto3 enum without a default case will not be able to catch all cases even if all the known fields are listed. This could lead to unexpected behavior including data corruption and runtime crashes. **Always add a default case or explicitly call `Foo_IsValid(int)` outside of the switch to handle unknown enum values.**
在 switch 语句中使用 proto3 枚举时要小心。Proto3 枚举是开放枚举类型，其可能值超出指定符号的范围。解析 proto3 消息时，将保留无法识别的枚举值，并由枚举字段访问器返回。即使列出了所有已知字段，没有默认情况的 proto3 枚举的 switch 语句也无法捕获所有情况。这可能导致意外行为，包括数据损坏和运行时崩溃。始终添加一个默认情况或在 switch 外部显式调用 `Foo_IsValid(int)` 来处理未知枚举值。

You can define an enum inside a message type. In this case, the protocol buffer compiler generates code that makes it appear that the enum type itself was declared nested inside the message’s class. The `Foo_descriptor()` and `Foo_IsValid()` functions are declared as static methods. In reality, the enum type itself and its values are declared at the global scope with mangled names, and are imported into the class’s scope with a typedef and a series of constant definitions. This is done only to get around problems with declaration ordering. Do not depend on the mangled top-level names; pretend the enum really is nested in the message class.
您可以在消息类型中定义枚举。在这种情况下，协议缓冲区编译器生成的代码会让枚举类型本身看起来是声明在消息的类中。函数 `Foo_descriptor()` 和 `Foo_IsValid()` 被声明为静态方法。实际上，枚举类型本身及其值在全局作用域中声明，名称经过混淆，并通过 typedef 和一系列常量定义导入到类的作用域中。这样做只是为了解决声明顺序问题。不要依赖混淆的顶级名称；假装枚举确实嵌套在消息类中。

## Extensions (proto2 only) 扩展（仅限 proto2）

Given a message with an extension range:
给定具有扩展范围的消息：

```proto
message Foo {
  extensions 100 to 199;
}
```

The protocol buffer compiler will generate some additional methods for `Foo`: `HasExtension()`, `ExtensionSize()`, `ClearExtension()`, `GetExtension()`, `SetExtension()`, `MutableExtension()`, `AddExtension()`, `SetAllocatedExtension()` and `ReleaseExtension()`. Each of these methods takes, as its first parameter, an extension identifier (described below), which identifies an extension field. The remaining parameters and the return value are exactly the same as those for the corresponding accessor methods that would be generated for a normal (non-extension) field of the same type as the extension identifier. (`GetExtension()` corresponds to the accessors with no special prefix.)
协议缓冲区编译器将为 `Foo` 生成一些其他方法： `HasExtension()` 、 `ExtensionSize()` 、 `ClearExtension()` 、 `GetExtension()` 、 `SetExtension()` 、 `MutableExtension()` 、 `AddExtension()` 、 `SetAllocatedExtension()` 和 `ReleaseExtension()` 。这些方法中的每一个都将扩展标识符（如下所述）作为其第一个参数，该标识符标识一个扩展字段。其余参数和返回值与为与扩展标识符类型相同的普通（非扩展）字段生成的相应访问器方法的参数和返回值完全相同。（ `GetExtension()` 对应于没有特殊前缀的访问器。）

Given an extension definition:
给定扩展定义：

```proto
extend Foo {
  optional int32 bar = 123;
  repeated int32 repeated_bar = 124;
  optional Bar message_bar = 125;
}
```

For the singular extension field `bar`, the protocol buffer compiler generates an “extension identifier” called `bar`, which you can use with `Foo`’s extension accessors to access this extension, like so:
对于单数扩展字段 `bar` ，协议缓冲区编译器会生成一个名为 `bar` 的“扩展标识符”，您可以将其与 `Foo` 的扩展访问器一起使用来访问此扩展，如下所示：

```cpp
Foo foo;
assert(!foo.HasExtension(bar));
foo.SetExtension(bar, 1);
assert(foo.HasExtension(bar));
assert(foo.GetExtension(bar) == 1);
foo.ClearExtension(bar);
assert(!foo.HasExtension(bar));
```

For the message extension field `message_bar`, if the field is not set `foo.GetExtension(message_bar)` returns a `Bar` with none of its fields set (possibly `Bar::default_instance()`).
对于消息扩展字段 `message_bar` ，如果字段未设置， `foo.GetExtension(message_bar)` 会返回一个未设置任何字段（可能是 `Bar::default_instance()` ）的 `Bar` 。

Similarly, for the repeated extension field `repeated_bar`, the compiler generates an extension identifier called `repeated_bar`, which you can also use with `Foo`’s extension accessors:
类似地，对于重复扩展字段 `repeated_bar` ，编译器会生成一个名为 `repeated_bar` 的扩展标识符，您也可以将其与 `Foo` 的扩展访问器一起使用：

```cpp
Foo foo;
for (int i = 0; i < kSize; ++i) {
  foo.AddExtension(repeated_bar, i)
}
assert(foo.ExtensionSize(repeated_bar) == kSize)
for (int i = 0; i < kSize; ++i) {
  assert(foo.GetExtension(repeated_bar, i) == i)
}
```

(The exact implementation of extension identifiers is complicated and involves magical use of templates—however, you don’t need to worry about how extension identifiers work to use them.)
（扩展标识符的确切实现很复杂，涉及模板的神奇用法——但是，您无需担心扩展标识符的工作原理即可使用它们。）

Extensions can be declared nested inside of another type. For example, a common pattern is to do something like this:
扩展可以声明为嵌套在另一个类型内部。例如，一个常见的模式是执行类似这样的操作：

```proto
message Baz {
  extend Foo {
    optional Baz foo_ext = 124;
  }
}
```

In this case, the extension identifier `foo_ext` is declared nested inside `Baz`. It can be used as follows:
在这种情况下，扩展标识符 `foo_ext` 声明为嵌套在 `Baz` 内部。它可以按如下方式使用：

```cpp
Foo foo;
Baz* baz = foo.MutableExtension(Baz::foo_ext);
FillInMyBaz(baz);
```

## Arena Allocation Arena 分配

Arena allocation is a C++-only feature that helps you optimize your memory usage and improve performance when working with protocol buffers. Enabling arena allocation in your `.proto` adds additional code for working with arenas to your C++ generated code. You can find out more about the arena allocation API in the [Arena Allocation Guide](https://protobuf.dev/reference/cpp/arenas).
Arena 分配是仅限 C++ 的一项功能，可帮助您优化内存使用情况，并在使用协议缓冲区时提高性能。在 `.proto` 中启用 arena 分配会为您的 C++ 生成的代码添加用于处理 arena 的其他代码。您可以在 Arena 分配指南中找到有关 arena 分配 API 的更多信息。

## Services 服务

If the `.proto` file contains the following line:
如果 `.proto` 文件包含以下行：

```proto
option cc_generic_services = true;
```

then the protocol buffer compiler will generate code based on the service definitions found in the file as described in this section. However, the generated code may be undesirable as it is not tied to any particular RPC system, and thus requires more levels of indirection than code tailored to one system. If you do NOT want this code to be generated, add this line to the file:
然后，协议缓冲区编译器将根据本节所述文件中找到的服务定义生成代码。但是，生成的代码可能不理想，因为它与任何特定的 RPC 系统无关，因此需要比针对一个系统定制的代码更多的间接级别。如果您不希望生成此代码，请将此行添加到文件中：

```proto
option cc_generic_services = false;
```

If neither of the above lines are given, the option defaults to `false`, as generic services are deprecated. (Note that prior to 2.4.0, the option defaults to `true`)
如果未给出上述任何一行，则该选项默认为 `false` ，因为通用服务已弃用。（请注意，在 2.4.0 之前，该选项默认为 `true` ）

RPC systems based on `.proto`-language service definitions should provide [plugins](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.compiler.plugin.pb) to generate code appropriate for the system. These plugins are likely to require that abstract services are disabled, so that they can generate their own classes of the same names.
基于 `.proto` 语言服务定义的 RPC 系统应提供插件来生成适用于该系统的代码。这些插件可能需要禁用抽象服务，以便它们可以生成具有相同名称的自己的类。

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

The protocol buffer compiler will generate a class `Foo` to represent this service. `Foo` will have a virtual method for each method defined in the service definition. In this case, the method `Bar` is defined as:
协议缓冲区编译器将生成一个类 `Foo` 来表示此服务。 `Foo` 将为服务定义中定义的每个方法提供一个虚拟方法。在这种情况下，方法 `Bar` 定义为：

```cpp
virtual void Bar(RpcController* controller, const FooRequest* request,
                 FooResponse* response, Closure* done);
```

The parameters are equivalent to the parameters of `Service::CallMethod()`, except that the `method` argument is implied and `request` and `response` specify their exact type.
这些参数等同于 `Service::CallMethod()` 的参数，但隐含了 `method` 参数，而 `request` 和 `response` 指定了它们的确切类型。

These generated methods are virtual, but not pure-virtual. The default implementations simply call `controller->SetFailed()` with an error message indicating that the method is unimplemented, then invoke the `done` callback. When implementing your own service, you must subclass this generated service and implement its methods as appropriate.
这些生成的方法是虚拟的，但不是纯虚拟的。默认实现只是使用错误消息调用 `controller->SetFailed()` ，指示该方法未实现，然后调用 `done` 回调。在实现您自己的服务时，您必须对这个生成的服务进行子类化并适当地实现其方法。

`Foo` subclasses the `Service` interface. The protocol buffer compiler automatically generates implementations of the methods of `Service` as follows:
`Foo` 对 `Service` 接口进行子类化。协议缓冲区编译器自动生成 `Service` 的方法的实现，如下所示：

- `GetDescriptor`: Returns the service’s [`ServiceDescriptor`](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.descriptor#ServiceDescriptor).
  `GetDescriptor` ：返回服务的 `ServiceDescriptor` 。
- `CallMethod`: Determines which method is being called based on the provided method descriptor and calls it directly, down-casting the request and response messages objects to the correct types.
  `CallMethod` ：根据提供的 method descriptor 确定正在调用哪个方法，并直接调用它，将请求和响应消息对象向下转换为正确的类型。
- `GetRequestPrototype` and `GetResponsePrototype`: Returns the default instance of the request or response of the correct type for the given method.
  `GetRequestPrototype` 和 `GetResponsePrototype` ：返回给定方法的正确类型的请求或响应的默认实例。

The following static method is also generated:
还会生成以下静态方法：

- `static ServiceDescriptor descriptor()`: Returns the type’s descriptor, which contains information about what methods this service has and what their input and output types are.
  `static ServiceDescriptor descriptor()` ：返回类型的描述符，其中包含有关此服务具有哪些方法以及它们的输入和输出类型是什么的信息。

### Stub 存根

The protocol buffer compiler also generates a “stub” implementation of every service interface, which is used by clients wishing to send requests to servers implementing the service. For the `Foo` service (above), the stub implementation `Foo_Stub` will be defined. As with nested message types, a typedef is used so that `Foo_Stub` can also be referred to as `Foo::Stub`.
协议缓冲区编译器还会为每个服务接口生成一个“存根”实现，该实现由希望向实现该服务的服务器发送请求的客户端使用。对于 `Foo` 服务（以上），将定义存根实现 `Foo_Stub` 。与嵌套消息类型一样，使用typedef，以便 `Foo_Stub` 也可以称为 `Foo::Stub` 。

`Foo_Stub` is a subclass of `Foo` which also implements the following methods:
`Foo_Stub` 是 `Foo` 的子类，它还实现了以下方法：

- `Foo_Stub(RpcChannel* channel)`: Constructs a new stub which sends requests on the given channel.
  `Foo_Stub(RpcChannel* channel)` ：构造一个在给定通道上发送请求的新存根。
- `Foo_Stub(RpcChannel* channel, ChannelOwnership ownership)`: Constructs a new stub which sends requests on the given channel and possibly owns that channel. If `ownership` is `Service::STUB_OWNS_CHANNEL` then when the stub object is deleted it will delete the channel as well.
  `Foo_Stub(RpcChannel* channel, ChannelOwnership ownership)` ：构造一个在给定通道上发送请求并可能拥有该通道的新存根。如果 `ownership` 是 `Service::STUB_OWNS_CHANNEL` ，那么当存根对象被删除时，它也将删除通道。
- `RpcChannel* channel()`: Returns this stub’s channel, as passed to the constructor.
  `RpcChannel* channel()` ：返回此存根的通道，如传递给构造函数。

The stub additionally implements each of the service’s methods as a wrapper around the channel. Calling one of the methods simply calls `channel->CallMethod()`.
此外，存根还将服务的每个方法实现为通道周围的包装器。调用其中一个方法只是调用 `channel->CallMethod()` 。

The Protocol Buffer library does not include an RPC implementation. However, it includes all of the tools you need to hook up a generated service class to any arbitrary RPC implementation of your choice. You need only provide implementations of [`RpcChannel`](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.service#RpcChannel) and [`RpcController`](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.service#RpcController). See the documentation for [`service.h`](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.service) for more information.
协议缓冲区库不包含 RPC 实现。但是，它包含将生成的 service 类连接到您选择的任何任意 RPC 实现所需的所有工具。您只需提供 `RpcChannel` 和 `RpcController` 的实现。有关更多信息，请参阅 `service.h` 的文档。

## Plugin Insertion Points 插件插入点

[Code generator plugins](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.compiler.plugin.pb) which want to extend the output of the C++ code generator may insert code of the following types using the given insertion point names. Each insertion point appears in both the `.pb.cc` file and the `.pb.h` file unless otherwise noted.
希望扩展 C++ 代码生成器输出的代码生成器插件可以使用给定的插入点名称插入以下类型的代码。除非另有说明，否则每个插入点都出现在 `.pb.cc` 文件和 `.pb.h` 文件中。

- `includes`: Include directives.
  `includes` ：包含指令。
- `namespace_scope`: Declarations that belong in the file’s package/namespace, but not within any particular class. Appears after all other namespace-scope code.
  `namespace_scope` ：属于文件包/命名空间的声明，但不属于任何特定类。出现在所有其他命名空间范围代码之后。
- `global_scope`: Declarations that belong at the top level, outside of the file’s namespace. Appears at the very end of the file.
  `global_scope` ：属于顶层的声明，位于文件命名空间之外。出现在文件的最后。
- `class_scope:TYPENAME`: Member declarations that belong in a message class. `TYPENAME` is the full proto name, e.g. `package.MessageType`. Appears after all other public declarations in the class. This insertion point appears only in the `.pb.h` file.
  `class_scope:TYPENAME` ：属于消息类的成员声明。 `TYPENAME` 是完整协议名称，例如 `package.MessageType` 。出现在类中的所有其他公共声明之后。此插入点仅出现在 `.pb.h` 文件中。

Do not generate code which relies on private class members declared by the standard code generator, as these implementation details may change in future versions of Protocol Buffers.
不要生成依赖于标准代码生成器声明的私有类成员的代码，因为这些实现细节可能会在 Protocol Buffers 的未来版本中发生变化。
