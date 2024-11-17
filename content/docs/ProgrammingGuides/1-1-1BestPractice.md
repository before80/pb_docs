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

# 1-1-1 Best Practice

All proto definitions should have one top-level element and build target per file.



The 1-1-1 best practice is to keep every proto_library and .proto file as small as is reasonable, with the ideal being:

- One `proto_library` build rule
- One source `.proto` file
- One top-level entity (message, enum, or extension)

Having the fewest number of message, enum, extension, and services as you reasonably can makes refactoring easier. Moving files when they’re separated is much easier than extracting messages from a file with other messages.

Following this practice can help build times and binary size by reducing the size of your transitive dependencies in practice: when some code only needs to use one enum, under a 1-1-1 design it can depend just on the .proto file that defines that enum and avoid incidentally pulling in a large set of transitive dependencies that may only be used by another message defined in the same file.

There are cases where the 1-1-1 ideal is not possible (circular dependencies), not ideal (extremely conceptually coupled messages which have readability benefits by being co-located), or where the some of the downsides don’t apply (when a .proto file has no imports, then there are no technical concerns about the size of transitive dependencies). As with any best practice, use good judgment for when to diverge from the guideline.

One place that modularity of proto schema files is important is when creating gRPC definitions. The following set of proto files shows modular structure.

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

In this example, `Student`, `StudentID`, and `FullName` are domain types that are reusable across requests and responses. The top-level request and response protos are unique to each service+method.

If you later need to add a `middle_name` field to the `FullName` message, you won’t need to update every individual top-level message with that new field. Likewise, if you need to update `Student` with more information, all the requests and responses get the update. Further, `StudentID` might update to be a multi-part ID.

Lastly, having even simple types like `StudentID` wrapped as a message means that you have created a type that has semantics and consolidated documentation. For something like `FullName` you’ll need to be careful with where this PII gets logged; this is another advantage of not repeating these fields in multiple top-level messages. You can tag those fields in one place as sensitive and exclude them from logging.
