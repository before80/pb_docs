+++
title = "Protocol Buffer Basics: C++"
weight = 210
linkTitle = "C++"
description = "C++ 程序员入门指南，介绍如何使用 Protocol Buffers。"
type = "docs"

+++

## Protocol Buffer Basics: C++ Protocol Buffer 基础知识：C++

A basic C++ programmers introduction to working with protocol buffers.

​	C++ 程序员入门指南，介绍如何使用 Protocol Buffers。



This tutorial provides a basic C++ programmers introduction to working with protocol buffers. By walking through creating a simple example application, it shows you how to

​	本教程为 C++ 程序员提供了一份入门指南，介绍如何使用 Protocol Buffers。通过逐步创建一个简单的示例应用程序，它向您展示如何

- Define message formats in a `.proto` file.
- 在 `.proto` 文件中定义消息格式。
- Use the protocol buffer compiler.
- 使用协议缓冲区编译器。
- Use the C++ protocol buffer API to write and read messages.
- 使用 C++ Protocol Buffer API 来编写和读取消息。

This isn’t a comprehensive guide to using protocol buffers in C++. For more detailed reference information, see the [Protocol Buffer Language Guide (proto2)](https://protobuf.dev/programming-guides/proto), the [Protocol Buffer Language Guide (proto3)](https://protobuf.dev/programming-guides/proto3), the [C++ API Reference](https://protobuf.dev/reference/cpp/api-docs), the [C++ Generated Code Guide](https://protobuf.dev/reference/cpp/cpp-generated), and the [Encoding Reference](https://protobuf.dev/programming-guides/encoding).

​	这不是一份关于如何在 C++ 中使用 Protocol Buffers 的综合指南。如需更详细的参考信息，请参阅 Protocol Buffer 语言指南 (proto2)、Protocol Buffer 语言指南 (proto3)、C++ API 参考、C++ 生成代码指南和编码参考。

## 问题域 The Problem Domain 

The example we’re going to use is a very simple “address book” application that can read and write people’s contact details to and from a file. Each person in the address book has a name, an ID, an email address, and a contact phone number.

​	我们将要使用的示例是一个非常简单的“通讯录”应用程序，它可以将人们的联系方式读写到文件。通讯录中的每个人都有一个姓名、一个 ID、一个电子邮件地址和一个联系电话号码。

How do you serialize and retrieve structured data like this? There are a few ways to solve this problem:

​	您如何序列化和检索这样的结构化数据？有几种方法可以解决此问题：

- The raw in-memory data structures can be sent/saved in binary form. Over time, this is a fragile approach, as the receiving/reading code must be compiled with exactly the same memory layout, endianness, etc. Also, as files accumulate data in the raw format and copies of software that are wired for that format are spread around, it’s very hard to extend the format.
- 原始内存数据结构可以以二进制形式发送/保存。随着时间的推移，这是一种脆弱的方法，因为接收/读取代码必须使用完全相同的内存布局、字节序等进行编译。此外，随着文件以原始格式累积数据，并且针对该格式进行布线的软件副本被四处传播，扩展格式非常困难。
- You can invent an ad-hoc way to encode the data items into a single string – such as encoding 4 ints as “12:3:-23:67”. This is a simple and flexible approach, although it does require writing one-off encoding and parsing code, and the parsing imposes a small run-time cost. This works best for encoding very simple data.
- 您可以发明一种临时方法将数据项编码为单个字符串——例如将 4 个整数编码为“12:3:-23:67”。这是一种简单而灵活的方法，尽管它确实需要编写一次性编码和解析代码，并且解析会产生少量运行时成本。这最适合编码非常简单的数据。
- Serialize the data to XML. This approach can be very attractive since XML is (sort of) human readable and there are binding libraries for lots of languages. This can be a good choice if you want to share data with other applications/projects. However, XML is notoriously space intensive, and encoding/decoding it can impose a huge performance penalty on applications. Also, navigating an XML DOM tree is considerably more complicated than navigating simple fields in a class normally would be.
- 将数据序列化为 XML。这种方法可能非常有吸引力，因为 XML（某种程度上）是人类可读的，并且有很多语言的绑定库。如果您想与其他应用程序/项目共享数据，这可能是一个不错的选择。但是，XML 以空间密集而臭名昭著，并且对它的编码/解码可能会给应用程序带来巨大的性能损失。此外，导航 XML DOM 树比通常在类中导航简单字段要复杂得多。

Instead of these options, you can use protocol buffers. Protocol buffers are the flexible, efficient, automated solution to solve exactly this problem. With protocol buffers, you write a `.proto` description of the data structure you wish to store. From that, the protocol buffer compiler creates a class that implements automatic encoding and parsing of the protocol buffer data with an efficient binary format. The generated class provides getters and setters for the fields that make up a protocol buffer and takes care of the details of reading and writing the protocol buffer as a unit. Importantly, the protocol buffer format supports the idea of extending the format over time in such a way that the code can still read data encoded with the old format.

​	除了这些选项，您还可以使用协议缓冲区。协议缓冲区是解决此问题的灵活、高效、自动化的解决方案。使用协议缓冲区，您可以编写一个 `.proto` 描述您想要存储的数据结构。由此，协议缓冲区编译器会创建一个类，该类实现协议缓冲区数据的自动编码和解析，并采用高效的二进制格式。生成的类提供获取器和设置器，用于构成协议缓冲区的字段，并负责以一个单元的形式读取和写入协议缓冲区的详细信息。重要的是，协议缓冲区格式支持随着时间的推移扩展格式的想法，这样代码仍然可以读取使用旧格式编码的数据。

## 在哪里找到示例代码 Where to Find the Example Code 

The example code is included in the source code package, under the [“examples” directory](https://github.com/protocolbuffers/protobuf/tree/main/examples).

​	示例代码包含在源代码包中，“examples”目录下。

## 定义协议格式 Defining Your Protocol Format 

To create your address book application, you’ll need to start with a `.proto` file. The definitions in a `.proto` file are simple: you add a *message* for each data structure you want to serialize, then specify a name and a type for each field in the message. Here is the `.proto` file that defines your messages, `addressbook.proto`.

​	要创建您的通讯录应用程序，您需要从 `.proto` 文件开始。在 `.proto` 文件中的定义很简单：为要序列化的每个数据结构添加一条消息，然后为消息中的每个字段指定一个名称和类型。以下是定义您的消息的 `.proto` 文件， `addressbook.proto` 。

```proto
syntax = "proto2";

package tutorial;

message Person {
  optional string name = 1;
  optional int32 id = 2;
  optional string email = 3;

  enum PhoneType {
    PHONE_TYPE_UNSPECIFIED = 0;
    PHONE_TYPE_MOBILE = 1;
    PHONE_TYPE_HOME = 2;
    PHONE_TYPE_WORK = 3;
  }

  message PhoneNumber {
    optional string number = 1;
    optional PhoneType type = 2 [default = PHONE_TYPE_HOME];
  }

  repeated PhoneNumber phones = 4;
}

message AddressBook {
  repeated Person people = 1;
}
```

As you can see, the syntax is similar to C++ or Java. Let’s go through each part of the file and see what it does.

​	正如您所见，语法类似于 C++ 或 Java。让我们逐个部分地了解该文件，看看它做了什么。

The `.proto` file starts with a package declaration, which helps to prevent naming conflicts between different projects. In C++, your generated classes will be placed in a namespace matching the package name.

​	文件以包声明开头，这有助于防止不同项目之间的命名冲突。在 C++ 中，生成的类将被放置在与包名称匹配的命名空间中。

Next, you have your message definitions. A message is just an aggregate containing a set of typed fields. Many standard simple data types are available as field types, including `bool`, `int32`, `float`, `double`, and `string`. You can also add further structure to your messages by using other message types as field types – in the above example the `Person` message contains `PhoneNumber` messages, while the `AddressBook` message contains `Person` messages. You can even define message types nested inside other messages – as you can see, the `PhoneNumber` type is defined inside `Person`. You can also define `enum` types if you want one of your fields to have one of a predefined list of values – here you want to specify that a phone number can be one of the following phone types: `PHONE_TYPE_MOBILE`, `PHONE_TYPE_HOME`, or `PHONE_TYPE_WORK`.

​	接下来，您有您的消息定义。消息只是一个包含一组类型化字段的聚合。许多标准的简单数据类型可用作字段类型，包括 `bool` 、 `int32` 、 `float` 、 `double` 和 `string` 。您还可以通过使用其他消息类型作为字段类型来为您的消息添加更多结构——在上面的示例中， `Person` 消息包含 `PhoneNumber` 消息，而 `AddressBook` 消息包含 `Person` 消息。您甚至可以定义嵌套在其他消息内部的消息类型——如您所见， `PhoneNumber` 类型定义在 `Person` 内部。如果您希望您的某个字段具有预定义值列表中的一个值，您还可以定义 `enum` 类型——在这里，您要指定电话号码可以是以下电话类型之一： `PHONE_TYPE_MOBILE` 、 `PHONE_TYPE_HOME` 或 `PHONE_TYPE_WORK` 。

The " = 1", " = 2" markers on each element identify the unique field number that field uses in the binary encoding. Field numbers 1-15 require one less byte to encode than higher numbers, so as an optimization you can decide to use those numbers for the commonly used or repeated elements, leaving field numbers 16 and higher for less-commonly used optional elements. Each element in a repeated field requires re-encoding the field number, so repeated fields are particularly good candidates for this optimization.

​	每个元素上的“ = 1”、“ = 2”标记标识该字段在二进制编码中使用的唯一字段编号。字段编号 1-15 比较高的数字编码时少一个字节，因此作为优化，您可以决定将这些数字用于常用或重复的元素，将字段编号 16 和更高编号留给不常用的可选元素。重复字段中的每个元素都需要重新编码字段编号，因此重复字段特别适合这种优化。

Each field must be annotated with one of the following modifiers:

​	每个字段都必须使用以下修饰符之一进行注释：

- `optional`: the field may or may not be set. If an optional field value isn’t set, a default value is used. For simple types, you can specify your own default value, as we’ve done for the phone number `type` in the example. Otherwise, a system default is used: zero for numeric types, the empty string for strings, false for bools. For embedded messages, the default value is always the “default instance” or “prototype” of the message, which has none of its fields set. Calling the accessor to get the value of an optional (or required) field which has not been explicitly set always returns that field’s default value.
- `optional` ：该字段可以设置，也可以不设置。如果未设置可选字段值，则使用默认值。对于简单类型，您可以指定自己的默认值，就像我们在示例中为电话号码 `type` 所做的那样。否则，将使用系统默认值：对于数字类型为零，对于字符串为空字符串，对于布尔值为 false。对于嵌入式消息，默认值始终是消息的“默认实例”或“原型”，其没有任何字段设置。调用访问器以获取尚未显式设置的可选（或必需）字段的值时，始终返回该字段的默认值。
- `repeated`: the field may be repeated any number of times (including zero). The order of the repeated values will be preserved in the protocol buffer. Think of repeated fields as dynamically sized arrays.
- `repeated` ：该字段可以重复任意多次（包括零次）。重复值在协议缓冲区中的顺序将被保留。将重复字段视为动态大小的数组。
- `required`: a value for the field must be provided, otherwise the message will be considered “uninitialized”. If `libprotobuf` is compiled in debug mode, serializing an uninitialized message will cause an assertion failure. In optimized builds, the check is skipped and the message will be written anyway. However, parsing an uninitialized message will always fail (by returning `false` from the parse method). Other than this, a required field behaves exactly like an optional field.
- `required` ：必须提供字段值，否则消息将被视为“未初始化”。如果在调试模式下编译 `libprotobuf` ，则序列化未初始化的消息将导致断言失败。在经过优化的版本中，将跳过检查，并且无论如何都将写入消息。但是，解析未初始化的消息将始终失败（通过从解析方法返回 `false` ）。除此之外，必需字段的行为与可选字段完全相同。

#### 重要 Important 

**Required Is Forever
必需永远是必需的** 

You should be very careful about marking fields as `required`. If at some point you wish to stop writing or sending a required field, it will be problematic to change the field to an optional field – old readers will consider messages without this field to be incomplete and may reject or drop them unintentionally. You should consider writing application-specific custom validation routines for your buffers instead. Within Google, `required` fields are strongly disfavored; most messages defined in proto2 syntax use `optional` and `repeated` only. (Proto3 does not support `required` fields at all.)

​	您应该非常小心地将字段标记为 `required` 。如果您在某个时刻希望停止编写或发送必填字段，将字段更改为可选字段会很麻烦——旧读者会认为没有此字段的消息不完整，可能会拒绝或无意中丢弃它们。您应该考虑为缓冲区编写特定于应用程序的自定义验证例程。在 Google 内部， `required` 字段非常不受欢迎；大多数在 proto2 语法中定义的消息仅使用 `optional` 和 `repeated` 。（Proto3 完全不支持 `required` 字段。）

You’ll find a complete guide to writing `.proto` files – including all the possible field types – in the [Protocol Buffer Language Guide](https://protobuf.dev/programming-guides/proto). Don’t go looking for facilities similar to class inheritance, though – protocol buffers don’t do that.

​	您可以在协议缓冲区语言指南中找到编写 `.proto` 文件的完整指南，包括所有可能的字段类型。不过，不要寻找类似于类继承的功能，协议缓冲区不具备该功能。

## 编译您的协议缓冲区 Compiling Your Protocol Buffers 

Now that you have a `.proto`, the next thing you need to do is generate the classes you’ll need to read and write `AddressBook` (and hence `Person` and `PhoneNumber`) messages. To do this, you need to run the protocol buffer compiler `protoc` on your `.proto`:

​	现在您已拥有 `.proto` ，接下来需要做的是生成您需要用来读写 `AddressBook` （进而读写 `Person` 和 `PhoneNumber` ）消息的类。为此，您需要在 `.proto` 上运行协议缓冲区编译器 `protoc` ：

1. If you haven’t installed the compiler, [download the package](https://protobuf.dev/downloads) and follow the instructions in the README.

2. 如果您尚未安装编译器，请下载软件包并按照自述文件中的说明进行操作。

3. Now run the compiler, specifying the source directory (where your application’s source code lives – the current directory is used if you don’t provide a value), the destination directory (where you want the generated code to go; often the same as `$SRC_DIR`), and the path to your `.proto`. In this case, you…:

4. 现在运行编译器，指定源目录（您的应用程序源代码所在的位置——如果您未提供值，则使用当前目录）、目标目录（您希望生成的代码所在的位置；通常与 `$SRC_DIR` 相同）以及 `.proto` 的路径。在这种情况下，您…：

   ```shell
   protoc -I=$SRC_DIR --cpp_out=$DST_DIR $SRC_DIR/addressbook.proto
   ```

   Because you want C++ classes, you use the `--cpp_out` option – similar options are provided for other supported languages.

   因为您想要 C++ 类，所以您使用 `--cpp_out` 选项——其他受支持的语言提供了类似的选项。

This generates the following files in your specified destination directory:

​	这将在您指定的目标目录中生成以下文件：

- `addressbook.pb.h`, the header which declares your generated classes.
- `addressbook.pb.h` ，声明您生成的类的头文件。
- `addressbook.pb.cc`, which contains the implementation of your classes.
- `addressbook.pb.cc` ，其中包含您类的实现。

## 协议缓冲区 API - The Protocol Buffer API 

Let’s look at some of the generated code and see what classes and functions the compiler has created for you. If you look in `addressbook.pb.h`, you can see that you have a class for each message you specified in `addressbook.proto`. Looking closer at the `Person` class, you can see that the compiler has generated accessors for each field. For example, for the `name`, `id`, `email`, and `phones` fields, you have these methods:

​	让我们看看一些生成的代码，看看编译器为您创建了哪些类和函数。如果您查看 `addressbook.pb.h` ，您会看到您为在 `addressbook.proto` 中指定的每条消息都有一个类。仔细查看 `Person` 类，您会看到编译器为每个字段生成了访问器。例如，对于 `name` 、 `id` 、 `email` 和 `phones` 字段，您有以下方法：

```cpp
  // name
  inline bool has_name() const;
  inline void clear_name();
  inline const ::std::string& name() const;
  inline void set_name(const ::std::string& value);
  inline void set_name(const char* value);
  inline ::std::string* mutable_name();

  // id
  inline bool has_id() const;
  inline void clear_id();
  inline int32_t id() const;
  inline void set_id(int32_t value);

  // email
  inline bool has_email() const;
  inline void clear_email();
  inline const ::std::string& email() const;
  inline void set_email(const ::std::string& value);
  inline void set_email(const char* value);
  inline ::std::string* mutable_email();

  // phones
  inline int phones_size() const;
  inline void clear_phones();
  inline const ::google::protobuf::RepeatedPtrField< ::tutorial::Person_PhoneNumber >& phones() const;
  inline ::google::protobuf::RepeatedPtrField< ::tutorial::Person_PhoneNumber >* mutable_phones();
  inline const ::tutorial::Person_PhoneNumber& phones(int index) const;
  inline ::tutorial::Person_PhoneNumber* mutable_phones(int index);
  inline ::tutorial::Person_PhoneNumber* add_phones();
```

As you can see, the getters have exactly the name as the field in lowercase, and the setter methods begin with `set_`. There are also `has_` methods for each singular (required or optional) field which return true if that field has been set. Finally, each field has a `clear_` method that un-sets the field back to its empty state.

​	如您所见，getter 的名称与字段的名称完全相同，只是小写，setter 方法以 `set_` 开头。对于每个单数（必需或可选）字段，还有一些 `has_` 方法，如果已设置该字段，则返回 true。最后，每个字段都有一个 `clear_` 方法，该方法将字段重置为空状态。

While the numeric `id` field just has the basic accessor set described above, the `name` and `email` fields have a couple of extra methods because they’re strings – a `mutable_` getter that lets you get a direct pointer to the string, and an extra setter. Note that you can call `mutable_email()` even if `email` is not already set; it will be initialized to an empty string automatically. If you had a repeated message field in this example, it would also have a `mutable_` method but not a `set_` method.

​	虽然数字 `id` 字段仅具有上面描述的基本访问器集，但 `name` 和 `email` 字段有一些额外的方法，因为它们是字符串 - 一个 `mutable_` 获取器，它允许您获取字符串的直接指针，以及一个额外的设置器。请注意，即使尚未设置 `email` ，您也可以调用 `mutable_email()` ；它将自动初始化为空字符串。如果在此示例中您有一个重复的消息字段，它还将具有 `mutable_` 方法，但没有 `set_` 方法。

Repeated fields also have some special methods – if you look at the methods for the repeated `phones` field, you’ll see that you can

​	重复字段也有一些特殊方法 - 如果你查看重复 `phones` 字段的方法，你会看到你可以

- check the repeated field’s `_size` (in other words, how many phone numbers are associated with this `Person`).
- 检查重复字段的 `_size` （换句话说，有多少个电话号码与此 `Person` 相关联）。
- get a specified phone number using its index.
- 使用其索引获取指定的电话号码。
- update an existing phone number at the specified index.
- 更新指定索引处的现有电话号码。
- add another phone number to the message which you can then edit (repeated scalar types have an `add_` that just lets you pass in the new value).
- 向消息中添加另一个电话号码，然后可以对其进行编辑（重复标量类型有一个 `add_` ，它只允许您传入新值）。

For more information on exactly what members the protocol compiler generates for any particular field definition, see the [C++ generated code reference](https://protobuf.dev/reference/cpp/cpp-generated).

​	有关协议编译器为任何特定字段定义生成的成员的详细信息，请参阅 C++ 生成的代码参考。

### 枚举和嵌套类 Enums and Nested Classes 

The generated code includes a `PhoneType` enum that corresponds to your `.proto` enum. You can refer to this type as `Person::PhoneType` and its values as `Person::PHONE_TYPE_MOBILE`, `Person::PHONE_TYPE_HOME`, and `Person::PHONE_TYPE_WORK` (the implementation details are a little more complicated, but you don’t need to understand them to use the enum).

​	生成的代码包含一个与您的 `.proto` 枚举相对应的 `PhoneType` 枚举。您可以将此类型称为 `Person::PhoneType` ，并将其值称为 `Person::PHONE_TYPE_MOBILE` 、 `Person::PHONE_TYPE_HOME` 和 `Person::PHONE_TYPE_WORK` （实现细节稍微复杂一些，但您无需了解它们即可使用枚举）。

The compiler has also generated a nested class for you called `Person::PhoneNumber`. If you look at the code, you can see that the “real” class is actually called `Person_PhoneNumber`, but a typedef defined inside `Person` allows you to treat it as if it were a nested class. The only case where this makes a difference is if you want to forward-declare the class in another file – you cannot forward-declare nested types in C++, but you can forward-declare `Person_PhoneNumber`.

​	编译器还为您生成了一个名为 `Person::PhoneNumber` 的嵌套类。如果您查看代码，您会看到“实际”类实际上称为 `Person_PhoneNumber` ，但 `Person` 内部定义的 typedef 允许您将其视为嵌套类。唯一有区别的情况是，如果您想在另一个文件中预先声明该类 - 您无法在 C++ 中预先声明嵌套类型，但您可以预先声明 `Person_PhoneNumber` 。

### 标准消息方法 Standard Message Methods 

Each message class also contains a number of other methods that let you check or manipulate the entire message, including:

​	每个消息类还包含许多其他方法，可让您检查或操作整个消息，包括：

- `bool IsInitialized() const;`: checks if all the required fields have been set.
- `bool IsInitialized() const;` ：检查是否已设置所有必需字段。
- `string DebugString() const;`: returns a human-readable representation of the message, particularly useful for debugging.
- `string DebugString() const;` ：返回消息的人类可读表示形式，特别适用于调试。
- `void CopyFrom(const Person& from);`: overwrites the message with the given message’s values.
- `void CopyFrom(const Person& from);` ：使用给定消息的值覆盖消息。
- `void Clear();`: clears all the elements back to the empty state.
- `void Clear();` ：将所有元素清除回空状态。

These and the I/O methods described in the following section implement the `Message` interface shared by all C++ protocol buffer classes. For more info, see the [complete API documentation for `Message`](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.message#Message).

​	以下部分中描述的这些方法和 I/O 方法实现了所有 C++ 协议缓冲区类共享的 `Message` 接口。有关更多信息，请参阅 `Message` 的完整 API 文档。

### 解析和序列化 Parsing and Serialization 

Finally, each protocol buffer class has methods for writing and reading messages of your chosen type using the protocol buffer [binary format](https://protobuf.dev/programming-guides/encoding). These include:

​	最后，每个协议缓冲区类都有用于使用协议缓冲区二进制格式写入和读取您选择类型的消息的方法。其中包括：

- `bool SerializeToString(string* output) const;`: serializes the message and stores the bytes in the given string. Note that the bytes are binary, not text; we only use the `string` class as a convenient container.
- `bool SerializeToString(string* output) const;` ：对消息进行序列化并将字节存储在给定的字符串中。请注意，字节是二进制的，而不是文本；我们仅将 `string` 类用作一个方便的容器。
- `bool ParseFromString(const string& data);`: parses a message from the given string.
- `bool ParseFromString(const string& data);` ：从给定的字符串中解析消息。
- `bool SerializeToOstream(ostream* output) const;`: writes the message to the given C++ `ostream`.
- `bool SerializeToOstream(ostream* output) const;` ：将消息写入给定的 C++ `ostream` 。
- `bool ParseFromIstream(istream* input);`: parses a message from the given C++ `istream`.
- `bool ParseFromIstream(istream* input);` ：从给定的 C++ `istream` 中解析消息。

These are just a couple of the options provided for parsing and serialization. Again, see the [`Message` API reference](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.message#Message) for a complete list.

​	这些只是为解析和序列化提供的几个选项。同样，请参阅 `Message` API 参考以获取完整列表。

#### 重要 Important 

**Protocol Buffers and Object Oriented Design
协议缓冲区和面向对象设计** 

Protocol buffer classes are basically data holders (like structs in C) that don’t provide additional functionality; they don’t make good first class citizens in an object model. If you want to add richer behavior to a generated class, the best way to do this is to wrap the generated protocol buffer class in an application-specific class. Wrapping protocol buffers is also a good idea if you don’t have control over the design of the `.proto` file (if, say, you’re reusing one from another project). In that case, you can use the wrapper class to craft an interface better suited to the unique environment of your application: hiding some data and methods, exposing convenience functions, etc. **You should never add behavior to the generated classes by inheriting from them**. This will break internal mechanisms and is not good object-oriented practice anyway.

​	协议缓冲区类基本上是数据持有者（类似于 C 中的结构），不提供其他功能；它们在对象模型中不是好的第一类公民。如果您想向生成的类添加更丰富的行为，最好的方法是将生成的协议缓冲区类包装在特定于应用程序的类中。如果您无法控制 `.proto` 文件的设计（例如，如果您正在从另一个项目中重用一个文件），那么包装协议缓冲区也是一个好主意。在这种情况下，您可以使用包装器类来构建更适合您应用程序的独特环境的接口：隐藏一些数据和方法，公开便利函数等。您绝不应通过继承生成类来向它们添加行为。这会破坏内部机制，而且无论如何也不是好的面向对象实践。

## 编写消息 Writing a Message 

Now let’s try using your protocol buffer classes. The first thing you want your address book application to be able to do is write personal details to your address book file. To do this, you need to create and populate instances of your protocol buffer classes and then write them to an output stream.

​	现在，我们尝试使用您的协议缓冲区类。您希望通讯录应用程序能够做的第一件事是将个人详细信息写入通讯录文件。为此，您需要创建和填充协议缓冲区类的实例，然后将它们写入输出流。

Here is a program which reads an `AddressBook` from a file, adds one new `Person` to it based on user input, and writes the new `AddressBook` back out to the file again. The parts which directly call or reference code generated by the protocol compiler are highlighted.

​	这是一个从文件中读取 `AddressBook` 、根据用户输入向其中添加一个新的 `Person` ，然后将新的 `AddressBook` 再次写入文件的程序。直接调用或引用协议编译器生成的代码的部分已突出显示。

```cpp
#include <iostream>
#include <fstream>
#include <string>
#include "addressbook.pb.h"
using namespace std;

// This function fills in a Person message based on user input.
void PromptForAddress(tutorial::Person* person) {
  cout << "Enter person ID number: ";
  int id;
  cin >> id;
  person->set_id(id);
  cin.ignore(256, '\n');

  cout << "Enter name: ";
  getline(cin, *person->mutable_name());

  cout << "Enter email address (blank for none): ";
  string email;
  getline(cin, email);
  if (!email.empty()) {
    person->set_email(email);
  }

  while (true) {
    cout << "Enter a phone number (or leave blank to finish): ";
    string number;
    getline(cin, number);
    if (number.empty()) {
      break;
    }

    tutorial::Person::PhoneNumber* phone_number = person->add_phones();
    phone_number->set_number(number);

    cout << "Is this a mobile, home, or work phone? ";
    string type;
    getline(cin, type);
    if (type == "mobile") {
      phone_number->set_type(tutorial::Person::PHONE_TYPE_MOBILE);
    } else if (type == "home") {
      phone_number->set_type(tutorial::Person::PHONE_TYPE_HOME);
    } else if (type == "work") {
      phone_number->set_type(tutorial::Person::PHONE_TYPE_WORK);
    } else {
      cout << "Unknown phone type.  Using default." << endl;
    }
  }
}

// Main function:  Reads the entire address book from a file,
//   adds one person based on user input, then writes it back out to the same
//   file.
int main(int argc, char* argv[]) {
  // Verify that the version of the library that we linked against is
  // compatible with the version of the headers we compiled against.
  GOOGLE_PROTOBUF_VERIFY_VERSION;

  if (argc != 2) {
    cerr << "Usage:  " << argv[0] << " ADDRESS_BOOK_FILE" << endl;
    return -1;
  }

  tutorial::AddressBook address_book;

  {
    // Read the existing address book.
    fstream input(argv[1], ios::in | ios::binary);
    if (!input) {
      cout << argv[1] << ": File not found.  Creating a new file." << endl;
    } else if (!address_book.ParseFromIstream(&input)) {
      cerr << "Failed to parse address book." << endl;
      return -1;
    }
  }

  // Add an address.
  PromptForAddress(address_book.add_people());

  {
    // Write the new address book back to disk.
    fstream output(argv[1], ios::out | ios::trunc | ios::binary);
    if (!address_book.SerializeToOstream(&output)) {
      cerr << "Failed to write address book." << endl;
      return -1;
    }
  }

  // Optional:  Delete all global objects allocated by libprotobuf.
  google::protobuf::ShutdownProtobufLibrary();

  return 0;
}
```

Notice the `GOOGLE_PROTOBUF_VERIFY_VERSION` macro. It is good practice – though not strictly necessary – to execute this macro before using the C++ Protocol Buffer library. It verifies that you have not accidentally linked against a version of the library which is incompatible with the version of the headers you compiled with. If a version mismatch is detected, the program will abort. Note that every `.pb.cc` file automatically invokes this macro on startup.

​	注意 `GOOGLE_PROTOBUF_VERIFY_VERSION` 宏。这是一个好习惯——虽然不是严格必要的——在使用 C++ 协议缓冲区库之前执行此宏。它会验证您是否意外地链接到与您编译时使用的头文件版本不兼容的库版本。如果检测到版本不匹配，程序将中止。请注意，每个 `.pb.cc` 文件都会在启动时自动调用此宏。

Also notice the call to `ShutdownProtobufLibrary()` at the end of the program. All this does is delete any global objects that were allocated by the Protocol Buffer library. This is unnecessary for most programs, since the process is just going to exit anyway and the OS will take care of reclaiming all of its memory. However, if you use a memory leak checker that requires that every last object be freed, or if you are writing a library which may be loaded and unloaded multiple times by a single process, then you may want to force Protocol Buffers to clean up everything.

​	还要注意程序末尾对 `ShutdownProtobufLibrary()` 的调用。这只是删除由 Protocol Buffer 库分配的任何全局对象。对于大多数程序来说，这是不必要的，因为进程无论如何都将退出，而操作系统将负责回收其所有内存。但是，如果您使用需要释放每个最后一个对象的内存泄漏检查器，或者如果您正在编写一个可能被单个进程多次加载和卸载的库，那么您可能希望强制 Protocol Buffers 清理所有内容。

## 读取消息 Reading a Message 

Of course, an address book wouldn’t be much use if you couldn’t get any information out of it! This example reads the file created by the above example and prints all the information in it.

​	当然，如果您无法从中获取任何信息，那么通讯录就没有多大用处！此示例读取由上述示例创建的文件并打印其中的所有信息。

```cpp
#include <iostream>
#include <fstream>
#include <string>
#include "addressbook.pb.h"
using namespace std;

// Iterates though all people in the AddressBook and prints info about them.
void ListPeople(const tutorial::AddressBook& address_book) {
  for (int i = 0; i < address_book.people_size(); i++) {
    const tutorial::Person& person = address_book.people(i);

    cout << "Person ID: " << person.id() << endl;
    cout << "  Name: " << person.name() << endl;
    if (person.has_email()) {
      cout << "  E-mail address: " << person.email() << endl;
    }

    for (int j = 0; j < person.phones_size(); j++) {
      const tutorial::Person::PhoneNumber& phone_number = person.phones(j);

      switch (phone_number.type()) {
        case tutorial::Person::PHONE_TYPE_MOBILE:
          cout << "  Mobile phone #: ";
          break;
        case tutorial::Person::PHONE_TYPE_HOME:
          cout << "  Home phone #: ";
          break;
        case tutorial::Person::PHONE_TYPE_WORK:
          cout << "  Work phone #: ";
          break;
      }
      cout << phone_number.number() << endl;
    }
  }
}

// Main function:  Reads the entire address book from a file and prints all
//   the information inside.
int main(int argc, char* argv[]) {
  // Verify that the version of the library that we linked against is
  // compatible with the version of the headers we compiled against.
  GOOGLE_PROTOBUF_VERIFY_VERSION;

  if (argc != 2) {
    cerr << "Usage:  " << argv[0] << " ADDRESS_BOOK_FILE" << endl;
    return -1;
  }

  tutorial::AddressBook address_book;

  {
    // Read the existing address book.
    fstream input(argv[1], ios::in | ios::binary);
    if (!address_book.ParseFromIstream(&input)) {
      cerr << "Failed to parse address book." << endl;
      return -1;
    }
  }

  ListPeople(address_book);

  // Optional:  Delete all global objects allocated by libprotobuf.
  google::protobuf::ShutdownProtobufLibrary();

  return 0;
}
```

## 扩展协议缓冲区 Extending a Protocol Buffer 

Sooner or later after you release the code that uses your protocol buffer, you will undoubtedly want to “improve” the protocol buffer’s definition. If you want your new buffers to be backwards-compatible, and your old buffers to be forward-compatible – and you almost certainly do want this – then there are some rules you need to follow. In the new version of the protocol buffer:

​	在您发布使用协议缓冲区的代码后，迟早您无疑会想要“改进”协议缓冲区的定义。如果您希望新缓冲区向后兼容，旧缓冲区向前兼容——您几乎肯定希望如此——那么您需要遵循一些规则。在协议缓冲区的新版本中：

- you *must not* change the field numbers of any existing fields.
- 您不得更改任何现有字段的字段编号。
- you *must not* add or delete any required fields.
- 您不得添加或删除任何必需的字段。
- you *may* delete optional or repeated fields.
- 您可以删除可选或重复的字段。
- you *may* add new optional or repeated fields but you must use fresh field numbers (that is, field numbers that were never used in this protocol buffer, not even by deleted fields).
- 您可以添加新的可选字段或重复字段，但您必须使用新的字段编号（即，此协议缓冲区中从未使用过的字段编号，即使是已删除的字段也不行）。

(There are [some exceptions](https://protobuf.dev/programming-guides/proto#updating) to these rules, but they are rarely used.)

​	（这些规则有一些例外，但很少使用。）

If you follow these rules, old code will happily read new messages and simply ignore any new fields. To the old code, optional fields that were deleted will simply have their default value, and deleted repeated fields will be empty. New code will also transparently read old messages. However, keep in mind that new optional fields will not be present in old messages, so you will need to either check explicitly whether they’re set with `has_`, or provide a reasonable default value in your `.proto` file with `[default = value]` after the field number. If the default value is not specified for an optional element, a type-specific default value is used instead: for strings, the default value is the empty string. For booleans, the default value is false. For numeric types, the default value is zero. Note also that if you added a new repeated field, your new code will not be able to tell whether it was left empty (by new code) or never set at all (by old code) since there is no `has_` flag for it.

​	如果您遵循这些规则，旧代码将乐于读取新消息，并简单地忽略任何新字段。对于旧代码，已删除的可选字段将简单地具有其默认值，已删除的重复字段将为空。新代码也将透明地读取旧消息。但是，请记住，新可选字段不会出现在旧消息中，因此您需要使用 `has_` 明确检查它们是否已设置，或在 `.proto` 文件中使用 `[default = value]` 在字段编号后提供合理默认值。如果未为可选元素指定默认值，则使用特定于类型的默认值：对于字符串，默认值为空字符串。对于布尔值，默认值为 false。对于数字类型，默认值为零。另请注意，如果您添加了新的重复字段，您的新代码将无法判断它是被（新代码）留空还是根本未设置（由旧代码），因为没有 `has_` 标志。

## 优化提示 Optimization Tips 

The C++ Protocol Buffers library is extremely heavily optimized. However, proper usage can improve performance even more. Here are some tips for squeezing every last drop of speed out of the library:

​	C++ 协议缓冲区库经过了极度优化。但是，正确使用可以进一步提高性能。以下是一些从库中榨取最后一点速度的技巧：

- Reuse message objects when possible. Messages try to keep around any memory they allocate for reuse, even when they are cleared. Thus, if you are handling many messages with the same type and similar structure in succession, it is a good idea to reuse the same message object each time to take load off the memory allocator. However, objects can become bloated over time, especially if your messages vary in “shape” or if you occasionally construct a message that is much larger than usual. You should monitor the sizes of your message objects by calling the `SpaceUsed` method and delete them once they get too big.
- 尽可能重用消息对象。消息会尝试保留其为重用而分配的任何内存，即使它们被清除。因此，如果您连续处理许多具有相同类型和相似结构的消息，最好每次都重用相同的消息对象，以减轻内存分配器的负担。但是，随着时间的推移，对象可能会变得臃肿，尤其是如果您的消息“形状”各异，或者如果您偶尔构建一个比平时大得多的消息。您应该通过调用 `SpaceUsed` 方法来监视消息对象的大小，并在它们变得太大时删除它们。
- Your system’s memory allocator may not be well-optimized for allocating lots of small objects from multiple threads. Try using [Google’s TCMalloc](https://github.com/google/tcmalloc) instead.
- 您的系统的内存分配器可能无法很好地优化从多个线程分配大量小对象。尝试改用 Google 的 TCMalloc。

## 高级用法 Advanced Usage 

Protocol buffers have uses that go beyond simple accessors and serialization. Be sure to explore the [C++ API reference](https://protobuf.dev/reference/cpp) to see what else you can do with them.

​	协议缓冲区的使用超出了简单的访问器和序列化。务必浏览 C++ API 参考，以了解您可以用它们做什么。

One key feature provided by protocol message classes is *reflection*. You can iterate over the fields of a message and manipulate their values without writing your code against any specific message type. One very useful way to use reflection is for converting protocol messages to and from other encodings, such as XML or JSON. A more advanced use of reflection might be to find differences between two messages of the same type, or to develop a sort of “regular expressions for protocol messages” in which you can write expressions that match certain message contents. If you use your imagination, it’s possible to apply Protocol Buffers to a much wider range of problems than you might initially expect!

​	协议消息类提供的一个关键功能是反射。您可以迭代消息的字段并操作其值，而无需针对任何特定消息类型编写代码。使用反射的一种非常有用的方法是将协议消息转换为其他编码（例如 XML 或 JSON），或将协议消息从其他编码转换回来。反射的更高级用法可能是查找两个相同类型消息之间的差异，或开发一种“协议消息的正则表达式”，您可以在其中编写与某些消息内容匹配的表达式。如果您发挥想象力，就可以将 Protocol Buffers 应用到比您最初预期的更广泛的问题上！

Reflection is provided by the [`Message::Reflection` interface](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.message#Reflection).

​	反射由 `Message::Reflection` 接口提供。
