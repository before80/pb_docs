+++
title = "Go"
date = 2024-11-17T09:35:36+08:00
weight = 30
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文：[https://protobuf.dev/getting-started/gotutorial/](https://protobuf.dev/getting-started/gotutorial/)
>
> 收录该文档的时间：`2024-11-17T09:35:36+08:00`

# Protocol Buffer Basics: Go - Protocol Buffer 基础：Go

A basic Go programmers introduction to working with protocol buffers.

​	面向 Go 程序员的协议缓冲基础教程。

This tutorial provides a basic Go programmer’s introduction to working with protocol buffers, using the [proto3](https://protobuf.dev/programming-guides/proto3) version of the protocol buffers language. By walking through creating a simple example application, it shows you how to

​	本教程为 Go 程序员提供了一个使用协议缓冲的基础入门，基于 [proto3](https://protobuf.dev/programming-guides/proto3) 版本的协议缓冲语言。通过构建一个简单的示例应用程序，本教程将展示如何：

- Define message formats in a `.proto` file.
  - 在 `.proto` 文件中定义消息格式。

- Use the protocol buffer compiler.
  - 使用协议缓冲编译器。

- Use the Go protocol buffer API to write and read messages.
  - 使用 Go 协议缓冲 API 读取和写入消息。


This isn’t a comprehensive guide to using protocol buffers in Go. For more detailed reference information, see the [Protocol Buffer Language Guide](https://protobuf.dev/programming-guides/proto3), the [Go API Reference](https://pkg.go.dev/google.golang.org/protobuf/proto), the [Go Generated Code Guide](https://protobuf.dev/reference/go/go-generated), and the [Encoding Reference](https://protobuf.dev/programming-guides/encoding).

​	本教程并非 Go 中协议缓冲使用的全面指南。有关更详细的参考信息，请参阅以下文档：[协议缓冲语言指南](https://protobuf.dev/programming-guides/proto3)、[Go API 参考](https://pkg.go.dev/google.golang.org/protobuf/proto)、[Go 生成代码指南](https://protobuf.dev/reference/go/go-generated) 和 [编码参考](https://protobuf.dev/programming-guides/encoding)。

## 问题领域 The Problem Domain

The example we’re going to use is a very simple “address book” application that can read and write people’s contact details to and from a file. Each person in the address book has a name, an ID, an email address, and a contact phone number.

​	我们将用一个简单的“地址簿”应用程序作为示例，该应用程序可以从文件中读取和写入联系人详细信息。地址簿中的每个人都有姓名、ID、电子邮件地址和联系电话号码。

How do you serialize and retrieve structured data like this? There are a few ways to solve this problem:

​	如何序列化和检索这样的结构化数据？有以下几种方法解决这个问题：

- Use [gobs](https://golang.org/pkg/encoding/gob/) to serialize Go data structures. This is a good solution in a Go-specific environment, but it doesn’t work well if you need to share data with applications written for other platforms.
  - 使用 [gobs](https://golang.org/pkg/encoding/gob/) 序列化 Go 数据结构。这种方式在 Go 特定环境下是不错的解决方案，但如果需要与其他平台上的应用程序共享数据，则效果不佳。

- You can invent an ad-hoc way to encode the data items into a single string – such as encoding 4 ints as “12:3:-23:67”. This is a simple and flexible approach, although it does require writing one-off encoding and parsing code, and the parsing imposes a small run-time cost. This works best for encoding very simple data.
  - 发明一种特定方法将数据项编码为单个字符串，例如将 4 个整数编码为 "12:3:-23:67"。这种方法简单灵活，但需要编写特定的编码和解析代码，并且解析会带来一定的运行时成本。这种方法适用于非常简单的数据编码。

- Serialize the data to XML. This approach can be very attractive since XML is (sort of) human readable and there are binding libraries for lots of languages. This can be a good choice if you want to share data with other applications/projects. However, XML is notoriously space intensive, and encoding/decoding it can impose a huge performance penalty on applications. Also, navigating an XML DOM tree is considerably more complicated than navigating simple fields in a class normally would be.
  - 将数据序列化为 XML。这种方法可能很吸引人，因为 XML 是（某种程度上）人类可读的，并且有适用于多种语言的绑定库。如果需要与其他应用程序/项目共享数据，这可能是不错的选择。然而，XML 的空间开销非常大，并且编码/解码会对应用程序的性能产生巨大影响。此外，遍历 XML DOM 树比通常在类中遍历简单字段要复杂得多。


Protocol buffers are the flexible, efficient, automated solution to solve exactly this problem. With protocol buffers, you write a `.proto` description of the data structure you wish to store. From that, the protocol buffer compiler creates a class that implements automatic encoding and parsing of the protocol buffer data with an efficient binary format. The generated class provides getters and setters for the fields that make up a protocol buffer and takes care of the details of reading and writing the protocol buffer as a unit. Importantly, the protocol buffer format supports the idea of extending the format over time in such a way that the code can still read data encoded with the old format.

​	协议缓冲提供了灵活、高效、自动化的解决方案，可以完美解决这个问题。使用协议缓冲，您只需编写一个 `.proto` 文件来描述要存储的数据结构。然后，协议缓冲编译器会生成一个类，该类实现了高效二进制格式的协议缓冲数据的自动编码和解析。生成的类为组成协议缓冲的字段提供了 getter 和 setter 方法，并负责将协议缓冲作为一个单元进行读取和写入。重要的是，协议缓冲格式支持随时间扩展格式的概念，使得代码仍然可以读取旧格式编码的数据。

## 示例代码位置 Where to Find the Example Code

Our example is a set of command-line applications for managing an address book data file, encoded using protocol buffers. The command `add_person_go` adds a new entry to the data file. The command `list_people_go` parses the data file and prints the data to the console.

​	我们的示例是一组用于管理协议缓冲编码的地址簿数据文件的命令行应用程序。命令 `add_person_go` 用于向数据文件添加新条目。命令 `list_people_go` 用于解析数据文件并将数据打印到控制台。

You can find the complete example in the [examples directory](https://github.com/protocolbuffers/protobuf/tree/master/examples) of the GitHub repository.

​	您可以在 GitHub 仓库的 [examples 目录](https://github.com/protocolbuffers/protobuf/tree/master/examples) 中找到完整示例。

## 定义您的协议格式 Defining Your Protocol Format

To create your address book application, you’ll need to start with a `.proto` file. The definitions in a `.proto` file are simple: you add a *message* for each data structure you want to serialize, then specify a name and a type for each field in the message. In our example, the `.proto` file that defines the messages is [`addressbook.proto`](https://github.com/protocolbuffers/protobuf/blob/master/examples/addressbook.proto).

​	要创建地址簿应用程序，您需要从 `.proto` 文件开始。`.proto` 文件中的定义很简单：为每个要序列化的数据结构添加一个 *message*，然后为消息中的每个字段指定名称和类型。在我们的示例中，定义消息的 `.proto` 文件是 [`addressbook.proto`](https://github.com/protocolbuffers/protobuf/blob/master/examples/addressbook.proto)。

The `.proto` file starts with a package declaration, which helps to prevent naming conflicts between different projects.

​	`.proto` 文件以包声明开始，有助于防止不同项目之间的命名冲突。

```proto
syntax = "proto3";
package tutorial;

import "google/protobuf/timestamp.proto";
```

The `go_package` option defines the import path of the package which will contain all the generated code for this file. The Go package name will be the last path component of the import path. For example, our example will use a package name of “tutorialpb”.

​	`go_package` 选项定义了包含此文件生成代码的包的导入路径。Go 包名将是导入路径的最后一个路径组件。例如，我们的示例将使用包名 “tutorialpb”。

```proto
option go_package = "github.com/protocolbuffers/protobuf/examples/go/tutorialpb";
```

Next, you have your message definitions. A message is just an aggregate containing a set of typed fields. Many standard simple data types are available as field types, including `bool`, `int32`, `float`, `double`, and `string`. You can also add further structure to your messages by using other message types as field types.

​	接下来是消息定义。消息是一个包含一组类型化字段的聚合体。许多标准简单数据类型可用作字段类型，包括 `bool`、`int32`、`float`、`double` 和 `string`。还可以通过使用其他消息类型作为字段类型，为消息添加进一步的结构。

```proto
message Person {
  string name = 1;
  int32 id = 2;  // Unique ID number for this person. 此人的唯一 ID 编号。
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
// 我们的地址簿文件仅包含这些。
message AddressBook {
  repeated Person people = 1;
}
```

In the above example, the `Person` message contains `PhoneNumber` messages, while the `AddressBook` message contains `Person` messages. You can even define message types nested inside other messages – as you can see, the `PhoneNumber` type is defined inside `Person`. You can also define `enum` types if you want one of your fields to have one of a predefined list of values – here you want to specify that a phone number can be one of `PHONE_TYPE_MOBILE`, `PHONE_TYPE_HOME`, or `PHONE_TYPE_WORK`.

​	在上述示例中，`Person` 消息包含 `PhoneNumber` 消息，而 `AddressBook` 消息包含 `Person` 消息。您甚至可以在其他消息内部定义消息类型——如所示，`PhoneNumber` 类型定义在 `Person` 内部。如果希望字段具有预定义值列表中的一个值，还可以定义 `enum` 类型——在这里，您可以指定电话号码可以是 `PHONE_TYPE_MOBILE`、`PHONE_TYPE_HOME` 或 `PHONE_TYPE_WORK`。

The " = 1", " = 2" markers on each element identify the unique “tag” that field uses in the binary encoding. Tag numbers 1-15 require one less byte to encode than higher numbers, so as an optimization you can decide to use those tags for the commonly used or repeated elements, leaving tags 16 and higher for less-commonly used optional elements. Each element in a repeated field requires re-encoding the tag number, so repeated fields are particularly good candidates for this optimization.

​	每个元素上的 `= 1`、`= 2` 标记标识字段在二进制编码中使用的唯一“标签”。标签编号 1-15 编码所需的字节少于更高编号，因此作为优化，可以选择将这些标签用于常用或重复的元素，将标签编号 16 及以上用于不常用的可选元素。重复字段中的每个元素都需要重新编码标签编号，因此重复字段特别适合这种优化。

If a field value isn’t set, a [default value](https://protobuf.dev/programming-guides/proto3#default) is used: zero for numeric types, the empty string for strings, false for bools. For embedded messages, the default value is always the “default instance” or “prototype” of the message, which has none of its fields set. Calling the accessor to get the value of a field which has not been explicitly set always returns that field’s default value.

​	如果字段值未设置，则使用[默认值](https://protobuf.dev/programming-guides/proto3#default)：数字类型为零，字符串为空字符串，布尔值为 `false`。对于嵌入消息，默认值始终是消息的“默认实例”或“原型”，其字段未设置。调用访问器获取未显式设置的字段的值时，总是返回该字段的默认值。

If a field is `repeated`, the field may be repeated any number of times (including zero). The order of the repeated values will be preserved in the protocol buffer. Think of repeated fields as dynamically sized arrays.

​	如果字段是 `repeated`，该字段可以重复任意次（包括零次）。协议缓冲中会保留重复值的顺序。可以将重复字段视为动态大小的数组。

You’ll find a complete guide to writing `.proto` files – including all the possible field types – in the [Protocol Buffer Language Guide](https://protobuf.dev/programming-guides/proto3). Don’t go looking for facilities similar to class inheritance, though – protocol buffers don’t do that.

​	有关编写 `.proto` 文件的完整指南（包括所有可能的字段类型），请参阅 [协议缓冲语言指南](https://protobuf.dev/programming-guides/proto3)。不过，协议缓冲不支持类似类继承的功能。

## 编译协议缓冲 Compiling Your Protocol Buffers

Now that you have a `.proto`, the next thing you need to do is generate the classes you’ll need to read and write `AddressBook` (and hence `Person` and `PhoneNumber`) messages. To do this, you need to run the protocol buffer compiler `protoc` on your `.proto`:

​	完成 `.proto` 文件后，下一步是生成类，用于读取和写入 `AddressBook`（因此也包括 `Person` 和 `PhoneNumber`）消息。为此，您需要在 `.proto` 文件上运行协议缓冲编译器 `protoc`：

1. If you haven’t installed the compiler, [download the package](https://protobuf.dev/downloads) and follow the instructions in the README. 如果尚未安装编译器，请[下载软件包](https://protobuf.dev/downloads)并按照 README 中的说明操作。

2. Run the following command to install the Go protocol buffers plugin: 运行以下命令安装 Go 协议缓冲插件：

   ```shell
   go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
   ```

   The compiler plugin `protoc-gen-go` will be installed in `$GOBIN`, defaulting to `$GOPATH/bin`. It must be in your `$PATH` for the protocol compiler `protoc` to find it.

   ​	编译器插件 `protoc-gen-go` 将安装在 `$GOBIN`，默认为 `$GOPATH/bin`。它必须位于 `$PATH` 中，以便协议编译器 `protoc` 能够找到它。

3. Now run the compiler, specifying the source directory (where your application’s source code lives – the current directory is used if you don’t provide a value), the destination directory (where you want the generated code to go; often the same as `$SRC_DIR`), and the path to your `.proto`. In this case, you would invoke: 现在运行编译器，指定源目录（应用程序源代码所在的目录，如果未提供值，则使用当前目录）、目标目录（生成代码的存放目录；通常与 `$SRC_DIR` 相同）以及 `.proto` 文件的路径。在本例中，您将运行以下命令：

   ```shell
   protoc -I=$SRC_DIR --go_out=$DST_DIR $SRC_DIR/addressbook.proto
   ```

   Because you want Go code, you use the `--go_out` option – similar options are provided for other supported languages.
   
   ​	因为需要生成 Go 代码，所以使用 `--go_out` 选项——对于其他支持的语言，也提供了类似的选项。

This generates `github.com/protocolbuffers/protobuf/examples/go/tutorialpb/addressbook.pb.go` in your specified destination directory.

​	这将在您指定的目标目录中生成 `github.com/protocolbuffers/protobuf/examples/go/tutorialpb/addressbook.pb.go`。

## 协议缓冲 API - The Protocol Buffer API

Generating `addressbook.pb.go` gives you the following useful types:

​	生成的 `addressbook.pb.go` 提供以下有用的类型：

- An `AddressBook` structure with a `People` field.
  - 一个包含 `People` 字段的 `AddressBook` 结构。

- A `Person` structure with fields for `Name`, `Id`, `Email` and `Phones`.
  - 一个包含 `Name`、`Id`、`Email` 和 `Phones` 字段的 `Person` 结构。

- A `Person_PhoneNumber` structure, with fields for `Number` and `Type`.
  - 一个包含 `Number` 和 `Type` 字段的 `Person_PhoneNumber` 结构。

- The type `Person_PhoneType` and a value defined for each value in the `Person.PhoneType` enum.
  - `Person_PhoneType` 类型，以及 `Person.PhoneType` 枚举中的每个值的定义。


You can read more about the details of exactly what’s generated in the [Go Generated Code guide](https://protobuf.dev/reference/go/go-generated), but for the most part you can treat these as perfectly ordinary Go types.

​	有关生成内容的详细信息，请参阅 [Go 生成代码指南](https://protobuf.dev/reference/go/go-generated)，但大多数情况下，您可以将这些类型视为普通的 Go 类型。

Here’s an example from the [`list_people` command’s unit tests](https://github.com/protocolbuffers/protobuf/blob/master/examples/go/cmd/list_people/list_people_test.go) of how you might create an instance of Person:

​	以下是 [`list_people` 命令单元测试](https://github.com/protocolbuffers/protobuf/blob/master/examples/go/cmd/list_people/list_people_test.go) 中的示例，展示了如何创建 `Person` 的实例：

```go
p := pb.Person{
    Id:    1234,
    Name:  "John Doe",
    Email: "jdoe@example.com",
    Phones: []*pb.Person_PhoneNumber{
        {Number: "555-4321", Type: pb.PhoneType_PHONE_TYPE_HOME},
    },
}
```

## 写入消息 Writing a Message

The whole purpose of using protocol buffers is to serialize your data so that it can be parsed elsewhere. In Go, you use the `proto` library’s [Marshal](https://pkg.go.dev/google.golang.org/protobuf/proto?tab=doc#Marshal) function to serialize your protocol buffer data. A pointer to a protocol buffer message’s `struct` implements the `proto.Message` interface. Calling `proto.Marshal` returns the protocol buffer, encoded in its wire format. For example, we use this function in the [`add_person` command](https://github.com/protocolbuffers/protobuf/blob/master/examples/go/cmd/add_person/add_person.go):

​	使用协议缓冲的主要目的是序列化数据，以便在其他地方解析。在 Go 中，可以使用 `proto` 库的 [Marshal](https://pkg.go.dev/google.golang.org/protobuf/proto?tab=doc#Marshal) 函数序列化协议缓冲数据。指向协议缓冲消息结构的指针实现了 `proto.Message` 接口。调用 `proto.Marshal` 返回编码为线格式的协议缓冲数据。例如，在 [`add_person` 命令](https://github.com/protocolbuffers/protobuf/blob/master/examples/go/cmd/add_person/add_person.go) 中，我们使用了以下代码：

```go
book := &pb.AddressBook{}
// ...

// Write the new address book back to disk.
// 将新的地址簿写回磁盘。
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

​	要解析已编码的消息，可以使用 `proto` 库的 [Unmarshal](https://pkg.go.dev/google.golang.org/protobuf/proto?tab=doc#Unmarshal) 函数。调用该函数会将 `in` 中的数据解析为协议缓冲，并将结果放入 `book`。例如，在 [`list_people` 命令](https://github.com/protocolbuffers/protobuf/blob/master/examples/go/cmd/list_people/list_people.go) 中，我们使用了以下代码：

```go
// Read the existing address book.
// 读取现有地址簿。
in, err := ioutil.ReadFile(fname)
if err != nil {
    log.Fatalln("Error reading file:", err)
}
book := &pb.AddressBook{}
if err := proto.Unmarshal(in, book); err != nil {
    log.Fatalln("Failed to parse address book:", err)
}
```

## 扩展协议缓冲 Extending a Protocol Buffer

Sooner or later after you release the code that uses your protocol buffer, you will undoubtedly want to “improve” the protocol buffer’s definition. If you want your new buffers to be backwards-compatible, and your old buffers to be forward-compatible – and you almost certainly do want this – then there are some rules you need to follow. In the new version of the protocol buffer:

​	在发布使用协议缓冲的代码后，不可避免地会希望“改进”协议缓冲的定义。如果希望新旧协议缓冲相互兼容（向后兼容和向前兼容），需要遵循一些规则。在协议缓冲的新版本中：

- you *must not* change the tag numbers of any existing fields.
  - *不得* 更改任何现有字段的标签编号。

- you *may* delete fields.
  - *可以* 删除字段。

- you *may* add new fields but you must use fresh tag numbers (i.e. tag numbers that were never used in this protocol buffer, not even by deleted fields).
  - *可以* 添加新字段，但必须使用未使用的标签编号（即，从未在此协议缓冲中使用过的标签编号，包括已删除字段的标签编号）。


(There are [some exceptions](https://protobuf.dev/programming-guides/proto3#updating) to these rules, but they are rarely used.)

​	（这些规则有[一些例外](https://protobuf.dev/programming-guides/proto3#updating)，但很少使用。）

If you follow these rules, old code will happily read new messages and simply ignore any new fields. To the old code, singular fields that were deleted will simply have their default value, and deleted repeated fields will be empty. New code will also transparently read old messages.

​	如果遵循这些规则，旧代码可以正常读取新消息并忽略任何新字段。对于旧代码，被删除的单字段将简单地具有其默认值，而被删除的重复字段将为空。新代码也可以透明地读取旧消息。

However, keep in mind that new fields will not be present in old messages, so you will need to do something reasonable with the default value. A type-specific [default value](https://protobuf.dev/programming-guides/proto3#default) is used: for strings, the default value is the empty string. For booleans, the default value is false. For numeric types, the default value is zero.

​	需要注意的是，新字段在旧消息中不存在，因此需要合理处理默认值。使用类型特定的[默认值](https://protobuf.dev/programming-guides/proto3#default)：字符串的默认值为空字符串；布尔值的默认值为 `false`；数字类型的默认值为 `0`。
