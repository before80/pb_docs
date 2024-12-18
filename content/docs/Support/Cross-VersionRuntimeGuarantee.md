+++
title = "Cross-Version Runtime Guarantee"
date = 2024-11-17T09:35:36+08:00
weight = 20
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文：[https://protobuf.dev/support/cross-version-runtime-guarantee/](https://protobuf.dev/support/cross-version-runtime-guarantee/)
>
> 收录该文档的时间：`2024-11-17T09:35:36+08:00`

# Cross-Version Runtime Guarantee

The guarantees that the language has for cross-version runtime compatibility.



Protobuf language bindings have two components. The generated code (typically produced from `protoc`) and the runtime libraries that must be included when using the generated code. When these come from different releases of protobuf, we are in a “cross version runtime” situation.

We intend to offer the following guarantees across all languages except [C++](https://protobuf.dev/support/cross-version-runtime-guarantee/#cpp). These are the default guarantees; however, owners of protobuf code generators and runtimes may explicitly override them with more specific guarantees for that language.

Protobuf cross-version usages outside the guarantees are **error-prone and not supported**. Version skews can lead to *flakes and undefined behaviors* that are hard to diagnose, even if it can often *seem* to work as long as nothing has changed in a source-incompatible way. For Protobuf, the proliferation of tools and services that rely on using unsupported Protobuf language bindings prevents the protobuf team from updating the protobuf implementation in response to bug reports or security vulnerabilities.

## New Gencode + Old Runtime = Never Allowed

We may add new runtime APIs in any kind of release (Major, Minor, or Patch). Gencode in that release is allowed to use those new APIs. The consequence is that gencode should never be paired with a runtime that predates the `protoc` and plugin that was used to generate those bindings.

We will add “poison pills” where possible to prevent attempts to pair newer gencode with an older runtime.

## Major Versions

Starting with the 2025Q1 release, the protobuf project will adopt a rolling compatibility window for major versions. Code generated for a major version V (full version: V.x.y) will be supported by protobuf runtimes of version V and V+1.

Protobuf will not support using gencode from version V with runtime >= V+2 and will be using a “poison pill” mechanism to fail with a clear error message when a software assembly attempts to use such a configuration.

## Minor Versions

Within a single major runtime version, generated code from an older version of `protoc` will run on a newer runtime.

## Security Exception

We reserve the right to violate the above promises if needed for security reasons. We expect these exceptions to be rare, but will always prioritize security above these guarantees. For example, [footmitten](https://cve.report/CVE-2022-3510) required paired updates to both the runtime and the generated code for Java. As a result, code generated by 3.20.3 (which contained the footmitten fix) would not load with runtime library 3.21.6 (which predates the footmitten fix), creating the following compatibility matrix:

|                 | Generated Code Version |        |            |            |            |
| --------------- | ---------------------- | ------ | ---------- | ---------- | ---------- |
| 3.20.2          | 3.20.3                 | 3.21.6 | 3.21.7     |            |            |
| Runtime Version | 3.20.2                 | Vuln   | **Broken** | **Vuln**   | **Broken** |
| 3.20.3          | Vuln                   | Works  | **Vuln**   | **Works?** |            |
| 3.21.6          | Vuln                   | Broken | Vuln       | **Broken** |            |
| 3.21.7          | Vuln                   | Works  | Vuln       | Works      |            |

- “Vuln” indicates that the combination will successfully start, but the security vulnerability still exists.
- “Works” indicates that the combination will successfully start and does not have the vulnerability.
- “Broken” indicates that the combination will not successfully start.
- **Bold** indicates configurations that were never deliberately intended to function together given the guarantees outlined in this topic.

## No Coexistence of Multiple Major Runtime Versions

Coexistence of multiple major versions in the same process is **not** supported.

## C++ Specific Guarantees

Protobuf C++ disclaims all cross-runtime support and requires an exact match between its generated code version and its runtime version at all times. Additionally, Protobuf C++ makes no guarantees about ABI stability across any releases (major, minor, or micro).
