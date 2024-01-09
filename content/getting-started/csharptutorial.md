+++
title = "Protocol Buffer Basics: C#"
weight = 220
linkTitle = "C#"
description = "A basic C# programmers introduction to working with protocol buffers."
type = "docs"
+++

# Protocol Buffer Basics: C# 协议缓冲区基础知识：C#

A basic C# programmers introduction to working with protocol buffers.
C# 程序员基本介绍如何使用协议缓冲区。



This tutorial provides a basic C# programmer’s introduction to working with protocol buffers, using the [proto3](https://protobuf.dev/programming-guides/proto3) version of the protocol buffers language. By walking through creating a simple example application, it shows you how to
本教程为 C# 程序员提供使用协议缓冲区的基本介绍，使用协议缓冲区语言的 proto3 版本。通过逐步创建一个简单的示例应用程序，它向您展示如何

- Define message formats in a `.proto` file.
  在 `.proto` 文件中定义消息格式。
- Use the protocol buffer compiler.
  使用协议缓冲区编译器。
- Use the C# protocol buffer API to write and read messages.
  使用 C# 协议缓冲区 API 来编写和读取消息。

This isn’t a comprehensive guide to using protocol buffers in C#. For more detailed reference information, see the [Protocol Buffer Language Guide](https://protobuf.dev/programming-guides/proto3), the [C# API Reference](https://protobuf.dev/reference/csharp/api-docs), the [C# Generated Code Guide](https://protobuf.dev/reference/csharp/csharp-generated), and the [Encoding Reference](https://protobuf.dev/programming-guides/encoding).
这不是有关在 C# 中使用协议缓冲区的综合指南。有关更详细的参考信息，请参阅协议缓冲区语言指南、C# API 参考、C# 生成代码指南和编码参考。

## The Problem Domain 问题域

The example we’re going to use is a very simple “address book” application that can read and write people’s contact details to and from a file. Each person in the address book has a name, an ID, an email address, and a contact phone number.
我们将要使用的示例是一个非常简单的“通讯录”应用程序，它可以将人们的联系方式读写到文件。通讯录中的每个人都有一个姓名、一个 ID、一个电子邮件地址和一个联系电话号码。

How do you serialize and retrieve structured data like this? There are a few ways to solve this problem:
您如何序列化和检索这样的结构化数据？有几种方法可以解决此问题：

- Use .NET binary serialization with `System.Runtime.Serialization.Formatters.Binary.BinaryFormatter` and associated classes. This ends up being very fragile in the face of changes, expensive in terms of data size in some cases. It also doesn’t work very well if you need to share data with applications written for other platforms.
  使用 .NET 二进制序列化与 `System.Runtime.Serialization.Formatters.Binary.BinaryFormatter` 和相关类。在面对更改时，这最终变得非常脆弱，在某些情况下，数据大小方面代价高昂。如果您需要与为其他平台编写的应用程序共享数据，它也不太好用。
- You can invent an ad-hoc way to encode the data items into a single string – such as encoding 4 ints as “12:3:-23:67”. This is a simple and flexible approach, although it does require writing one-off encoding and parsing code, and the parsing imposes a small run-time cost. This works best for encoding very simple data.
  您可以发明一种临时方法将数据项编码为单个字符串——例如将 4 个整数编码为“12:3:-23:67”。这是一种简单而灵活的方法，尽管它确实需要编写一次性编码和解析代码，并且解析会产生少量运行时成本。这最适合编码非常简单的数据。
- Serialize the data to XML. This approach can be very attractive since XML is (sort of) human readable and there are binding libraries for lots of languages. This can be a good choice if you want to share data with other applications/projects. However, XML is notoriously space intensive, and encoding/decoding it can impose a huge performance penalty on applications. Also, navigating an XML DOM tree is considerably more complicated than navigating simple fields in a class normally would be.
  将数据序列化为 XML。这种方法可能非常有吸引力，因为 XML（某种程度上）是人类可读的，并且有很多语言的绑定库。如果您想与其他应用程序/项目共享数据，这可能是一个不错的选择。但是，XML 以空间密集而臭名昭著，并且对它的编码/解码可能会给应用程序带来巨大的性能损失。此外，导航 XML DOM 树比通常在类中导航简单字段要复杂得多。

Protocol buffers are the flexible, efficient, automated solution to solve exactly this problem. With protocol buffers, you write a `.proto` description of the data structure you wish to store. From that, the protocol buffer compiler creates a class that implements automatic encoding and parsing of the protocol buffer data with an efficient binary format. The generated class provides getters and setters for the fields that make up a protocol buffer and takes care of the details of reading and writing the protocol buffer as a unit. Importantly, the protocol buffer format supports the idea of extending the format over time in such a way that the code can still read data encoded with the old format.
协议缓冲区是解决此问题的灵活、高效、自动化的解决方案。使用协议缓冲区，您可以编写要存储的数据结构的 `.proto` 描述。由此，协议缓冲区编译器会创建一个类，该类实现协议缓冲区数据的自动编码和解析，并采用高效的二进制格式。生成的类提供获取器和设置器，用于构成协议缓冲区的字段，并负责以一个单元的形式读取和写入协议缓冲区的详细信息。重要的是，协议缓冲区格式支持随着时间的推移扩展格式的想法，这样代码仍然可以读取使用旧格式编码的数据。

## Where to Find the Example Code 在哪里找到示例代码

Our example is a command-line application for managing an address book data file, encoded using protocol buffers. The command `AddressBook` (see: [Program.cs](https://github.com/protocolbuffers/protobuf/blob/master/csharp/src/AddressBook/Program.cs)) can add a new entry to the data file or parse the data file and print the data to the console.
我们的示例是一个用于管理通讯录数据文件的命令行应用程序，该文件使用协议缓冲区进行编码。命令 `AddressBook` （请参阅：Program.cs）可以向数据文件添加新条目，或解析数据文件并将数据打印到控制台。

You can find the complete example in the [examples directory](https://github.com/protocolbuffers/protobuf/tree/master/examples) and [`csharp/src/AddressBook` directory](https://github.com/protocolbuffers/protobuf/tree/master/csharp/src/AddressBook) of the GitHub repository.
您可以在 GitHub 存储库的 examples 目录和 `csharp/src/AddressBook` 目录中找到完整示例。

## Defining Your Protocol Format 定义协议格式

To create your address book application, you’ll need to start with a `.proto` file. The definitions in a `.proto` file are simple: you add a *message* for each data structure you want to serialize, then specify a name and a type for each field in the message. In our example, the `.proto` file that defines the messages is [`addressbook.proto`](https://github.com/protocolbuffers/protobuf/blob/master/examples/addressbook.proto).
要创建您的地址簿应用程序，您需要从一个 `.proto` 文件开始。在 `.proto` 文件中的定义很简单：您为要序列化的每个数据结构添加一条消息，然后为消息中的每个字段指定一个名称和一个类型。在我们的示例中，定义消息的 `.proto` 文件是 `addressbook.proto` 。

The `.proto` file starts with a package declaration, which helps to prevent naming conflicts between different projects.
`.proto` 文件以包声明开头，这有助于防止不同项目之间的命名冲突。

```proto
syntax = "proto3";
package tutorial;

import "google/protobuf/timestamp.proto";
```

In C#, your generated classes will be placed in a namespace matching the `package` name if `csharp_namespace` is not specified. In our example, the `csharp_namespace` option has been specified to override the default, so the generated code uses a namespace of `Google.Protobuf.Examples.AddressBook` instead of `Tutorial`.
在 C# 中，如果未指定 `csharp_namespace` ，生成的类将被置于与 `package` 名称匹配的命名空间中。在我们的示例中，已指定 `csharp_namespace` 选项以覆盖默认值，因此生成的代码使用 `Google.Protobuf.Examples.AddressBook` 命名空间，而不是 `Tutorial` 。

```csharp
option csharp_namespace = "Google.Protobuf.Examples.AddressBook";
```

Next, you have your message definitions. A message is just an aggregate containing a set of typed fields. Many standard simple data types are available as field types, including `bool`, `int32`, `float`, `double`, and `string`. You can also add further structure to your messages by using other message types as field types.
接下来，您有您的消息定义。消息只是一个包含一组类型化字段的聚合。许多标准的简单数据类型可作为字段类型使用，包括 `bool` 、 `int32` 、 `float` 、 `double` 和 `string` 。您还可以通过使用其他消息类型作为字段类型来为您的消息添加更多结构。

```proto
message Person {
  string name = 1;
  int32 id = 2;  // Unique ID number for this person.
  string email = 3;

  enum PhoneType {
    PHONE_TYPE_UNSPECIFIED = 0;
    PHONE_TYPE_MOBILE = 1;
    PHONE_TYPE_HOME = 2;
    PHONE_TYPE_WORK = 3;
  }

  message PhoneNumber {
    string number = 1;
    PhoneType type = 2;
  }

  repeated PhoneNumber phones = 4;

  google.protobuf.Timestamp last_updated = 5;
}

// Our address book file is just one of these.
message AddressBook {
  repeated Person people = 1;
}
```

In the above example, the `Person` message contains `PhoneNumber` messages, while the `AddressBook` message contains `Person` messages. You can even define message types nested inside other messages – as you can see, the `PhoneNumber` type is defined inside `Person`. You can also define `enum` types if you want one of your fields to have one of a predefined list of values – here you want to specify that a phone number can be one of `PHONE_TYPE_MOBILE`, `PHONE_TYPE_HOME`, or `PHONE_TYPE_WORK`.
在上面的示例中， `Person` 消息包含 `PhoneNumber` 消息，而 `AddressBook` 消息包含 `Person` 消息。您甚至可以定义嵌套在其他消息内部的消息类型——如您所见， `PhoneNumber` 类型定义在 `Person` 内部。如果您希望某个字段具有预定义值列表中的某个值，还可以定义 `enum` 类型——此处您要指定电话号码可以是 `PHONE_TYPE_MOBILE` 、 `PHONE_TYPE_HOME` 或 `PHONE_TYPE_WORK` 之一。

The " = 1", " = 2" markers on each element identify the unique “tag” that field uses in the binary encoding. Tag numbers 1-15 require one less byte to encode than higher numbers, so as an optimization you can decide to use those tags for the commonly used or repeated elements, leaving tags 16 and higher for less-commonly used optional elements. Each element in a repeated field requires re-encoding the tag number, so repeated fields are particularly good candidates for this optimization.
每个元素上的“= 1”、“= 2”标记标识该字段在二进制编码中使用的唯一“标记”。标记编号 1-15 比较高的编号编码时少一个字节，因此作为优化，您可以决定将这些标记用于常用或重复的元素，将标记 16 及更高编号留给不常用的可选元素。重复字段中的每个元素都需要重新编码标记编号，因此重复字段特别适合这种优化。

If a field value isn’t set, a [default value](https://protobuf.dev/programming-guides/proto3#default) is used: zero for numeric types, the empty string for strings, false for bools. For embedded messages, the default value is always the “default instance” or “prototype” of the message, which has none of its fields set. Calling the accessor to get the value of a field which has not been explicitly set always returns that field’s default value.
如果未设置字段值，则使用默认值：对于数字类型为零，对于字符串为空字符串，对于布尔值为 false。对于嵌入式消息，默认值始终是消息的“默认实例”或“原型”，其没有任何已设置的字段。调用访问器以获取尚未显式设置的字段的值时，始终返回该字段的默认值。

If a field is `repeated`, the field may be repeated any number of times (including zero). The order of the repeated values will be preserved in the protocol buffer. Think of repeated fields as dynamically sized arrays.
如果字段是 `repeated` ，则该字段可以重复任意多次（包括零次）。重复值在协议缓冲区中的顺序将被保留。将重复字段视为动态大小的数组。

You’ll find a complete guide to writing `.proto` files – including all the possible field types – in the [Protocol Buffer Language Guide](https://protobuf.dev/programming-guides/proto3). Don’t go looking for facilities similar to class inheritance, though – protocol buffers don’t do that.
您可以在协议缓冲区语言指南中找到编写 `.proto` 文件的完整指南，包括所有可能的字段类型。不过，不要寻找类似于类继承的功能，协议缓冲区不具备该功能。

## Compiling Your Protocol Buffers 编译您的协议缓冲区

Now that you have a `.proto`, the next thing you need to do is generate the classes you’ll need to read and write `AddressBook` (and hence `Person` and `PhoneNumber`) messages. To do this, you need to run the protocol buffer compiler `protoc` on your `.proto`:
现在您已拥有 `.proto` ，接下来需要做的是生成您需要用来读写 `AddressBook` （进而读写 `Person` 和 `PhoneNumber` ）消息的类。为此，您需要在 `.proto` 上运行协议缓冲区编译器 `protoc` ：

1. If you haven’t installed the compiler, [download the package](https://protobuf.dev/downloads) and follow the instructions in the README.
   如果您尚未安装编译器，请下载软件包并按照自述文件中的说明进行操作。

2. Now run the compiler, specifying the source directory (where your application’s source code lives – the current directory is used if you don’t provide a value), the destination directory (where you want the generated code to go; often the same as `$SRC_DIR`), and the path to your `.proto`. In this case, you would invoke:
   现在运行编译器，指定源目录（您的应用程序源代码所在的位置 - 如果您未提供值，则使用当前目录）、目标目录（您希望生成的代码所在的位置；通常与 `$SRC_DIR` 相同）和 `.proto` 的路径。在这种情况下，您将调用：

   ```shell
   protoc -I=$SRC_DIR --csharp_out=$DST_DIR $SRC_DIR/addressbook.proto
   ```

   Because you want C# code, you use the `--csharp_out` option – similar options are provided for other supported languages.
   因为您想要 C# 代码，所以您使用 `--csharp_out` 选项 - 为其他受支持语言提供了类似的选项。

This generates `Addressbook.cs` in your specified destination directory. To compile this code, you’ll need a project with a reference to the `Google.Protobuf` assembly.
这会在您指定的目标目录中生成 `Addressbook.cs` 。要编译此代码，您需要一个引用 `Google.Protobuf` 程序集的项目。

## The Addressbook Classes 通讯录类

Generating `Addressbook.cs` gives you five useful types:
生成 `Addressbook.cs` 会为您提供五种有用的类型：

- A static `Addressbook` class that contains metadata about the protocol buffer messages.
  一个包含有关协议缓冲区消息的元数据的静态 `Addressbook` 类。
- An `AddressBook` class with a read-only `People` property.
  一个具有只读 `People` 属性的 `AddressBook` 类。
- A `Person` class with properties for `Name`, `Id`, `Email` and `Phones`.
  一个 `Person` 类，其中包含 `Name` 、 `Id` 、 `Email` 和 `Phones` 的属性。
- A `PhoneNumber` class, nested in a static `Person.Types` class.
  一个 `PhoneNumber` 类，嵌套在一个静态 `Person.Types` 类中。
- A `PhoneType` enum, also nested in `Person.Types`.
  一个 `PhoneType` 枚举，也嵌套在 `Person.Types` 中。

You can read more about the details of exactly what’s generated in the [C# Generated Code guide](https://protobuf.dev/reference/csharp/csharp-generated), but for the most part you can treat these as perfectly ordinary C# types. One point to highlight is that any properties corresponding to repeated fields are read-only. You can add items to the collection or remove items from it, but you can’t replace it with an entirely separate collection. The collection type for repeated fields is always `RepeatedField<T>`. This type is like `List<T>` but with a few extra convenience methods, such as an `Add` overload accepting a collection of items, for use in collection initializers.
您可以在 C# 生成的代码指南中阅读有关确切生成内容的详细信息，但大多数情况下，您可以将这些视为完全普通的 C# 类型。需要强调的一点是，任何对应于重复字段的属性都是只读的。您可以向集合中添加项或从中删除项，但不能用一个完全独立的集合替换它。重复字段的集合类型始终是 `RepeatedField<T>` 。此类型类似于 `List<T>` ，但有一些额外的便捷方法，例如接受项集合的 `Add` 重载，用于集合初始化器。

Here’s an example of how you might create an instance of Person:
以下是如何创建 Person 实例的示例：

```csharp
Person john = new Person
{
    Id = 1234,
    Name = "John Doe",
    Email = "jdoe@example.com",
    Phones = { new Person.Types.PhoneNumber { Number = "555-4321", Type = Person.Types.PhoneType.Home } }
};
```

Note that with C# 6, you can use `using static` to remove the `Person.Types` ugliness:
请注意，使用 C# 6，您可以使用 `using static` 删除 `Person.Types` 的丑陋：

```csharp
// Add this to the other using directives
using static Google.Protobuf.Examples.AddressBook.Person.Types;
...
// The earlier Phones assignment can now be simplified to:
Phones = { new PhoneNumber { Number = "555-4321", Type = PhoneType.HOME } }
```

## Parsing and Serialization 解析和序列化

The whole purpose of using protocol buffers is to serialize your data so that it can be parsed elsewhere. Every generated class has a `WriteTo(CodedOutputStream)` method, where `CodedOutputStream` is a class in the protocol buffer runtime library. However, usually you’ll use one of the extension methods to write to a regular `System.IO.Stream` or convert the message to a byte array or `ByteString`. These extension messages are in the `Google.Protobuf.MessageExtensions` class, so when you want to serialize you’ll usually want a `using` directive for the `Google.Protobuf` namespace. For example:
使用协议缓冲区的全部目的是序列化您的数据，以便可以在其他地方解析它。每个生成的类都有一个 `WriteTo(CodedOutputStream)` 方法，其中 `CodedOutputStream` 是协议缓冲区运行时库中的一个类。但是，通常您会使用其中一个扩展方法写入常规 `System.IO.Stream` 或将消息转换为字节数组或 `ByteString` 。这些扩展消息位于 `Google.Protobuf.MessageExtensions` 类中，因此当您想要序列化时，通常需要一个 `using` 指令用于 `Google.Protobuf` 命名空间。例如：

```csharp
using Google.Protobuf;
...
Person john = ...; // Code as before
using (var output = File.Create("john.dat"))
{
    john.WriteTo(output);
}
```

Parsing is also simple. Each generated class has a static `Parser` property which returns a `MessageParser<T>` for that type. That in turn has methods to parse streams, byte arrays and `ByteString`s. So to parse the file we’ve just created, we can use:
解析也很简单。每个生成的类都有一个静态 `Parser` 属性，该属性返回该类型的 `MessageParser<T>` 。它反过来具有解析流、字节数组和 `ByteString` 的方法。因此，要解析我们刚刚创建的文件，我们可以使用：

```csharp
Person john;
using (var input = File.OpenRead("john.dat"))
{
    john = Person.Parser.ParseFrom(input);
}
```

A full example program to maintain an addressbook (adding new entries and listing existing ones) using these messages is available [in the Github repository](https://github.com/protocolbuffers/protobuf/tree/master/csharp/src/AddressBook).
可以在 Github 存储库中找到一个完整的示例程序，该程序使用这些消息维护通讯录（添加新条目并列出现有条目）。

## Extending a Protocol Buffer 扩展协议缓冲区

Sooner or later after you release the code that uses your protocol buffer, you will undoubtedly want to “improve” the protocol buffer’s definition. If you want your new buffers to be backwards-compatible, and your old buffers to be forward-compatible – and you almost certainly do want this – then there are some rules you need to follow. In the new version of the protocol buffer:
在您发布使用协议缓冲区的代码后，迟早您无疑会想要“改进”协议缓冲区的定义。如果您希望新缓冲区向后兼容，旧缓冲区向前兼容——您几乎肯定希望如此——那么您需要遵循一些规则。在协议缓冲区的新版本中：

- you *must not* change the tag numbers of any existing fields.
  您不得更改任何现有字段的标记号。
- you *may* delete fields.
  您可以删除字段。
- you *may* add new fields but you must use fresh tag numbers (i.e. tag numbers that were never used in this protocol buffer, not even by deleted fields).
  您可以添加新字段，但您必须使用新的标记号（即此协议缓冲区中从未使用过的标记号，即使是已删除的字段也不行）。

(There are [some exceptions](https://protobuf.dev/programming-guides/proto3#updating) to these rules, but they are rarely used.)
（这些规则有一些例外，但很少使用。）

If you follow these rules, old code will happily read new messages and simply ignore any new fields. To the old code, singular fields that were deleted will simply have their default value, and deleted repeated fields will be empty. New code will also transparently read old messages.
如果您遵循这些规则，旧代码将乐于读取新消息，并简单地忽略任何新字段。对于旧代码，已删除的单数字段将简单地具有其默认值，已删除的重复字段将为空。新代码也将透明地读取旧消息。

However, keep in mind that new fields will not be present in old messages, so you will need to do something reasonable with the default value. A type-specific [default value](https://protobuf.dev/programming-guides/proto3#default) is used: for strings, the default value is the empty string. For booleans, the default value is false. For numeric types, the default value is zero.
但是，请记住，新字段不会出现在旧消息中，因此您需要对默认值执行一些合理的操作。使用特定于类型的默认值：对于字符串，默认值为空字符串。对于布尔值，默认值为 false。对于数字类型，默认值为零。

## Reflection 反射

Message descriptors (the information in the `.proto` file) and instances of messages can be examined programmatically using the reflection API. This can be useful when writing generic code such as a different text format or a smart diff tool. Each generated class has a static `Descriptor` property, and the descriptor for any instance can be retrieved using the `IMessage.Descriptor` property. As a quick example of how these can be used, here is a short method to print the top-level fields of any message.
消息描述符（ `.proto` 文件中的信息）和消息实例可以使用反射 API 以编程方式进行检查。这在编写通用代码（例如不同的文本格式或智能差异工具）时非常有用。每个生成的类都有一个静态 `Descriptor` 属性，并且可以使用 `IMessage.Descriptor` 属性检索任何实例的描述符。作为一个快速示例，说明如何使用这些属性，这里有一个简短的方法来打印任何消息的顶级字段。

```csharp
public void PrintMessage(IMessage message)
{
    var descriptor = message.Descriptor;
    foreach (var field in descriptor.Fields.InDeclarationOrder())
    {
        Console.WriteLine(
            "Field {0} ({1}): {2}",
            field.FieldNumber,
            field.Name,
            field.Accessor.GetValue(message);
    }
}
```
