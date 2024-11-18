+++
title = "API 最佳实践"
date = 2024-11-17T09:35:36+08:00
weight = 160
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

> 原文：[https://protobuf.dev/programming-guides/api/](https://protobuf.dev/programming-guides/api/)
>
> 收录该文档的时间：`2024-11-17T09:35:36+08:00`

# API Best Practices - API 最佳实践

A future-proof API is surprisingly hard to get right. The suggestions in this document make trade-offs to favor long-term, bug-free evolution.

​	设计一个面向未来的 API 非常困难。本文件中的建议旨在权衡长期的无错误演化。

Updated for proto3. Patches welcome!

​	更新至 proto3。如有改进建议，欢迎提交补丁！

This doc is a complement to [Proto Best Practices](https://protobuf.dev/programming-guides/dos-donts). It’s not a prescription for Java/C++/Go and other APIs.

​	本文是 [Proto 最佳实践](https://protobuf.dev/programming-guides/dos-donts) 的补充内容，而非 Java/C++/Go 等 API 的使用规范。

If you see a proto straying from these guidelines in a code review, point the author to this topic and help spread the word.

​	如果在代码评审中发现 proto 偏离了这些指南，请指引作者查看此主题并帮助传播这些规范。

> Note
>
> These guidelines are just that and many have documented exceptions. For example, if you’re writing a performance-critical backend, you might want to sacrifice flexibility or safety for speed. This topic will help you better understand the trade-offs and make a decision that works for your situation.
>
> ​	这些指南并非硬性规定，其中许多有明确的例外情况。例如，如果您正在编写一个性能关键的后端，可能需要为了速度牺牲灵活性或安全性。本文将帮助您更好地理解这些权衡，并为您的具体场景做出合理决策。

## 精确简洁地记录大多数字段和消息 Precisely, Concisely Document Most Fields and Messages

Chances are good your proto will be inherited and used by people who don’t know what you were thinking when you wrote or modified it. Document each field in terms that will be useful to a new team-member or client with little knowledge of your system.

​	您的 proto 可能会被那些不了解您编写或修改初衷的人继承和使用。以对新团队成员或对系统了解不多的客户有用的方式记录每个字段。

Some concrete examples:

​	以下是一些具体示例：

```proto
// Bad: Option to enable Foo
// 错误示例：启用 Foo 的选项
// Good: Configuration controlling the behavior of the Foo feature.
// 正确示例：控制 Foo 功能行为的配置。
message FeatureFooConfig {
  // Bad: Sets whether the feature is enabled
  // Good: Required field indicating whether the Foo feature
  // is enabled for account_id.  Must be false if account_id's
  // FOO_OPTIN Gaia bit is not set.
  // 错误示例：设置是否启用功能
  // 正确示例：必填字段，指示是否为 account_id 启用了 Foo 功能。
  // 如果 account_id 的 FOO_OPTIN Gaia 位未设置，则必须为 false。
  optional bool enabled;
}

// Bad: Foo object.
// Good: Client-facing representation of a Foo (what/foo) exposed in APIs.
// 错误示例：Foo 对象。
// 正确示例：API 中公开的面向客户的 Foo 表示。
message Foo {
  // Bad: Title of the foo.
  // Good: Indicates the user-supplied title of this Foo, with no
  // normalization or escaping.
  // An example title: "Picture of my cat in a box <3 <3 !!!"
  // 错误示例：Foo 的标题。
  // 正确示例：表示用户提供的 Foo 标题，不做规范化或转义。
  // 示例标题："Picture of my cat in a box <3 <3 !!!"
  optional string title [(max_length) = 512];
}

// Bad: Foo config.
// Less-Bad: If the most useful comment is re-stating the name, better to omit
// the comment.
// 错误示例：Foo 配置。
// 较好示例：如果最有用的注释仅重述名称，则最好省略该注释。
FooConfig foo_config = 3;
```

Document the constraints, expectations and interpretation of each field in as few words as possible.

​	请用尽可能少的文字记录每个字段的约束、期望和解释。

You can use custom proto annotations. See [Custom Options](https://protobuf.dev/programming-guides/proto2#options) to define cross-language constants like `max_length` in the example above. Supported in proto2 and proto3.

​	您可以使用自定义 proto 注释。请参阅 [自定义选项](https://protobuf.dev/programming-guides/proto2#options)，以定义跨语言的常量，例如上例中的 `max_length`。支持 proto2 和 proto3。

Over time, documentation of an interface can get longer and longer. The length takes away from the clarity. When the documentation is genuinely unclear, fix it, but look at it holistically and aim for brevity.

​	随着时间推移，接口文档可能会变得越来越长，影响清晰度。如果文档确实不清楚，请修复它，但要整体考虑并追求简洁。

## 区分传输消息和存储消息 Use Different Messages for Wire and Storage

If a top-level proto you expose to clients is the same one you store on disk, you’re headed for trouble. More and more binaries will depend on your API over time, making it harder to change. You’ll want the freedom to change your storage format without impacting your clients. Layer your code so that modules deal either with client protos, storage protos, or translation.

​	如果你向客户端公开的顶层 proto 与存储在磁盘上的 proto 相同，那就会带来麻烦。随着时间的推移，越来越多的二进制文件将依赖于你的 API，这将使更改变得更加困难。你需要有自由更改存储格式而不影响客户端的能力。将代码分层，使模块分别处理客户端 proto、存储 proto 或转换逻辑。

Why? You might want to swap your underlying storage system. You might want to normalize—or denormalize—data differently. You might realize that parts of your client-exposed proto make sense to store in RAM while other parts make sense to go on disk.

​	为什么？你可能希望更换底层存储系统。你可能需要以不同的方式规范化或去规范化数据。你可能会发现，某些向客户端公开的 proto 部分更适合存储在 RAM 中，而其他部分更适合存储在磁盘上。

When it comes to protos nested one or more levels within a top-level request or response, the case for separating storage and wire protos isn’t as strong, and depends on how closely you’re willing to couple your clients to those protos.

​	对于嵌套在顶层请求或响应中的 proto，一层或多层的情况下，将存储和传输 proto 分离的必要性不那么强，这取决于你愿意将客户端与这些 proto 紧密耦合的程度。

There’s a cost in maintaining the translation layer, but it quickly pays off once you have clients and have to do your first storage changes.

​	维护转换层是有成本的，但一旦你有了客户端并需要进行第一次存储更改时，这个成本就很快会得到回报。

You might be tempted to share protos and diverge “when you need to.” With a perceived high cost to diverge and no clear place to put internal fields, your API will accrue fields clients either don’t understand or begin to depend on without your knowledge.

​	你可能会倾向于“在需要时”共享 proto 并进行分歧。然而，由于分歧的感知成本较高且没有明确的位置来放置内部字段，你的 API 会累积一些客户端无法理解或在你不知情的情况下开始依赖的字段。

By starting with separate proto files, your team will know where to add internal fields without polluting your API. In the early days, the wire proto can be tag-for-tag identical with an automatic translation layer (think: byte copying or proto reflection). Proto annotations can also power an automatic translation layer.

​	从单独的 proto 文件开始，你的团队将知道在哪里添加内部字段而不会污染你的 API。在早期，传输 proto 可以通过自动转换层（例如字节复制或 proto 反射）实现逐字段一致。Proto 注释也可以支持自动转换层。

The following are exceptions to the rule:

​	以下是规则的例外情况：

- If the proto field is one of a common type, such as `google.type` or `google.protobuf`, then using that type both as storage and API is acceptable.

  - 如果 proto 字段是 `google.type` 或 `google.protobuf` 等常见类型之一，那么同时将其用作存储和 API 是可以接受的。

- If your service is extremely performance-sensitive, it may be worth trading flexibility for execution speed. If your service doesn’t have millions of QPS with millisecond latency, you’re probably not the exception.

  - 如果你的服务对性能极为敏感，可能值得用灵活性换取执行速度。如果你的服务没有数百万 QPS 和毫秒级延迟，你可能不属于例外情况。

- If all of the following are true: 如果满足以下所有条件：

  - your service *is* the storage system
    - 你的服务*就是*存储系统。

  - your system doesn’t make decisions based on your clients’ structured data
    - 你的系统不基于客户端的结构化数据做出决策。

  - your system simply stores, loads, and perhaps provides queries at your client’s request
    - 你的系统只是存储、加载，并可能按客户端请求提供查询。


  Note that if you are implementing something like a logging system or a proto-based wrapper around a generic storages system, then you probably want to aim to have your clients’ messages transit into your storage backend as opaquely as possible so that you don’t create a dependency nexus. Consider using extensions or [Encode Opaque Data in Strings by Web-safe Encoding Binary Proto Serialization](https://protobuf.dev/programming-guides/api#encode-opaque-data-in-strings).

  ​	请注意，如果你正在实现类似日志记录系统或基于 proto 的通用存储系统包装器，那么你可能希望让客户端的消息尽可能透明地传递到你的存储后端，以避免创建依赖关系网络。考虑使用扩展或[通过网络安全编码二进制 Proto 序列化将不透明数据编码到字符串中](https://protobuf.dev/programming-guides/api#encode-opaque-data-in-strings)。

## 对于修改，支持部分更新或仅附加更新，而不是完全替换 For Mutations, Support Partial Updates or Append-Only Updates, Not Full Replaces

Don’t make an `UpdateFooRequest` that only takes a `Foo`.

​	不要创建仅接收 `Foo` 的 `UpdateFooRequest`。

If a client doesn’t preserve unknown fields, they will not have the newest fields of `GetFooResponse` leading to data loss on a round-trip. Some systems don’t preserve unknown fields. Proto2 and proto3 implementations do preserve unknown fields unless the application drops the unknown fields explicitly. In general, public APIs should drop unknown fields on server-side to prevent security attack via unknown fields. For example, garbage unknown fields may cause a server to fail when it starts to use them as new fields in the future.

​	如果客户端不保留未知字段，它们将不会拥有 `GetFooResponse` 的最新字段，从而导致往返过程中数据丢失。一些系统不会保留未知字段。Proto2 和 Proto3 实现会保留未知字段，除非应用程序明确删除未知字段。一般来说，公共 API 应在服务器端删除未知字段，以防止通过未知字段进行的安全攻击。例如，垃圾未知字段可能会导致服务器在开始将其用作新字段时出现故障。

Absent documentation, handling of optional fields is ambiguous. Will `UpdateFoo` clear the field? That leaves you open to data loss when the client doesn’t know about the field. Does it not touch a field? Then how can clients clear the field? Neither are good.

​	没有文档的情况下，处理可选字段是不明确的。`UpdateFoo` 是否会清除该字段？如果会，这将导致当客户端不知道该字段时的数据丢失。是否不触及字段？那么客户端如何清除该字段？两种情况都不好。

### 

### Fix #1: 使用更新字段掩码 Use an Update Field-mask

Have your client pass which fields it wants to modify and include only those fields in the update request. Your server leaves other fields alone and updates only those specified by the mask. In general, the structure of your mask should mirror the structure of the response proto; that is, if `Foo` contains `Bar`, `FooMask` contains `BarMask`.

​	让你的客户端传递它想要修改的字段，并仅在更新请求中包含这些字段。你的服务器保持其他字段不变，只更新掩码中指定的字段。通常，掩码的结构应与响应 proto 的结构相匹配；即，如果 `Foo` 包含 `Bar`，则 `FooMask` 包含 `BarMask`。

### 

### Fix #2: 公开更窄范围的变更，仅更改个别部分 Expose More Narrow Mutations That Change Individual Pieces

For example, instead of `UpdateEmployeeRequest`, you might have: `PromoteEmployeeRequest`, `SetEmployeePayRequest`, `TransferEmployeeRequest`, etc.

​	例如，替代 `UpdateEmployeeRequest`，你可能会有：`PromoteEmployeeRequest`、`SetEmployeePayRequest`、`TransferEmployeeRequest` 等。

Custom update methods are easier to monitor, audit, and secure than a very flexible update method. They’re also easier to implement and call. A *large* number of them can increase the cognitive load of an API.

​	自定义更新方法比一个非常灵活的更新方法更容易监控、审计和保护。它们也更容易实现和调用。*大量*此类方法可能会增加 API 的认知负担。

## 不要在顶层请求或响应 proto 中包含原始类型 Don’t Include Primitive Types in a Top-level Request or Response Proto

Many of the pitfalls described elsewhere in this doc are solved with this rule. For example:

​	此文档中描述的许多陷阱都可以通过此规则解决。例如：

Telling clients that a repeated field is unset in storage versus not-populated in this particular call can be done by wrapping the repeated field in a message.

​	告诉客户端存储中未设置的重复字段与此特定调用中未填充的字段之间的区别，可以通过将重复字段包装在消息中来完成。

Common request options that are shared between requests naturally fall out of following this rule. Read and write field masks fall out of this.

​	在请求之间共享的通用请求选项自然会因遵循此规则而产生。读写字段掩码也是如此。

Your top-level proto should almost always be a container for other messages that can grow independently.

​	你的顶层 proto 应几乎总是其他可以独立增长的消息的容器。

Even when you only need a single primitive type today, having it wrapped in a message gives you a clear path to expand that type and share the type among other methods that return the similar values. For example:

​	即使你今天只需要一个原始类型，将其包装在消息中也会为你提供一个清晰的扩展路径，并允许在返回类似值的其他方法之间共享该类型。例如：

```proto
message MultiplicationResponse {
  // Bad: What if you later want to return complex numbers and have an
  // AdditionResponse that returns the same multi-field type?
  // 错误：如果你以后想返回复数，并且还有一个返回相同多字段类型的 AdditionResponse，该怎么办？
  optional double result;


  // Good: Other methods can share this type and it can grow as your
  // service adds new features (units, confidence intervals, etc.).
  // 正确：其他方法可以共享此类型，并且随着服务添加新功能（单位、置信区间等），它可以增长。
  optional NumericResult result;
}

message NumericResult {
  optional double real_value;
  optional double complex_value;
  optional UnitType units;
}
```

One exception to top-level primitives: Opaque strings (or bytes) that encode a proto but are only built and parsed on the server. Continuation tokens, version info tokens and IDs can all be returned as strings *if* the string is actually an encoding of a structured proto.

​	顶层原始类型的一个例外是：不透明字符串（或字节），它们编码了一个 proto，但仅在服务器上构建和解析。例如，连续性令牌、版本信息令牌和 ID 都可以作为字符串返回，*如果*该字符串实际上是结构化 proto 的编码。

## 永远不要为现在只有两个状态但将来可能有更多状态的内容使用布尔值 Never Use Booleans for Something That Has Two States Now, but Might Have More Later

If you are using boolean for a field, make sure that the field is indeed describing just two possible states (for all time, not just now and the near future). Often, the flexibility of an enum, int, or message turns out to be worth it.

​	如果你为一个字段使用布尔值，请确保该字段确实只描述两个可能的状态（永远如此，不仅仅是现在和近期）。通常，枚举、整数或消息的灵活性会证明其值得。

For example, in returning a stream of posts a developer may need to indicate whether a post should be rendered in two-columns or not based on the current mocks from UX. Even though a boolean is all that’s needed today, nothing prevents UX from introducing two-row posts, three-column posts or four-square posts in a future version.

​	例如，返回一系列帖子时，开发者可能需要指示某个帖子是否应该根据当前的 UX 模型以两列方式呈现。即使今天只需要布尔值，也没有什么可以阻止 UX 在未来版本中引入两行帖子、三列帖子或四格帖子。

```proto
message GooglePlusPost {
  // Bad: Whether to render this post across two columns.
  // 错误：是否将此帖子呈现在两列中。
  optional bool big_post;

  // Good: Rendering hints for clients displaying this post.
  // Clients should use this to decide how prominently to render this
  // post. If absent, assume a default rendering.
  // 正确：显示此帖子的客户端的渲染提示。
  // 客户端应使用此来决定以多显著的方式呈现此帖子。如果缺失，假设为默认渲染。
  optional LayoutConfig layout_config;
}

message Photo {
  // Bad: True if it's a GIF.
  // 错误：如果它是 GIF，则为 true。
  optional bool gif;

  // Good: File format of the referenced photo (for example, GIF, WebP, PNG).
  // 正确：所引用照片的文件格式（例如，GIF、WebP、PNG）。
  optional PhotoType type;
}
```

Be cautious about adding states to an enum that conflates concepts.

​	在枚举中增加状态时要谨慎。

If a state introduces a new dimension to the enum or implies multiple application behaviors, you almost certainly want another field.

​	如果一个状态引入了新的维度或意味着多个应用程序行为，你几乎肯定需要另一个字段。

## 很少使用整数字段作为 ID - Rarely Use an Integer Field for an ID

It’s tempting to use an int64 as an identifier for an object. Opt instead for a string.

​	使用 `int64` 作为对象标识符可能很有诱惑力，但更好的选择是使用字符串。

This lets you change your ID space if you need to and reduces the chance of collisions. 2^64 isn’t as big as it used to be.

​	这可以让你在需要时更改 ID 空间，并减少冲突的可能性。2^64 不再像过去那么大。

You can also encode a structured identifier as a string which encourages clients to treat it as an opaque blob. You still must have a proto backing the string, but you can serialize the proto to a string field (encoded as web-safe Base64) which removes any of the internal details from the client-exposed API. In this case follow the guidelines [below](https://protobuf.dev/programming-guides/api/#encode-opaque-data-in-strings).

​	你还可以将结构化标识符编码为字符串，这鼓励客户端将其视为不透明的 blob。你仍然需要一个 proto 来支持该字符串，但可以将 proto 序列化为字符串字段（使用网络安全的 Base64 编码），从而消除客户端 API 中的内部细节。在这种情况下，请遵循[以下指南](https://protobuf.dev/programming-guides/api/#encode-opaque-data-in-strings)。

```proto
message GetFooRequest {
  // Which Foo to fetch.
  // 要获取的 Foo。
  optional string foo_id;
}

// Serialized and websafe-base64-encoded into the GetFooRequest.foo_id field.
// 序列化并使用网络安全的 base64 编码到 GetFooRequest.foo_id 字段。
message InternalFooRef {
  // Only one of these two is set. Foos that have already been
  // migrated use the spanner_foo_id and Foos still living in
  // Caribou Storage Server have a classic_foo_id.
  // 仅设置其中之一。已迁移的 Foo 使用 spanner_foo_id，
  // 而仍存储在 Caribou Storage Server 的 Foo 使用 classic_foo_id。
  optional bytes spanner_foo_id;
  optional int64 classic_foo_id;
}
```

If you start off with your own serialization scheme to represent your IDs as strings, things can get weird quickly. That’s why it’s often best to start with an internal proto that backs your string field.

​	如果你从自己的序列化方案开始，将 ID 表示为字符串，事情可能会迅速变得混乱。因此，最好从支持字符串字段的内部 proto 开始。

## 不要将需要客户端构造或解析的数据编码为字符串 Don’t Encode Data in a String That You Expect a Client to Construct or Parse

It’s less efficient over the wire, more work for the consumer of the proto, and confusing for someone reading your documentation. Your clients also have to wonder about the encoding: Are lists comma-separated? Did I escape this untrusted data correctly? Are numbers base-10? Better to have clients send an actual message or primitive type. It’s more compact over the wire and clearer for your clients.

​	在传输中效率较低，会增加 proto 使用者的工作量，并使阅读文档的人感到困惑。你的客户端还需要考虑编码问题：列表是用逗号分隔的吗？我是否正确地转义了这些不可信数据？数字是基于十进制的吗？最好让客户端发送实际的消息或原始类型。它在传输中更紧凑，并且对客户端更清晰。

This gets especially bad when your service acquires clients in several languages. Now each will have to choose the right parser or builder—or worse—write one.

​	尤其是在你的服务有多个语言客户端时，问题会变得更加糟糕。现在每个客户端都需要选择正确的解析器或构建器——甚至更糟——自己编写一个。

More generally, choose the right primitive type. See the Scalar Value Types table in the [Protocol Buffer Language Guide](https://protobuf.dev/programming-guides/proto2#scalar).

​	更一般地说，选择正确的原始类型。请参阅[协议缓冲区语言指南](https://protobuf.dev/programming-guides/proto2#scalar)中的标量值类型表。

### 在前端 Proto 中返回 HTML - Returning HTML in a Front-End Proto

With a JavaScript client, it’s tempting to return HTML or JSON in a field of your API. This is a slippery slope towards tying your API to a specific UI. Here are three concrete dangers:

​	对于 JavaScript 客户端，将 HTML 或 JSON 作为 API 字段返回很有诱惑力。这是一条滑坡，会将你的 API 与特定的 UI 绑定在一起。以下是三个具体的风险：

- A “scrappy” non-web client will end up parsing your HTML or JSON to get the data they want leading to fragility if you change formats and vulnerabilities if their parsing is bad.
  - 非 Web 客户端最终会解析你的 HTML 或 JSON 以获取他们想要的数据，这会导致如果你更改格式时的脆弱性，以及如果他们的解析不佳时的漏洞。

- Your web-client is now vulnerable to an XSS exploit if that HTML is ever returned unsanitized.
  - 如果 HTML 未经过清理返回，Web 客户端可能会受到 XSS 攻击的威胁。

- The tags and classes you’re returning expect a particular style-sheet and DOM structure. From release to release, that structure will change, and you risk a version-skew problem where the JavaScript client is older than the server and the HTML the server returns no longer renders properly on old clients. For projects that release often, this is not an edge case.
  - 你返回的标签和类期望特定的样式表和 DOM 结构。随着版本变化，这种结构会发生变化，你可能面临版本偏差问题，即 JavaScript 客户端比服务器版本旧，而服务器返回的 HTML 不再正确渲染在旧客户端上。对于频繁发布的项目，这不是边缘案例。


Other than the initial page load, it’s usually better to return data and use client-side templating to construct HTML on the client .

​	除了初始页面加载外，通常最好返回数据并使用客户端模板在客户端构建 HTML。

## 通过网络安全编码二进制 Proto 序列化，将不透明数据编码为字符串 Encode Opaque Data in Strings by Web-Safe Encoding Binary Proto Serialization

If you do encode *opaque* data in a client-visible field (continuation tokens, serialized IDs, version infos, and so on), document that clients should treat it as an opaque blob. *Always use binary proto serialization, never text-format or something of your own devising for these fields.* When you need to expand the data encoded in an opaque field, you’ll find yourself reinventing protocol buffer serialization if you’re not already using it.

​	如果你确实在客户端可见字段中编码了*不透明*数据（如连续性令牌、序列化 ID、版本信息等），请记录客户端应将其视为不透明的 blob。*始终使用二进制 Proto 序列化，绝不要对这些字段使用文本格式或你自己设计的格式。* 如果你需要扩展编码在不透明字段中的数据，而你没有使用 Proto 序列化，你会发现自己在重新发明 Proto 序列化。

Define an internal proto to hold the fields that will go in the opaque field (even if you only need one field), serialize this internal proto to bytes then web-safe base-64 encode the result into your string field .

​	定义一个内部 proto 来保存将进入不透明字段的字段（即使你只需要一个字段），将此内部 proto 序列化为字节，然后将结果网络安全地 Base64 编码到字符串字段中。

One rare exception to using proto serialization: *Very* occasionally, the compactness wins from a carefully constructed alternative format are worth it.

​	一种罕见的例外是：*在非常偶然的情况下*，通过精心构建的替代格式获得的紧凑性胜出，这可能是值得的。

## 不要包含客户端无法使用的字段 Don’t Include Fields that Your Clients Can’t Possibly Have a Use for

The API you expose to clients should only be for describing how to interact with your system. Including anything else in it adds cognitive overhead to someone trying to understand it.

​	你向客户端公开的 API 应仅用于描述如何与系统交互。包含其他任何内容都会增加理解它的人的认知负担。

Returning debug data in response protos used to be a common practice, but we have a better way. RPC response extensions (also called “side channels”) let you describe your client interface with one proto and your debugging surface with another.

​	返回响应 proto 中的调试数据曾经是一种常见做法，但我们有更好的方式。RPC 响应扩展（也称为“旁路”）允许你使用一个 proto 描述客户端接口，另一个 proto 描述调试界面。

Similarly, returning experiment names in response protos used to be a logging convenience–the unwritten contract was the client would send those experiments back on subsequent actions. The accepted way of accomplishing the same is to do log joining in the analysis pipeline.

​	类似地，在响应 proto 中返回实验名称曾经是为了日志记录的便利——未明文规定的契约是客户端会在后续操作中发送回这些实验。实现相同目标的公认方法是在分析管道中进行日志连接。

One exception:

​	一个例外：

If you need continuous, real-time analytics *and* are on a small machine budget, running log joins might be prohibitive. In cases where cost is a deciding factor, denormalizing log data ahead of time can be a win. If you need log data round-tripped to you, send it to clients as an opaque blob and document the request and response fields.

​	如果你需要连续的实时分析*并且*预算有限，运行日志连接可能会受到限制。在成本是决定性因素的情况下，提前对日志数据进行非规范化处理可能是一个好选择。如果需要对日志数据进行回传，可以将其作为不透明的 blob 发送给客户端，并记录请求和响应字段。

**Caution:** If you need to return or round-trip hidden data on *every* request , you’re hiding the true cost of using your service and that’s not good either.

​	**注意：** 如果你需要在*每次*请求中返回或回传隐藏数据，那你实际上是在隐藏使用服务的真实成本，这样做是不好的。

## **很少**定义没有连续性令牌的分页 API -  Rarely Define a Pagination API Without a Continuation Token

```proto
message FooQuery {
  // Bad: If the data changes between the first query and second, each of
  // these strategies can cause you to miss results. In an eventually
  // consistent world (that is, storage backed by Bigtable), it's not uncommon
  // to have old data appear after the new data. Also, the offset- and
  // page-based approaches all assume a sort-order, taking away some
  // flexibility.
  // 错误：如果数据在第一次查询和第二次查询之间发生变化，每种策略都可能导致你遗漏结果。
  // 在最终一致性（例如，存储由 Bigtable 支持的系统）中，
  // 旧数据在新数据之后出现并不罕见。此外，基于偏移量和分页的方法都假设一种排序方式，限制了灵活性。
  optional int64 max_timestamp_ms;
  optional int32 result_offset;
  optional int32 page_number;
  optional int32 page_size;

  // Good: You've got flexibility! Return this in a FooQueryResponse and
  // have clients pass it back on the next query.
  // 正确：你有了灵活性！将此返回到 FooQueryResponse 中，
  // 并让客户端在下一次查询中传回它。
  optional string next_page_token;
}
```

The best practice for a pagination API is to use an opaque continuation token (called next_page_token ) backed by an internal proto that you serialize and then `WebSafeBase64Escape` (C++) or `BaseEncoding.base64Url().encode` (Java). That internal proto could include many fields. The important thing is it buys you flexibility and–if you choose–it can buy your clients stability in the results.

​	分页 API 的最佳实践是使用一个不透明的连续性令牌（称为 `next_page_token`），并由内部 proto 支持，该 proto 可以被序列化，然后通过 `WebSafeBase64Escape`（C++）或 `BaseEncoding.base64Url().encode`（Java）编码。该内部 proto 可以包含许多字段。关键在于它提供了灵活性——如果你选择，它还可以为你的客户端提供结果的稳定性。

Do not forget to validate the fields of this proto as untrustworthy inputs (see note in [Encode opaque data in strings](https://protobuf.dev/programming-guides/api/#encode-opaque-data-in-strings)).

​	不要忘记将该 proto 的字段验证为不可信输入（请参阅[在字符串中编码不透明数据](https://protobuf.dev/programming-guides/api/#encode-opaque-data-in-strings)的说明）。

```proto
message InternalPaginationToken {
  // Track which IDs have been seen so far. This gives perfect recall at the
  // expense of a larger continuation token--especially as the user pages
  // back.
  // 跟踪到目前为止已看到的 ID。这提供了完美的回忆性，
  // 代价是较大的连续性令牌——特别是当用户向后分页时。
  repeated FooRef seen_ids;

  // Similar to the seen_ids strategy, but puts the seen_ids in a Bloom filter
  // to save bytes and sacrifice some precision.
  // 类似于 seen_ids 策略，但将 seen_ids 放入布隆过滤器以节省字节并牺牲一些精度。
  optional bytes bloom_filter;

  // A reasonable first cut and it may work for longer. Having it embedded in
  // a continuation token lets you change it later without affecting clients.
  // 一个合理的初步方案，它可能会工作更长时间。
  // 将其嵌入到连续性令牌中，允许你以后更改它，而不会影响客户端。
  optional int64 max_timestamp_ms;
}
```

## 将相关字段分组到一个新消息中，仅嵌套高度内聚的字段 Group Related Fields into a New Message. Nest Only Fields with High Cohesion

```proto
message Foo {
  // Bad: The price and currency of this Foo.
  // 错误：该 Foo 的价格和货币。
  optional int price;
  optional CurrencyType currency;

  // Better: Encapsulates the price and currency of this Foo.
  // 更好：封装该 Foo 的价格和货币。
  optional CurrencyAmount price;
}
```

Only fields with high cohesion should be nested. If the fields are genuinely related, you’ll often want to pass them around together inside a server. That’s easier if they’re defined together in a message. Think:

​	仅嵌套高度内聚的字段。如果这些字段确实相关，你通常希望将它们一起传递到服务器中。如果它们定义在同一个消息中，这会更容易。比如：

```java
CurrencyAmount calculateLocalTax(CurrencyAmount price, Location where)
```

If your CL introduces one field, but that field might have related fields later, preemptively put it in its own message to avoid this:

​	如果你的 CL 引入了一个字段，但该字段后来可能会有相关字段，预先将其放入自己的消息中以避免以下问题：

```proto
message Foo {
  // DEPRECATED! Use currency_amount.
  // 已废弃！使用 currency_amount。
  optional int price [deprecated = true];

  // The price and currency of this Foo.
  // 该 Foo 的价格和货币。
  optional google.type.Money currency_amount;
}
```

The problem with a nested message is that while `CurrencyAmount` might be a popular candidate for reuse in other places of your API, `Foo.CurrencyAmount` might not. In the worst case, `Foo.CurrencyAmount` *is* reused, but `Foo`-specific fields leak into it.

​	嵌套消息的问题在于，虽然 `CurrencyAmount` 可能是你的 API 中其他地方重用的热门候选项，但 `Foo.CurrencyAmount` 可能不会。在最坏的情况下，`Foo.CurrencyAmount` *被*重用，但 `Foo`-特定的字段泄漏到了其中。

While [loose coupling](https://en.wikipedia.org/wiki/Loose_coupling) is generally accepted as a best practice when developing systems, that practice may not always apply when designing `.proto` files. There may be cases in which tightly coupling two units of information (by nesting one unit inside of the other) may make sense. For example, if you are creating a set of fields that appear fairly generic right now but which you anticipate adding specialized fields into at a later time, nesting the message would dissuade others from referencing that message from elsewhere in this or other `.proto` files.

​	尽管[松耦合](https://en.wikipedia.org/wiki/Loose_coupling)通常被认为是开发系统时的最佳实践，但在设计 `.proto` 文件时，这一实践可能并不总是适用。在某些情况下，将两个信息单元紧密耦合（通过将一个单元嵌套在另一个单元中）可能是合理的。例如，如果你现在创建了一组看起来相当通用的字段，但你预计以后会向其中添加专用字段，那么嵌套消息可以劝阻其他人在此 `.proto` 文件或其他文件中引用该消息。

```proto
message Photo {
  // Bad: It's likely PhotoMetadata will be reused outside the scope of Photo,
  // so it's probably a good idea not to nest it and make it easier to access.
  // 错误：PhotoMetadata 可能在 Photo 的范围之外重用，
  // 因此最好不要嵌套它以便更容易访问。
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
  // 正确：在 FooConfiguration 范围外重用 FooConfiguration.Rule
  // 会将其与可能无关的组件紧密耦合，而嵌套它可以避免这种情况。
  message Rule {
    optional float multiplier = 1;
  }
  repeated Rule rules = 1;
}
```

## 在读取请求中包含字段读取掩码 Include a Field Read Mask in Read Requests

```proto
// Recommended: use google.protobuf.FieldMask
// 推荐：使用 google.protobuf.FieldMask

// Alternative one:
// 备选方案 1：
message FooReadMask {
  optional bool return_field1;
  optional bool return_field2;
}

// Alternative two:
// 备选方案 2：
message BarReadMask {
  // Tag numbers of the fields in Bar to return.
  // 要返回的 Bar 中字段的标记号。
  repeated int32 fields_to_return;
}
```

If you use the recommended `google.protobuf.FieldMask`, you can use the `FieldMaskUtil` ([Java](https://protobuf.dev/reference/java/api-docs/com/google/protobuf/util/FieldMaskUtil.html)/[C++](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.util.field_mask_util.md)) libraries to automatically filter a proto.

​	如果你使用推荐的 `google.protobuf.FieldMask`，可以使用 `FieldMaskUtil` 库（[Java](https://protobuf.dev/reference/java/api-docs/com/google/protobuf/util/FieldMaskUtil.html)/[C++](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.util.field_mask_util.md)）来自动过滤 proto。

Read masks set clear expectations on the client side, give them control of how much data they want back and allow the backend to only fetch data the client needs.

​	字段读取掩码在客户端明确设置期望值，给予它们控制返回数据量的能力，同时允许后端只获取客户端需要的数据。

The acceptable alternative is to always populate every field; that is, treat the request as if there were an implicit read mask with all fields set to true. This can get costly as your proto grows.

​	可接受的替代方案是始终填充所有字段，即将请求视为隐含的读取掩码，其中所有字段均设置为 true。但随着 proto 的增长，这可能会变得昂贵。

The worst failure mode is to have an implicit (undeclared) read mask that varies depending on which method populated the message. This anti-pattern leads to apparent data loss on clients that build a local cache from response protos.

​	最糟糕的失败模式是具有隐式（未声明）的读取掩码，这取决于哪个方法填充了消息。这种反模式会导致客户端在构建本地缓存时出现明显的数据丢失问题。

## 包含一个版本字段以实现一致性读取 Include a Version Field to Allow for Consistent Reads

When a client does a write followed by a read of the same object, they expect to get back what they wrote–even when the expectation isn’t reasonable for the underlying storage system.

​	当客户端执行写操作后紧接着进行读取操作时，他们期望返回的数据与写入的数据一致——即使这一期望对于底层存储系统来说不合理。

Your server will read the local value and if the local version_info is less than the expected version_info, it will read from remote replicas to find the latest value. Typically version_info is a [proto encoded as a string](https://protobuf.dev/programming-guides/api/#encode-opaque-data-in-strings) that includes the datacenter the mutation went to and the timestamp at which it was committed.

​	你的服务器将读取本地值，并且如果本地的 `version_info` 小于预期的 `version_info`，则它将从远程副本中读取以找到最新值。通常，`version_info` 是一个[编码为字符串的 proto](https://protobuf.dev/programming-guides/api/#encode-opaque-data-in-strings)，其中包括变更提交的时间戳和数据中心。

Even systems backed by consistent storage often want a token to trigger the more expensive read-consistent path rather than incurring the cost on every read.

​	即使是由一致性存储支持的系统，也经常需要一个标记来触发更昂贵的读取一致性路径，而不是在每次读取时承担这种开销。

## 为返回相同数据类型的 RPC 使用一致的请求选项 Use Consistent Request Options for RPCs that Return the Same Data Type

An example failure pattern is the request options for a service in which each RPC returns the same data type, but has separate request options for specifying things like maximum comments, embeds supported types list, and so on.

​	一个典型的失败模式是服务中每个 RPC 返回相同的数据类型，但有单独的请求选项来指定例如最大评论数、支持的嵌入类型列表等内容。

The cost of approaching this ad hoc is increased complexity on the client from figuring out how to fill out each request and increased complexity on the server transforming the N request options into a common internal one. A not-small number of real-life bugs are traceable to this example.

​	这种临时的方式会导致客户端在填写每个请求时的复杂性增加，也会增加服务器将 N 个请求选项转换为一个通用内部选项的复杂性。许多真实场景中的 bug 都可以追溯到这一点。

Instead, create a single, separate message to hold request options and include that in each of the top-level request messages. Here’s a better-practices example:

​	相反，为请求选项创建一个单独的消息，并在每个顶级请求消息中包含它。这是一个更好的实践示例：

```proto
message FooRequestOptions {
  // Field-level read mask of which fields to return. Only fields that
  // were requested will be returned in the response. Clients should only
  // ask for fields they need to help the backend optimize requests.
  // 字段级别的读取掩码，指定要返回的字段。
  // 只有请求的字段会在响应中返回。
  // 客户端应只请求它们需要的字段以帮助后端优化请求。
  optional FooReadMask read_mask;

  // Up to this many comments will be returned on each Foo in the response.
  // Comments that are marked as spam don't count towards the maximum
  // comments. By default, no comments are returned.
  // 每个 Foo 中返回的最大评论数。
  // 被标记为垃圾的评论不计入最大评论数。
  // 默认情况下，不返回评论。
  optional int max_comments_to_return;

  // Foos that include embeds that are not on this supported types list will
  // have the embeds down-converted to an embed specified in this list. If no
  // supported types list is specified, no embeds will be returned. If an embed
  // can't be down-converted to one of the supplied supported types, no embed
  // will be returned. Clients are strongly encouraged to always include at
  // least the THING_V2 embed type from EmbedTypes.proto.
  // 包含嵌入的 Foos 如果嵌入类型不在支持的类型列表中，
  // 将降级为此列表中指定的嵌入。
  // 如果未指定支持的类型列表，则不返回嵌入。
  // 如果无法将嵌入降级为提供的支持类型之一，则不返回嵌入。
  // 强烈建议客户端始终包括 EmbedTypes.proto 中的 THING_V2 类型。
  repeated EmbedType embed_supported_types_list;
}

message GetFooRequest {
  // What Foo to read. If the viewer doesn't have access to the Foo or the
  // Foo has been deleted, the response will be empty but will succeed.
  // 要读取的 Foo。如果查看者没有权限访问 Foo，或 Foo 已被删除，
  // 响应将为空但成功。
  optional string foo_id;

  // Clients are required to include this field. Server returns
  // INVALID_ARGUMENT if FooRequestOptions is left empty.
  // 客户端必须包含此字段。如果 FooRequestOptions 为空，
  // 服务器将返回 INVALID_ARGUMENT。
  optional FooRequestOptions params;
}

message ListFooRequest {
  // Which Foos to return. Searches have 100% recall, but more clauses
  // impact performance.
  // 要返回的 Foos。搜索具有 100% 的召回率，
  // 但更多的子句会影响性能。
  optional FooQuery query;

  // Clients are required to include this field. The server returns
  // INVALID_ARGUMENT if FooRequestOptions is left empty.
  // 客户端必须包含此字段。如果 FooRequestOptions 为空，
  // 服务器将返回 INVALID_ARGUMENT。
  optional FooRequestOptions params;
}
```

## 批量/多阶段请求 Batch/multi-phase Requests

Where possible, make mutations atomic. Even more important, make [mutations idempotent](https://protobuf.dev/programming-guides/api/#prefer-idempotency). A full retry of a partial failure shouldn’t corrupt/duplicate data.

​	在可能的情况下，使变更操作（mutations）原子化。更重要的是，使变更操作[幂等](https://protobuf.dev/programming-guides/api/#prefer-idempotency)。部分失败的完整重试不应导致数据损坏或重复。

Occasionally, you’ll need a single RPC that encapsulates multiple operations for performance reasons. What to do on a partial failure? If some succeeded and some failed, it’s best to let clients know.

​	有时，为了性能原因，你可能需要一个单独的 RPC 来封装多个操作。发生部分失败时该怎么办？如果某些操作成功了，而某些失败了，最好让客户端知道。

Consider setting the RPC as failed and return details of both the successes and failures in an RPC status proto.

​	可以将 RPC 标记为失败，并在 RPC 状态 proto 中返回成功和失败的详细信息。

In general, you want clients who are unaware of your handling of partial failures to still behave correctly and clients who are aware to get extra value.

​	一般来说，对于不了解部分失败处理的客户端，它们应该仍然表现正常；对于了解这一点的客户端，它们会获得额外的价值。

## 创建返回或操作小数据块的方法，并期望客户端通过批量组合这些请求来构建 UI - Create Methods that Return or Manipulate Small Bits of Data and Expect Clients to Compose UIs from Batching Multiple Such Requests

The ability to query many narrowly specified bits of data in a single round-trip allows a wider range of UX options without server changes by letting the client compose what they need.

​	在单次往返中查询许多范围较窄的数据块的能力允许更广泛的 UX 选项，而无需对服务器进行更改，因为客户端可以组合它们所需的内容。

This is most relevant for front-end and middle-tier servers.

​	这对于前端和中间层服务器尤其重要。

Many services expose their own batching API.

​	许多服务公开自己的批量 API。

## 当替代方案是移动或 Web 上的串行往返时，创建一次性 RPC - Make a One-off RPC when the Alternative is Serial Round-trips on Mobile or Web

In cases where a *web or mobile* client needs to make two queries with a data dependency between them, the current best practice is to create a new RPC that protects the client from the round trip.

​	在 *Web 或移动* 客户端需要执行两个数据之间有依赖关系的查询时，当前的最佳实践是创建一个新的 RPC，以保护客户端免受往返的影响。

In the case of mobile, it’s almost always worth saving your client the cost of an extra round-trip by bundling the two service methods together in one new one. For server-to-server calls, the case may not be as clear; it depends on how performance-sensitive your service is and how much cognitive overhead the new method introduces.

​	在移动场景中，通过将两个服务方法捆绑在一个新的方法中，几乎总是值得为客户端节省一次额外的往返成本。对于服务器间调用，情况可能不太明确；这取决于服务的性能敏感性以及新方法引入的认知负担。

## 将重复字段设为消息类型，而非标量或枚举类型 Make Repeated Fields Messages, Not Scalars or Enums

A common evolution is that a single repeated field needs to become multiple related repeated fields. If you start with a repeated primitive your options are limited–you either create parallel repeated fields, or define a new repeated field with a new message that holds the values and migrate clients to it.

​	一个常见的演化是，单个重复字段需要变成多个相关的重复字段。如果一开始你选择了一个重复的基本类型（primitive），你的选择就受限了——你要么创建并行的重复字段，要么定义一个新的重复字段，使用包含这些值的新消息类型，然后让客户端迁移到它。

If you start with a repeated message, evolution becomes trivial.

​	如果从重复消息开始，演化就会变得简单。

```proto
// Describes a type of enhancement applied to a photo
// 描述应用于照片的一种增强类型
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
  // 正确：PhotoEnhancement 可以扩展以描述需要
  // 更多字段的增强。
  repeated PhotoEnhancement enhancements;

  // Bad: If we ever want to return parameters associated with the
  // enhancement, we'd have to introduce a parallel array (terrible) or
  // deprecate this field and introduce a repeated message.
  // 错误：如果我们需要返回与增强相关的参数，
  // 我们必须引入一个并行数组（非常糟糕），
  // 或者废弃该字段并引入一个重复消息。
  repeated EnhancementType enhancement_types;
}
```

Imagine the following feature request: “We need to know which enhancements were performed by the user and which enhancements were automatically applied by the system.”

​	假设以下功能请求：“我们需要知道哪些增强是由用户执行的，以及哪些增强是系统自动应用的。”

If the enhancement field in `PhotoEnhancementReply` were a scalar or enum, this would be much harder to support.

​	如果 `PhotoEnhancementReply` 中的增强字段是标量或枚举，这将很难支持。

This applies equally to maps. It is much easier to add additional fields to a map value if it’s already a message rather than having to migrate from `map<string, string>` to `map<string, MyProto>`.

​	这同样适用于映射（maps）。如果映射的值已经是一个消息类型，添加额外的字段就容易得多，而不需要从 `map<string, string>` 迁移到 `map<string, MyProto>`。

One exception:

​	一个例外：

Latency-critical applications will find parallel arrays of primitive types are faster to construct and delete than a single array of messages; they can also be smaller over the wire if you use [[packed=true\]](https://protobuf.dev/programming-guides/encoding#packed) (eliding field tags). Allocating a fixed number of arrays is less work than allocating N messages. Bonus: in [Proto3](https://protobuf.dev/programming-guides/proto3), packing is automatic; you don’t need to explicitly specify it.

​	对于延迟敏感的应用程序，并行的基本类型数组比单个消息数组构造和删除更快；如果使用 [[packed=true\]](https://protobuf.dev/programming-guides/encoding#packed)（省略字段标签），它们在线传输时也可以更小。分配固定数量的数组比分配 N 个消息更省事。额外的好处是，在 [Proto3](https://protobuf.dev/programming-guides/proto3) 中，打包是自动的；你不需要显式指定它。

## Use Proto Maps

Prior to the introduction in [Proto3](https://protobuf.dev/programming-guides/proto3) of [Proto3 maps](https://protobuf.dev/programming-guides/proto3#maps), services would sometimes expose data as pairs using an ad-hoc KVPair message with scalar fields. Eventually clients would need a deeper structure and would end up devising keys or values that need to be parsed in some way. See [Don’t encode data in a string](https://protobuf.dev/programming-guides/api/#dont-encode-data-in-a-string).

​	在 [Proto3](https://protobuf.dev/programming-guides/proto3) 引入 [Proto3 maps](https://protobuf.dev/programming-guides/proto3#maps) 之前，服务有时会以键值对（KVPair）消息的形式暴露数据，消息的字段是标量。最终，客户端需要更深的结构，并最终设计出需要某种方式解析的键或值。参见 [不要在字符串中编码数据](https://protobuf.dev/programming-guides/api/#dont-encode-data-in-a-string)。

So, using a (extensible) message type for the value is an immediate improvement over the naive design.

​	因此，使用一种（可扩展的）消息类型作为值，比简单设计要立即改进。

Maps were back-ported to proto2 in all languages, so using `map<scalar, **message**>` is better than inventing your own KVPair for the same purpose[1](https://protobuf.dev/programming-guides/api/#fn:1).

​	Maps 被回移植到了 proto2 中的所有语言，因此使用 `map<scalar, **message**>` 比发明自己的 KVPair 用于相同目的要好 [1](https://protobuf.dev/programming-guides/api/#fn:1)。

If you want to represent *arbitrary* data whose structure you don’t know ahead of time, use [`google.protobuf.Any`](https://protobuf.dev/reference/protobuf/textformat-spec#any).

​	如果需要表示结构未知的*任意*数据，请使用 [`google.protobuf.Any`](https://protobuf.dev/reference/protobuf/textformat-spec#any)。

## 优先保证幂等性 Prefer Idempotency

Somewhere in the stack above you, a client may have retry logic. If the retry is a mutation, the user could be in for a surprise. Duplicate comments, build requests, edits, and so on aren’t good for anyone.

​	在你的栈层之上可能有客户端的重试逻辑。如果重试涉及到变更操作（mutation），用户可能会遇到问题，比如重复的评论、构建请求、编辑等。

A simple way to avoid duplicate writes is to allow clients to specify a client-created request ID that your server dedupes on (for example, hash of content or UUID).

​	避免重复写入的一个简单方法是允许客户端指定一个客户端创建的请求 ID，服务器通过此 ID 去重（例如，内容的哈希值或 UUID）。

## 注意你的服务名称，并使其全局唯一 Be Mindful of Your Service Name, and Make it Globally Unique

A service name (that is, the part after the `service` keyword in your `.proto` file) is used in surprisingly many places, not just to generate the service class name. This makes this name more important than one might think.

​	服务名称（即 `.proto` 文件中 `service` 关键字之后的部分）被用在意想不到的许多地方，不仅仅是生成服务类名称。这使得服务名称比看起来更重要。

What’s tricky is that these tools make the implicit assumption that your service name is unique across a network . Worse, the service name they use is the *unqualified* service name (for example, `MyService`), not the qualified service name (for example, `my_package.MyService`).

​	棘手的是，这些工具隐含地假设你的服务名称在网络中是唯一的。更糟糕的是，它们使用的是*未限定*服务名称（例如，`MyService`），而不是限定名称（例如，`my_package.MyService`）。

For this reason, it makes sense to take steps to prevent naming conflicts on your service name, even if it is defined inside a specific package. For example, a service named `Watcher` is likely to cause problems; something like `MyProjectWatcher` would be better.

​	因此，即使服务名称定义在特定包内，也建议采取措施防止命名冲突。例如，服务名称 `Watcher` 很可能会引发问题；像 `MyProjectWatcher` 这样的名称会更好。

## 确保每个 RPC 都指定并强制执行一个（宽松的）截止时间 Ensure Every RPC Specifies and Enforces a (Permissive) Deadline

By default, an RPC does not have a timeout. Since a request may tie up backend resources that are only released on completion, setting a default deadline that allows all well-behaving requests to finish is a good defensive practice. Not enforcing one has in the past caused severe problems for major services . RPC clients should still set a deadline on outgoing RPCs and will typically do so by default when they use standard frameworks. A deadline may and typically will be overwritten by a shorter deadline attached to a request.

​	默认情况下，RPC 没有超时时间。由于请求可能占用后端资源，直到完成才释放，设置一个允许所有正常请求完成的默认截止时间是一个很好的防御性实践。不设置截止时间在过去曾导致一些主要服务的严重问题。RPC 客户端仍应为发出的 RPC 设置截止时间，并且在使用标准框架时通常会默认这样做。请求附加的较短截止时间通常会覆盖原始设置。

Setting the `deadline` option clearly communicates the RPC deadline to your clients, and is respected and enforced by standard frameworks:

​	通过设置 `deadline` 选项，可以清晰地向客户端传达 RPC 截止时间，并由标准框架尊重和强制执行：

```proto
rpc Foo(FooRequest) returns (FooResponse) {
  option deadline = x; // there is no globally good default 没有全局适用的默认值
}
```

Choosing a deadline value will especially impact how your system acts under load. For existing services, it is critical to evaluate existing client behavior before enforcing new deadlines to avoid breaking clients (consult SRE). In some cases, it may not be possible to enforce a shorter deadline after the fact.

​	选择截止时间的值会特别影响系统在负载下的行为。对于现有服务，在强制执行新的截止时间之前，评估现有客户端行为至关重要，以避免破坏客户端（请咨询 SRE）。在某些情况下，在事后强制执行较短的截止时间可能不可行。

## 限制请求和响应的大小 Bound Request and Response Sizes

Request and response sizes should be bounded. We recommend a bound in the ballpark of 8 MiB, and 2 GiB is a hard limit at which many proto implementations break . Many storage systems have a limit on message sizes .

​	请求和响应的大小应有界。推荐的限制大约在 8 MiB 左右，而 2 GiB 是许多 proto 实现的硬限制，超过这个大小可能会导致崩溃。许多存储系统也对消息大小有上限。

Also, unbounded messages

​	此外，无界的消息可能会导致以下问题：

- bloat both client and server,
  - 增加客户端和服务器的负担；

- cause high and unpredictable latency,
  - 导致高且不可预测的延迟；

- decrease resiliency by relying on a long-lived connection between a single client and a single server.
  - 降低服务的弹性，因为它依赖于单个客户端和单个服务器之间的长时间连接。


Here are a few ways to bound all messages in an API:

​	以下是为 API 中所有消息设置边界的几种方法：

- Define RPCs that return bounded messages, where each RPC call is logically independent from the others.
  - 定义返回有界消息的 RPC，其中每次 RPC 调用在逻辑上是独立的。

- Define RPCs that operate on a single object, instead of on an unbounded, client-specified list of objects.
  - 定义操作单个对象的 RPC，而不是针对无界、由客户端指定的对象列表。

- Avoid encoding unbounded data in string, byte, or repeated fields.
  - 避免在字符串、字节或重复字段中编码无界数据。

- Define a long-running operation . Store the result in a storage system designed for scalable, concurrent reads .
  - 定义一个长时间运行的操作，将结果存储在支持可扩展并发读取的存储系统中。

- Use a pagination API (see [Rarely define a pagination API without a continuation token](https://protobuf.dev/programming-guides/api/#define-pagination-api)).
  - 使用分页 API（参见 [很少定义没有续页令牌的分页 API](https://protobuf.dev/programming-guides/api/#define-pagination-api)）。

- Use streaming RPCs.
  - 使用流式 RPC。


If you are working on a UI, see also [Create methods that return or manipulate small bits of data](https://protobuf.dev/programming-guides/api/#create-methods-manipulate-small-bits).

​	如果您正在处理用户界面，还可以参见 [创建返回或操作小块数据的方法](https://protobuf.dev/programming-guides/api/#create-methods-manipulate-small-bits)。

## 谨慎传播状态码 Propagate Status Codes Carefully

RPC services should take care at RPC boundaries to interrogate errors, and return meaningful status errors to their callers.

​	RPC 服务在 RPC 边界上应仔细检查错误，并返回有意义的状态错误给调用者。

Let’s examine a toy example to illustrate the point:

​	让我们通过一个简单的例子来说明这个问题：

Consider a client that calls `ProductService.GetProducts`, which takes no arguments. As part of `GetProducts`, `ProductService` might get all the products, and call `LocaleService.LocaliseNutritionFacts` for each product.

​	假设客户端调用了 `ProductService.GetProducts`，它不需要参数。在 `GetProducts` 的执行过程中，`ProductService` 可能会获取所有产品，并调用 `LocaleService.LocaliseNutritionFacts` 为每个产品本地化营养信息。

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

​	如果 `ProductService` 的实现有误，它可能会向 `LocaleService` 发送错误的参数，导致返回 `INVALID_ARGUMENT` 错误。

If `ProductService` carelessly returns errors to its callers, the client will receive `INVALID_ARGUMENT`, since status codes propagate across RPC boundaries. But, the client didn’t pass any arguments to `ProductService.GetProducts`. So, the error is worse than useless: it will cause a great deal of confusion!

​	如果 `ProductService` 不负责任地将错误传递给调用者，客户端将收到 `INVALID_ARGUMENT` 错误，因为状态码会跨 RPC 边界传播。然而，客户端并没有向 `ProductService.GetProducts` 传递任何参数。因此，这个错误不仅没有用，反而会引起极大的困惑！

Instead, `ProductService` should interrogate errors it receives at the RPC boundary; that is, the `ProductService` RPC handler it implements. It should return meaningful errors to users: if *it received* invalid arguments from the caller, it should return `INVALID_ARGUMENT`. If *something downstream* received invalid arguments, it should convert the `INVALID_ARGUMENT` to `INTERNAL` before returning the error to the caller.

​	相反，`ProductService` 应该在 RPC 边界检查接收到的错误。例如，如果 *它* 收到来自调用方的无效参数，它应该返回 `INVALID_ARGUMENT`。如果 *下游* 的某些服务接收到无效参数，则应将 `INVALID_ARGUMENT` 转换为 `INTERNAL`，然后将错误返回给调用方。

Carelessly propagating status errors leads to confusion, which can be very expensive to debug. Worse, it can lead to an invisible outage where every service forwards a client error without causing any alerts to happen .

​	不加思索地传播状态错误会导致混乱，调试成本非常高。更糟糕的是，这可能导致隐形宕机——每个服务都将客户端错误传递出去，却没有触发任何警报。

The general rule is: at RPC boundaries, take care to interrogate errors, and return meaningful status errors to callers, with appropriate status codes. To convey meaning, each RPC method should document what error codes it returns in which circumstances. The implementation of each method should conform to the documented API contract.

​	一般规则：在 RPC 边界上，仔细检查错误，并向调用者返回有意义的状态错误，使用适当的状态码。为了传达意义，每个 RPC 方法应记录它在何种情况下返回哪些错误代码。每个方法的实现应符合文档中的 API 合约。

## 为每个方法创建唯一的 Protos - Create Unique Protos per Method

Create a unique request and response proto for each RPC method. Discovering later that you need to diverge the top-level request or response can be expensive. This includes “empty” responses; create a unique empty response proto rather than reusing the [well-known Empty message type](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/empty.proto).

​	为每个 RPC 方法创建唯一的请求和响应 proto。如果在后期发现需要分离顶级请求或响应，成本会非常高。这包括“空”响应；创建一个独特的空响应 proto，而不是重复使用 [众所周知的 Empty 消息类型](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/empty.proto)。

### 重用消息 Reusing Messages

To reuse messages, create shared “domain” message types to include in multiple Request and Response protos. Write your application logic in terms of those types rather than the request and response types.

​	为了重用消息，可以创建共享的“领域”消息类型，并将其包含在多个请求和响应 protos 中。在应用程序逻辑中基于这些类型进行开发，而不是直接使用请求和响应类型。

This gives you the flexibility to evolve your method request/response types independently, but share code for logical sub-units.

​	这样可以让你的方法请求/响应类型独立演化，同时为逻辑子单元共享代码。

## 附录 Appendix

### 返回重复字段 Returning Repeated Fields

When a repeated field is empty, the client can’t tell if the field just wasn’t populated by the server or if the backing data for the field is genuinely empty. In other words, there’s no `hasFoo` method for repeated fields.

​	当一个重复字段为空时，客户端无法判断该字段是服务器未填充还是底层数据确实为空。换句话说，重复字段没有 `hasFoo` 方法。

Wrapping a repeated field in a message is an easy way to get a hasFoo method.

​	将重复字段包装在一个消息中是获取 `hasFoo` 方法的简单方法。

```proto
message FooList {
  repeated Foo foos;
}
```

The more holistic way to solve it is with a field [read mask](https://protobuf.dev/programming-guides/api/#include-field-read-mask). If the field was requested, an empty list means there’s no data. If the field wasn’t requested the client should ignore the field in the response.

​	更全面的解决方案是使用字段 [读取掩码](https://protobuf.dev/programming-guides/api/#include-field-read-mask)。如果请求了该字段，空列表意味着没有数据。如果未请求该字段，客户端应忽略响应中的该字段。

### 更新重复字段 Updating Repeated Fields

The worst way to update a repeated field is to force the client to supply a replacement list. The dangers with forcing the client to supply the entire array are manyfold. Clients that don’t preserve unknown fields cause data loss. Concurrent writes cause data loss. Even if those problems don’t apply, your clients will need to carefully read your documentation to know how the field is interpreted on the server side. Does an empty field mean the server won’t update it, or that the server will clear it?

​	更新重复字段的最糟糕方式是强制客户端提供一个替换列表。强制客户端提供整个数组会带来许多风险。无法保留未知字段的客户端会导致数据丢失，并发写入会导致数据丢失。即使这些问题不存在，客户端仍需要仔细阅读文档以了解服务器端如何解释字段：空字段是否表示服务器不会更新它，还是表示服务器会清除它？

**Fix #1**: Use a repeated update mask that permits the client to replace, delete, or insert elements into the array without supplying the entire array on a write.

​	使用一个重复更新掩码，允许客户端在写入时替换、删除或插入数组中的元素，而无需提供整个数组。

**Fix #2**: Create separate append, replace, delete arrays in the request proto.

​	在请求 proto 中创建独立的追加、替换、删除数组。

**Fix #3**: Allow only appending or clearing. You can do this by wrapping the repeated field in a message. A present, but empty, message means clear, otherwise, any repeated elements mean append.

​	仅允许追加或清除。可以通过将重复字段包装在消息中来实现。存在但为空的消息表示清除，否则，任何重复元素表示追加。

### 重复字段的顺序独立性 Order Independence in Repeated Fields

*Try* to avoid order dependence in general. It’s an extra layer of fragility. An especially bad type of order dependence is parallel arrays. Parallel arrays make it more difficult for clients to interpret the results and make it unnatural to pass the two related fields around inside your own service.

​	*尽量*避免顺序依赖。顺序依赖增加了额外的脆弱性层。尤其糟糕的一种顺序依赖是并行数组。并行数组让客户端难以解释结果，也让在服务内部传递两个相关字段变得不自然。

```proto
message BatchEquationSolverResponse {
  // Bad: Solved values are returned in the order of the equations given in
  // the request.
  // 错误：解决值以请求中提供的方程顺序返回。
  repeated double solved_values;
  // (Usually) Bad: Parallel array for solved_values.
  // （通常）错误：与 solved_values 对应的并行数组。
  repeated double solved_complex_values;
}

// Good: A separate message that can grow to include more fields and be
// shared among other methods. No order dependence between request and
// response, no order dependence between multiple repeated fields.
// 正确：一个独立的消息可以扩展以包含更多字段，并可被其他方法共享。
// 请求和响应之间没有顺序依赖，多个重复字段之间也没有顺序依赖。
message BatchEquationSolverResponse {
  // Deprecated, this will continue to be populated in responses until Q2
  // 2014, after which clients must move to using the solutions field below.
  // 废弃字段：此字段将在 2014 年 Q2 之前继续在响应中填充，之后客户端必须转向使用 solutions 字段。
  repeated double solved_values [deprecated = true];

  // Good: Each equation in the request has a unique identifier that's
  // included in the EquationSolution below so that the solutions can be
  // correlated with the equations themselves. Equations are solved in
  // parallel and as the solutions are made they are added to this array.
  // 正确：请求中的每个方程都有唯一标识符，该标识符包含在下面的 EquationSolution 中，
  // 这样解决方案就可以与方程本身相关联。方程并行求解，解决方案在生成时被添加到此数组中。
  repeated EquationSolution solutions;
}
```

### 因为 Proto 出现在移动构建中而导致的功能泄漏 Leaking Features Because Your Proto is in a Mobile Build

Android and iOS runtimes both support reflection. To do that, the unfiltered names of fields and messages are embedded in the application binary (APK, IPA) as strings.

​	Android 和 iOS 的运行时均支持反射。为此，字段和消息的未过滤名称会作为字符串嵌入到应用程序二进制文件（APK，IPA）中。

```proto
message Foo {
  // This will leak existence of Google Teleport project on Android and iOS
  // 在 Android 和 iOS 上泄露 Google Teleport 项目的存在。
  optional FeatureStatus google_teleport_enabled;
}
```

Several mitigation strategies:

​	多种缓解策略：

- ProGuard obfuscation on Android. As of Q3 2014. iOS has no obfuscation option: once you have the IPA on a desktop, piping it through `strings` will reveal field names of included protos. [iOS Chrome tear-down](https://github.com/Bensge/Chrome-for-iOS-Headers)
  - 在 Android 上使用 ProGuard 混淆。截至 2014 年 Q3，iOS 没有混淆选项：一旦在桌面上获得 IPA，通过 `strings` 命令即可揭示包含的 proto 的字段名称。[iOS Chrome 逆向分析](https://github.com/Bensge/Chrome-for-iOS-Headers)

- Curate precisely which fields are sent to mobile clients .
  - 精确策划发送给移动客户端的字段。

- If plugging the leak isn’t feasible on an acceptable timescale, get buy-in from the feature owner to risk it.
  - 如果在可接受的时间范围内无法堵住泄漏，请获得功能所有者的支持，以承担风险。


*Never* use this as an excuse to obfuscate the meaning of a field with a code-name. Either plug the leak or get buy-in to risk it.

​	*绝不要*以字段使用代码名称为借口来掩盖其含义。要么堵住泄漏，要么获得风险承担的支持。

### 性能优化 Performance Optimizations

You can trade type safety or clarity for performance wins in some cases. For example, a proto with hundreds of fields–particularly message-type fields–is going to be slower to parse than one with fewer fields. A very deeply-nested message can be slow to deserialize just from the memory management. A handful of techniques teams have used to speed deserialization:

​	在某些情况下，可以以牺牲类型安全性或清晰性来换取性能提升。例如，具有数百个字段的 proto（尤其是消息类型字段）解析速度会比具有较少字段的 proto 更慢。非常深层嵌套的消息仅因内存管理也会导致反序列化变慢。以下是一些团队用来加速反序列化的技术：

- Create a parallel, trimmed proto that mirrors the larger proto but has only some of the tags declared. Use this for parsing when you don’t need all the fields. Add tests to enforce that tag numbers continue to match as the trimmed proto accumulates numbering “holes.”
  - 创建一个与较大 proto 平行的精简 proto，仅声明其中的部分标签。仅在不需要所有字段时用于解析。添加测试以确保在精简 proto 累积编号“空洞”时标签号继续匹配。

- Annotate the fields as “lazily parsed” with [[lazy=true\]](https://github.com/protocolbuffers/protobuf/blob/cacb096002994000f8ccc6d9b8e1b5b0783ee561/src/google/protobuf/descriptor.proto#L609).
  - 使用 [[lazy=true\]](https://github.com/protocolbuffers/protobuf/blob/cacb096002994000f8ccc6d9b8e1b5b0783ee561/src/google/protobuf/descriptor.proto#L609) 注解将字段标记为“延迟解析”。

- Declare a field as bytes and document its type. Clients who care to parse the field can do so manually. The danger with this approach is there’s nothing preventing someone from putting a message of the wrong type in the bytes field. You should never do this with a proto that’s written to any logs, as it prevents the proto from being vetted for PII or scrubbed for policy or privacy reasons.
  - 将字段声明为字节类型并记录其类型。关心字段解析的客户端可以手动解析。此方法的风险在于，没有任何机制防止错误类型的消息被放入字节字段。不要在任何写入日志的 proto 中使用此方法，因为这会阻止 proto 被审核隐私信息或根据策略或隐私原因进行清理。


------

1. A gotcha with protos that contain `map<k,v>` fields. Don’t use them as reduce keys in a MapReduce. The wire format and iteration order of proto3 map items are *unspecified* which leads to inconsistent map shards. 使用包含 `map<k,v>` 字段的 protos 时的陷阱。不要在 MapReduce 中将它们用作 reduce 键。proto3 映射项的线格式和迭代顺序*未指定*，这会导致不一致的映射分片。
