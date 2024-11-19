+++
title = "Version Support"
date = 2024-11-17T09:35:36+08:00
weight = 1
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文：[https://protobuf.dev/support/version-support/](https://protobuf.dev/support/version-support/)
>
> 收录该文档的时间：`2024-11-17T09:35:36+08:00`

# Version Support

A list of the support windows provided for language implementations.



Support windows for protoc and the various languages are covered in the tables later in this topic. Version numbers throughout this topic use [SemVer](https://semver.org/) conventions; in the version “3.21.7,” we say that “3” is the major version, “21” is the minor version, and “7” is the micro or patch number.

Starting with the v20.x protoc release, we changed our versioning scheme to enable nimbler updates to language-specific parts of Protocol Buffers. In the new scheme, each language has its own major version that can be incremented independently of other languages. The minor and patch versions, however, remain coupled. This allows us to introduce breaking changes into some languages without requiring a bump of the major version in languages that do not experience a breaking change. For example, a single release might include protoc version 24.0, Java runtime version 4.24.0 and C# runtime version 3.24.0.

The first instance of this new versioning scheme was the 4.21.0 version of the Python API, which followed the preceding version, 3.20.1. Other language APIs released at the same time were released as 3.21.0.

## Release Cadence

Protobuf strives to release updates quarterly. We may add a release if there is an urgent need such as a security fix that requires new APIs. Skipping a release should be a very rare event.

Major (breaking) releases will be targeted to the Q1 release. We may introduce a major breaking change at any time if there is an urgent need, but this should be very rare.

Our support windows are defined by our [library breaking change policy](https://opensource.google/documentation/policies/library-breaking-change).

Protobuf does *not* consider enforcement of its documented language, tooling, platform, and library support policies to be a breaking change. For example, a release may drop support for an EOL language version without bumping major versions.

## What Changes in a Release?

**The binary wire format does not change** even in major version updates. You will continue to be able to read old binary wire format proto data from newer versions of Protocol Buffers. Newly generated protobuf bindings serialized to binary wire format will be parseable by older binaries. This is a fundamental design principle of Protocol Buffers. Note that JSON and textproto formats *do not* offer the same stability guarantees.

**The descriptor.proto schema can change.** In minor or patch releases, we may add new message, fields, enums, enum values, editions, editions [features]({{< ref "/docs/ProtobufEditions/FeatureSettingsforEditions" >}}) etc. We may also mark existing elements as deprecated. In a major release, we may remove deprecated options, enums, enum values, messages, fields, etc.

**The .proto language grammar can change.** In minor or patch releases, we may add new language constructs and alternative syntax for existing features. We may also mark certain features as deprecated. This could result in new warnings that were not previously emitted by `protoc`. In a major release, we may remove support for obsolete features, syntax, editions in a way that will require updates to client code.

**Gencode and runtime APIs can change.** In minor or patch releases, changes will either be purely additive for new functionality or source-compatible updates. Simply recompiling code should work. In a major release, the gencode or runtime API can change in incompatible ways that require callsite changes. We try to minimize these. Changes fixing or otherwise affecting undefined behavior are not considered breaking, and do not require a major release.

**Operating system, programming language, and tooling version support can change.** In a minor or patch release, we may add or drop support for particular versions of an operating system, programming language, or tooling. See [foundational support matrices](https://github.com/google/oss-policies-info/tree/main) for our supported languages.

In general:

- Minor or patch releases should only contain purely additive or source-compatible updates per our [Cross Version Runtime Guarantee](https://protobuf.dev/support/cross-version-runtime-guarantee/#minor)
- Major releases may remove functionality, features, or change APIs in ways that require updates to callsites.

## Support Duration

The most recent release is always supported. Support for earlier minor versions ends when a new minor version under the same major version is released. Support for earlier major versions ends four quarters beyond the quarter that the breaking release is introduced. For example, when Protobuf Python 5.26.0 was released in Q1 of 2024, that set the end of support of Protobuf Python 4.25.x at the [end of Q1 2025](https://protobuf.dev/support/version-support/#python).

The following sections provide a guide to the support for each language.

## C++

C++ will target making major version bumps annually in Q1 of each year.

The protoc version can be inferred from the Protobuf C++ minor version number. Example: Protobuf C++ version 4.25.x uses protoc version 25.x.

Future plans are shown in *italics* and are subject to change.

**Release support dates**

| Protobuf C++ | Release date | End of support |
| :----------: | :----------: | :------------: |
|     3.x      | 25 May 2022  |  31 Mar 2024   |
|     4.x      | 16 Feb 2023  |  31 Mar 2025   |
|     5.x      | 13 Mar 2024  |  31 Mar 2026   |
|     6.x      |   Q1 2025    |  31 Mar 2027   |

**Release support chart**

| Protobuf C++ |  protoc   | 23Q1 | 23Q2 | 23Q3 | 23Q4 | 24Q1 | 24Q2 |      | 24Q4 | 25Q1 | 25Q2 | 25Q3 | 25Q4 |
| :----------: | :-------: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: |
|     3.x      |   21.x    | 3.21 | 3.21 | 3.21 | 3.21 | 3.21 |      |      |      |      |      |      |      |
|     4.x      | 22.x-25.x | 4.22 | 4.23 | 4.24 | 4.25 | 4.25 | 4.25 | 4.25 | 4.25 | 4.25 |      |      |      |
|     5.x      | 26.x-29.x |      |      |      |      | 5.26 | 5.27 | 5.28 | 5.29 | 5.29 | 5.29 | 5.29 | 5.29 |
|     6.x      | 30.x-33.x |      |      |      |      |      |      |      |      | 6.30 | 6.31 | 6.32 | 6.33 |

**Legend**

|   Active    | Minor and patch releases with new features, compatible changes and bug fixes. |
| :---------: | ------------------------------------------------------------ |
| Maintenance | Patch releases with critical bug fixes.                      |
| End of life | Release is unsupported. Users should upgrade to a supported release. |
|   Future    | Projected release. Shown for planning purposes.              |

### C++ Tooling, Platform, and Library Support

Protobuf is committed to following the tooling, platform, and library support policy described in [Foundational C++ Support Policy](https://opensource.google/documentation/policies/cplusplus-support). For specific versions supported, see [Foundational C++ Support Matrix](https://github.com/google/oss-policies-info/blob/main/foundational-cxx-support-matrix.md).

## C#

The protoc version can be inferred from the Protobuf C# minor version number. Example: Protobuf C# version 3.25.x uses protoc version 25.x.

Future plans are shown in *italics* and are subject to change.

**Release support dates**

| Protobuf C# | Release date | End of support |
| :---------: | :----------: | :------------: |
|     3.x     | 16 Feb 2023  |      TBD       |

**Release support chart**

| Protobuf C# |  protoc   | 23Q1 | 23Q2 | 23Q3 | 23Q4 | 24Q1 | 24Q2 |      | 24Q4 | 25Q1 | 25Q2 | 25Q3 | 25Q4 |
| :---------: | :-------: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: |
|     3.x     | 22.x-33.x | 3.22 | 3.23 | 3.24 | 3.25 | 3.26 | 3.27 | 3.28 | 3.29 | 3.30 | 3.31 | 3.32 | 3.33 |

**Legend**

|   Active    | Minor and patch releases with new features, compatible changes and bug fixes. |
| :---------: | ------------------------------------------------------------ |
| Maintenance | Patch releases with critical bug fixes.                      |
| End of life | Release is unsupported. Users should upgrade to a supported release. |
|   Future    | Projected release. Shown for planning purposes.              |

### C# Platform and Library Support

Protobuf is committed to following the platform and library support policy described in [.NET Support Policy](https://opensource.google/documentation/policies/dotnet-support). For specific versions supported, see [Foundational .NET Support Matrix](https://github.com/google/oss-policies-info/blob/main/foundational-dotnet-support-matrix.md).

## Java

Java will target making major version bumps annually in Q1 of each year.

The protoc version can be inferred from the Protobuf Java minor version number. Example: Protobuf Java version 3.25.x uses protoc version 25.x.

Future plans are shown in *italics* and are subject to change.

**Release support dates**

| Protobuf Java | Release date | End of support |
| :-----------: | :----------: | :------------: |
|      3.x      | 16 Feb 2023  |  31 Mar 2026*  |
|      4.x      | 13 Mar 2024  |  31 Mar 2027   |
|      5.x      |   Q1 2026*   |  31 Mar 2028   |

**NOTE:** The maintenance support window for the Protobuf Java 3.x release will be 24 months rather than the typical 12 months for the final release in a major version line. Future major version updates (5.x, planned for Q1 2026) will adopt an improved [“rolling compatibility window”](https://protobuf.dev/support/cross-version-runtime-guarantee/#major) that should allow a return to 12-month support windows. There will be no major version bump in Q1 2025.

**Release support chart**

| Protobuf Java |  protoc   | 23Q1 | 23Q2 | 23Q3 | 23Q4 | 24Q1 | 24Q2 |      | 24Q4 | 25Q1 | 25Q2 | 25Q3 | 25Q4 |
| :-----------: | :-------: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: |
|      3.x      | 22.x-25.x | 3.22 | 3.23 | 3.24 | 3.25 | 3.25 | 3.25 | 3.25 | 3.25 | 3.25 | 3.25 | 3.25 | 3.25 |
|      4.x      | 26.x-33.x |      |      |      |      | 4.26 | 4.27 | 4.28 | 4.29 | 4.30 | 4.31 | 4.32 | 4.33 |

**Legend**

|   Active    | Minor and patch releases with new features, compatible changes and bug fixes. |
| :---------: | ------------------------------------------------------------ |
| Maintenance | Patch releases with critical bug fixes.                      |
| End of life | Release is unsupported. Users should upgrade to a supported release. |
|   Future    | Projected release. Shown for planning purposes.              |

### Java Platform and Library Support

Protobuf is committed to following the platform and library support policy described in [Java Support Policy](https://cloud.google.com/java/docs/supported-java-versions). For specific versions supported, see [Foundational Java Support Matrix](https://github.com/google/oss-policies-info/blob/main/foundational-java-support-matrix.md).

## Objective-C

The protoc version can be inferred from the Protobuf Objective-C minor version number. Example: Protobuf Objective-C version 3.25.x uses protoc version 25.x.

Future plans are shown in *italics* and are subject to change.

**Release support dates**

| Protobuf Objective-C | Release date | End of support |
| :------------------: | :----------: | :------------: |
|         3.x          | 16 Feb 2023  |  31 Mar 2026   |
|         4.x          |   Q1 2025    |      TBD       |

**Release support chart**

| Protobuf Objective-C |  protoc   | 23Q1 | 23Q2 | 23Q3 | 23Q4 | 24Q1 | 24Q2 |      | 24Q4 | 25Q1 | 25Q2 | 25Q3 | 25Q4 |
| :------------------: | :-------: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: |
|         3.x          | 22.x-29.x | 3.22 | 3.23 | 3.24 | 3.25 | 3.26 | 3.27 | 3.28 | 3.29 | 3.29 | 3.29 | 3.29 | 3.29 |
|         4.x          |   30.x+   |      |      |      |      |      |      |      |      | 4.30 | 4.31 | 4.32 | 4.33 |

**Legend**

|   Active    | Minor and patch releases with new features, compatible changes and bug fixes. |
| :---------: | ------------------------------------------------------------ |
| Maintenance | Patch releases with critical bug fixes.                      |
| End of life | Release is unsupported. Users should upgrade to a supported release. |
|   Future    | Projected release. Shown for planning purposes.              |

## PHP

The protoc version can be inferred from the Protobuf PHP minor version number. Example: Protobuf PHP version 3.25.x uses protoc version 25.x.

Future plans are shown in *italics* and are subject to change.

**Release support dates**

| Protobuf PHP | Release date | End of support |
| :----------: | :----------: | :------------: |
|     3.x      | 16 Feb 2023  |  31 Mar 2025   |
|     4.x      | 13 Mar 2024  |      TBD       |

**Release support chart**

| Protobuf PHP |  protoc   | 23Q1 | 23Q2 | 23Q3 | 23Q4 | 24Q1 | 24Q2 |      | 24Q4 | 25Q1 | 25Q2 | 25Q3 | 25Q4 |
| :----------: | :-------: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: |
|     3.x      | 22.x-25.x | 3.22 | 3.23 | 3.24 | 3.25 | 3.25 | 3.25 | 3.25 | 3.25 | 3.25 |      |      |      |
|     4.x      |   26.x+   |      |      |      |      | 4.26 | 4.27 | 4.28 | 4.29 | 4.30 | 4.31 | 4.32 | 4.33 |

**Legend**

|   Active    | Minor and patch releases with new features, compatible changes and bug fixes. |
| :---------: | ------------------------------------------------------------ |
| Maintenance | Patch releases with critical bug fixes.                      |
| End of life | Release is unsupported. Users should upgrade to a supported release. |
|   Future    | Projected release. Shown for planning purposes.              |

### PHP Platform and Library Support

Protobuf is committed to following the platform and library support policy described in [PHP Support Policy](https://cloud.google.com/php/getting-started/supported-php-versions). For specific versions supported, see [Foundational PHP Support Matrix](https://github.com/google/oss-policies-info/blob/main/foundational-php-support-matrix.md).

## Python

The protoc version can be inferred from the Protobuf Python minor version number. Example: Protobuf Python version 4.25.x uses protoc version 25.x.

Future plans are shown in *italics* and are subject to change.

**Release support dates**

| Protobuf Python | Release date | End of support |
| :-------------: | :----------: | :------------: |
|       4.x       | 16 Feb 2023  |  31 Mar 2025   |
|       5.x       | 13 Mar 2024  |  31 Mar 2026   |
|       6.x       |   Q1 2025    |      TBD       |

**Release support chart**

| Protobuf Python |  protoc   | 23Q1 | 23Q2 | 23Q3 | 23Q4 | 24Q1 | 24Q2 |      | 24Q4 | 25Q1 | 25Q2 | 25Q3 | 25Q4 |
| :-------------: | :-------: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: |
|       4.x       | 22.x-25.x | 4.22 | 4.23 | 4.24 | 4.25 | 4.25 | 4.25 | 4.25 | 4.25 | 4.25 |      |      |      |
|       5.x       | 26.x-29.x |      |      |      |      | 5.26 | 5.27 | 5.28 | 5.29 | 5.29 | 5.29 | 5.29 | 5.29 |
|       6.x       |   30.x+   |      |      |      |      |      |      |      |      | 6.30 | 6.31 | 6.32 | 6.33 |

**Legend**

|   Active    | Minor and patch releases with new features, compatible changes and bug fixes. |
| :---------: | ------------------------------------------------------------ |
| Maintenance | Patch releases with critical bug fixes.                      |
| End of life | Release is unsupported. Users should upgrade to a supported release. |
|   Future    | Projected release. Shown for planning purposes.              |

### Python Platform and Library Support

Protobuf is committed to following the platform and library support policy described in [Python Support Policy](https://cloud.google.com/python/docs/supported-python-versions). For specific versions supported, see [Foundational Python Support Matrix](https://github.com/google/oss-policies-info/blob/main/foundational-python-support-matrix.md).

## Ruby

The protoc version can be inferred from the Protobuf Ruby minor version number. Example: Protobuf Ruby version 3.25.x uses protoc version 25.x.

Future plans are shown in *italics* and are subject to change.

**Release support dates**

| Protobuf Ruby | Release date | End of support |
| :-----------: | :----------: | :------------: |
|      3.x      | 16 Feb 2023  |  31 Mar 2025   |
|      4.x      | 13 Mar 2024  |      TBD       |

**Release support chart**

| Protobuf Ruby |  protoc   | 23Q1 | 23Q2 | 23Q3 | 23Q4 | 24Q1 | 24Q2 |      | 24Q4 | 25Q1 | 25Q2 | 25Q3 | 25Q4 |
| :-----------: | :-------: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: |
|      3.x      | 22.x-25.x | 3.22 | 3.23 | 3.24 | 3.25 | 3.25 | 3.25 | 3.25 | 3.25 | 3.25 |      |      |      |
|      4.x      |   26.x+   |      |      |      |      | 4.26 | 4.27 | 4.28 | 4.29 | 4.30 | 4.31 | 4.32 | 4.33 |

**Legend**

|   Active    | Minor and patch releases with new features, compatible changes and bug fixes. |
| :---------: | ------------------------------------------------------------ |
| Maintenance | Patch releases with critical bug fixes.                      |
| End of life | Release is unsupported. Users should upgrade to a supported release. |
|   Future    | Projected release. Shown for planning purposes.              |

### Ruby Platform and Library Support

Protobuf is committed to following the platform and library support policy described in [Ruby Support Policy](https://cloud.google.com/ruby/getting-started/supported-ruby-versions). For specific versions supported, see [Foundational Ruby Support Matrix](https://github.com/google/oss-policies-info/blob/main/foundational-ruby-support-matrix.md).

JRuby is not officially supported, but we provide unofficial support for the latest JRuby version targeting compatibility with our minimum Ruby version or above on a best-effort basis.
