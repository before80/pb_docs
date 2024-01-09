+++
title = "API 最佳实践"
weight = 100
description = "令人惊讶的是，很难获得一个面向未来的 API。本文档中的建议权衡利弊，以支持长期、无错误的演进。"
type = "docs"

+++

> 原文网址： https://protobuf.dev/programming-guides/api/

## API Best Practices - API 最佳实践

A future-proof API is surprisingly hard to get right. The suggestions in this document make trade-offs to favor long-term, bug-free evolution.

​	令人惊讶的是，很难获得一个面向未来的 API。本文档中的建议权衡利弊，以支持长期、无错误的演进。



Updated for proto3. Patches welcome!

​	已针对 proto3 更新。欢迎提交补丁程序！

This doc is a complement to [Proto Best Practices](https://protobuf.dev/programming-guides/dos-donts). It’s not a prescription for Java/C++/Go and other APIs.

​	本文档是对 Proto 最佳实践的补充。它不是针对 Java/C++/Go 和其他 API 的处方。

If you see a proto straying from these guidelines in a code review, point the author to this topic and help spread the word.

​	如果您在代码审查中看到偏离这些准则的 proto，请将作者指向此主题并帮助传播。

#### 注意 Note 

These guidelines are just that and many have documented exceptions. For example, if you’re writing a performance-critical backend, you might want to sacrifice flexibility or safety for speed. This topic will help you better understand the trade-offs and make a decision that works for your situation.

​	这些准则只是这样，许多准则都有记录的例外情况。例如，如果您正在编写对性能至关重要的后端，您可能希望牺牲灵活性或安全性以换取速度。本主题将帮助您更好地理解权衡利弊，并为您的情况做出决策。

## 精确、简洁地记录大多数字段和消息 Precisely, Concisely Document Most Fields and Messages 

Chances are good your proto will be inherited and used by people who don’t know what you were thinking when you wrote or modified it. Document each field in terms that will be useful to a new team-member or client with little knowledge of your system.

​	您的 proto 很可能会被继承并被那些不知道您在编写或修改它时在想什么的人使用。使用对新团队成员或对您的系统了解不多的客户有用的术语记录每个字段。

Some concrete examples:

​	一些具体示例：

```proto
// Bad: Option to enable Foo
// Good: Configuration controlling the behavior of the Foo feature.
message FeatureFooConfig {
  // Bad: Sets whether the feature is enabled
  // Good: Required field indicating whether the Foo feature
  // is enabled for account_id.  Must be false if account_id's
  // FOO_OPTIN Gaia bit is not set.
  optional bool enabled;
}

// Bad: Foo object.
// Good: Client-facing representation of a Foo (what/foo) exposed in APIs.
message Foo {
  // Bad: Title of the foo.
  // Good: Indicates the user-supplied title of this Foo, with no
  // normalization or escaping.
  // An example title: "Picture of my cat in a box <3 <3 !!!"
  optional string title [(max_length) = 512];
}

// Bad: Foo config.
// Less-Bad: If the most useful comment is re-stating the name, better to omit
// the comment.
FooConfig foo_config = 3;
```

Document the constraints, expectations and interpretation of each field in as few words as possible.

​	用尽可能少的文字记录每个字段的约束、期望和解释。

You can use custom proto annotations. See [Custom Options](https://protobuf.dev/programming-guides/proto#options) to define cross-language constants like `max_length` in the example above. Supported in proto2 and proto3.

​	您可以使用自定义的 proto 注释。请参阅自定义选项，以定义跨语言常量，如上面的示例中的 `max_length` 。在 proto2 和 proto3 中受支持。

Over time, documentation of an interface can get longer and longer. The length takes away from the clarity. When the documentation is genuinely unclear, fix it, but look at it holistically and aim for brevity.

​	随着时间的推移，接口的文档可能会越来越长。长度会降低清晰度。当文档确实不清楚时，请修复它，但要从整体上考虑它并力求简洁。

## 对传输和存储使用不同的消息 Use Different Messages for Wire and Storage 

If a top-level proto you expose to clients is the same one you store on disk, you’re headed for trouble. More and more binaries will depend on your API over time, making it harder to change. You’ll want the freedom to change your storage format without impacting your clients. Layer your code so that modules deal either with client protos, storage protos, or translation.

​	如果您向客户端公开的顶级 proto 与您存储在磁盘上的 proto 相同，那么您将面临麻烦。随着时间的推移，越来越多的二进制文件将依赖您的 API，这使得更改变得更加困难。您希望能够自由地更改存储格式，而不会影响您的客户端。对代码进行分层，以便模块处理客户端 proto、存储 proto 或翻译。

Why? You might want to swap your underlying storage system. You might want to normalize—or denormalize—data differently. You might realize that parts of your client-exposed proto make sense to store in RAM while other parts make sense to go on disk.

​	为什么？您可能希望交换底层存储系统。您可能希望以不同的方式对数据进行规范化或非规范化。您可能会意识到，客户端公开的 proto 的某些部分适合存储在 RAM 中，而其他部分适合存储在磁盘上。

When it comes to protos nested one or more levels within a top-level request or response, the case for separating storage and wire protos isn’t as strong, and depends on how closely you’re willing to couple your clients to those protos.

​	对于嵌套在一个或多个级别内的顶级请求或响应中的 proto，分离存储和网络 proto 的理由并不那么充分，并且取决于您愿意将客户端与这些 proto 耦合到何种程度。

There’s a cost in maintaining the translation layer, but it quickly pays off once you have clients and have to do your first storage changes.

​	维护转换层需要成本，但一旦您拥有客户端并必须进行第一次存储更改，成本就会迅速得到补偿。

You might be tempted to share protos and diverge “when you need to.” With a perceived high cost to diverge and no clear place to put internal fields, your API will accrue fields clients either don’t understand or begin to depend on without your knowledge.

​	您可能倾向于共享 proto，并在“需要时”进行分歧。由于分歧的感知成本很高，并且没有明确的位置来放置内部字段，因此您的 API 将累积字段，而客户端要么不理解，要么在您不知情的情况下开始依赖这些字段。

By starting with separate proto files, your team will know where to add internal fields without polluting your API. In the early days, the wire proto can be tag-for-tag identical with an automatic translation layer (think: byte copying or proto reflection). Proto annotations can also power an automatic translation layer.

​	通过从单独的 proto 文件开始，您的团队将知道在哪里添加内部字段，而不会污染您的 API。在早期，网络 proto 可以与自动转换层完全相同（例如：字节复制或 proto 反射）。Proto 注释还可以支持自动转换层。

The following are exceptions to the rule:

​	以下是规则的例外情况：

- If the proto field is one of a common type, such as `google.type` or `google.protobuf`, then using that type both as storage and API is acceptable.
  
- 如果 proto 字段是常见类型之一，例如 `google.type` 或 `google.protobuf` ，那么将该类型同时用作存储和 API 是可以接受的。
  
- If your service is extremely performance-sensitive, it may be worth trading flexibility for execution speed. If your service doesn’t have millions of QPS with millisecond latency, you’re probably not the exception.
  
- 如果您的服务对性能极其敏感，那么值得用灵活性来换取执行速度。如果您的服务没有百万 QPS 和毫秒级延迟，那么您可能不是例外。
  
- If all of the following are true:
  
- 如果所有以下情况都为真：
  
  - your service *is* the storage system
  - 您的服务是存储系统
  - your system doesn’t make decisions based on your clients’ structured data
  - 您的系统不会根据客户端的结构化数据做出决策
  - your system simply stores, loads, and perhaps provides queries at your client’s request
  - 您的系统只是存储、加载，并可能根据客户端的请求提供查询
  
  Note that if you are implementing something like a logging system or a proto-based wrapper around a generic storages system, then you probably want to aim to have your clients’ messages transit into your storage backend as opaquely as possible so that you don’t create a dependency nexus. Consider using extensions or [Encode Opaque Data in Strings by Web-safe Encoding Binary Proto Serialization](https://protobuf.dev/programming-guides/api#encode-opaque-data-in-strings).
  
  ​	请注意，如果您正在实现类似日志记录系统或基于 proto 的通用存储系统包装器，那么您可能希望让客户端的消息尽可能不透明地传输到您的存储后端，以便您不会创建依赖关系。考虑使用扩展或通过网络安全编码二进制 Proto 序列化对字符串中的不透明数据进行编码。

## 对于突变，支持部分更新或仅追加更新，而不是完全替换 For Mutations, Support Partial Updates or Append-Only Updates, Not Full Replaces 

Don’t make an `UpdateFooRequest` that only takes a `Foo`.

​	不要做出只占 `UpdateFooRequest` 的 `Foo` 。

If a client doesn’t preserve unknown fields, they will not have the newest fields of `GetFooResponse` leading to data loss on a round-trip. Some systems don’t preserve unknown fields. Proto2 and proto3 implementations do preserve unknown fields unless the application drops the unknown fields explicitly. In general, public APIs should drop unknown fields on server-side to prevent security attack via unknown fields. For example, garbage unknown fields may cause a server to fail when it starts to use them as new fields in the future.

​	如果客户端不保留未知字段，它们将不会拥有 `GetFooResponse` 的最新字段，从而导致往返数据丢失。某些系统不保留未知字段。除非应用程序明确删除未知字段，否则 Proto2 和 proto3 实现会保留未知字段。通常，公共 API 应在服务器端删除未知字段，以防止通过未知字段进行安全攻击。例如，垃圾未知字段可能会导致服务器在将来开始将它们用作新字段时发生故障。

Absent documentation, handling of optional fields is ambiguous. Will `UpdateFoo` clear the field? That leaves you open to data loss when the client doesn’t know about the field. Does it not touch a field? Then how can clients clear the field? Neither are good.

​	在没有文档的情况下，对可选字段的处理是模棱两可的。 `UpdateFoo` 会清除该字段吗？当客户端不知道该字段时，这会使您面临数据丢失的风险。它不接触字段吗？那么客户端如何清除该字段？两者都不好。

### 修复方法 #1：使用更新字段掩码 Fix #1: Use an Update Field-mask 

Have your client pass which fields it wants to modify and include only those fields in the update request. Your server leaves other fields alone and updates only those specified by the mask. In general, the structure of your mask should mirror the structure of the response proto; that is, if `Foo` contains `Bar`, `FooMask` contains `BarMask`.

​	让您的客户端传递它想要修改的字段，并在更新请求中仅包含这些字段。您的服务器会保留其他字段，并且仅更新掩码指定的那些字段。通常，掩码的结构应反映响应原型的结构；也就是说，如果 `Foo` 包含 `Bar` ，则 `FooMask` 包含 `BarMask` 。

### 修复方法 2：公开更多更改各个部分的细粒度变异 Fix #2: Expose More Narrow Mutations That Change Individual Pieces 

For example, instead of `UpdateEmployeeRequest`, you might have: `PromoteEmployeeRequest`, `SetEmployeePayRequest`, `TransferEmployeeRequest`, etc.

​	例如，您可以使用 `PromoteEmployeeRequest` 、 `SetEmployeePayRequest` 、 `TransferEmployeeRequest` 等，而不是 `UpdateEmployeeRequest` 。

Custom update methods are easier to monitor, audit, and secure than a very flexible update method. They’re also easier to implement and call. A *large* number of them can increase the cognitive load of an API.

​	自定义更新方法比非常灵活的更新方法更容易监控、审核和保护。它们也更容易实现和调用。大量使用它们会增加 API 的认知负载。

## 不要在顶级请求或响应协议中包含基本类型 Don’t Include Primitive Types in a Top-level Request or Response Proto 

Many of the pitfalls described elsewhere in this doc are solved with this rule. For example:

​	本文档中其他位置描述的许多陷阱都可以通过此规则解决。例如：

Telling clients that a repeated field is unset in storage versus not-populated in this particular call can be done by wrapping the repeated field in a message.

​	通过将重复字段包装在消息中，可以告诉客户端重复字段在存储中未设置，而不是在此特定调用中未填充。

Common request options that are shared between requests naturally fall out of following this rule. Read and write field masks fall out of this.

​	在请求之间共享的常见请求选项自然会遵循此规则。读写字段掩码由此产生。

Your top-level proto should almost always be a container for other messages that can grow independently.

​	您的顶级协议几乎总是其他可以独立增长的消息的容器。

Even when you only need a single primitive type today, having it wrapped in a message gives you a clear path to expand that type and share the type among other methods that return the similar values. For example:

​	即使您今天只需要一个基本类型，将其包装在消息中也会为您提供一条清晰的路径来扩展该类型，并在返回类似值的其他方法之间共享该类型。例如：

```proto
message MultiplicationResponse {
  // Bad: What if you later want to return complex numbers and have an
  // AdditionResponse that returns the same multi-field type?
  optional double result;


  // Good: Other methods can share this type and it can grow as your
  // service adds new features (units, confidence intervals, etc.).
  optional NumericResult result;
}

message NumericResult {
  optional double real_value;
  optional double complex_value;
  optional UnitType units;
}
```

One exception to top-level primitives: Opaque strings (or bytes) that encode a proto but are only built and parsed on the server. Continuation tokens, version info tokens and IDs can all be returned as strings *if* the string is actually an encoding of a structured proto.

​	一个顶级原语的例外：不透明字符串（或字节），它对协议进行编码，但仅在服务器上构建和解析。如果字符串实际上是结构化协议的编码，则可以将延续令牌、版本信息令牌和 ID 全部作为字符串返回。

## 现在不要将布尔值用于具有两种状态但以后可能会有更多状态的内容 Never Use Booleans for Something That Has Two States Now, but Might Have More Later 

If you are using boolean for a field, make sure that the field is indeed describing just two possible states (for all time, not just now and the near future). Often, the flexibility of an enum, int, or message turns out to be worth it.

​	如果您对某个字段使用布尔值，请确保该字段确实只描述两种可能的状态（永远，而不仅仅是现在和不久的将来）。通常，枚举、整数或消息的灵活性被证明是值得的。

For example, in returning a stream of posts a developer may need to indicate whether a post should be rendered in two-columns or not based on the current mocks from UX. Even though a boolean is all that’s needed today, nothing prevents UX from introducing two-row posts, three-column posts or four-square posts in a future version.

​	例如，在返回帖子流时，开发人员可能需要根据 UX 的当前模型指示帖子是否应以两栏形式呈现。尽管现在只需要布尔值，但没有任何因素阻止 UX 在未来版本中引入两行帖子、三栏帖子或四方形帖子。

```proto
message GooglePlusPost {
  // Bad: Whether to render this post across two columns.
  optional bool big_post;

  // Good: Rendering hints for clients displaying this post.
  // Clients should use this to decide how prominently to render this
  // post. If absent, assume a default rendering.
  optional LayoutConfig layout_config;
}

message Photo {
  // Bad: True if it's a GIF.
  optional bool gif;

  // Good: File format of the referenced photo (for example, GIF, WebP, PNG).
  optional PhotoType type;
}
```

Be cautious about adding states to an enum that conflates concepts.

​	在将概念混为一谈的枚举中添加状态时要谨慎。

If a state introduces a new dimension to the enum or implies multiple application behaviors, you almost certainly want another field.

​	如果某个状态为枚举引入了新维度或暗示了多种应用程序行为，那么您几乎肯定需要另一个字段。

## 很少将整数字段用于 ID - Rarely Use an Integer Field for an ID 

It’s tempting to use an int64 as an identifier for an object. Opt instead for a string.

​	将 int64 用作对象的标识符很诱人。相反，选择使用字符串。

This lets you change your ID space if you need to and reduces the chance of collisions. 2^64 isn’t as big as it used to be.

​	这使您可以在需要时更改 ID 空间，并减少冲突的可能性。2^64 不再像以前那么大了。

You can also encode a structured identifier as a string which encourages clients to treat it as an opaque blob. You still must have a proto backing the string, but you can serialize the proto to a string field (encoded as web-safe Base64) which removes any of the internal details from the client-exposed API. In this case follow the guidelines [below](https://protobuf.dev/programming-guides/api/#encode-opaque-data-in-strings).

​	您还可以将结构化标识符编码为字符串，这会鼓励客户端将其视为不透明的二进制大对象。您仍然必须让一个 proto 支持该字符串，但您可以将 proto 序列化为字符串字段（编码为网络安全 Base64），这会从客户端公开的 API 中移除所有内部详细信息。在这种情况下，请遵循以下准则。

```proto
message GetFooRequest {
  // Which Foo to fetch.
  optional string foo_id;
}

// Serialized and websafe-base64-encoded into the GetFooRequest.foo_id field.
message InternalFooRef {
  // Only one of these two is set. Foos that have already been
  // migrated use the spanner_foo_id and Foos still living in
  // Caribou Storage Server have a classic_foo_id.
  optional bytes spanner_foo_id;
  optional int64 classic_foo_id;
}
```

If you start off with your own serialization scheme to represent your IDs as strings, things can get weird quickly. That’s why it’s often best to start with an internal proto that backs your string field.

​	如果您一开始就使用自己的序列化方案将 ID 表示为字符串，那么事情可能会很快变得奇怪。这就是为什么通常最好从支持字符串字段的内部 proto 开始。

## 不要在您希望客户端构建或解析的字符串中对数据进行编码 Don’t Encode Data in a String That You Expect a Client to Construct or Parse 

It’s less efficient over the wire, more work for the consumer of the proto, and confusing for someone reading your documentation. Your clients also have to wonder about the encoding: Are lists comma-separated? Did I escape this untrusted data correctly? Are numbers base-10? Better to have clients send an actual message or primitive type. It’s more compact over the wire and clearer for your clients.

​	通过网络传输效率较低，协议消费者需要做更多工作，并且对于阅读文档的人来说会感到困惑。您的客户端还必须考虑编码：列表是否以逗号分隔？我是否正确地转义了此不受信任的数据？数字是否为十进制？最好让客户端发送实际消息或原始类型。通过网络传输时更紧凑，并且对您的客户端来说更清晰。

This gets especially bad when your service acquires clients in several languages. Now each will have to choose the right parser or builder—or worse—write one.

​	当您的服务获取多种语言的客户端时，情况会变得尤其糟糕。现在，每个客户端都必须选择正确的解析器或构建器，或者更糟糕的是，编写一个解析器或构建器。

More generally, choose the right primitive type. See the Scalar Value Types table in the [Protocol Buffer Language Guide](https://protobuf.dev/programming-guides/proto#scalar).

​	更一般地说，选择正确的原始类型。请参阅《Protocol Buffer 语言指南》中的标量值类型表。

### 在前端协议中返回 HTML Returning HTML in a Front-End Proto 

With a JavaScript client, it’s tempting to return HTML or JSON in a field of your API. This is a slippery slope towards tying your API to a specific UI. Here are three concrete dangers:

​	使用 JavaScript 客户端时，很容易在 API 的字段中返回 HTML 或 JSON。这是将 API 绑定到特定 UI 的一个危险做法。这里有三个具体危险：

- A “scrappy” non-web client will end up parsing your HTML or JSON to get the data they want leading to fragility if you change formats and vulnerabilities if their parsing is bad.
- 一个“粗制滥造”的非 Web 客户端最终会解析您的 HTML 或 JSON 以获取所需的数据，如果您更改格式，则会导致脆弱性，如果其解析不当，则会导致漏洞。
- Your web-client is now vulnerable to an XSS exploit if that HTML is ever returned unsanitized.
- 如果该 HTML 未经过清理就返回，那么您的 Web 客户端现在容易受到 XSS 攻击。
- The tags and classes you’re returning expect a particular style-sheet and DOM structure. From release to release, that structure will change, and you risk a version-skew problem where the JavaScript client is older than the server and the HTML the server returns no longer renders properly on old clients. For projects that release often, this is not an edge case.
- 您返回的标签和类需要特定的样式表和 DOM 结构。从一个版本到另一个版本，该结构将会发生变化，并且您可能会遇到版本偏差问题，即 JavaScript 客户端比服务器旧，服务器返回的 HTML 在旧客户端上无法正常呈现。对于经常发布的项目，这不是一个极端情况。

Other than the initial page load, it’s usually better to return data and use client-side templating to construct HTML on the client .

​	除了初始页面加载之外，通常最好返回数据并使用客户端模板在客户端构建 HTML。

## 通过 Web 安全编码二进制 Proto 序列化对字符串中的不透明数据进行编码 Encode Opaque Data in Strings by Web-Safe Encoding Binary Proto Serialization 

If you do encode *opaque* data in a client-visible field (continuation tokens, serialized IDs, version infos, and so on), document that clients should treat it as an opaque blob. *Always use binary proto serialization, never text-format or something of your own devising for these fields.* When you need to expand the data encoded in an opaque field, you’ll find yourself reinventing protocol buffer serialization if you’re not already using it.

​	如果您确实在客户端可见字段（延续令牌、序列化 ID、版本信息等）中对不透明数据进行编码，请记录客户端应将其视为不透明 blob。始终使用二进制 proto 序列化，切勿对这些字段使用文本格式或您自己设计的内容。当您需要扩展不透明字段中编码的数据时，如果您尚未使用协议缓冲区序列化，您会发现自己正在重新发明它。

Define an internal proto to hold the fields that will go in the opaque field (even if you only need one field), serialize this internal proto to bytes then web-safe base-64 encode the result into your string field .

​	定义一个内部 proto 来保存将进入不透明字段的字段（即使您只需要一个字段），将此内部 proto 序列化为字节，然后将结果 web 安全 base-64 编码到您的字符串字段中。

One rare exception to using proto serialization: *Very* occasionally, the compactness wins from a carefully constructed alternative format are worth it.

​	使用 proto 序列化的一个罕见例外：非常偶尔地，紧凑性胜过精心构建的备用格式是值得的。

## 不要包含您的客户端不可能用到的字段 Don’t Include Fields that Your Clients Can’t Possibly Have a Use for 

The API you expose to clients should only be for describing how to interact with your system. Including anything else in it adds cognitive overhead to someone trying to understand it.

​	您向客户端公开的 API 应该仅用于描述如何与您的系统交互。在其中包含任何其他内容都会给试图理解它的人增加认知负担。

Returning debug data in response protos used to be a common practice, but we have a better way. RPC response extensions (also called “side channels”) let you describe your client interface with one proto and your debugging surface with another.

​	在响应 proto 中返回调试数据曾经是一种常见做法，但我们有更好的方法。RPC 响应扩展（也称为“辅助通道”）允许您使用一个 proto 来描述您的客户端界面，并使用另一个 proto 来描述您的调试界面。

Similarly, returning experiment names in response protos used to be a logging convenience–the unwritten contract was the client would send those experiments back on subsequent actions. The accepted way of accomplishing the same is to do log joining in the analysis pipeline.

​	同样，在响应 proto 中返回实验名称曾经是一种日志记录的便利——不成文的契约是客户端将在后续操作中发送回这些实验。实现相同目标的公认方法是在分析管道中执行日志联接。

One exception:

​	一个例外：

If you need continuous, real-time analytics *and* are on a small machine budget, running log joins might be prohibitive. In cases where cost is a deciding factor, denormalizing log data ahead of time can be a win. If you need log data round-tripped to you, send it to clients as an opaque blob and document the request and response fields.

​	如果您需要连续的实时分析并且机器预算很小，那么运行日志联接可能会受到限制。在成本是决定性因素的情况下，预先反规范化日志数据可能是一个胜利。如果您需要往返日志数据，请将其作为不透明的二进制大对象发送给客户端，并记录请求和响应字段。

**Caution:** If you need to return or round-trip hidden data on *every* request , you’re hiding the true cost of using your service and that’s not good either.

​	注意：如果您需要在每次请求时返回或往返隐藏数据，那么您就是在隐藏使用服务的真实成本，这也不好。

## 很少定义没有延续令牌的分页 API - *Rarely* Define a Pagination API Without a Continuation Token 

```proto
message FooQuery {
  // Bad: If the data changes between the first query and second, each of
  // these strategies can cause you to miss results. In an eventually
  // consistent world (that is, storage backed by Bigtable), it's not uncommon
  // to have old data appear after the new data. Also, the offset- and
  // page-based approaches all assume a sort-order, taking away some
  // flexibility.
  optional int64 max_timestamp_ms;
  optional int32 result_offset;
  optional int32 page_number;
  optional int32 page_size;

  // Good: You've got flexibility! Return this in a FooQueryResponse and
  // have clients pass it back on the next query.
  optional string next_page_token;
}
```

The best practice for a pagination API is to use an opaque continuation token (called next_page_token ) backed by an internal proto that you serialize and then `WebSafeBase64Escape` (C++) or `BaseEncoding.base64Url().encode` (Java). That internal proto could include many fields. The important thing is it buys you flexibility and–if you choose–it can buy your clients stability in the results.

​	分页 API 的最佳做法是使用不透明的延续令牌（称为 next_page_token ），该令牌由您序列化的内部协议支持，然后 `WebSafeBase64Escape` (C++) 或 `BaseEncoding.base64Url().encode` (Java)。该内部协议可以包含许多字段。重要的是，它为您带来了灵活性，如果您选择，它可以为您的客户端带来结果的稳定性。

Do not forget to validate the fields of this proto as untrustworthy inputs (see note in [Encode opaque data in strings](https://protobuf.dev/programming-guides/api/#encode-opaque-data-in-strings)).

​	不要忘记将此协议的字段验证为不可信输入（请参阅在字符串中对不透明数据进行编码中的注释）。

```proto
message InternalPaginationToken {
  // Track which IDs have been seen so far. This gives perfect recall at the
  // expense of a larger continuation token--especially as the user pages
  // back.
  repeated FooRef seen_ids;

  // Similar to the seen_ids strategy, but puts the seen_ids in a Bloom filter
  // to save bytes and sacrifice some precision.
  optional bytes bloom_filter;

  // A reasonable first cut and it may work for longer. Having it embedded in
  // a continuation token lets you change it later without affecting clients.
  optional int64 max_timestamp_ms;
}
```

## 将相关字段分组到新消息中。仅嵌套具有高内聚性的字段 Group Related Fields into a New Message. Nest Only Fields with High Cohesion 

```proto
message Foo {
  // Bad: The price and currency of this Foo.
  optional int price;
  optional CurrencyType currency;

  // Better: Encapsulates the price and currency of this Foo.
  optional CurrencyAmount price;
}
```

Only fields with high cohesion should be nested. If the fields are genuinely related, you’ll often want to pass them around together inside a server. That’s easier if they’re defined together in a message. Think:

​	只有内聚性高的字段才应该嵌套。如果字段确实相关，通常需要在服务器内一起传递它们。如果它们一起在消息中定义，则更容易。请考虑：

```java
CurrencyAmount calculateLocalTax(CurrencyAmount price, Location where)
```

If your CL introduces one field, but that field might have related fields later, preemptively put it in its own message to avoid this:

​	如果 CL 引入了某个字段，但该字段可能以后有相关字段，则预先将其放入自己的消息中以避免这种情况：

```proto
message Foo {
  // DEPRECATED! Use currency_amount.
  optional int price [deprecated = true];

  // The price and currency of this Foo.
  optional google.type.Money currency_amount;
}
```

The problem with a nested message is that while `CurrencyAmount` might be a popular candidate for reuse in other places of your API, `Foo.CurrencyAmount` might not. In the worst case, `Foo.CurrencyAmount` *is* reused, but `Foo`-specific fields leak into it.

​	嵌套消息的问题在于，虽然 `CurrencyAmount` 可能是 API 中其他地方的热门重用候选，但 `Foo.CurrencyAmount` 可能不是。最糟糕的情况下， `Foo.CurrencyAmount` 被重用，但 `Foo` 特定的字段泄漏到其中。

While [loose coupling](https://en.wikipedia.org/wiki/Loose_coupling) is generally accepted as a best practice when developing systems, that practice may not always apply when designing `.proto` files. There may be cases in which tightly coupling two units of information (by nesting one unit inside of the other) may make sense. For example, if you are creating a set of fields that appear fairly generic right now but which you anticipate adding specialized fields into at a later time, nesting the message would dissuade others from referencing that message from elsewhere in this or other `.proto` files.

​	虽然在开发系统时通常将松散耦合视为最佳实践，但设计 `.proto` 文件时可能并不总是适用该实践。在某些情况下，紧密耦合两个信息单元（通过将一个单元嵌套在另一个单元内）可能是有意义的。例如，如果您要创建一组现在看起来相当通用的字段，但您预计以后会添加专门的字段，则嵌套消息会阻止其他人从此处或其他 `.proto` 文件中的其他位置引用该消息。

```proto
message Photo {
  // Bad: It's likely PhotoMetadata will be reused outside the scope of Photo,
  // so it's probably a good idea not to nest it and make it easier to access.
  message PhotoMetadata {
    optional int32 width = 1;
    optional int32 height = 2;
  }
  optional PhotoMetadata metadata = 1;
}

message FooConfiguration {
  // Good: Reusing FooConfiguration.Rule outside the scope of FooConfiguration
  // tightly-couples it with likely unrelated components, nesting it dissuades
  // from doing that.
  message Rule {
    optional float multiplier = 1;
  }
  repeated Rule rules = 1;
}
```

## 在读取请求中包含字段读取掩码 Include a Field Read Mask in Read Requests 

```proto
// Recommended: use google.protobuf.FieldMask

// Alternative one:
message FooReadMask {
  optional bool return_field1;
  optional bool return_field2;
}

// Alternative two:
message BarReadMask {
  // Tag numbers of the fields in Bar to return.
  repeated int32 fields_to_return;
}
```

If you use the recommended `google.protobuf.FieldMask`, you can use the `FieldMaskUtil` ([Java](https://protobuf.dev/reference/java/api-docs/com/google/protobuf/util/FieldMaskUtil.html)/[C++](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.util.field_mask_util.md)) libraries to automatically filter a proto.

​	如果您使用推荐的 `google.protobuf.FieldMask` ，则可以使用 `FieldMaskUtil` （Java/C++）库自动过滤 proto。

Read masks set clear expectations on the client side, give them control of how much data they want back and allow the backend to only fetch data the client needs.

​	读取掩码在客户端设置明确的预期，让客户端控制他们想要返回的数据量，并允许后端仅获取客户端需要的数据。

The acceptable alternative is to always populate every field; that is, treat the request as if there were an implicit read mask with all fields set to true. This can get costly as your proto grows.

​	可接受的替代方法是始终填充每个字段；也就是说，将请求视为具有隐式读取掩码，其中所有字段都设置为 true。随着 proto 的增长，这可能会变得代价高昂。

The worst failure mode is to have an implicit (undeclared) read mask that varies depending on which method populated the message. This anti-pattern leads to apparent data loss on clients that build a local cache from response protos.

​	最糟糕的故障模式是具有隐式（未声明）读取掩码，该掩码会根据填充消息的方法而有所不同。这种反模式会导致在从响应 proto 构建本地缓存的客户端上出现明显的数据丢失。

## 包含版本字段以允许一致读取 Include a Version Field to Allow for Consistent Reads 

When a client does a write followed by a read of the same object, they expect to get back what they wrote–even when the expectation isn’t reasonable for the underlying storage system.

​	当客户端对同一对象执行写入操作后又执行读取操作时，他们希望取回他们写入的内容，即使对于底层存储系统而言，这种预期不合理。

Your server will read the local value and if the local version_info is less than the expected version_info, it will read from remote replicas to find the latest value. Typically version_info is a [proto encoded as a string](https://protobuf.dev/programming-guides/api/#encode-opaque-data-in-strings) that includes the datacenter the mutation went to and the timestamp at which it was committed.

​	您的服务器将读取本地值，如果本地 version_info 小于预期的 version_info，它将从远程副本中读取以查找最新值。通常，version_info 是一个编码为字符串的协议，其中包括突变发生的数据中心以及提交该突变时的时间戳。

Even systems backed by consistent storage often want a token to trigger the more expensive read-consistent path rather than incurring the cost on every read.

​	即使是受一致性存储支持的系统通常也需要一个令牌来触发更昂贵的读取一致性路径，而不是在每次读取时都产生成本。

## 对返回相同数据类型的 RPC 使用一致的请求选项 Use Consistent Request Options for RPCs that Return the Same Data Type 

An example failure pattern is the request options for a service in which each RPC returns the same data type, but has separate request options for specifying things like maximum comments, embeds supported types list, and so on.

​	一个示例故障模式是服务中每个 RPC 都返回相同数据类型的请求选项，但有单独的请求选项用于指定诸如最大评论数、嵌入支持的类型列表等内容。

The cost of approaching this ad hoc is increased complexity on the client from figuring out how to fill out each request and increased complexity on the server transforming the N request options into a common internal one. A not-small number of real-life bugs are traceable to this example.

​	采用这种临时方法的成本是客户端在弄清楚如何填写每个请求时复杂度增加，以及服务器在将 N 个请求选项转换为一个通用的内部选项时复杂度增加。相当多的现实生活中的错误可以追溯到此示例。

Instead, create a single, separate message to hold request options and include that in each of the top-level request messages. Here’s a better-practices example:

​	相反，创建一个单独的独立消息来保存请求选项，并将其包含在每个顶级请求消息中。这是一个最佳实践示例：

```proto
message FooRequestOptions {
  // Field-level read mask of which fields to return. Only fields that
  // were requested will be returned in the response. Clients should only
  // ask for fields they need to help the backend optimize requests.
  optional FooReadMask read_mask;

  // Up to this many comments will be returned on each Foo in the response.
  // Comments that are marked as spam don't count towards the maximum
  // comments. By default, no comments are returned.
  optional int max_comments_to_return;

  // Foos that include embeds that are not on this supported types list will
  // have the embeds down-converted to an embed specified in this list. If no
  // supported types list is specified, no embeds will be returned. If an embed
  // can't be down-converted to one of the supplied supported types, no embed
  // will be returned. Clients are strongly encouraged to always include at
  // least the THING_V2 embed type from EmbedTypes.proto.
  repeated EmbedType embed_supported_types_list;
}

message GetFooRequest {
  // What Foo to read. If the viewer doesn't have access to the Foo or the
  // Foo has been deleted, the response will be empty but will succeed.
  optional string foo_id;

  // Clients are required to include this field. Server returns
  // INVALID_ARGUMENT if FooRequestOptions is left empty.
  optional FooRequestOptions params;
}

message ListFooRequest {
  // Which Foos to return. Searches have 100% recall, but more clauses
  // impact performance.
  optional FooQuery query;

  // Clients are required to include this field. The server returns
  // INVALID_ARGUMENT if FooRequestOptions is left empty.
  optional FooRequestOptions params;
}
```

## 批处理/多阶段请求 Batch/multi-phase Requests 

Where possible, make mutations atomic. Even more important, make [mutations idempotent](https://protobuf.dev/programming-guides/api/#prefer-idempotency). A full retry of a partial failure shouldn’t corrupt/duplicate data.

​	尽可能使突变原子化。更重要的是，使突变具有幂等性。部分失败的完全重试不应损坏/复制数据。

Occasionally, you’ll need a single RPC that encapsulates multiple operations for performance reasons. What to do on a partial failure? If some succeeded and some failed, it’s best to let clients know.

​	有时，出于性能原因，您需要一个封装多个操作的单个 RPC。部分失败时该怎么办？如果有些成功而有些失败，最好让客户端知道。

Consider setting the RPC as failed and return details of both the successes and failures in an RPC status proto.

​	考虑将 RPC 设置为失败，并在 RPC 状态协议中返回成功和失败的详细信息。

In general, you want clients who are unaware of your handling of partial failures to still behave correctly and clients who are aware to get extra value.

​	通常，您希望不知道您如何处理部分失败的客户端仍然表现正确，而知道此情况的客户端则会获得额外价值。

## 创建返回或处理少量数据的方法，并期望客户端通过批处理多个此类请求来组合 UI Create Methods that Return or Manipulate Small Bits of Data and Expect Clients to Compose UIs from Batching Multiple Such Requests 

The ability to query many narrowly specified bits of data in a single round-trip allows a wider range of UX options without server changes by letting the client compose what they need.

​	通过让客户端组合他们需要的内容，能够在单次往返中查询许多狭窄指定的数据位，从而无需服务器更改即可提供更广泛的 UX 选项。

This is most relevant for front-end and middle-tier servers.

​	这与前端和中间层服务器最相关。

Many services expose their own batching API.

​	许多服务都公开了自己的批处理 API。

## 在移动端或 Web 端的替代方案是串行往返时进行一次性 RPC Make a One-off RPC when the Alternative is Serial Round-trips on Mobile or Web 

In cases where a *web or mobile* client needs to make two queries with a data dependency between them, the current best practice is to create a new RPC that protects the client from the round trip.

​	如果 Web 或移动端客户端需要进行两次查询，并且这两次查询之间存在数据依赖关系，那么当前的最佳做法是创建一个新的 RPC，以保护客户端免受往返影响。

In the case of mobile, it’s almost always worth saving your client the cost of an extra round-trip by bundling the two service methods together in one new one. For server-to-server calls, the case may not be as clear; it depends on how performance-sensitive your service is and how much cognitive overhead the new method introduces.

​	对于移动端而言，将两个服务方法捆绑到一个新方法中，几乎总是值得为客户端节省一次额外往返的成本。对于服务器到服务器的调用，情况可能并不那么明显；这取决于您的服务对性能有多敏感，以及新方法引入了多少认知开销。

## 创建重复字段消息，而不是标量或枚举 Make Repeated Fields Messages, Not Scalars or Enums 

A common evolution is that a single repeated field needs to become multiple related repeated fields. If you start with a repeated primitive your options are limited–you either create parallel repeated fields, or define a new repeated field with a new message that holds the values and migrate clients to it.

​	一个常见的演变是，单个重复字段需要变成多个相关的重复字段。如果您从重复基元开始，那么您的选择是有限的——您要么创建并行重复字段，要么使用包含这些值的新消息定义一个新的重复字段，并将客户端迁移到该字段。

If you start with a repeated message, evolution becomes trivial.

​	如果您从重复消息开始，那么演变将变得微不足道。

```proto
// Describes a type of enhancement applied to a photo
enum EnhancementType {
  ENHANCEMENT_TYPE_UNSPECIFIED;
  RED_EYE_REDUCTION;
  SKIN_SOFTENING;
}

message PhotoEnhancement {
  optional EnhancementType type;
}

message PhotoEnhancementReply {
  // Good: PhotoEnhancement can grow to describe enhancements that require
  // more fields than just an enum.
  repeated PhotoEnhancement enhancements;

  // Bad: If we ever want to return parameters associated with the
  // enhancement, we'd have to introduce a parallel array (terrible) or
  // deprecate this field and introduce a repeated message.
  repeated EnhancementType enhancement_types;
}
```

Imagine the following feature request: “We need to know which enhancements were performed by the user and which enhancements were automatically applied by the system.”

​	设想以下功能请求：“我们需要知道哪些增强功能是由用户执行的，哪些增强功能是由系统自动应用的。”

If the enhancement field in `PhotoEnhancementReply` were a scalar or enum, this would be much harder to support.

​	如果 `PhotoEnhancementReply` 中的增强字段是标量或枚举，则支持起来会困难得多。

This applies equally to maps. It is much easier to add additional fields to a map value if it’s already a message rather than having to migrate from `map<string, string>` to `map<string, MyProto>`.

​	这同样适用于映射。如果映射值已经是消息，则添加其他字段要容易得多，而不必从 `map<string, string>` 迁移到 `map<string, MyProto>` 。

One exception:

​	一个例外：

Latency-critical applications will find parallel arrays of primitive types are faster to construct and delete than a single array of messages; they can also be smaller over the wire if you use [[packed=true\]](https://protobuf.dev/programming-guides/encoding#packed) (eliding field tags). Allocating a fixed number of arrays is less work than allocating N messages. Bonus: in [Proto3](https://protobuf.dev/programming-guides/proto3), packing is automatic; you don’t need to explicitly specify it.

​	对于延迟关键型应用程序，发现基本类型的并行数组比单个消息数组构建和删除速度更快；如果您使用[packed=true]（省略字段标记），它们在网络上也可能更小。分配固定数量的数组比分配N条消息的工作量更少。奖励：在Proto3中，打包是自动的；您无需明确指定它。

## 使用Proto映射 Use Proto Maps 

Prior to the introduction in [Proto3](https://protobuf.dev/programming-guides/proto3) of [Proto3 maps](https://protobuf.dev/programming-guides/proto3#maps), services would sometimes expose data as pairs using an ad-hoc KVPair message with scalar fields. Eventually clients would need a deeper structure and would end up devising keys or values that need to be parsed in some way. See [Don’t encode data in a string](https://protobuf.dev/programming-guides/api/#dont-encode-data-in-a-string).

​	在Proto3中引入Proto3映射之前，服务有时会使用具有标量字段的临时KVPair消息将数据公开为对。最终，客户端将需要更深层次的结构，并最终设计出需要以某种方式解析的键或值。请参阅不要在字符串中对数据进行编码。

So, using a (extensible) message type for the value is an immediate improvement over the naive design.

​	因此，对值使用（可扩展的）消息类型是比朴素设计立即改进。

Maps were back-ported to proto2 in all languages, so using `map<scalar, **message**>` is better than inventing your own KVPair for the same purpose[1](https://protobuf.dev/programming-guides/api/#fn:1).

​	Maps 已在所有语言中反向移植到 proto2，因此使用 `map<scalar, **message**>` 比出于相同目的发明自己的 KVPair 更好 [1](https://protobuf.dev/programming-guides/api/#fn:1) 。

If you want to represent *arbitrary* data whose structure you don’t know ahead of time, use [`google.protobuf.Any`](https://protobuf.dev/reference/protobuf/textformat-spec#any).

​	如果您想表示您事先不知道其结构的任意数据，请使用 `google.protobuf.Any` 。

## 更喜欢幂等性 Prefer Idempotency 

Somewhere in the stack above you, a client may have retry logic. If the retry is a mutation, the user could be in for a surprise. Duplicate comments, build requests, edits, and so on aren’t good for anyone.

​	在您上方的某个堆栈中，客户端可能具有重试逻辑。如果重试是突变，用户可能会感到惊讶。重复的评论、构建请求、编辑等对任何人都不好。

A simple way to avoid duplicate writes is to allow clients to specify a client-created request ID that your server dedupes on (for example, hash of content or UUID).

​	避免重复写入的一个简单方法是允许客户端指定服务器对其实施去重处理的客户端创建的请求 ID（例如，内容或 UUID 的哈希值）。

## 注意您的服务名称，并使其全局唯一 Be Mindful of Your Service Name, and Make it Globally Unique 

A service name (that is, the part after the `service` keyword in your `.proto` file) is used in surprisingly many places, not just to generate the service class name. This makes this name more important than one might think.

​	服务名称（即 `.proto` 文件中 `service` 关键字后面的部分）在很多地方使用，而不仅仅是生成服务类名称。这使得此名称比人们想象的更重要。

What’s tricky is that these tools make the implicit assumption that your service name is unique across a network . Worse, the service name they use is the *unqualified* service name (for example, `MyService`), not the qualified service name (for example, `my_package.MyService`).

​	棘手的是，这些工具会隐式假设您的服务名称在网络中是唯一的。更糟糕的是，它们使用的服务名称是不合格的服务名称（例如， `MyService` ），而不是合格的服务名称（例如， `my_package.MyService` ）。

For this reason, it makes sense to take steps to prevent naming conflicts on your service name, even if it is defined inside a specific package. For example, a service named `Watcher` is likely to cause problems; something like `MyProjectWatcher` would be better.

​	因此，即使是在特定软件包内定义服务名称，采取措施防止服务名称命名冲突也是合理的。例如，名为 `Watcher` 的服务可能会导致问题；类似 `MyProjectWatcher` 的名称会更好。

## 确保每个 RPC 指定并强制执行（宽松的）截止时间 Ensure Every RPC Specifies and Enforces a (Permissive) Deadline 

By default, an RPC does not have a timeout. Since a request may tie up backend resources that are only released on completion, setting a default deadline that allows all well-behaving requests to finish is a good defensive practice. Not enforcing one has in the past caused severe problems for major services . RPC clients should still set a deadline on outgoing RPCs and will typically do so by default when they use standard frameworks. A deadline may and typically will be overwritten by a shorter deadline attached to a request.

​	默认情况下，RPC 没有超时时间。由于请求可能会占用仅在完成时才会释放的后端资源，因此设置一个允许所有行为良好的请求完成的默认截止时间是一种良好的防御性做法。过去，不强制执行此操作已给主要服务造成了严重问题。RPC 客户端仍应为传出的 RPC 设置截止时间，并且在使用标准框架时通常会默认这样做。截止时间可能会被附加到请求的更短截止时间覆盖，并且通常会这样做。

Setting the `deadline` option clearly communicates the RPC deadline to your clients, and is respected and enforced by standard frameworks:

​	设置 `deadline` 选项可以清楚地向您的客户端传达 RPC 截止时间，并且受到标准框架的尊重和强制执行：

```proto
rpc Foo(FooRequest) returns (FooResponse) {
  option deadline = x; // there is no globally good default
}
```

Choosing a deadline value will especially impact how your system acts under load. For existing services, it is critical to evaluate existing client behavior before enforcing new deadlines to avoid breaking clients (consult SRE). In some cases, it may not be possible to enforce a shorter deadline after the fact.

​	选择截止时间值将特别影响您的系统在负载下的行为。对于现有服务，在强制执行新截止时间之前评估现有客户端行为以避免破坏客户端（咨询 SRE）至关重要。在某些情况下，可能无法在事后强制执行更短的截止时间。

## 请求和响应大小的限制 Bound Request and Response Sizes 

Request and response sizes should be bounded. We recommend a bound in the ballpark of 8 MiB, and 2 GiB is a hard limit at which many proto implementations break . Many storage systems have a limit on message sizes .

​	请求和响应大小应受到限制。我们建议限制在 8 MiB 左右，而 2 GiB 是许多 proto 实现中断的硬限制。许多存储系统对消息大小有限制。

Also, unbounded messages

​	此外，无限制的消息

- bloat both client and server,
- 会使客户端和服务器都变得臃肿，
- cause high and unpredictable latency,
- 导致高且不可预测的延迟，
- decrease resiliency by relying on a long-lived connection between a single client and a single server.
- 通过依赖单个客户端和单个服务器之间的长期连接来降低弹性。

Here are a few ways to bound all messages in an API:

​	以下是一些在 API 中限制所有消息的方法：

- Define RPCs that return bounded messages, where each RPC call is logically independent from the others.
- 定义返回受限消息的 RPC，其中每个 RPC 调用在逻辑上独立于其他调用。
- Define RPCs that operate on a single object, instead of on an unbounded, client-specified list of objects.
- 定义针对单个对象（而不是针对客户端指定的对象的无界列表）进行操作的 RPC。
- Avoid encoding unbounded data in string, byte, or repeated fields.
- 避免在字符串、字节或重复字段中对无界数据进行编码。
- Define a long-running operation . Store the result in a storage system designed for scalable, concurrent reads .
- 定义一项长期运行的操作。将结果存储在专为可扩展并发读取而设计的存储系统中。
- Use a pagination API (see [Rarely define a pagination API without a continuation token](https://protobuf.dev/programming-guides/api/#define-pagination-api)).
- 使用分页 API（请参阅很少在没有延续令牌的情况下定义分页 API）。
- Use streaming RPCs.
- 使用流式 RPC。

If you are working on a UI, see also [Create methods that return or manipulate small bits of data](https://protobuf.dev/programming-guides/api/#create-methods-manipulate-small-bits).

​	如果您正在处理 UI，请参阅创建返回或处理少量数据的方法。

## 谨慎传播状态代码 Propagate Status Codes Carefully 

RPC services should take care at RPC boundaries to interrogate errors, and return meaningful status errors to their callers.

​	RPC 服务应在 RPC 边界处小心检查错误，并向调用方返回有意义的状态错误。

Let’s examine a toy example to illustrate the point:

​	我们来看一个玩具示例来说明这一点：

Consider a client that calls `ProductService.GetProducts`, which takes no arguments. As part of `GetProducts`, `ProductService` might get all the products, and call `LocaleService.LocaliseNutritionFacts` for each product.

​	考虑一个调用 `ProductService.GetProducts` 的客户端，它不接受任何参数。作为 `GetProducts` 的一部分， `ProductService` 可能会获取所有产品，并针对每个产品调用 `LocaleService.LocaliseNutritionFacts` 。

```fallback
digraph toy_example {
  node [style=filled]
  client [label="Client"];
  product [label="ProductService"];
  locale [label="LocaleService"];
  client -> product [label="GetProducts"]
  product -> locale [label="LocaliseNutritionFacts"]
}
```

If `ProductService` is incorrectly implemented, it might send the wrong arguments to `LocaleService`, resulting in an `INVALID_ARGUMENT`.

​	如果 `ProductService` 实现不正确，它可能会向 `LocaleService` 发送错误的参数，从而导致 `INVALID_ARGUMENT` 。

If `ProductService` carelessly returns errors to its callers, the client will receive `INVALID_ARGUMENT`, since status codes propagate across RPC boundaries. But, the client didn’t pass any arguments to `ProductService.GetProducts`. So, the error is worse than useless: it will cause a great deal of confusion!

​	如果 `ProductService` 粗心地向其调用者返回错误，客户端将收到 `INVALID_ARGUMENT` ，因为状态代码会跨 RPC 边界传播。但是，客户端并未向 `ProductService.GetProducts` 传递任何参数。因此，错误比无用更糟糕：它会导致极大的混乱！

Instead, `ProductService` should interrogate errors it receives at the RPC boundary; that is, the `ProductService` RPC handler it implements. It should return meaningful errors to users: if *it received* invalid arguments from the caller, it should return `INVALID_ARGUMENT`. If *something downstream* received invalid arguments, it should convert the `INVALID_ARGUMENT` to `INTERNAL` before returning the error to the caller.

​	相反， `ProductService` 应该询问它在 RPC 边界收到的错误；也就是说，它实现的 `ProductService` RPC 处理程序。它应该向用户返回有意义的错误：如果它从调用者那里收到无效参数，它应该返回 `INVALID_ARGUMENT` 。如果下游某个内容收到无效参数，它应该在将错误返回给调用者之前将 `INVALID_ARGUMENT` 转换为 `INTERNAL` 。

Carelessly propagating status errors leads to confusion, which can be very expensive to debug. Worse, it can lead to an invisible outage where every service forwards a client error without causing any alerts to happen .

​	粗心地传播状态错误会导致混乱，而调试起来可能非常昂贵。更糟糕的是，它可能导致无形的停机，其中每个服务都会转发客户端错误，而不会导致任何警报发生。

The general rule is: at RPC boundaries, take care to interrogate errors, and return meaningful status errors to callers, with appropriate status codes. To convey meaning, each RPC method should document what error codes it returns in which circumstances. The implementation of each method should conform to the documented API contract.

​	一般规则是：在 RPC 边界，注意询问错误，并向调用方返回有意义的状态错误，并带有适当的状态代码。为了传达含义，每个 RPC 方法都应记录在何种情况下返回哪些错误代码。每个方法的实现应符合记录的 API 契约。

## 附录 Appendix 

### 返回重复字段 Returning Repeated Fields 

When a repeated field is empty, the client can’t tell if the field just wasn’t populated by the server or if the backing data for the field is genuinely empty. In other words, there’s no `hasFoo` method for repeated fields.

​	当重复字段为空时，客户端无法判断该字段只是未由服务器填充还是该字段的支持数据确实为空。换句话说，对于重复字段没有 `hasFoo` 方法。

Wrapping a repeated field in a message is an easy way to get a hasFoo method.

​	将重复字段包装在消息中是一种获取 hasFoo 方法的简单方法。

```proto
message FooList {
  repeated Foo foos;
}
```

The more holistic way to solve it is with a field [read mask](https://protobuf.dev/programming-guides/api/#include-field-read-mask). If the field was requested, an empty list means there’s no data. If the field wasn’t requested the client should ignore the field in the response.

​	解决此问题的更全面方法是使用字段读取掩码。如果请求了该字段，则空列表表示没有数据。如果未请求该字段，则客户端应忽略响应中的该字段。

### 更新重复字段 Updating Repeated Fields 

The worst way to update a repeated field is to force the client to supply a replacement list. The dangers with forcing the client to supply the entire array are manyfold. Clients that don’t preserve unknown fields cause data loss. Concurrent writes cause data loss. Even if those problems don’t apply, your clients will need to carefully read your documentation to know how the field is interpreted on the server side. Does an empty field mean the server won’t update it, or that the server will clear it?

​	更新重复字段的最糟糕方法是强制客户端提供替换列表。强制客户端提供整个数组的危险是多方面的。不保留未知字段的客户端会导致数据丢失。并发写入会导致数据丢失。即使这些问题不适用，您的客户端也需要仔细阅读您的文档，以了解字段在服务器端是如何解释的。空字段是否意味着服务器不会更新它，或者服务器会清除它？

**Fix #1**: Use a repeated update mask that permits the client to replace, delete, or insert elements into the array without supplying the entire array on a write.

​	修复 #1：使用重复更新掩码，允许客户端在写入时替换、删除或向数组中插入元素，而无需提供整个数组。

**Fix #2**: Create separate append, replace, delete arrays in the request proto.

​	修复 #2：在请求协议中创建单独的追加、替换、删除数组。

**Fix #3**: Allow only appending or clearing. You can do this by wrapping the repeated field in a message. A present, but empty, message means clear, otherwise, any repeated elements mean append.

​	修复 #3：仅允许追加或清除。您可以通过将重复字段包装在消息中来做到这一点。一个存在但为空的消息表示清除，否则，任何重复的元素都表示追加。

### 重复字段中的顺序独立性 Order Independence in Repeated Fields 

*Try* to avoid order dependence in general. It’s an extra layer of fragility. An especially bad type of order dependence is parallel arrays. Parallel arrays make it more difficult for clients to interpret the results and make it unnatural to pass the two related fields around inside your own service.

​	一般来说，尽量避免顺序依赖。这是脆弱性的额外一层。一种特别糟糕的顺序依赖类型是并行数组。并行数组让客户端更难解释结果，并且在您自己的服务中传递两个相关字段变得不自然。

```proto
message BatchEquationSolverResponse {
  // Bad: Solved values are returned in the order of the equations given in
  // the request.
  repeated double solved_values;
  // (Usually) Bad: Parallel array for solved_values.
  repeated double solved_complex_values;
}

// Good: A separate message that can grow to include more fields and be
// shared among other methods. No order dependence between request and
// response, no order dependence between multiple repeated fields.
message BatchEquationSolverResponse {
  // Deprecated, this will continue to be populated in responses until Q2
  // 2014, after which clients must move to using the solutions field below.
  repeated double solved_values [deprecated = true];

  // Good: Each equation in the request has a unique identifier that's
  // included in the EquationSolution below so that the solutions can be
  // correlated with the equations themselves. Equations are solved in
  // parallel and as the solutions are made they are added to this array.
  repeated EquationSolution solutions;
}
```

### 由于您的 Proto 在移动版本中而泄露功能 Leaking Features Because Your Proto is in a Mobile Build 

Android and iOS runtimes both support reflection. To do that, the unfiltered names of fields and messages are embedded in the application binary (APK, IPA) as strings.

​	Android 和 iOS 运行时都支持反射。为此，字段和消息的未过滤名称作为字符串嵌入到应用程序二进制文件 (APK、IPA) 中。

```proto
message Foo {
  // This will leak existence of Google Teleport project on Android and iOS
  optional FeatureStatus google_teleport_enabled;
}
```

Several mitigation strategies:

​	几种缓解策略：

- ProGuard obfuscation on Android. As of Q3 2014. iOS has no obfuscation option: once you have the IPA on a desktop, piping it through `strings` will reveal field names of included protos. [iOS Chrome tear-down](https://github.com/Bensge/Chrome-for-iOS-Headers)
- Android 上的 ProGuard 混淆。截至 2014 年第三季度。iOS 没有混淆选项：一旦您在桌面上拥有 IPA，通过 `strings` 传递它将显示所包含 proto 的字段名称。iOS Chrome 拆解
- Curate precisely which fields are sent to mobile clients .
- 精心策划哪些字段发送到移动客户端。
- If plugging the leak isn’t feasible on an acceptable timescale, get buy-in from the feature owner to risk it.
- 如果在可接受的时间范围内无法堵住泄漏，请获得功能所有者的认可以承担风险。

*Never* use this as an excuse to obfuscate the meaning of a field with a code-name. Either plug the leak or get buy-in to risk it.

切勿以此为借口使用代号来混淆字段的含义。堵住泄漏或获得认可以承担风险。

### 性能优化 Performance Optimizations 

You can trade type safety or clarity for performance wins in some cases. For example, a proto with hundreds of fields–particularly message-type fields–is going to be slower to parse than one with fewer fields. A very deeply-nested message can be slow to deserialize just from the memory management. A handful of techniques teams have used to speed deserialization:

​	在某些情况下，您可以用类型安全或清晰度来换取性能提升。例如，一个包含数百个字段的 proto（尤其是消息类型字段）比包含较少字段的 proto 解析起来要慢。一个非常深层嵌套的消息可能仅仅由于内存管理而导致反序列化速度变慢。团队用来加速反序列化的几种技术：

- Create a parallel, trimmed proto that mirrors the larger proto but has only some of the tags declared. Use this for parsing when you don’t need all the fields. Add tests to enforce that tag numbers continue to match as the trimmed proto accumulates numbering “holes.”
- 创建一个并行的、经过修剪的 proto，它镜像较大的 proto，但仅声明了部分标记。当您不需要所有字段时，使用此 proto 进行解析。添加测试以强制标记号继续匹配，因为修剪后的 proto 会累积编号“空洞”。
- Annotate the fields as “lazily parsed” with [[lazy=true\]](https://github.com/protocolbuffers/protobuf/blob/cacb096002994000f8ccc6d9b8e1b5b0783ee561/src/google/protobuf/descriptor.proto#L609).
- 使用 [lazy=true] 将字段注释为“延迟解析”。
- Declare a field as bytes and document its type. Clients who care to parse the field can do so manually. The danger with this approach is there’s nothing preventing someone from putting a message of the wrong type in the bytes field. You should never do this with a proto that’s written to any logs, as it prevents the proto from being vetted for PII or scrubbed for policy or privacy reasons.
- 将字段声明为字节并记录其类型。关心解析字段的客户端可以手动执行此操作。这种方法的危险之处在于，没有任何东西可以阻止某人将错误类型的消息放入字节字段中。您绝不应将此方法用于写入任何日志的 proto，因为它会阻止对 proto 进行 PII 审查或出于政策或隐私原因进行清除。

------

1. A gotcha with protos that contain `map<k,v>` fields. Don’t use them as reduce keys in a MapReduce. The wire format and iteration order of proto3 map items are *unspecified* which leads to inconsistent map shards. [↩︎](https://protobuf.dev/programming-guides/api/#fnref:1)
2. 包含 `map<k,v>` 字段的协议存在一个问题。不要在 MapReduce 中将它们用作归约键。proto3 映射项的线格式和迭代顺序未指定，这会导致映射分片不一致。↩︎
