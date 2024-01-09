+++
title = "Protocol Buffer Basics: Python"
weight = 270
linkTitle = "Python"
description = "A basic Python programmers introduction to working with protocol buffers."
type = "docs"
+++

> 原文网址： https://protobuf.dev/getting-started/pythontutorial/

# Protocol Buffer Basics: Python 协议缓冲区基础知识：Python

A basic Python programmers introduction to working with protocol buffers.
Python 程序员基本入门，了解如何使用协议缓冲区。



This tutorial provides a basic Python programmer’s introduction to working with protocol buffers. By walking through creating a simple example application, it shows you how to
本教程为 Python 程序员提供了使用协议缓冲区的基本入门知识。通过逐步创建一个简单的示例应用程序，它向您展示了如何

- Define message formats in a `.proto` file.
  在 `.proto` 文件中定义消息格式。
- Use the protocol buffer compiler.
  使用协议缓冲区编译器。
- Use the Python protocol buffer API to write and read messages.
  使用 Python 协议缓冲区 API 来编写和读取消息。

This isn’t a comprehensive guide to using protocol buffers in Python. For more detailed reference information, see the [Protocol Buffer Language Guide (proto2)](https://protobuf.dev/programming-guides/proto), the [Protocol Buffer Language Guide (proto3)](https://protobuf.dev/programming-guides/proto3), the [Python API Reference](https://googleapis.dev/python/protobuf/latest/), the [Python Generated Code Guide](https://protobuf.dev/reference/python/python-generated), and the [Encoding Reference](https://protobuf.dev/programming-guides/encoding).
这不是使用 Python 中的协议缓冲区的综合指南。有关更详细的参考信息，请参阅协议缓冲区语言指南 (proto2)、协议缓冲区语言指南 (proto3)、Python API 参考、Python 生成代码指南和编码参考。

## The Problem Domain 问题域

The example we’re going to use is a very simple “address book” application that can read and write people’s contact details to and from a file. Each person in the address book has a name, an ID, an email address, and a contact phone number.
我们将要使用的示例是一个非常简单的“通讯录”应用程序，它可以将人们的联系方式读写到文件。通讯录中的每个人都有一个姓名、一个 ID、一个电子邮件地址和一个联系电话号码。

How do you serialize and retrieve structured data like this? There are a few ways to solve this problem:
您如何序列化和检索这样的结构化数据？有几种方法可以解决此问题：

- Use Python pickling. This is the default approach since it’s built into the language, but it doesn’t deal well with schema evolution, and also doesn’t work very well if you need to share data with applications written in C++ or Java.
  使用 Python pickle。这是默认方法，因为它内置于语言中，但它不能很好地处理模式演变，而且如果您需要与用 C++ 或 Java 编写的应用程序共享数据，它也不能很好地工作。
- You can invent an ad-hoc way to encode the data items into a single string – such as encoding 4 ints as “12:3:-23:67”. This is a simple and flexible approach, although it does require writing one-off encoding and parsing code, and the parsing imposes a small run-time cost. This works best for encoding very simple data.
  您可以发明一种临时方法将数据项编码为单个字符串——例如将 4 个整数编码为“12:3:-23:67”。这是一种简单而灵活的方法，尽管它确实需要编写一次性编码和解析代码，并且解析会产生少量运行时成本。这最适合编码非常简单的数据。
- Serialize the data to XML. This approach can be very attractive since XML is (sort of) human readable and there are binding libraries for lots of languages. This can be a good choice if you want to share data with other applications/projects. However, XML is notoriously space intensive, and encoding/decoding it can impose a huge performance penalty on applications. Also, navigating an XML DOM tree is considerably more complicated than navigating simple fields in a class normally would be.
  将数据序列化为 XML。这种方法可能非常有吸引力，因为 XML（某种程度上）是人类可读的，并且有很多语言的绑定库。如果您想与其他应用程序/项目共享数据，这可能是一个不错的选择。但是，XML 以空间密集而臭名昭著，并且对它的编码/解码可能会给应用程序带来巨大的性能损失。此外，导航 XML DOM 树比通常在类中导航简单字段要复杂得多。

Instead of these options, you can use protocol buffers. Protocol buffers are the flexible, efficient, automated solution to solve exactly this problem. With protocol buffers, you write a `.proto` description of the data structure you wish to store. From that, the protocol buffer compiler creates a class that implements automatic encoding and parsing of the protocol buffer data with an efficient binary format. The generated class provides getters and setters for the fields that make up a protocol buffer and takes care of the details of reading and writing the protocol buffer as a unit. Importantly, the protocol buffer format supports the idea of extending the format over time in such a way that the code can still read data encoded with the old format.
除了这些选项，您还可以使用协议缓冲区。协议缓冲区是解决此问题的灵活、高效、自动化的解决方案。使用协议缓冲区，您可以编写一个 `.proto` 描述您想要存储的数据结构。由此，协议缓冲区编译器会创建一个类，该类实现协议缓冲区数据的自动编码和解析，并采用高效的二进制格式。生成的类提供获取器和设置器，用于构成协议缓冲区的字段，并负责以一个单元的形式读取和写入协议缓冲区的详细信息。重要的是，协议缓冲区格式支持随着时间的推移扩展格式的想法，这样代码仍然可以读取使用旧格式编码的数据。

## Where to Find the Example Code 在哪里找到示例代码

The example code is included in the source code package, under the “examples” directory. [Download it here.](https://protobuf.dev/downloads)
示例代码包含在源代码包中，“示例”目录下。在此处下载。

## Defining Your Protocol Format 定义协议格式

To create your address book application, you’ll need to start with a `.proto` file. The definitions in a `.proto` file are simple: you add a *message* for each data structure you want to serialize, then specify a name and a type for each field in the message. Here is the `.proto` file that defines your messages, `addressbook.proto`.
要创建您的通讯录应用程序，您需要从 `.proto` 文件开始。在 `.proto` 文件中的定义很简单：为要序列化的每个数据结构添加一条消息，然后为消息中的每个字段指定一个名称和类型。以下是定义您的消息的 `.proto` 文件， `addressbook.proto` 。

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
正如您所见，语法类似于 C++ 或 Java。让我们逐个部分地了解该文件，看看它做了什么。

The `.proto` file starts with a package declaration, which helps to prevent naming conflicts between different projects. In Python, packages are normally determined by directory structure, so the `package` you define in your `.proto` file will have no effect on the generated code. However, you should still declare one to avoid name collisions in the Protocol Buffers name space as well as in non-Python languages.
文件以包声明开头，这有助于防止不同项目之间的命名冲突。在 Python 中，包通常由目录结构确定，因此您在 `.proto` 文件中定义的 `package` 对生成的代码没有任何影响。但是，您仍应声明一个，以避免在 Protocol Buffers 名称空间以及非 Python 语言中发生名称冲突。

Next, you have your message definitions. A message is just an aggregate containing a set of typed fields. Many standard simple data types are available as field types, including `bool`, `int32`, `float`, `double`, and `string`. You can also add further structure to your messages by using other message types as field types – in the above example the `Person` message contains `PhoneNumber` messages, while the `AddressBook` message contains `Person` messages. You can even define message types nested inside other messages – as you can see, the `PhoneNumber` type is defined inside `Person`. You can also define `enum` types if you want one of your fields to have one of a predefined list of values – here you want to specify that a phone number can be one of the following phone types: `PHONE_TYPE_MOBILE`, `PHONE_TYPE_HOME`, or `PHONE_TYPE_WORK`.
接下来，您有您的消息定义。消息只是一个包含一组类型化字段的聚合。许多标准的简单数据类型可用作字段类型，包括 `bool` 、 `int32` 、 `float` 、 `double` 和 `string` 。您还可以通过使用其他消息类型作为字段类型来为您的消息添加更多结构——在上面的示例中， `Person` 消息包含 `PhoneNumber` 消息，而 `AddressBook` 消息包含 `Person` 消息。您甚至可以定义嵌套在其他消息内部的消息类型——如您所见， `PhoneNumber` 类型定义在 `Person` 内部。如果您希望您的某个字段具有预定义值列表中的一个值，您还可以定义 `enum` 类型——在这里，您要指定电话号码可以是以下电话类型之一： `PHONE_TYPE_MOBILE` 、 `PHONE_TYPE_HOME` 或 `PHONE_TYPE_WORK` 。

The " = 1", " = 2" markers on each element identify the unique “tag” that field uses in the binary encoding. Tag numbers 1-15 require one less byte to encode than higher numbers, so as an optimization you can decide to use those tags for the commonly used or repeated elements, leaving tags 16 and higher for less-commonly used optional elements. Each element in a repeated field requires re-encoding the tag number, so repeated fields are particularly good candidates for this optimization.
每个元素上的“= 1”、“= 2”标记标识该字段在二进制编码中使用的唯一“标记”。标记编号 1-15 比较高的编号编码时少一个字节，因此作为优化，您可以决定将这些标记用于常用或重复的元素，将标记 16 及更高编号留给不常用的可选元素。重复字段中的每个元素都需要重新编码标记编号，因此重复字段特别适合这种优化。

Each field must be annotated with one of the following modifiers:
每个字段都必须使用以下修饰符之一进行注释：

- `optional`: the field may or may not be set. If an optional field value isn’t set, a default value is used. For simple types, you can specify your own default value, as we’ve done for the phone number `type` in the example. Otherwise, a system default is used: zero for numeric types, the empty string for strings, false for bools. For embedded messages, the default value is always the “default instance” or “prototype” of the message, which has none of its fields set. Calling the accessor to get the value of an optional (or required) field which has not been explicitly set always returns that field’s default value.
  `optional` ：该字段可以设置，也可以不设置。如果未设置可选字段值，则使用默认值。对于简单类型，您可以指定自己的默认值，就像我们在示例中为电话号码 `type` 所做的那样。否则，将使用系统默认值：对于数字类型为零，对于字符串为空字符串，对于布尔值为 false。对于嵌入式消息，默认值始终是消息的“默认实例”或“原型”，其没有任何字段设置。调用访问器以获取尚未显式设置的可选（或必需）字段的值时，始终返回该字段的默认值。
- `repeated`: the field may be repeated any number of times (including zero). The order of the repeated values will be preserved in the protocol buffer. Think of repeated fields as dynamically sized arrays.
  `repeated` ：该字段可以重复任意多次（包括零次）。重复值在协议缓冲区中的顺序将被保留。将重复字段视为动态大小的数组。
- `required`: a value for the field must be provided, otherwise the message will be considered “uninitialized”. Serializing an uninitialized message will raise an exception. Parsing an uninitialized message will fail. Other than this, a required field behaves exactly like an optional field.
  `required` ：必须提供字段的值，否则该消息将被视为“未初始化”。序列化未初始化的消息将引发异常。解析未初始化的消息将失败。除此之外，必需字段的行为与可选字段完全相同。

#### Important 重要

**Required Is Forever
必需永远是必需的** You should be very careful about marking fields as `required`. If at some point you wish to stop writing or sending a required field, it will be problematic to change the field to an optional field – old readers will consider messages without this field to be incomplete and may reject or drop them unintentionally. You should consider writing application-specific custom validation routines for your buffers instead. Within Google, `required` fields are strongly disfavored; most messages defined in proto2 syntax use `optional` and `repeated` only. (Proto3 does not support `required` fields at all.)
您应该非常小心地将字段标记为 `required` 。如果您在某个时刻希望停止编写或发送必填字段，将字段更改为可选字段会很麻烦——旧读者会认为没有此字段的消息不完整，可能会拒绝或无意中丢弃它们。您应该考虑为缓冲区编写特定于应用程序的自定义验证例程。在 Google 内部， `required` 字段非常不受欢迎；大多数在 proto2 语法中定义的消息仅使用 `optional` 和 `repeated` 。（Proto3 完全不支持 `required` 字段。）

You’ll find a complete guide to writing `.proto` files – including all the possible field types – in the [Protocol Buffer Language Guide](https://protobuf.dev/programming-guides/proto). Don’t go looking for facilities similar to class inheritance, though – protocol buffers don’t do that.
您可以在协议缓冲区语言指南中找到编写 `.proto` 文件的完整指南，包括所有可能的字段类型。不过，不要寻找类似于类继承的功能，协议缓冲区不具备该功能。

## Compiling Your Protocol Buffers 编译您的协议缓冲区

Now that you have a `.proto`, the next thing you need to do is generate the classes you’ll need to read and write `AddressBook` (and hence `Person` and `PhoneNumber`) messages. To do this, you need to run the protocol buffer compiler `protoc` on your `.proto`:
现在您已拥有 `.proto` ，接下来需要做的是生成您需要用来读写 `AddressBook` （进而读写 `Person` 和 `PhoneNumber` ）消息的类。为此，您需要在 `.proto` 上运行协议缓冲区编译器 `protoc` ：

1. If you haven’t installed the compiler, [download the package](https://protobuf.dev/downloads) and follow the instructions in the README.
   如果您尚未安装编译器，请下载软件包并按照自述文件中的说明进行操作。

2. Now run the compiler, specifying the source directory (where your application’s source code lives – the current directory is used if you don’t provide a value), the destination directory (where you want the generated code to go; often the same as `$SRC_DIR`), and the path to your `.proto`. In this case, you…:
   现在运行编译器，指定源目录（您的应用程序源代码所在的位置——如果您未提供值，则使用当前目录）、目标目录（您希望生成的代码所在的位置；通常与 `$SRC_DIR` 相同）以及 `.proto` 的路径。在这种情况下，您…：

   ```shell
   protoc -I=$SRC_DIR --python_out=$DST_DIR $SRC_DIR/addressbook.proto
   ```

   Because you want Python classes, you use the `--python_out` option – similar options are provided for other supported languages.
   因为您想要 Python 类，所以您使用 `--python_out` 选项——为其他受支持的语言提供了类似的选项。

This generates `addressbook_pb2.py` in your specified destination directory.
这会在您指定的目标目录中生成 `addressbook_pb2.py` 。

## The Protocol Buffer API 协议缓冲区 API

Unlike when you generate Java and C++ protocol buffer code, the Python protocol buffer compiler doesn’t generate your data access code for you directly. Instead (as you’ll see if you look at `addressbook_pb2.py`) it generates special descriptors for all your messages, enums, and fields, and some mysteriously empty classes, one for each message type:
与生成 Java 和 C++ 协议缓冲区代码不同，Python 协议缓冲区编译器不会直接为您生成数据访问代码。相反（如果您查看 `addressbook_pb2.py` ，您会看到），它会为所有消息、枚举和字段生成特殊描述符，以及一些神秘的空类，每个消息类型一个：

```python
class Person(message.Message):
  __metaclass__ = reflection.GeneratedProtocolMessageType

  class PhoneNumber(message.Message):
    __metaclass__ = reflection.GeneratedProtocolMessageType
    DESCRIPTOR = _PERSON_PHONENUMBER
  DESCRIPTOR = _PERSON

class AddressBook(message.Message):
  __metaclass__ = reflection.GeneratedProtocolMessageType
  DESCRIPTOR = _ADDRESSBOOK
```

The important line in each class is `__metaclass__ = reflection.GeneratedProtocolMessageType`. While the details of how Python metaclasses work is beyond the scope of this tutorial, you can think of them as like a template for creating classes. At load time, the `GeneratedProtocolMessageType` metaclass uses the specified descriptors to create all the Python methods you need to work with each message type and adds them to the relevant classes. You can then use the fully-populated classes in your code.
每个类中的重要行是 `__metaclass__ = reflection.GeneratedProtocolMessageType` 。虽然 Python 元类的详细工作原理超出了本教程的范围，但您可以将它们视为创建类的模板。在加载时， `GeneratedProtocolMessageType` 元类使用指定的描述符创建您处理每种消息类型所需的所有 Python 方法，并将它们添加到相关类中。然后，您可以在代码中使用填充完整的类。

The end effect of all this is that you can use the `Person` class as if it defined each field of the `Message` base class as a regular field. For example, you could write:
所有这一切的最终效果是，您可以使用 `Person` 类，就好像它将 `Message` 基类的每个字段定义为常规字段一样。例如，您可以编写：

```python
import addressbook_pb2
person = addressbook_pb2.Person()
person.id = 1234
person.name = "John Doe"
person.email = "jdoe@example.com"
phone = person.phones.add()
phone.number = "555-4321"
phone.type = addressbook_pb2.Person.PHONE_TYPE_HOME
```

Note that these assignments are not just adding arbitrary new fields to a generic Python object. If you were to try to assign a field that isn’t defined in the `.proto` file, an `AttributeError` would be raised. If you assign a field to a value of the wrong type, a `TypeError` will be raised. Also, reading the value of a field before it has been set returns the default value.
请注意，这些赋值不仅仅是向通用 Python 对象添加任意新字段。如果您尝试赋值一个未在 `.proto` 文件中定义的字段，则会引发 `AttributeError` 。如果您将字段赋值为错误类型的某个值，则会引发 `TypeError` 。此外，在设置字段之前读取字段的值会返回默认值。

```python
person.no_such_field = 1  # raises AttributeError
person.id = "1234"        # raises TypeError
```

For more information on exactly what members the protocol compiler generates for any particular field definition, see the [Python generated code reference](https://protobuf.dev/reference/python/python-generated).
有关协议编译器为任何特定字段定义生成的成员的详细信息，请参阅 Python 生成的代码参考。

### Enums 枚举

Enums are expanded by the metaclass into a set of symbolic constants with integer values. So, for example, the constant `addressbook_pb2.Person.PhoneType.PHONE_TYPE_WORK` has the value 2.
元类将枚举扩展为一组具有整数值的符号常量。因此，例如，常量 `addressbook_pb2.Person.PhoneType.PHONE_TYPE_WORK` 的值为 2。

### Standard Message Methods 标准消息方法

Each message class also contains a number of other methods that let you check or manipulate the entire message, including:
每个消息类还包含许多其他方法，可让您检查或操作整个消息，包括：

- `IsInitialized()`: checks if all the required fields have been set.
  `IsInitialized()` ：检查是否已设置所有必需字段。
- `__str__()`: returns a human-readable representation of the message, particularly useful for debugging. (Usually invoked as `str(message)` or `print message`.)
  `__str__()` ：返回消息的人类可读表示形式，这对于调试特别有用。（通常调用为 `str(message)` 或 `print message` 。）
- `CopyFrom(other_msg)`: overwrites the message with the given message’s values.
  `CopyFrom(other_msg)` ：使用给定消息的值覆盖消息。
- `Clear()`: clears all the elements back to the empty state.
  `Clear()` ：将所有元素清除回空状态。

These methods implement the `Message` interface. For more information, see the [complete API documentation for `Message`](https://googleapis.dev/python/protobuf/latest/google/protobuf/message.html#google.protobuf.message.Message).
这些方法实现了 `Message` 接口。有关更多信息，请参阅 `Message` 的完整 API 文档。

### Parsing and Serialization 解析和序列化

Finally, each protocol buffer class has methods for writing and reading messages of your chosen type using the protocol buffer [binary format](https://protobuf.dev/programming-guides/encoding). These include:
最后，每个协议缓冲区类都有用于使用协议缓冲区二进制格式写入和读取您选择类型的消息的方法。其中包括：

- `SerializeToString()`: serializes the message and returns it as a string. Note that the bytes are binary, not text; we only use the `str` type as a convenient container.
  `SerializeToString()` ：序列化消息并将其作为字符串返回。请注意，字节是二进制的，不是文本；我们仅将 `str` 类型用作方便的容器。
- `ParseFromString(data)`: parses a message from the given string.
  `ParseFromString(data)` ：从给定的字符串中解析消息。

These are just a couple of the options provided for parsing and serialization. Again, see the [`Message` API reference](https://googleapis.dev/python/protobuf/latest/google/protobuf/message.html#google.protobuf.message.Message) for a complete list.
这些只是为解析和序列化提供的几个选项。同样，请参阅 `Message` API 参考以获取完整列表。

#### Important 重要

**Protocol Buffers and Object Oriented Design
协议缓冲区和面向对象设计** Protocol buffer classes are basically data holders (like structs in C) that don’t provide additional functionality; they don’t make good first class citizens in an object model. If you want to add richer behavior to a generated class, the best way to do this is to wrap the generated protocol buffer class in an application-specific class. Wrapping protocol buffers is also a good idea if you don’t have control over the design of the `.proto` file (if, say, you’re reusing one from another project). In that case, you can use the wrapper class to craft an interface better suited to the unique environment of your application: hiding some data and methods, exposing convenience functions, etc. **You should never add behavior to the generated classes by inheriting from them**. This will break internal mechanisms and is not good object-oriented practice anyway.
协议缓冲区类基本上是数据持有者（类似于 C 中的结构），不提供其他功能；它们在对象模型中不是好的第一类公民。如果您想向生成的类添加更丰富的行为，最好的方法是将生成的协议缓冲区类包装在特定于应用程序的类中。如果您无法控制 `.proto` 文件的设计（例如，如果您正在从另一个项目中重用一个文件），那么包装协议缓冲区也是一个好主意。在这种情况下，您可以使用包装器类来构建更适合您应用程序的独特环境的接口：隐藏一些数据和方法，公开便利函数等。您绝不应通过继承生成类来向它们添加行为。这会破坏内部机制，而且无论如何也不是好的面向对象实践。

## Writing a Message 编写消息

Now let’s try using your protocol buffer classes. The first thing you want your address book application to be able to do is write personal details to your address book file. To do this, you need to create and populate instances of your protocol buffer classes and then write them to an output stream.
现在，我们尝试使用您的协议缓冲区类。您希望通讯录应用程序能够做的第一件事是将个人详细信息写入通讯录文件。为此，您需要创建和填充协议缓冲区类的实例，然后将它们写入输出流。

Here is a program which reads an `AddressBook` from a file, adds one new `Person` to it based on user input, and writes the new `AddressBook` back out to the file again. The parts which directly call or reference code generated by the protocol compiler are highlighted.
这是一个从文件中读取 `AddressBook` 、根据用户输入向其中添加一个新的 `Person` ，然后将新的 `AddressBook` 再次写入文件的程序。直接调用或引用协议编译器生成的代码的部分已突出显示。

```python
#!/usr/bin/env python3

import addressbook_pb2
import sys

# This function fills in a Person message based on user input.
def PromptForAddress(person):
  person.id = int(input("Enter person ID number: "))
  person.name = input("Enter name: ")

  email = input("Enter email address (blank for none): ")
  if email != "":
    person.email = email

  while True:
    number = input("Enter a phone number (or leave blank to finish): ")
    if number == "":
      break

    phone_number = person.phones.add()
    phone_number.number = number

    phone_type = input("Is this a mobile, home, or work phone? ")
    if phone_type == "mobile":
      phone_number.type = addressbook_pb2.Person.PhoneType.PHONE_TYPE_MOBILE
    elif phone_type == "home":
      phone_number.type = addressbook_pb2.Person.PhoneType.PHONE_TYPE_HOME
    elif phone_type == "work":
      phone_number.type = addressbook_pb2.Person.PhoneType.PHONE_TYPE_WORK
    else:
      print("Unknown phone type; leaving as default value.")

# Main procedure:  Reads the entire address book from a file,
#   adds one person based on user input, then writes it back out to the same
#   file.
if len(sys.argv) != 2:
  print("Usage:", sys.argv[0], "ADDRESS_BOOK_FILE")
  sys.exit(-1)

address_book = addressbook_pb2.AddressBook()

# Read the existing address book.
try:
  with open(sys.argv[1], "rb") as f:
    address_book.ParseFromString(f.read())
except IOError:
  print(sys.argv[1] + ": Could not open file.  Creating a new one.")

# Add an address.
PromptForAddress(address_book.people.add())

# Write the new address book back to disk.
with open(sys.argv[1], "wb") as f:
  f.write(address_book.SerializeToString())
```

## Reading a Message 读取消息

Of course, an address book wouldn’t be much use if you couldn’t get any information out of it! This example reads the file created by the above example and prints all the information in it.
当然，如果您无法从中获取任何信息，那么通讯录就没有多大用处！此示例读取由上述示例创建的文件并打印其中的所有信息。

```python
#!/usr/bin/env python3

import addressbook_pb2
import sys

# Iterates though all people in the AddressBook and prints info about them.
def ListPeople(address_book):
  for person in address_book.people:
    print("Person ID:", person.id)
    print("  Name:", person.name)
    if person.HasField('email'):
      print("  E-mail address:", person.email)

    for phone_number in person.phones:
      if phone_number.type == addressbook_pb2.Person.PhoneType.PHONE_TYPE_MOBILE:
        print("  Mobile phone #: ", end="")
      elif phone_number.type == addressbook_pb2.Person.PhoneType.PHONE_TYPE_HOME:
        print("  Home phone #: ", end="")
      elif phone_number.type == addressbook_pb2.Person.PhoneType.PHONE_TYPE_WORK:
        print("  Work phone #: ", end="")
      print(phone_number.number)

# Main procedure:  Reads the entire address book from a file and prints all
#   the information inside.
if len(sys.argv) != 2:
  print("Usage:", sys.argv[0], "ADDRESS_BOOK_FILE")
  sys.exit(-1)

address_book = addressbook_pb2.AddressBook()

# Read the existing address book.
with open(sys.argv[1], "rb") as f:
  address_book.ParseFromString(f.read())

ListPeople(address_book)
```

## Extending a Protocol Buffer 扩展协议缓冲区

Sooner or later after you release the code that uses your protocol buffer, you will undoubtedly want to “improve” the protocol buffer’s definition. If you want your new buffers to be backwards-compatible, and your old buffers to be forward-compatible – and you almost certainly do want this – then there are some rules you need to follow. In the new version of the protocol buffer:
在您发布使用协议缓冲区的代码后，迟早您无疑会想要“改进”协议缓冲区的定义。如果您希望新缓冲区向后兼容，旧缓冲区向前兼容——您几乎肯定希望如此——那么您需要遵循一些规则。在协议缓冲区的新版本中：

- you *must not* change the tag numbers of any existing fields.
  您不得更改任何现有字段的标记号。
- you *must not* add or delete any required fields.
  您不得添加或删除任何必需的字段。
- you *may* delete optional or repeated fields.
  您可以删除可选或重复的字段。
- you *may* add new optional or repeated fields but you must use fresh tag numbers (that is, tag numbers that were never used in this protocol buffer, not even by deleted fields).
  您可以添加新的可选或重复的字段，但您必须使用新的标记号（即，此协议缓冲区中从未使用过的标记号，即使是已删除的字段也不行）。

(There are [some exceptions](https://protobuf.dev/programming-guides/proto#updating) to these rules, but they are rarely used.)
（这些规则有一些例外，但很少使用。）

If you follow these rules, old code will happily read new messages and simply ignore any new fields. To the old code, optional fields that were deleted will simply have their default value, and deleted repeated fields will be empty. New code will also transparently read old messages. However, keep in mind that new optional fields will not be present in old messages, so you will need to either check explicitly whether they’re set with `has_`, or provide a reasonable default value in your `.proto` file with `[default = value]` after the tag number. If the default value is not specified for an optional element, a type-specific default value is used instead: for strings, the default value is the empty string. For booleans, the default value is false. For numeric types, the default value is zero. Note also that if you added a new repeated field, your new code will not be able to tell whether it was left empty (by new code) or never set at all (by old code) since there is no `has_` flag for it.
如果您遵循这些规则，旧代码将乐于读取新消息，并简单地忽略任何新字段。对于旧代码，已删除的可选字段将简单地具有其默认值，而已删除的重复字段将为空。新代码也将透明地读取旧消息。但是，请记住，新可选字段不会出现在旧消息中，因此您需要使用 `has_` 明确检查它们是否已设置，或在 `.proto` 文件中使用 `[default = value]` 在标记编号后提供合理默认值。如果未为可选元素指定默认值，则使用特定于类型的默认值：对于字符串，默认值为空字符串。对于布尔值，默认值是 false。对于数字类型，默认值是零。另请注意，如果您添加了一个新的重复字段，您的新代码将无法判断它是被（新代码）留空还是根本未设置（由旧代码），因为没有 `has_` 标志。

## Advanced Usage 高级用法

Protocol buffers have uses that go beyond simple accessors and serialization. Be sure to explore the [Python API reference](https://googleapis.dev/python/protobuf/latest/) to see what else you can do with them.
协议缓冲区的使用超出了简单的访问器和序列化。务必浏览 Python API 参考，以了解您可以用它们做什么。

One key feature provided by protocol message classes is *reflection*. You can iterate over the fields of a message and manipulate their values without writing your code against any specific message type. One very useful way to use reflection is for converting protocol messages to and from other encodings, such as XML or JSON. A more advanced use of reflection might be to find differences between two messages of the same type, or to develop a sort of “regular expressions for protocol messages” in which you can write expressions that match certain message contents. If you use your imagination, it’s possible to apply Protocol Buffers to a much wider range of problems than you might initially expect!
协议消息类提供的一个关键功能是反射。您可以迭代消息的字段并操作其值，而无需针对任何特定消息类型编写代码。使用反射的一种非常有用的方法是将协议消息转换为其他编码（例如 XML 或 JSON），或将协议消息从其他编码转换回来。反射的更高级用法可能是查找两个相同类型消息之间的差异，或开发一种“协议消息的正则表达式”，您可以在其中编写与某些消息内容匹配的表达式。如果您发挥想象力，就可以将 Protocol Buffers 应用到比您最初预期的更广泛的问题上！

Reflection is provided as part of the [`Message` interface](https://googleapis.dev/python/protobuf/latest/google/protobuf/message.html#google.protobuf.message.Message).
反射作为 `Message` 接口的一部分提供。
