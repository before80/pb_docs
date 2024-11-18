+++
title = "1-1-1 Best Practice"
date = 2024-11-17T09:35:36+08:00
weight = 170
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

> 原文：[https://protobuf.dev/programming-guides/1-1-1/](https://protobuf.dev/programming-guides/1-1-1/)
>
> 收录该文档的时间：`2024-11-17T09:35:36+08:00`

# 1-1-1 Best Practice - 最佳实践

All proto definitions should have one top-level element and build target per file.

​	所有 proto 定义应在每个文件中具有一个顶层元素和一个构建目标。

The 1-1-1 best practice is to keep every proto_library and .proto file as small as is reasonable, with the ideal being:

​	1-1-1 最佳实践旨在将每个 `proto_library` 和 `.proto` 文件尽可能保持简洁，理想状态是：

- One `proto_library` build rule
  - 一个 `proto_library` 构建规则

- One source `.proto` file
  - 一个源 `.proto` 文件

- One top-level entity (message, enum, or extension)
  - 一个顶层实体（消息、枚举或扩展）


Having the fewest number of message, enum, extension, and services as you reasonably can makes refactoring easier. Moving files when they’re separated is much easier than extracting messages from a file with other messages.

​	尽可能减少消息、枚举、扩展和服务的数量可以使重构更加简单。当文件是分离的时，移动文件比从包含多个消息的文件中提取消息容易得多。

Following this practice can help build times and binary size by reducing the size of your transitive dependencies in practice: when some code only needs to use one enum, under a 1-1-1 design it can depend just on the .proto file that defines that enum and avoid incidentally pulling in a large set of transitive dependencies that may only be used by another message defined in the same file.

​	遵循此实践还有助于缩短构建时间并减小二进制文件的大小，因为它在实际中减少了传递依赖的规模。例如，当某些代码只需要使用一个枚举时，按照 1-1-1 设计，它只需依赖定义该枚举的 `.proto` 文件，而不会意外引入包含其他大量传递依赖的文件，这些依赖可能仅由同一文件中的其他消息使用。

There are cases where the 1-1-1 ideal is not possible (circular dependencies), not ideal (extremely conceptually coupled messages which have readability benefits by being co-located), or where the some of the downsides don’t apply (when a .proto file has no imports, then there are no technical concerns about the size of transitive dependencies). As with any best practice, use good judgment for when to diverge from the guideline.

​	但在某些情况下，1-1-1 理想状态可能无法实现（例如循环依赖）、不理想（例如概念上高度耦合的消息放在一起可提高可读性），或者其中一些缺点并不适用（例如，当 `.proto` 文件没有导入时，就没有关于传递依赖大小的技术问题）。与任何最佳实践一样，需根据具体情况判断何时偏离该指南。

One place that modularity of proto schema files is important is when creating gRPC definitions. The following set of proto files shows modular structure.

​	在创建 gRPC 定义时，proto 模式文件的模块化显得尤为重要。以下是一组展示模块化结构的 proto 文件：

**student_id.proto**

```proto
edition = "2023";

package my.package;

message StudentID {
  string value = 1;
}
```

**full_name.proto**

```proto
edition = "2023";

package my.package;

message FullName {
  string family_name = 1;
  string given_name = 2;
}
```

**student.proto**

```proto
edition = "2023";

package my.package;

import "student_id.proto";
import "full_name.proto";

message Student {
  StudentId id = 1;
  FullName name = 2;
}
```

**create_student_request.proto**

```proto
edition = "2023";

package my.package;

import "full_name.proto";

message CreateStudentRequest {
  FullName name = 1;
}
```

**create_student_response.proto**

```proto
edition = "2023";

package my.package;

import "student.proto";

message CreateStudentResponse {
  Student student = 1;
}
```

**get_student_request.proto**

```proto
edition = "2023";

package my.package;

import "student_id.proto";

message GetStudentRequest {
  StudentID id = 1;
}
```

**get_student_response.proto**

```proto
edition = "2023";

package my.package;

import "student.proto";

message GetStudentResponse {
  Student student = 1;
}
```

**student_service.proto**

```proto
edition = "2023";

package my.package;

import "create_student_request.proto";
import "create_student_response.proto";
import "get_student_request.proto";
import "get_student_response.proto";

service StudentService {
  rpc CreateStudent(CreateStudentRequest) returns (CreateStudentResponse);
  rpc GetStudent(GetStudentRequest) returns (GetStudentResponse);
}
```

The service definition and each of the message definitions are each in their own file, and you use includes to give access to the messages from other schema files.

​	服务定义和每个消息定义都在各自的文件中，您可以通过 `import` 来访问其他模式文件中的消息。

In this example, `Student`, `StudentID`, and `FullName` are domain types that are reusable across requests and responses. The top-level request and response protos are unique to each service+method.

​	在此示例中，`Student`、`StudentID` 和 `FullName` 是跨请求和响应可重用的领域类型。顶层的请求和响应 protos 则是每个服务与方法独有的。

If you later need to add a `middle_name` field to the `FullName` message, you won’t need to update every individual top-level message with that new field. Likewise, if you need to update `Student` with more information, all the requests and responses get the update. Further, `StudentID` might update to be a multi-part ID.

​	如果您稍后需要为 `FullName` 消息添加 `middle_name` 字段，无需更新每个顶层消息中的新字段。同样，如果需要为 `Student` 增加更多信息，所有请求和响应都会自动更新。此外，`StudentID` 可能会更新为一个多部分 ID。

Lastly, having even simple types like `StudentID` wrapped as a message means that you have created a type that has semantics and consolidated documentation. For something like `FullName` you’ll need to be careful with where this PII gets logged; this is another advantage of not repeating these fields in multiple top-level messages. You can tag those fields in one place as sensitive and exclude them from logging.

​	最后，即使是像 `StudentID` 这样的简单类型，通过将其封装为消息，您可以创建具有语义的类型并集中文档说明。例如，对于 `FullName` 这样的字段，您需要注意 PII（个人可识别信息）的日志记录位置；这也是避免在多个顶层消息中重复这些字段的另一个优势。您可以在一个地方标记这些字段为敏感数据并排除其日志记录。
