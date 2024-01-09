+++
title = "Protocol Buffer Basics: Kotlin"
weight = 260
linkTitle = "Kotlin"
description = "A basic Kotlin programmers introduction to working with protocol buffers."
type = "docs"
+++

> 原文网址： https://protobuf.dev/getting-started/kotlintutorial/

# Protocol Buffer Basics: Kotlin 协议缓冲区基础知识：Kotlin

A basic Kotlin programmers introduction to working with protocol buffers.
Kotlin 程序员入门指南，介绍如何使用协议缓冲区。



This tutorial provides a basic Kotlin programmer’s introduction to working with protocol buffers, using the [proto3](https://protobuf.dev/programming-guides/proto3) version of the protocol buffers language. By walking through creating a simple example application, it shows you how to
本教程通过使用协议缓冲区语言的 proto3 版本，为 Kotlin 程序员提供有关如何使用协议缓冲区的基本入门知识。通过逐步创建一个简单的示例应用程序，本教程将向您展示如何

- Define message formats in a `.proto` file.
  在 `.proto` 文件中定义消息格式。
- Use the protocol buffer compiler.
  使用协议缓冲区编译器。
- Use the Kotlin protocol buffer API to write and read messages.
  使用 Kotlin 协议缓冲区 API 来编写和读取消息。

This isn’t a comprehensive guide to using protocol buffers in Kotlin. For more detailed reference information, see the [Protocol Buffer Language Guide](https://protobuf.dev/programming-guides/proto3), the [Kotlin API Reference](https://protobuf.dev/reference/kotlin/api-docs), the [Kotlin Generated Code Guide](https://protobuf.dev/reference/kotlin/kotlin-generated), and the [Encoding Reference](https://protobuf.dev/programming-guides/encoding).
这不是有关如何在 Kotlin 中使用协议缓冲区的综合指南。有关更详细的参考信息，请参阅《协议缓冲区语言指南》、《Kotlin API 参考》、《Kotlin 生成代码指南》和《编码参考》。

## The Problem Domain 问题域

The example we’re going to use is a very simple “address book” application that can read and write people’s contact details to and from a file. Each person in the address book has a name, an ID, an email address, and a contact phone number.
我们将要使用的示例是一个非常简单的“通讯录”应用程序，它可以将人们的联系方式读写到文件。通讯录中的每个人都有一个姓名、一个 ID、一个电子邮件地址和一个联系电话号码。

How do you serialize and retrieve structured data like this? There are a few ways to solve this problem:
您如何序列化和检索这样的结构化数据？有几种方法可以解决此问题：

- Use kotlinx.serialization. This does not work very well if you need to share data with applications written in C++ or Python. kotlinx.serialization has a [protobuf mode](https://github.com/Kotlin/kotlinx.serialization/blob/master/docs/formats.md#protobuf-experimental), but this does not offer the full features of protocol buffers.
  使用 kotlinx.serialization。如果您需要与用 C++ 或 Python 编写的应用程序共享数据，则此方法效果不佳。kotlinx.serialization 具有 protobuf 模式，但这并不能提供协议缓冲区的全部功能。
- You can invent an ad-hoc way to encode the data items into a single string – such as encoding 4 ints as “12:3:-23:67”. This is a simple and flexible approach, although it does require writing one-off encoding and parsing code, and the parsing imposes a small run-time cost. This works best for encoding very simple data.
  您可以发明一种临时方法将数据项编码为单个字符串——例如将 4 个整数编码为“12:3:-23:67”。这是一种简单而灵活的方法，尽管它确实需要编写一次性编码和解析代码，并且解析会产生少量运行时成本。这最适合编码非常简单的数据。
- Serialize the data to XML. This approach can be very attractive since XML is (sort of) human readable and there are binding libraries for lots of languages. This can be a good choice if you want to share data with other applications/projects. However, XML is notoriously space intensive, and encoding/decoding it can impose a huge performance penalty on applications. Also, navigating an XML DOM tree is considerably more complicated than navigating simple fields in a class normally would be.
  将数据序列化为 XML。这种方法可能非常有吸引力，因为 XML（某种程度上）是人类可读的，并且有很多语言的绑定库。如果您想与其他应用程序/项目共享数据，这可能是一个不错的选择。但是，XML 以空间密集而臭名昭著，并且对它的编码/解码可能会给应用程序带来巨大的性能损失。此外，导航 XML DOM 树比通常在类中导航简单字段要复杂得多。

Protocol buffers are the flexible, efficient, automated solution to solve exactly this problem. With protocol buffers, you write a `.proto` description of the data structure you wish to store. From that, the protocol buffer compiler creates a class that implements automatic encoding and parsing of the protocol buffer data with an efficient binary format. The generated class provides getters and setters for the fields that make up a protocol buffer and takes care of the details of reading and writing the protocol buffer as a unit. Importantly, the protocol buffer format supports the idea of extending the format over time in such a way that the code can still read data encoded with the old format.
协议缓冲区是解决此问题的灵活、高效、自动化的解决方案。使用协议缓冲区，您可以编写要存储的数据结构的 `.proto` 描述。由此，协议缓冲区编译器会创建一个类，该类实现协议缓冲区数据的自动编码和解析，并采用高效的二进制格式。生成的类提供获取器和设置器，用于构成协议缓冲区的字段，并负责以一个单元的形式读取和写入协议缓冲区的详细信息。重要的是，协议缓冲区格式支持随着时间的推移扩展格式的想法，这样代码仍然可以读取使用旧格式编码的数据。

## Where to Find the Example Code 在哪里找到示例代码

Our example is a set of command-line applications for managing an address book data file, encoded using protocol buffers. The command `add_person_kotlin` adds a new entry to the data file. The command `list_people_kotlin` parses the data file and prints the data to the console.
我们的示例是一组用于管理地址簿数据文件的命令行应用程序，该文件使用协议缓冲区进行编码。命令 `add_person_kotlin` 向数据文件添加新条目。命令 `list_people_kotlin` 解析数据文件并将数据打印到控制台。

You can find the complete example in the [examples directory](https://github.com/protocolbuffers/protobuf/tree/master/examples) of the GitHub repository.
您可以在 GitHub 存储库的 examples 目录中找到完整示例。

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
   protoc -I=$SRC_DIR --java_out=$DST_DIR --kotlin_out=$DST_DIR $SRC_DIR/addressbook.proto
   ```

   Because you want Kotlin code, you use the `--kotlin_out` option – similar options are provided for other supported languages.
   由于您想要 Kotlin 代码，因此您使用 `--kotlin_out` 选项 - 为其他受支持的语言提供了类似的选项。

Note that if you want to generate Kotlin code you must use both `--java_out` and `--kotlin_out`. This generates a `com/example/tutorial/protos/` subdirectory in your specified Java destination directory, containing a few generated `.java` files and a `com/example/tutorial/protos/` subdirectory in your specified Kotlin destination directory, containing a few generated `.kt` files.
请注意，如果您想生成 Kotlin 代码，则必须同时使用 `--java_out` 和 `--kotlin_out` 。这会在您指定的 Java 目标目录中生成一个 `com/example/tutorial/protos/` 子目录，其中包含一些生成的 `.java` 文件，以及在您指定的 Kotlin 目标目录中生成一个 `com/example/tutorial/protos/` 子目录，其中包含一些生成的 `.kt` 文件。

## The Protocol Buffer API 协议缓冲区 API

The protocol buffer compiler for Kotlin generates Kotlin APIs that add to the existing APIs generated for protocol buffers for Java. This ensures that codebases written in a mix of Java and Kotlin can interact with the same protocol buffer message objects without any special handling or conversion.
Kotlin 的协议缓冲区编译器会生成 Kotlin API，这些 API 会添加到为 Java 协议缓冲区生成的现有 API 中。这可确保以 Java 和 Kotlin 混合编写的代码库可以与相同的协议缓冲区消息对象进行交互，而无需任何特殊处理或转换。

Protocol buffers for other Kotlin compilation targets, such as JavaScript and native, are not currently supported.
目前不支持针对其他 Kotlin 编译目标（例如 JavaScript 和原生）的协议缓冲区。

Compiling `addressbook.proto` gives you the following APIs in Java:
编译 `addressbook.proto` 会在 Java 中为您提供以下 API：

- The

   

  ```
  AddressBook
  ```

   

  class

  
  `AddressBook` 类

  - which, from Kotlin, has the `peopleList : List<Person>` property
    该类在 Kotlin 中具有 `peopleList : List<Person>` 属性

- The

   

  ```
  Person
  ```

   

  class

  
  `Person` 类

  - which, from Kotlin, has `name`, `id`, `email`, and `phonesList` properties
    其中，从 Kotlin，具有 `name` 、 `id` 、 `email` 和 `phonesList` 属性
  - the `Person.PhoneNumber` nested class with `number` and `type` properties
    具有 `Person.PhoneNumber` 和 `number` 属性的 `Person.PhoneNumber` 嵌套类
  - the `Person.PhoneType` nested enum
    嵌套枚举 `Person.PhoneType`

but also generates the following Kotlin APIs:
但也会生成以下 Kotlin API：

- The `addressBook { ... }` and `person { ... }` factory methods
  工厂方法 `addressBook { ... }` 和 `person { ... }`
- A `PersonKt` object, with a `phoneNumber { ... }` factory method
  具有工厂方法 `phoneNumber { ... }` 的对象 `PersonKt`

You can read more about the details of exactly what’s generated in the [Kotlin Generated Code guide](https://protobuf.dev/reference/kotlin/kotlin-generated).
您可以在 Kotlin 生成的代码指南中阅读有关确切生成内容的详细信息。

## Writing a Message 编写消息

Now let’s try using your protocol buffer classes. The first thing you want your address book application to be able to do is write personal details to your address book file. To do this, you need to create and populate instances of your protocol buffer classes and then write them to an output stream.
现在，我们尝试使用您的协议缓冲区类。您希望通讯录应用程序能够做的第一件事是将个人详细信息写入通讯录文件。为此，您需要创建和填充协议缓冲区类的实例，然后将它们写入输出流。

Here is a program which reads an `AddressBook` from a file, adds one new `Person` to it based on user input, and writes the new `AddressBook` back out to the file again. The parts which directly call or reference code generated by the protocol compiler are highlighted.
这是一个从文件中读取 `AddressBook` 、根据用户输入向其中添加一个新的 `Person` ，然后将新的 `AddressBook` 再次写入文件的程序。直接调用或引用协议编译器生成的代码的部分已突出显示。

```kotlin
import com.example.tutorial.Person
import com.example.tutorial.AddressBook
import com.example.tutorial.person
import com.example.tutorial.addressBook
import com.example.tutorial.PersonKt.phoneNumber
import java.util.Scanner

// This function fills in a Person message based on user input.
fun promptPerson(): Person = person {
  print("Enter person ID: ")
  id = readLine().toInt()

  print("Enter name: ")
  name = readLine()

  print("Enter email address (blank for none): ")
  val email = readLine()
  if (email.isNotEmpty()) {
    this.email = email
  }

  while (true) {
    print("Enter a phone number (or leave blank to finish): ")
    val number = readLine()
    if (number.isEmpty()) break

    print("Is this a mobile, home, or work phone? ")
    val type = when (readLine()) {
      "mobile" -> Person.PhoneType.PHONE_TYPE_MOBILE
      "home" -> Person.PhoneType.PHONE_TYPE_HOME
      "work" -> Person.PhoneType.PHONE_TYPE_WORK
      else -> {
        println("Unknown phone type.  Using home.")
        Person.PhoneType.PHONE_TYPE_HOME
      }
    }
    phones += phoneNumber {
      this.number = number
      this.type = type
    }
  }
}

// Reads the entire address book from a file, adds one person based
// on user input, then writes it back out to the same file.
fun main(args: List) {
  if (arguments.size != 1) {
    println("Usage: add_person ADDRESS_BOOK_FILE")
    exitProcess(-1)
  }
  val path = Path(arguments.single())
  val initialAddressBook = if (!path.exists()) {
    println("File not found. Creating new file.")
    addressBook {}
  } else {
    path.inputStream().use {
      AddressBook.newBuilder().mergeFrom(it).build()
    }
  }
  path.outputStream().use {
    initialAddressBook.copy { peopleList += promptPerson() }.writeTo(it)
  }
}
```

## Reading a Message 读取消息

Of course, an address book wouldn’t be much use if you couldn’t get any information out of it! This example reads the file created by the above example and prints all the information in it.
当然，如果您无法从中获取任何信息，那么通讯录就没有多大用处！此示例读取由上述示例创建的文件并打印其中的所有信息。

```kotlin
import com.example.tutorial.Person
import com.example.tutorial.AddressBook

// Iterates though all people in the AddressBook and prints info about them.
fun print(addressBook: AddressBook) {
  for (person in addressBook.peopleList) {
    println("Person ID: ${person.id}")
    println("  Name: ${person.name}")
    if (person.hasEmail()) {
      println("  Email address: ${person.email}")
    }
    for (phoneNumber in person.phonesList) {
      val modifier = when (phoneNumber.type) {
        Person.PhoneType.PHONE_TYPE_MOBILE -> "Mobile"
        Person.PhoneType.PHONE_TYPE_HOME -> "Home"
        Person.PhoneType.PHONE_TYPE_WORK -> "Work"
        else -> "Unknown"
      }
      println("  $modifier phone #: ${phoneNumber.number}")
    }
  }
}

fun main(args: List) {
  if (arguments.size != 1) {
    println("Usage: list_person ADDRESS_BOOK_FILE")
    exitProcess(-1)
  }
  Path(arguments.single()).inputStream().use {
    print(AddressBook.newBuilder().mergeFrom(it).build())
  }
}
```

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
