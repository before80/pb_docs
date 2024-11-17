+++
title = "Proto Limits"
date = 2024-11-17T09:35:36+08:00
weight = 30
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文：[https://protobuf.dev/programming-guides/proto-limits/](https://protobuf.dev/programming-guides/proto-limits/)
>
> 收录该文档的时间：`2024-11-17T09:35:36+08:00`

# Proto Limits

Covers the limits to number of supported elements in proto schemas.



This topic documents the limits to the number of supported elements (fields, enum values, and so on) in proto schemas.

This information is a collection of discovered limitations by many engineers, but is not exhaustive and may be incorrect/outdated in some areas. As you discover limitations in your work, contribute those to this document to help others.

## Number of Fields

Message with only singular proto fields (such as Boolean):

- ~2100 fields (proto2)
- ~3100 (proto3 without using optional fields)

Empty message extended by singular fields (such as Boolean):

- ~4100 fields (proto2)

Extensions are supported [only by proto2](https://protobuf.dev/programming-guides/version-comparison#extensionsany).

To test this limitation, create a proto message with more than the upper bound number of fields and compile using a Java proto rule. The limit comes from JVM specs.

## Number of Values in an Enum

The lowest limit is ~1700 values, in Java . Other languages have different limits.

## Total Size of the Message

Any proto in serialized form must be <2GiB, as that is the maximum size supported by all implementations. It’s recommended to bound request and response sizes.

## Depth Limit for Proto Unmarshaling

- Java: 100
- C++: 100
- Go: 10000 (there is a plan to reduce this to 100)

If you try to unmarshal a message that is nested deeper than the depth limit, the unmarshaling will fail.
