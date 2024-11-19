+++
title = "不支持可空的 Setter 和 Getter"
date = 2024-11-17T09:35:36+08:00
weight = 130
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

> 原文：[https://protobuf.dev/programming-guides/nullable-getters-setters/](https://protobuf.dev/programming-guides/nullable-getters-setters/)
>
> 收录该文档的时间：`2024-11-17T09:35:36+08:00`

# No Nullable Setters/Getters Support - 不支持可空的 Setter 和 Getter

Covers why Protobuf doesn’t support nullable setters and getters

​	解释为什么 Protobuf 不支持可空的 Setter 和 Getter

We have heard feedback that some folks would like protobuf to support nullable getters/setters in their null-friendly language of choice (particularly Kotlin, C#, and Rust). While this does seem to be a helpful feature for folks using those languages, the design choice has tradeoffs which have led to the Protobuf team choosing not to implement them.

​	我们收到了一些反馈，部分用户希望 Protobuf 在支持空值的语言（特别是 Kotlin、C# 和 Rust）中提供可空的 Getter/Setter。虽然这对这些语言的用户来说似乎是一个很有帮助的功能，但这种设计选择存在权衡，最终促使 Protobuf 团队决定不实现它们。

The biggest reason not to have nullable fields is the intended behavior of default values specified in a `.proto` file. By design, calling a getter on an unset field will return the default value of that field.

​	**不支持可空字段的最大原因是 `.proto` 文件中指定的默认值的预期行为**。根据设计，对未设置的字段调用 Getter 时会返回该字段的默认值。

As an example, consider this `.proto` file:

​	以下是一个 `.proto` 文件的示例：

```proto
message Msg { optional Child child = 1; }
message Child { optional Grandchild grandchild = 1; }
message Grandchild { optional int32 foo = 1 [default = 72]; }
```

and corresponding Kotlin getters:

​	以及相应的 Kotlin Getter 示例：

```kotlin
// With our API where getters are always non-nullable:
// 在我们当前的 API 中，Getter 始终是非空的：
msg.child.grandchild.foo == 72

// With nullable submessages the ?. operator fails to get the default value:
// 如果子消息支持可空，则 ?. 操作符无法获取默认值：
msg?.child?.grandchild?.foo == null

// Or verbosely duplicating the default value at the usage site:
// 或者需要在使用点冗长地重复默认值：
(msg?.child?.grandchild?.foo ?: 72)
```

and corresponding Rust getters:

​	对应的 Rust Getter 示例：

```rust
// With our API:
msg.child().grandchild().foo()   // == 72

// Where every getter is an Option<T>, verbose and no default observed
// 如果每个 Getter 都返回 Option<T>，则代码冗长且无法观察到默认值：
msg.child().map(|c| c.grandchild()).map(|gc| gc.foo()) // == Option::None

// For the rare situations where code may want to observe both the presence and
// value of a field, the _opt() accessor which returns a custom Optional type
// can also be used here (the Optional type is similar to Option except can also
// be aware of the default value):
// 对于某些需要同时观察字段的存在性和值的少数情况，可以使用 `_opt()` 访问器（返回自定义的 Optional 类型，类似于 Option，但也可以感知默认值）：
msg.child().grandchild().foo_opt() // Optional::Unset(72)
```

If a nullable getter existed, it would necessarily ignore the user-specified defaults (to return null instead) which would lead to surprising and inconsistent behavior. If users of nullable getters want to access the default value of the field, they would have to write their own custom handling to use the default if null is returned, which removes the supposed benefit of cleaner/easier code with null getters.

​	如果存在可空的 Getter，则它必然会忽略用户指定的默认值（返回 null），这将导致令人困惑且不一致的行为。如果可空 Getter 的用户希望访问字段的默认值，他们必须编写自己的自定义处理逻辑以在返回 null 时使用默认值，这反而消除了使用可空 Getter 提供的简化代码的好处。

Similarly, we do not provide nullable setters as the behavior would be unintuitive. Performing a set and then get would not always give the same value back, and calling a set would only sometimes affect the has-bit for the field.

​	类似地，我们不提供可空的 Setter，因为这种行为会不直观。执行一次 set 操作后再调用 get 操作，不一定会返回相同的值；调用 set 操作仅在某些情况下会影响字段的存在位（has-bit）。

Note that message-typed fields are always explicit presence fields (with hazzers). Proto3 defaults to scalar fields having implicit presence (without hazzers) unless they are explicitly marked `optional`, while Proto2 does not support implicit presence. With [Editions]({{< ref "/docs/ProtobufEditions/FeatureSettingsforEditions#field_presence" >}}), explicit presence is the default behavior unless an implicit presence feature is used. With the forward expectation that almost all fields will have explicit presence, the ergonomic concerns that come with nullable getters are expected to be more of a concern than they may have been for Proto3 users.

​	需要注意的是，消息类型的字段始终是显式存在字段（具有 hazzers）。Proto3 默认标量字段为隐式存在（无 hazzers），除非明确标记为 `optional`，而 Proto2 不支持隐式存在。通过 [版本]({{< ref "/docs/ProtobufEditions/FeatureSettingsforEditions#field_presence" >}}) 支持的功能，显式存在成为默认行为，除非使用隐式存在特性。考虑到未来几乎所有字段都会具有显式存在，可空 Getter 带来的使用问题预计会比 Proto3 用户所面临的更严重。

Due to these issues, nullable setters/getters would radically change the way default values can be used. While we understand the possible utility, we have decided it’s not worth the inconsistencies and difficulty it introduces.

​	由于这些问题，可空 Setter/Getter 将从根本上改变默认值的使用方式。虽然我们理解其可能的实用性，但我们认为不值得引入这种不一致性和复杂性，因此决定不支持该功能。
