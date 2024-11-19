+++
title = "生成代码指南"
date = 2024-11-17T12:07:24+08:00
weight = 1
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

> 原文：[https://protobuf.dev/reference/go/go-generated/](https://protobuf.dev/reference/go/go-generated/)
>
> 收录该文档的时间：`2024-11-17T12:07:24+08:00`

# Go Generated Code Guide - Go 生成代码指南

Describes exactly what Go code the protocol buffer compiler generates for any given protocol definition.

​	描述协议缓冲区编译器为任意协议定义生成的 Go 代码。

Any differences between proto2 and proto3 generated code are highlighted - note that these differences are in the generated code as described in this document, not the base API, which are the same in both versions. You should read the [proto2 language guide]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2" >}}) and/or the [proto3 language guide]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto3" >}}) before reading this document.

​	在本文档中，proto2 和 proto3 生成代码的任何差异都会被突出显示——请注意，这些差异仅存在于生成代码中，而基础 API 在两个版本中是相同的。在阅读本文档之前，您应该先阅读 [proto2 语言指南]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2" >}}) 和/或 [proto3 语言指南]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto3" >}})。

## 编译器调用 Compiler Invocation

The protocol buffer compiler requires a plugin to generate Go code. Install it using Go 1.16 or higher by running:

​	协议缓冲区编译器需要一个插件来生成 Go 代码。使用 Go 1.16 或更高版本，通过运行以下命令安装：



```shell
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
```

This will install a `protoc-gen-go` binary in `$GOBIN`. Set the `$GOBIN` environment variable to change the installation location. It must be in your `$PATH` for the protocol buffer compiler to find it.

​	这将在 `$GOBIN` 中安装一个名为 `protoc-gen-go` 的二进制文件。设置 `$GOBIN` 环境变量可以更改安装位置。该文件必须在您的 `$PATH` 中，以便协议缓冲区编译器找到它。

The protocol buffer compiler produces Go output when invoked with the `go_out` flag. The argument to the `go_out` flag is the directory where you want the compiler to write your Go output. The compiler creates a single source file for each `.proto` file input. The name of the output file is created by replacing the `.proto` extension with `.pb.go`.

​	协议缓冲区编译器通过 `go_out` 标志生成 Go 输出。`go_out` 标志的参数是您希望编译器写入 Go 输出的目录。对于每个输入的 `.proto` 文件，编译器会生成一个源文件。输出文件的名称是通过将 `.proto` 扩展名替换为 `.pb.go` 创建的。

Where in the output directory the generated `.pb.go` file is placed depends on the compiler flags. There are several output modes:

​	生成的 `.pb.go` 文件在输出目录中的位置取决于编译器标志。有以下几种输出模式：

- If the `paths=import` flag is specified, the output file is placed in a directory named after the Go package’s import path (such as one provided by the `go_package` option within the `.proto` file). For example, an input file `protos/buzz.proto` with a Go import path of `example.com/project/protos/fizz` results in an output file at `example.com/project/protos/fizz/buzz.pb.go`. This is the default output mode if a `paths` flag is not specified.
  - 如果指定了 `paths=import` 标志，输出文件将被放置在一个以 Go 包导入路径命名的目录中（例如，在 `.proto` 文件中的 `go_package` 选项提供的路径）。例如，一个输入文件 `protos/buzz.proto` 的 Go 导入路径为 `example.com/project/protos/fizz`，输出文件将位于 `example.com/project/protos/fizz/buzz.pb.go`。如果未指定 `paths` 标志，这是默认输出模式。

- If the `module=$PREFIX` flag is specified, the output file is placed in a directory named after the Go package’s import path (such as one provided by the `go_package` option within the `.proto` file), but with the specified directory prefix removed from the output filename. For example, an input file `protos/buzz.proto` with a Go import path of `example.com/project/protos/fizz` and `example.com/project` specified as the `module` prefix results in an output file at `protos/fizz/buzz.pb.go`. Generating any Go packages outside the module path results in an error. This mode is useful for outputting generated files directly into a Go module.
  - 如果指定了 `module=$PREFIX` 标志，输出文件将被放置在以 Go 包导入路径命名的目录中（例如，在 `.proto` 文件中的 `go_package` 选项提供的路径），但输出文件名中会移除指定的目录前缀。例如，一个输入文件 `protos/buzz.proto` 的 Go 导入路径为 `example.com/project/protos/fizz`，并且指定 `example.com/project` 作为 `module` 前缀，输出文件将位于 `protos/fizz/buzz.pb.go`。如果生成的 Go 包超出了模块路径，则会报错。此模式用于将生成的文件直接输出到 Go 模块中。

- If the `paths=source_relative` flag is specified, the output file is placed in the same relative directory as the input file. For example, an input file `protos/buzz.proto` results in an output file at `protos/buzz.pb.go`.
  - 如果指定了 `paths=source_relative` 标志，输出文件将被放置在与输入文件相同的相对目录中。例如，一个输入文件 `protos/buzz.proto` 的输出文件为 `protos/buzz.pb.go`。


Flags specific to `protoc-gen-go` are provided by passing a `go_opt` flag when invoking `protoc`. Multiple `go_opt` flags may be passed. For example, when running:

​	通过在调用 `protoc` 时传递 `go_opt` 标志可以提供 `protoc-gen-go` 特定的标志。可以传递多个 `go_opt` 标志。例如，运行以下命令：

```shell
protoc --proto_path=src --go_out=out --go_opt=paths=source_relative foo.proto bar/baz.proto
```

the compiler will read input files `foo.proto` and `bar/baz.proto` from within the `src` directory, and write output files `foo.pb.go` and `bar/baz.pb.go` to the `out` directory. The compiler automatically creates nested output sub-directories if necessary, but will not create the output directory itself.

​	编译器将从 `src` 目录中读取输入文件 `foo.proto` 和 `bar/baz.proto`，并将输出文件 `foo.pb.go` 和 `bar/baz.pb.go` 写入 `out` 目录。编译器会根据需要自动创建嵌套输出子目录，但不会创建输出目录本身。

## Packages

In order to generate Go code, the Go package’s import path must be provided for every `.proto` file (including those transitively depended upon by the `.proto` files being generated). There are two ways to specify the Go import path:

​	为了生成 Go 代码，必须为每个 `.proto` 文件（包括生成文件依赖的所有传递文件）提供 Go 包导入路径。有两种方式可以指定 Go 导入路径：

- by declaring it within the `.proto` file, or
  - 在 `.proto` 文件中声明，或

- by declaring it on the command line when invoking `protoc`.
  - 在调用 `protoc` 时通过命令行声明。


We recommend declaring it within the `.proto` file so that the Go packages for `.proto` files can be centrally identified with the `.proto` files themselves and to simplify the set of flags passed when invoking `protoc`. If the Go import path for a given `.proto` file is provided by both the `.proto` file itself and on the command line, then the latter takes precedence over the former.

​	我们建议在 `.proto` 文件中声明，以便 `.proto` 文件的 Go 包可以集中标识，并简化调用 `protoc` 时传递的标志。如果 `.proto` 文件本身和命令行都提供了 Go 导入路径，则后者优先。

The Go import path is locally specified in a `.proto` file by declaring a `go_package` option with the full import path of the Go package. Example usage:

​	在 `.proto` 文件中，可以通过声明 `go_package` 选项并指定 Go 包的完整导入路径来本地指定 Go 导入路径。例如：



```proto
option go_package = "example.com/project/protos/fizz";
```

The Go import path may be specified on the command line when invoking the compiler, by passing one or more `M${PROTO_FILE}=${GO_IMPORT_PATH}` flags. Example usage:

​	在调用编译器时，可以通过传递一个或多个 `M${PROTO_FILE}=${GO_IMPORT_PATH}` 标志来指定 Go 导入路径。例如：

```shell
protoc --proto_path=src \
  --go_opt=Mprotos/buzz.proto=example.com/project/protos/fizz \
  --go_opt=Mprotos/bar.proto=example.com/project/protos/foo \
  protos/buzz.proto protos/bar.proto
```

Since the mapping of all `.proto` files to their Go import paths can be quite large, this mode of specifying the Go import paths is generally performed by some build tool (e.g., [Bazel](https://bazel.build/)) that has control over the entire dependency tree. If there are duplicate entries for a given `.proto` file, then the last one specified takes precedence.

​	由于将所有 `.proto` 文件映射到其 Go 导入路径的映射可能非常大，这种指定方式通常由一些构建工具（例如 [Bazel](https://bazel.build/)）自动完成，工具控制整个依赖树。如果为同一个 `.proto` 文件提供了重复条目，则最后一个条目优先。

For both the `go_package` option and the `M` flag, the value may include an explicit package name separated from the import path by a semicolon. For example: `"example.com/protos/foo;package_name"`. This usage is discouraged since the package name will be derived by default from the import path in a reasonable manner.

​	对于 `go_package` 选项和 `M` 标志，值可以包括由分号分隔的显式包名。例如：`"example.com/protos/foo;package_name"`。不建议使用这种用法，因为包名通常可以从导入路径中合理地派生出来。

The import path is used to determine which import statements must be generated when one `.proto` file imports another `.proto` file. For example, if `a.proto` imports `b.proto`, then the generated `a.pb.go` file needs to import the Go package which contains the generated `b.pb.go` file (unless both files are in the same package). The import path is also used to construct output filenames. See the "Compiler Invocation" section above for details.

​	导入路径用于确定在一个 `.proto` 文件导入另一个 `.proto` 文件时需要生成哪些导入语句。例如，如果 `a.proto` 导入了 `b.proto`，则生成的 `a.pb.go` 文件需要导入包含生成的 `b.pb.go` 文件的 Go 包（除非两个文件在同一个包中）。导入路径还用于构造输出文件名。详情参见上文“编译器调用”部分。

There is no correlation between the Go import path and the [`package` specifier]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto3#packages" >}}) in the `.proto` file. The latter is only relevant to the protobuf namespace, while the former is only relevant to the Go namespace. Also, there is no correlation between the Go import path and the `.proto` import path.

​	`.proto` 文件中的 Go 导入路径与其 [`package` 指定符]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto3#packages" >}})之间没有关联。后者仅与 protobuf 命名空间相关，而前者仅与 Go 命名空间相关。此外，Go 导入路径与 `.proto` 导入路径之间也没有关联。

## 消息 Messages

Given a simple message declaration:

​	给定一个简单的消息声明：

```proto
message Artist {}
```

the protocol buffer compiler generates a struct called `Artist`. An `*Artist` implements the [`proto.Message`](https://pkg.go.dev/google.golang.org/protobuf/proto?tab=doc#Message) interface.

​	协议缓冲区编译器会生成一个名为 `Artist` 的结构体。`*Artist` 实现了 [`proto.Message`](https://pkg.go.dev/google.golang.org/protobuf/proto?tab=doc#Message) 接口。

The [`proto` package](https://pkg.go.dev/google.golang.org/protobuf/proto?tab=doc) provides functions which operate on messages, including conversion to and from binary format.

​	[`proto` 包](https://pkg.go.dev/google.golang.org/protobuf/proto?tab=doc) 提供了用于操作消息的函数，包括消息的二进制格式转换。

The `proto.Message` interface defines a `ProtoReflect` method. This method returns a [`protoreflect.Message`](https://pkg.go.dev/google.golang.org/protobuf/reflect/protoreflect?tab=doc#Message) which provides a reflection-based view of the message.

​	`proto.Message` 接口定义了一个 `ProtoReflect` 方法。此方法返回一个 [`protoreflect.Message`](https://pkg.go.dev/google.golang.org/protobuf/reflect/protoreflect?tab=doc#Message)，提供基于反射的消息视图。

The `optimize_for` option does not affect the output of the Go code generator.

​	`optimize_for` 选项不会影响 Go 代码生成器的输出。

### 嵌套类型 Nested Types

A message can be declared inside another message. For example:

​	消息可以声明在另一条消息内部。例如：

```proto
message Artist {
  message Name {
  }
}
```

In this case, the compiler generates two structs: `Artist` and `Artist_Name`.

​	在这种情况下，编译器会生成两个结构体：`Artist` 和 `Artist_Name`。

## Fields

The protocol buffer compiler generates a struct field for each field defined within a message. The exact nature of this field depends on its type and whether it is a singular, repeated, map, or oneof field.

​	协议缓冲区编译器会为消息中定义的每个字段生成一个结构字段。字段的具体特性取决于其类型是单一、重复、映射或 oneof。

Note that the generated Go field names always use camel-case naming, even if the field name in the `.proto` file uses lower-case with underscores ([as it should]({{< ref "/docs/ProgrammingGuides/StyleGuide#message-field-names" >}})). The case-conversion works as follows:

​	请注意，生成的 Go 字段名称始终使用驼峰式命名，即使 `.proto` 文件中的字段名称使用的是下划线格式（[推荐使用]({{< ref "/docs/ProgrammingGuides/StyleGuide#message-field-names" >}})）。大小写转换规则如下：

1. The first letter is capitalized for export. If the first character is an underscore, it is removed and a capital X is prepended. 第一个字母大写以便导出。如果第一个字符是下划线，则将其移除，并在前面添加一个大写的 `X`。
2. If an interior underscore is followed by a lower-case letter, the underscore is removed, and the following letter is capitalized. 如果内部的下划线后跟一个小写字母，则移除下划线，并将后面的字母大写。

Thus, the proto field `birth_year` becomes `BirthYear` in Go, and `_birth_year_2` becomes `XBirthYear_2`.

​	因此，proto 字段 `birth_year` 在 Go 中变为 `BirthYear`，而 `_birth_year_2` 变为 `XBirthYear_2`。

### 单一标量字段（proto2） Singular Scalar Fields (proto2)

For either of these field definitions:

​	对于以下任意字段定义：

```proto
optional int32 birth_year = 1;
required int32 birth_year = 1;
```

the compiler generates a struct with an `*int32` field named `BirthYear` and an accessor method `GetBirthYear()` which returns the `int32` value in `Artist` or the default value if the field is unset. If the default is not explicitly set, the [zero value](https://golang.org/ref/spec#The_zero_value) of that type is used instead (`0` for numbers, the empty string for strings).

​	编译器会生成一个带有 `*int32` 字段 `BirthYear` 的结构体，以及一个访问器方法 `GetBirthYear()`，该方法返回 `Artist` 中的 `int32` 值，或字段未设置时的默认值。如果未显式设置默认值，则使用该类型的[零值](https://golang.org/ref/spec#The_zero_value)（数字为 `0`，字符串为空字符串）。

For other scalar field types (including `bool`, `bytes`, and `string`), `*int32` is replaced with the corresponding Go type according to the [scalar value types table]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2#scalar" >}}).

​	对于其他标量字段类型（包括 `bool`、`bytes` 和 `string`），`*int32` 会被替换为对应的 Go 类型，具体参考[标量值类型表]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto2#scalar" >}})。

### 单一标量字段（proto3） Singular Scalar Fields (proto3)

For this field definition:

​	对于以下字段定义：

```proto
int32 birth_year = 1;
optional int32 first_active_year = 2;
```

The compiler will generate a struct with an `int32` field named `BirthYear` and an accessor method `GetBirthYear()` which returns the `int32` value in `birth_year` or the [zero value](https://golang.org/ref/spec#The_zero_value) of that type if the field is unset (`0` for numbers, the empty string for strings).

​	编译器会生成一个带有 `int32` 字段 `BirthYear` 的结构体，以及一个访问器方法 `GetBirthYear()`，该方法返回 `birth_year` 的 `int32` 值，或字段未设置时的[零值](https://golang.org/ref/spec#The_zero_value)（数字为 `0`，字符串为空字符串）。

For other scalar field types (including `bool`, `bytes`, and `string`), `int32` is replaced with the corresponding Go type according to the [scalar value types table]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto3#scalar" >}}). Unset values in the proto will be represented as the [zero value](https://golang.org/ref/spec#The_zero_value) of that type (`0` for numbers, the empty string for strings).

​	对于其他标量字段类型（包括 `bool`、`bytes` 和 `string`），`int32` 会被替换为对应的 Go 类型，具体参考[标量值类型表]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto3#scalar" >}})。proto 中未设置的值会表示为该类型的[零值](https://golang.org/ref/spec#The_zero_value)（数字为 `0`，字符串为空字符串）。

### 单一消息字段 Singular Message Fields

Given the message type:

​	给定以下消息类型：

```proto
message Band {}
```

For a message with a `Band` field:

​	对于包含 `Band` 字段的消息：

```proto
// proto2
message Concert {
  optional Band headliner = 1;
  // The generated code is the same result if required instead of optional.
  // 生成的代码结果与将 optional 替换为 required 相同。
}

// proto3
message Concert {
  Band headliner = 1;
}
```

The compiler will generate a Go struct

​	编译器会生成如下 Go 结构体：

```go
type Concert struct {
    Headliner *Band
}
```

Message fields can be set to `nil`, which means that the field is unset, effectively clearing the field. This is not equivalent to setting the value to an "empty" instance of the message struct.

​	消息字段可以设置为 `nil`，这意味着该字段未设置，相当于清除此字段。这与将值设置为消息结构体的“空”实例不同。

The compiler also generates a `func (m *Concert) GetHeadliner() *Band` helper function. This function returns a `nil` `*Band` if `m` is nil or `headliner` is unset. This makes it possible to chain get calls without intermediate `nil` checks:

​	编译器还会生成一个辅助函数 `func (m *Concert) GetHeadliner() *Band`。如果 `m` 为 nil 或 `headliner` 未设置，则该函数返回一个 nil 的 `*Band`。这使得可以在不进行中间 `nil` 检查的情况下链式调用 `get`：

```go
var m *Concert // defaults to nil 默认值为 nil
log.Infof("GetFoundingYear() = %d (no panic!)", m.GetHeadliner().GetFoundingYear())
```

### 重复字段 Repeated Fields

Each repeated field generates a slice of `T` field in the struct in Go, where `T` is the field’s element type. For this message with a repeated field:

​	对于一个重复字段，编译器会在 Go 结构体中生成一个类型为 `[]T` 的字段，其中 `T` 是字段的元素类型。对于以下带有重复字段的消息：

```proto
message Concert {
  // Best practice: use pluralized names for repeated fields:
  // 最佳实践：对重复字段使用复数名称：
  // /programming-guides/style#repeated-fields
  repeated Band support_acts = 1;
}
```

the compiler generates the Go struct:

​	编译器会生成如下 Go 结构体：

```go
type Concert struct {
    SupportActs []*Band
}
```

Likewise, for the field definition `repeated bytes band_promo_images = 1;` the compiler will generate a Go struct with a `[][]byte` field named `BandPromoImage`. For a repeated [enumeration](https://protobuf.dev/reference/go/go-generated/#enum) like `repeated MusicGenre genres = 2;`, the compiler generates a struct with a `[]MusicGenre` field called `Genre`.

​	同样，对于字段定义 `repeated bytes band_promo_images = 1;`，编译器会生成一个 `[][]byte` 类型的 Go 字段，名为 `BandPromoImage`。对于一个重复的[枚举](https://protobuf.dev/reference/go/go-generated/#enum)字段，例如 `repeated MusicGenre genres = 2;`，编译器会生成一个带有 `[]MusicGenre` 字段 `Genre` 的结构体。

The following example shows how to set the field:

​	以下示例展示了如何设置字段：

```go
concert := &Concert{
  SupportActs: []*Band{
    {}, // First element.
    {}, // Second element.
  },
}
```

To access the field, you can do the following:

​	要访问字段，可以这样做：

```go
support := concert.GetSupportActs() // support type is []*Band.
b1 := support[0] // b1 type is *Band, the first element in support_acts.
```

### 映射字段 Map Fields

Each map field generates a field in the struct of type `map[TKey]TValue` where `TKey` is the field’s key type and `TValue` is the field’s value type. For this message with a map field:

​	对于映射字段，编译器会在 Go 结构体中生成一个 `map[TKey]TValue` 类型的字段，其中 `TKey` 是字段的键类型，`TValue` 是字段的值类型。对于以下带有映射字段的消息：

```proto
message MerchItem {}

message MerchBooth {
  // items maps from merchandise item name ("Signed T-Shirt") to
  // a MerchItem message with more details about the item.
  // items 将商品名称（如“签名 T 恤”）映射到
  // 一个带有该商品详细信息的 MerchItem 消息。
  map<string, MerchItem> items = 1;
}
```

the compiler generates the Go struct:

​	编译器会生成如下 Go 结构体：

```go
type MerchBooth struct {
    Items map[string]*MerchItem
}
```

### Oneof Fields

For a oneof field, the protobuf compiler generates a single field with an interface type `isMessageName_MyField`. It also generates a struct for each of the [singular fields](https://protobuf.dev/reference/go/go-generated/#singular-scalar-proto2) within the oneof. These all implement this `isMessageName_MyField` interface.

​	对于一个 oneof 字段，protobuf 编译器会生成一个接口类型 `isMessageName_MyField` 的单字段。它还会为 oneof 内的每个[单一字段](https://protobuf.dev/reference/go/go-generated/#singular-scalar-proto2)生成一个结构体。这些结构体都实现了 `isMessageName_MyField` 接口。

For this message with a oneof field:

​	对于以下带有 oneof 字段的消息：

```proto
package account;
message Profile {
  oneof avatar {
    string image_url = 1;
    bytes image_data = 2;
  }
}
```

the compiler generates the structs:

​	编译器生成以下结构体：

```go
type Profile struct {
    // Types that are valid to be assigned to Avatar:
    // 以下类型可以分配给 Avatar：
    //  *Profile_ImageUrl
    //  *Profile_ImageData
    Avatar isProfile_Avatar `protobuf_oneof:"avatar"`
}

type Profile_ImageUrl struct {
        ImageUrl string
}
type Profile_ImageData struct {
        ImageData []byte
}
```

Both `*Profile_ImageUrl` and `*Profile_ImageData` implement `isProfile_Avatar` by providing an empty `isProfile_Avatar()` method.

​	`*Profile_ImageUrl` 和 `*Profile_ImageData` 都通过提供一个空的 `isProfile_Avatar()` 方法实现了 `isProfile_Avatar` 接口。

The following example shows how to set the field:

​	以下示例展示了如何设置字段：

```go
p1 := &account.Profile{
  Avatar: &account.Profile_ImageUrl{ImageUrl: "http://example.com/image.png"},
}

// imageData is []byte
imageData := getImageData()
p2 := &account.Profile{
  Avatar: &account.Profile_ImageData{ImageData: imageData},
}
```

To access the field, you can use a type switch on the value to handle the different message types.

​	要访问字段，可以对值使用类型判断以处理不同的消息类型：

```go
switch x := m.Avatar.(type) {
case *account.Profile_ImageUrl:
    // Load profile image based on URL
    // using x.ImageUrl
    // 根据 URL 加载头像
    // 使用 x.ImageUrl
case *account.Profile_ImageData:
    // Load profile image based on bytes
    // using x.ImageData
    // 根据字节加载头像
    // 使用 x.ImageData
case nil:
    // The field is not set.
    // 字段未设置。
default:
    return fmt.Errorf("Profile.Avatar has unexpected type %T", x)
}
```

The compiler also generates get methods `func (m *Profile) GetImageUrl() string` and `func (m *Profile) GetImageData() []byte`. Each get function returns the value for that field or the zero value if it is not set.

​	编译器还会生成以下获取方法：`func (m *Profile) GetImageUrl() string` 和 `func (m *Profile) GetImageData() []byte`。每个获取方法返回该字段的值，或者如果字段未设置则返回零值。

### 枚举类型 Enumerations

Given an enumeration like:

​	对于如下定义的枚举类型：

```proto
message Venue {
  enum Kind {
    KIND_UNSPECIFIED = 0;
    KIND_CONCERT_HALL = 1;
    KIND_STADIUM = 2;
    KIND_BAR = 3;
    KIND_OPEN_AIR_FESTIVAL = 4;
  }
  Kind kind = 1;
  // ...
}
```

the protocol buffer compiler generates a type and a series of constants with that type:

​	协议缓冲区编译器生成一个类型和一组具有该类型的常量：

```go
type Venue_Kind int32

const (
    Venue_KIND_UNSPECIFIED       Venue_Kind = 0
    Venue_KIND_CONCERT_HALL      Venue_Kind = 1
    Venue_KIND_STADIUM           Venue_Kind = 2
    Venue_KIND_BAR               Venue_Kind = 3
    Venue_KIND_OPEN_AIR_FESTIVAL Venue_Kind = 4
)
```

For enums within a message (like the one above), the type name begins with the message name:

​	对于消息内的枚举（如上所示），类型名称以消息名称开头：

```go
type Venue_Kind int32
```

For a package-level enum:

​	对于包级别的枚举：

```proto
enum Genre {
  GENRE_UNSPECIFIED = 0;
  GENRE_ROCK = 1;
  GENRE_INDIE = 2;
  GENRE_DRUM_AND_BASS = 3;
  // ...
}
```

the Go type name is unmodified from the proto enum name:

​	Go 类型名称与 proto 枚举名称一致：

```go
type Genre int32
```

This type has a `String()` method that returns the name of a given value.

​	该类型具有一个 `String()` 方法，可以返回给定值的名称。

The `Enum()` method initializes freshly allocated memory with a given value and returns the corresponding pointer:

​	`Enum()` 方法用于初始化新分配的内存并返回相应的指针：

```go
func (Genre) Enum() *Genre
```

The protocol buffer compiler generates a constant for each value in the enum. For enums within a message, the constants begin with the enclosing message’s name:

​	协议缓冲区编译器为枚举中的每个值生成一个常量。对于消息中的枚举，常量名称以封闭消息的名称开头：

```go
const (
    Venue_KIND_UNSPECIFIED       Venue_Kind = 0
    Venue_KIND_CONCERT_HALL      Venue_Kind = 1
    Venue_KIND_STADIUM           Venue_Kind = 2
    Venue_KIND_BAR               Venue_Kind = 3
    Venue_KIND_OPEN_AIR_FESTIVAL Venue_Kind = 4
)
```

For a package-level enum, the constants begin with the enum name instead:

​	对于包级别的枚举，常量名称以枚举名称开头：

```go
const (
    Genre_GENRE_UNSPECIFIED   Genre = 0
    Genre_GENRE_ROCK          Genre = 1
    Genre_GENRE_INDIE         Genre = 2
    Genre_GENRE_DRUM_AND_BASS Genre = 3
)
```

The protobuf compiler also generates a map from integer values to the string names and a map from the names to the values:

​	协议缓冲区编译器还生成从整数值到字符串名称的映射，以及从名称到值的映射：

```go
var Genre_name = map[int32]string{
    0: "GENRE_UNSPECIFIED",
    1: "GENRE_ROCK",
    2: "GENRE_INDIE",
    3: "GENRE_DRUM_AND_BASS",
}
var Genre_value = map[string]int32{
    "GENRE_UNSPECIFIED":   0,
    "GENRE_ROCK":          1,
    "GENRE_INDIE":         2,
    "GENRE_DRUM_AND_BASS": 3,
}
```

Note that the `.proto` language allows multiple enum symbols to have the same numeric value. Symbols with the same numeric value are synonyms. These are represented in Go in exactly the same way, with multiple names corresponding to the same numeric value. The reverse mapping contains a single entry for the numeric value to the name which appears first in the .proto file.

​	请注意，`.proto` 语言允许多个枚举符号具有相同的数值。这些符号是同义词。在 Go 中，这些同义词以完全相同的方式表示，即多个名称对应于相同的数值。反向映射只包含该数值的第一个名称（在 `.proto` 文件中首先出现的名称）。

## 扩展（proto2） Extensions (proto2)

Given an extension definition:

​	给定一个扩展定义：

```proto
extend Concert {
  optional int32 promo_id = 123;
}
```

The protocol buffer compiler will generate a [`protoreflect.ExtensionType`](https://pkg.go.dev/google.golang.org/protobuf/reflect/protoreflect?tab=doc#ExtensionType) value named `E_Promo_id`. This value may be used with the [`proto.GetExtension`](https://pkg.go.dev/google.golang.org/protobuf/proto?tab=doc#GetExtension), [`proto.SetExtension`](https://pkg.go.dev/google.golang.org/protobuf/proto?tab=doc#SetExtension), [`proto.HasExtension`](https://pkg.go.dev/google.golang.org/protobuf/proto?tab=doc#HasExtension), and [`proto.ClearExtension`](https://pkg.go.dev/google.golang.org/protobuf/proto?tab=doc#ClearExtension) functions to access an extension in a message. The `GetExtension` function and `SetExtension` functions respectively return and accept an `interface{}` value containing the extension value type.

​	协议缓冲区编译器将生成一个 [`protoreflect.ExtensionType`](https://pkg.go.dev/google.golang.org/protobuf/reflect/protoreflect?tab=doc#ExtensionType) 值，名为 `E_Promo_id`。此值可以与以下函数一起使用来访问消息中的扩展字段：[`proto.GetExtension`](https://pkg.go.dev/google.golang.org/protobuf/proto?tab=doc#GetExtension)、[`proto.SetExtension`](https://pkg.go.dev/google.golang.org/protobuf/proto?tab=doc#SetExtension)、[`proto.HasExtension`](https://pkg.go.dev/google.golang.org/protobuf/proto?tab=doc#HasExtension) 和 [`proto.ClearExtension`](https://pkg.go.dev/google.golang.org/protobuf/proto?tab=doc#ClearExtension)。`GetExtension` 函数和 `SetExtension` 函数分别返回和接受一个 `interface{}` 值，其中包含扩展值类型。

For singular scalar extension fields, the extension value type is the corresponding Go type from the [scalar value types table]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto3#scalar" >}}).

​	对于单一标量扩展字段，扩展值类型是 [标量值类型表]({{< ref "/docs/ProgrammingGuides/LanguageGuideproto3#scalar" >}}) 中对应的 Go 类型。

For singular embedded message extension fields, the extension value type is `*M`, where `M` is the field message type.

​	对于单一嵌套消息扩展字段，扩展值类型是 `*M`，其中 `M` 是字段的消息类型。

For repeated extension fields, the extension value type is a slice of the singular type.

​	对于重复扩展字段，扩展值类型是单一类型的切片。

For example, given the following definition:

​	例如，给定以下定义：

```proto
extend Concert {
  optional int32 singular_int32 = 1;
  repeated bytes repeated_strings = 2;
  optional Band singular_message = 3;
}
```

Extension values may be accessed as:

​	可以按如下方式访问扩展值：

```go
m := &somepb.Concert{}
proto.SetExtension(m, extpb.E_SingularInt32, int32(1))
proto.SetExtension(m, extpb.E_RepeatedString, []string{"a", "b", "c"})
proto.SetExtension(m, extpb.E_SingularMessage, &extpb.Band{})

v1 := proto.GetExtension(m, extpb.E_SingularInt32).(int32)
v2 := proto.GetExtension(m, extpb.E_RepeatedString).([][]byte)
v3 := proto.GetExtension(m, extpb.E_SingularMessage).(*extpb.Band)
```

Extensions can be declared nested inside of another type. For example, a common pattern is to do something like this:

​	扩展可以嵌套声明在另一类型中。例如，一个常见的模式如下：

```proto
message Promo {
  extend Concert {
    optional int32 promo_id = 124;
  }
}
```

In this case, the `ExtensionType` value is named `E_Promo_Concert`.

​	在这种情况下，`ExtensionType` 值被命名为 `E_Promo_Concert`。

## 服务 Services

The Go code generator does not produce output for services by default. If you enable the [gRPC](https://www.grpc.io/) plugin (see the [gRPC Go Quickstart guide](https://github.com/grpc/grpc-go/tree/master/examples)) then code will be generated to support gRPC.

​	默认情况下，Go 代码生成器不会为服务生成代码。如果启用了 [gRPC](https://www.grpc.io/) 插件（请参阅 [gRPC Go 快速入门指南](https://github.com/grpc/grpc-go/tree/master/examples)），则将生成支持 gRPC 的代码。
