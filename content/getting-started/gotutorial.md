+++
title = "协议缓冲区基础知识：Go"
weight = 240
linkTitle = "Go"
description = "Go 程序员使用协议缓冲区的入门介绍。"
type = "docs"

+++

> 原文网址： https://protobuf.dev/getting-started/gotutorial/

## Protocol Buffer Basics: Go 协议缓冲区基础知识：Go

A basic Go programmers introduction to working with protocol buffers.

​	Go 程序员使用协议缓冲区的入门介绍。

This tutorial provides a basic Go programmer’s introduction to working with protocol buffers, using the [proto3](https://protobuf.dev/programming-guides/proto3) version of the protocol buffers language. By walking through creating a simple example application, it shows you how to

​	本教程提供了 Go 程序员使用协议缓冲区的基本介绍，使用协议缓冲区语言的 proto3 版本。通过逐步创建一个简单的示例应用程序，它向您展示如何

- Define message formats in a `.proto` file.
- 在 `.proto` 文件中定义消息格式。
- Use the protocol buffer compiler.
- 使用协议缓冲区编译器。
- Use the Go protocol buffer API to write and read messages.
- 使用 Go 协议缓冲区 API 来编写和读取消息。

This isn’t a comprehensive guide to using protocol buffers in Go. For more detailed reference information, see the [Protocol Buffer Language Guide](https://protobuf.dev/programming-guides/proto3), the [Go API Reference](https://pkg.go.dev/google.golang.org/protobuf/proto), the [Go Generated Code Guide](https://protobuf.dev/reference/go/go-generated), and the [Encoding Reference](https://protobuf.dev/programming-guides/encoding).

​	这不是有关在 Go 中使用协议缓冲区的综合指南。有关更详细的参考信息，请参阅《协议缓冲区语言指南》、《Go API 参考》、《Go 生成的代码指南》和《编码参考》。

## 问题域 The Problem Domain 

The example we’re going to use is a very simple “address book” application that can read and write people’s contact details to and from a file. Each person in the address book has a name, an ID, an email address, and a contact phone number.

​	我们将要使用的示例是一个非常简单的“通讯录”应用程序，它可以将人们的联系方式读写到文件。通讯录中的每个人都有一个姓名、一个 ID、一个电子邮件地址和一个联系电话号码。

How do you serialize and retrieve structured data like this? There are a few ways to solve this problem:

​	您如何序列化和检索这样的结构化数据？有几种方法可以解决此问题：

- Use [gobs](https://golang.org/pkg/encoding/gob/) to serialize Go data structures. This is a good solution in a Go-specific environment, but it doesn’t work well if you need to share data with applications written for other platforms.
- 使用 gobs 序列化 Go 数据结构。在特定于 Go 的环境中，这是一个不错的解决方案，但如果您需要与为其他平台编写的应用程序共享数据，则效果不佳。
- You can invent an ad-hoc way to encode the data items into a single string – such as encoding 4 ints as “12:3:-23:67”. This is a simple and flexible approach, although it does require writing one-off encoding and parsing code, and the parsing imposes a small run-time cost. This works best for encoding very simple data.
- 您可以发明一种临时方法将数据项编码为单个字符串——例如将 4 个整数编码为“12:3:-23:67”。这是一种简单而灵活的方法，尽管它确实需要编写一次性编码和解析代码，并且解析会产生少量运行时成本。这最适合编码非常简单的数据。
- Serialize the data to XML. This approach can be very attractive since XML is (sort of) human readable and there are binding libraries for lots of languages. This can be a good choice if you want to share data with other applications/projects. However, XML is notoriously space intensive, and encoding/decoding it can impose a huge performance penalty on applications. Also, navigating an XML DOM tree is considerably more complicated than navigating simple fields in a class normally would be.
- 将数据序列化为 XML。这种方法可能非常有吸引力，因为 XML（某种程度上）是人类可读的，并且有很多语言的绑定库。如果您想与其他应用程序/项目共享数据，这可能是一个不错的选择。但是，XML 以空间密集而臭名昭著，并且对它的编码/解码可能会给应用程序带来巨大的性能损失。此外，导航 XML DOM 树比通常在类中导航简单字段要复杂得多。

Protocol buffers are the flexible, efficient, automated solution to solve exactly this problem. With protocol buffers, you write a `.proto` description of the data structure you wish to store. From that, the protocol buffer compiler creates a class that implements automatic encoding and parsing of the protocol buffer data with an efficient binary format. The generated class provides getters and setters for the fields that make up a protocol buffer and takes care of the details of reading and writing the protocol buffer as a unit. Importantly, the protocol buffer format supports the idea of extending the format over time in such a way that the code can still read data encoded with the old format.

​	协议缓冲区是解决此问题的灵活、高效、自动化的解决方案。使用协议缓冲区，您可以编写要存储的数据结构的 `.proto` 描述。由此，协议缓冲区编译器会创建一个类，该类实现协议缓冲区数据的自动编码和解析，并采用高效的二进制格式。生成的类提供获取器和设置器，用于构成协议缓冲区的字段，并负责以一个单元的形式读取和写入协议缓冲区的详细信息。重要的是，协议缓冲区格式支持随着时间的推移扩展格式的想法，这样代码仍然可以读取使用旧格式编码的数据。

## 在哪里找到示例代码 Where to Find the Example Code 

Our example is a set of command-line applications for managing an address book data file, encoded using protocol buffers. The command `add_person_go` adds a new entry to the data file. The command `list_people_go` parses the data file and prints the data to the console.

​	我们的示例是一组用于管理地址簿数据文件的命令行应用程序，该文件使用协议缓冲区进行编码。命令 `add_person_go` 向数据文件添加新条目。命令 `list_people_go` 解析数据文件并将数据打印到控制台。

You can find the complete example in the [examples directory](https://github.com/protocolbuffers/protobuf/tree/master/examples) of the GitHub repository.

​	您可以在 GitHub 存储库的 examples 目录中找到完整示例。

## 定义协议格式 Defining Your Protocol Format 

To create your address book application, you’ll need to start with a `.proto` file. The definitions in a `.proto` file are simple: you add a *message* for each data structure you want to serialize, then specify a name and a type for each field in the message. In our example, the `.proto` file that defines the messages is [`addressbook.proto`](https://github.com/protocolbuffers/protobuf/blob/master/examples/addressbook.proto).

​	要创建您的地址簿应用程序，您需要从一个 `.proto` 文件开始。在 `.proto` 文件中的定义很简单：您为要序列化的每个数据结构添加一条消息，然后为消息中的每个字段指定一个名称和一个类型。在我们的示例中，定义消息的 `.proto` 文件是 `addressbook.proto` 。

The `.proto` file starts with a package declaration, which helps to prevent naming conflicts between different projects.

​	`.proto` 文件以包声明开头，这有助于防止不同项目之间的命名冲突。

```proto
syntax = "proto3";
package tutorial;

import "google/protobuf/timestamp.proto";
```

The `go_package` option defines the import path of the package which will contain all the generated code for this file. The Go package name will be the last path component of the import path. For example, our example will use a package name of “tutorialpb”.

​	`go_package` 选项定义了将包含此文件的所有生成代码的包的导入路径。Go 包名称将是导入路径的最后一个路径组件。例如，我们的示例将使用“tutorialpb”的包名称。

```proto
option go_package = "github.com/protocolbuffers/protobuf/examples/go/tutorialpb";
```

Next, you have your message definitions. A message is just an aggregate containing a set of typed fields. Many standard simple data types are available as field types, including `bool`, `int32`, `float`, `double`, and `string`. You can also add further structure to your messages by using other message types as field types.

​	接下来，您有您的消息定义。消息只是一个包含一组类型化字段的聚合。许多标准的简单数据类型可作为字段类型使用，包括 `bool` 、 `int32` 、 `float` 、 `double` 和 `string` 。您还可以通过使用其他消息类型作为字段类型来为您的消息添加更多结构。

```proto
message Person {
  string name = 1;
  int32 id = 2;  // Unique ID number for this person.
  string email = 3;

  message PhoneNumber {
    string number = 1;
    PhoneType type = 2;
  }

  repeated PhoneNumber phones = 4;

  google.protobuf.Timestamp last_updated = 5;
}

enum PhoneType {
  PHONE_TYPE_UNSPECIFIED = 0;
  PHONE_TYPE_MOBILE = 1;
  PHONE_TYPE_HOME = 2;
  PHONE_TYPE_WORK = 3;
}

// Our address book file is just one of these.
message AddressBook {
  repeated Person people = 1;
}
```

In the above example, the `Person` message contains `PhoneNumber` messages, while the `AddressBook` message contains `Person` messages. You can even define message types nested inside other messages – as you can see, the `PhoneNumber` type is defined inside `Person`. You can also define `enum` types if you want one of your fields to have one of a predefined list of values – here you want to specify that a phone number can be one of `PHONE_TYPE_MOBILE`, `PHONE_TYPE_HOME`, or `PHONE_TYPE_WORK`.

​	在上面的示例中， `Person` 消息包含 `PhoneNumber` 消息，而 `AddressBook` 消息包含 `Person` 消息。您甚至可以定义嵌套在其他消息内部的消息类型——如您所见， `PhoneNumber` 类型定义在 `Person` 内部。如果您希望某个字段具有预定义值列表中的某个值，还可以定义 `enum` 类型——此处您要指定电话号码可以是 `PHONE_TYPE_MOBILE` 、 `PHONE_TYPE_HOME` 或 `PHONE_TYPE_WORK` 之一。

The " = 1", " = 2" markers on each element identify the unique “tag” that field uses in the binary encoding. Tag numbers 1-15 require one less byte to encode than higher numbers, so as an optimization you can decide to use those tags for the commonly used or repeated elements, leaving tags 16 and higher for less-commonly used optional elements. Each element in a repeated field requires re-encoding the tag number, so repeated fields are particularly good candidates for this optimization.

​	每个元素上的“= 1”、“= 2”标记标识该字段在二进制编码中使用的唯一“标记”。标记编号 1-15 比较高的编号编码时少一个字节，因此作为优化，您可以决定将这些标记用于常用或重复的元素，将标记 16 及更高编号留给不常用的可选元素。重复字段中的每个元素都需要重新编码标记编号，因此重复字段特别适合这种优化。

If a field value isn’t set, a [default value](https://protobuf.dev/programming-guides/proto3#default) is used: zero for numeric types, the empty string for strings, false for bools. For embedded messages, the default value is always the “default instance” or “prototype” of the message, which has none of its fields set. Calling the accessor to get the value of a field which has not been explicitly set always returns that field’s default value.

​	如果未设置字段值，则使用默认值：对于数字类型为零，对于字符串为空字符串，对于布尔值为 false。对于嵌入式消息，默认值始终是消息的“默认实例”或“原型”，其没有任何已设置的字段。调用访问器以获取尚未显式设置的字段的值时，始终返回该字段的默认值。

If a field is `repeated`, the field may be repeated any number of times (including zero). The order of the repeated values will be preserved in the protocol buffer. Think of repeated fields as dynamically sized arrays.

​	如果字段是 `repeated` ，则该字段可以重复任意多次（包括零次）。重复值在协议缓冲区中的顺序将被保留。将重复字段视为动态大小的数组。

You’ll find a complete guide to writing `.proto` files – including all the possible field types – in the [Protocol Buffer Language Guide](https://protobuf.dev/programming-guides/proto3). Don’t go looking for facilities similar to class inheritance, though – protocol buffers don’t do that.

​	您可以在协议缓冲区语言指南中找到编写 `.proto` 文件的完整指南，包括所有可能的字段类型。不过，不要寻找类似于类继承的功能，协议缓冲区不具备该功能。

## 编译您的协议缓冲区 Compiling Your Protocol Buffers 

Now that you have a `.proto`, the next thing you need to do is generate the classes you’ll need to read and write `AddressBook` (and hence `Person` and `PhoneNumber`) messages. To do this, you need to run the protocol buffer compiler `protoc` on your `.proto`:

​	现在您已拥有 `.proto` ，接下来需要做的是生成您需要用来读写 `AddressBook` （进而读写 `Person` 和 `PhoneNumber` ）消息的类。为此，您需要在 `.proto` 上运行协议缓冲区编译器 `protoc` ：

1. If you haven’t installed the compiler, [download the package](https://protobuf.dev/downloads) and follow the instructions in the README.

2. 如果您尚未安装编译器，请下载软件包并按照自述文件中的说明进行操作。

3. Run the following command to install the Go protocol buffers plugin:

4. 运行以下命令以安装 Go 协议缓冲区插件：

   ```shell
   go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
   ```

   The compiler plugin `protoc-gen-go` will be installed in `$GOBIN`, defaulting to `$GOPATH/bin`. It must be in your `$PATH` for the protocol compiler `protoc` to find it.

   编译器插件 `protoc-gen-go` 将安装在 `$GOBIN` 中，默认为 `$GOPATH/bin` 。它必须在您的 `$PATH` 中，以便协议编译器 `protoc` 找到它。

5. Now run the compiler, specifying the source directory (where your application’s source code lives – the current directory is used if you don’t provide a value), the destination directory (where you want the generated code to go; often the same as `$SRC_DIR`), and the path to your `.proto`. In this case, you would invoke:

6. 现在运行编译器，指定源目录（您的应用程序源代码所在的位置 - 如果您未提供值，则使用当前目录）、目标目录（您希望生成的代码所在的位置；通常与 `$SRC_DIR` 相同）和 `.proto` 的路径。在这种情况下，您将调用：

   ```shell
   protoc -I=$SRC_DIR --go_out=$DST_DIR $SRC_DIR/addressbook.proto
   ```

   Because you want Go code, you use the `--go_out` option – similar options are provided for other supported languages.

   ​	因为您想要 Go 代码，所以您使用 `--go_out` 选项 - 为其他受支持的语言提供了类似的选项。

This generates `github.com/protocolbuffers/protobuf/examples/go/tutorialpb/addressbook.pb.go` in your specified destination directory.

​	这会在您指定的目标目录中生成 `github.com/protocolbuffers/protobuf/examples/go/tutorialpb/addressbook.pb.go` 。

## 协议缓冲区 API - The Protocol Buffer API 

Generating `addressbook.pb.go` gives you the following useful types:

​	生成 `addressbook.pb.go` 为您提供了以下有用的类型：

- An `AddressBook` structure with a `People` field.
- 一个具有 `People` 字段的 `AddressBook` 结构。
- A `Person` structure with fields for `Name`, `Id`, `Email` and `Phones`.
- 一个 `Person` 结构，其中包含 `Name` 、 `Id` 、 `Email` 和 `Phones` 的字段。
- A `Person_PhoneNumber` structure, with fields for `Number` and `Type`.
- 一个 `Person_PhoneNumber` 结构，其中包含 `Number` 和 `Type` 的字段。
- The type `Person_PhoneType` and a value defined for each value in the `Person.PhoneType` enum.
- 类型 `Person_PhoneType` 和为 `Person.PhoneType` 枚举中的每个值定义的值。

You can read more about the details of exactly what’s generated in the [Go Generated Code guide](https://protobuf.dev/reference/go/go-generated), but for the most part you can treat these as perfectly ordinary Go types.

​	您可以在 Go 生成的代码指南中阅读有关确切生成内容的详细信息，但大多数情况下，您可以将这些视为完全普通的 Go 类型。

Here’s an example from the [`list_people` command’s unit tests](https://github.com/protocolbuffers/protobuf/blob/master/examples/go/cmd/list_people/list_people_test.go) of how you might create an instance of Person:

​	以下是从 `list_people` 命令的单元测试中创建 Person 实例的示例：

```go
p := pb.Person{
    Id:    1234,
    Name:  "John Doe",
    Email: "jdoe@example.com",
    Phones: []*pb.Person_PhoneNumber{
        {Number: "555-4321", Type: pb.Person_PHONE_TYPE_HOME},
    },
}
```

## 编写消息 Writing a Message 

The whole purpose of using protocol buffers is to serialize your data so that it can be parsed elsewhere. In Go, you use the `proto` library’s [Marshal](https://pkg.go.dev/google.golang.org/protobuf/proto?tab=doc#Marshal) function to serialize your protocol buffer data. A pointer to a protocol buffer message’s `struct` implements the `proto.Message` interface. Calling `proto.Marshal` returns the protocol buffer, encoded in its wire format. For example, we use this function in the [`add_person` command](https://github.com/protocolbuffers/protobuf/blob/master/examples/go/cmd/add_person/add_person.go):

​	使用协议缓冲区的全部目的是序列化您的数据，以便可以在其他地方解析它。在 Go 中，您使用 `proto` 库的 Marshal 函数来序列化您的协议缓冲区数据。指向协议缓冲区消息的 `struct` 的指针实现了 `proto.Message` 接口。调用 `proto.Marshal` 返回以其线路格式编码的协议缓冲区。例如，我们在 `add_person` 命令中使用此函数：

```go
book := &pb.AddressBook{}
// ...

// Write the new address book back to disk.
out, err := proto.Marshal(book)
if err != nil {
    log.Fatalln("Failed to encode address book:", err)
}
if err := ioutil.WriteFile(fname, out, 0644); err != nil {
    log.Fatalln("Failed to write address book:", err)
}
```

## 读取消息 Reading a Message 

To parse an encoded message, you use the `proto` library’s [Unmarshal](https://pkg.go.dev/google.golang.org/protobuf/proto?tab=doc#Unmarshal) function. Calling this parses the data in `in` as a protocol buffer and places the result in `book`. So to parse the file in the [`list_people` command](https://github.com/protocolbuffers/protobuf/blob/master/examples/go/cmd/list_people/list_people.go), we use:

​	要解析编码的消息，请使用 `proto` 库的 Unmarshal 函数。调用此函数将 `in` 中的数据解析为协议缓冲区，并将结果放入 `book` 中。因此，要解析 `list_people` 命令中的文件，我们使用：

```go
// Read the existing address book.
in, err := ioutil.ReadFile(fname)
if err != nil {
    log.Fatalln("Error reading file:", err)
}
book := &pb.AddressBook{}
if err := proto.Unmarshal(in, book); err != nil {
    log.Fatalln("Failed to parse address book:", err)
}
```

## 扩展协议缓冲区 Extending a Protocol Buffer 

Sooner or later after you release the code that uses your protocol buffer, you will undoubtedly want to “improve” the protocol buffer’s definition. If you want your new buffers to be backwards-compatible, and your old buffers to be forward-compatible – and you almost certainly do want this – then there are some rules you need to follow. In the new version of the protocol buffer:

​	在您发布使用协议缓冲区的代码后，迟早您无疑会想要“改进”协议缓冲区的定义。如果您希望新缓冲区向后兼容，旧缓冲区向前兼容——您几乎肯定希望如此——那么您需要遵循一些规则。在协议缓冲区的新版本中：

- you *must not* change the tag numbers of any existing fields.
- 您不得更改任何现有字段的标记号。
- you *may* delete fields.
- 您可以删除字段。
- you *may* add new fields but you must use fresh tag numbers (i.e. tag numbers that were never used in this protocol buffer, not even by deleted fields).
- 您可以添加新字段，但您必须使用新的标记号（即此协议缓冲区中从未使用过的标记号，即使是已删除的字段也不行）。

(There are [some exceptions](https://protobuf.dev/programming-guides/proto3#updating) to these rules, but they are rarely used.)

​	（这些规则有一些例外，但很少使用。）

If you follow these rules, old code will happily read new messages and simply ignore any new fields. To the old code, singular fields that were deleted will simply have their default value, and deleted repeated fields will be empty. New code will also transparently read old messages.

​	如果您遵循这些规则，旧代码将乐于读取新消息，并简单地忽略任何新字段。对于旧代码，已删除的单数字段将简单地具有其默认值，已删除的重复字段将为空。新代码也将透明地读取旧消息。

However, keep in mind that new fields will not be present in old messages, so you will need to do something reasonable with the default value. A type-specific [default value](https://protobuf.dev/programming-guides/proto3#default) is used: for strings, the default value is the empty string. For booleans, the default value is false. For numeric types, the default value is zero.

​	但是，请记住，新字段不会出现在旧消息中，因此您需要对默认值执行一些合理的操作。使用特定于类型的默认值：对于字符串，默认值为空字符串。对于布尔值，默认值为 false。对于数字类型，默认值为零。
