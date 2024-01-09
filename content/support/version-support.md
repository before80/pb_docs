+++
title = "版本支持"
weight = 910
description = "为语言实现提供的支持窗口列表。"
type = "docs"

+++

## Version Support 版本支持

A list of the support windows provided for language implementations.

​	为语言实现提供的支持窗口列表。



Support windows for protoc and the various languages are covered in the tables later in this topic. Version numbers throughout this topic use [SemVer](https://semver.org/) conventions; in the version “3.21.7,” we say that “3” is the major version, “21” is the minor version, and “7” is the micro or patch number.

​	支持 protoc 和各种语言的 Windows 版本将在本主题后面的表格中介绍。本主题中的版本号使用 SemVer 约定；在版本“3.21.7”中，我们称“3”为主要版本，“21”为次要版本，“7”为微版本或修补程序编号。

Starting with the v20.x protoc release, we changed our versioning scheme to enable nimbler updates to language-specific parts of Protocol Buffers. In the new scheme, each language has its own major version that can be incremented independently of other languages. The minor and patch versions, however, remain coupled. This allows us to introduce breaking changes into some languages without requiring a bump of the major version in languages that do not experience a breaking change. For example, a single release might include protoc version 24.0, Java runtime version 4.24.0 and C# runtime version 3.24.0.

​	从 v20.x protoc 版本开始，我们更改了版本控制方案，以便对 Protocol Buffers 的特定于语言的部分进行更灵活的更新。在新方案中，每种语言都有自己的主要版本，可以独立于其他语言进行递增。但是，次要版本和修补程序版本仍然是耦合的。这使我们能够在某些语言中引入重大更改，而无需在未经历重大更改的语言中增加主要版本。例如，单个版本可能包括 protoc 版本 24.0、Java 运行时版本 4.24.0 和 C# 运行时版本 3.24.0。

The first instance of this new versioning scheme was the 4.21.0 version of the Python API, which followed the preceding version, 3.20.1. Other language APIs released at the same time were released as 3.21.0.

​	这种新版本控制方案的第一个实例是 Python API 的 4.21.0 版本，它紧随前一个版本 3.20.1。同时发布的其他语言 API 发布为 3.21.0。

## 发布节奏 Release Cadence 

Protobuf does not officially have a release cadence; however, we strive to release updates quarterly, on a best-effort basis. Our support windows are defined by our [library breaking change policy](https://opensource.google/documentation/policies/library-breaking-change).

​	Protobuf 并没有官方发布节奏；但是，我们努力在季度内发布更新，尽最大努力。我们的支持窗口由我们的库重大变更政策定义。

## 支持期限 Support Duration 

The most recent release is always supported. Support for earlier minor versions ends when a new minor version under the same major version is released. Support for earlier major versions ends four quarters beyond the quarter that the breaking release is introduced. For example, when Python 4.21.0 was released in May of 2022, that set the end of public support of Python 3.20.1 at the [end of 2023 Q2](https://protobuf.dev/support/version-support/#python).

​	始终支持最新版本。早期小版本的支持在同一主要版本下发布新的小版本时结束。早期主要版本的支持在引入重大变更版本的季度后四个季度结束。例如，当 Python 4.21.0 于 2022 年 5 月发布时，这将把 Python 3.20.1 的公开支持结束时间定为 2023 年第二季度末。

The following sections provide a visual guide to the support for each language.

​	以下部分为每种语言的支持提供了一个可视化指南。

## C++

The C++ 4.25.x runtime was first released in 2023 Q4.

​	C++ 4.25.x 运行时首次发布在 2023 年第四季度。

The C++ 3.21.x runtime was first released in 2022 Q2 and has support until 2024 Q1.

​	C++ 3.21.x 运行时首次发布在 2022 年第二季度，并支持到 2024 年第一季度。

C++ will target making major version bumps annually in Q1 of each year.

​	C++ 将以每年第一季度进行重大版本升级为目标。

|                            protoc                            |  C++   | 22Q2 | 22Q3 | 22Q4 | 23Q1 | 23Q2 | 23Q3 | 23Q4 | 24Q1 | 24Q2 |
| :----------------------------------------------------------: | :----: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: |
|                             21.x                             | 3.21.x |  IR  |  PS  |  PS  |  PS  |  PS  |  PS  |  PS  |  SE  |      |
|                             22.x                             | 4.22.x |      |      |      |  IR  |      |      |      |      |      |
|                             23.x                             | 4.23.x |      |      |      |      |  IR  |      |      |      |      |
|                             24.x                             | 4.24.x |      |      |      |      |      |  IR  |      |      |      |
|                             25.x                             | 4.25.x |      |      |      |      |      |      |  IR  |  PS  |  PS  |
| The cells below are projections of future releases, but are not guarantees 下方的单元格是未来版本的预测，但不能保证 that those releases will happen, or that they will happen on that schedule. 这些版本会发布，或者会按照该时间表发布。 |        |      |      |      |      |      |      |      |      |      |
|                             26.x                             | 5.26.x |      |      |      |      |      |      |      |  IR  |      |
|                             27.x                             | 5.27.x |      |      |      |      |      |      |      |      |  IR  |

| **Legend 图例**                    |
| ---------------------------------- |
| Initial release (IR) 初始版本 (IR) |
| Public support (PS) 公开支持 (PS)  |
| Support ends (SE) 支持结束 (SE)    |

### C++ 工具、平台和库支持 C++ Tooling, Platform, and Library Support 

Protobuf is committed to following the tooling, platform, and library support policy described in [Foundational C++ Support Policy](https://opensource.google/documentation/policies/cplusplus-support). For specific versions supported, see [Foundational C++ Support Matrix](https://github.com/google/oss-policies-info/blob/main/foundational-cxx-support-matrix.md).

​	Protobuf 致力于遵循基本 C++ 支持政策中描述的工具、平台和库支持政策。有关受支持的特定版本，请参阅基本 C++ 支持矩阵。

## C#

The C# 3.25.x runtime was first released in 2023 Q4.

​	C# 3.25.x 运行时首次发布在 2023 年第四季度。

|                            protoc                            |   C#   | 23Q1 | 23Q2 | 23Q3 | 23Q4 | 24Q1 | 24Q2 |
| :----------------------------------------------------------: | :----: | :--: | :--: | :--: | :--: | :--: | :--: |
|                             22.x                             | 3.22.x |  IR  |      |      |      |      |      |
|                             23.x                             | 3.23.x |      |  IR  |      |      |      |      |
|                             24.x                             | 3.24.x |      |      |  IR  |      |      |      |
|                             25.x                             | 3.25.x |      |      |      |  IR  |  PS  |  PS  |
| The cells below are projections of future releases, but are not guarantees 下表是未来版本的预测，但不能保证 that those releases will happen, or that they will happen on that schedule. 这些版本会发布，或者会按照该时间表发布。 |        |      |      |      |      |      |      |
|                             26.x                             |        |      |      |      |      |  IR  |      |
|                             27.x                             |        |      |      |      |      |      |  IR  |

| **Legend 图例**                    |
| ---------------------------------- |
| Initial release (IR) 初始版本 (IR) |
| Public support (PS) 公共支持 (PS)  |
| Support ends (SE) 支持结束 (SE)    |

### C# 平台和库支持 C# Platform and Library Support

Protobuf is committed to following the platform and library support policy described in [.NET Support Policy](https://opensource.google/documentation/policies/dotnet-support).

​	Protobuf 致力于遵循 .NET 支持政策中描述的平台和库支持政策。

## Java

Java will target making major version bumps annually in Q1 of each year.

​	Java 将在每年的第一季度针对主要版本进行年度升级。

|                            protoc                            |  Java  | 22Q2 | 22Q3 | 22Q4 | 23Q1 | 23Q2 | 23Q3 | 23Q4 | 24Q1 | 24Q2 | 24Q3 |
| :----------------------------------------------------------: | :----: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: |
|                             21.x                             | 3.21.x |  IR  |  PS  |  PS  |      |      |      |      |      |      |      |
|                             22.x                             | 3.22.x |      |      |      |  IR  |      |      |      |      |      |      |
|                             23.x                             | 3.23.x |      |      |      |      |  IR  |      |      |      |      |      |
|                             24.x                             | 3.24.x |      |      |      |      |      |  IR  |      |      |      |      |
|                             25.x                             | 3.25.x |      |      |      |      |      |      |  IR  |  PS  |  PS  |  PS  |
| The cells below are projections of future releases, but are not guarantees 下方的单元格是未来版本的预测，但不能保证 that those releases will happen, or that they will happen on that schedule. 这些版本会发布，或者会按照该时间表发布。 |        |      |      |      |      |      |      |      |      |      |      |
|                             26.x                             | 4.26.x |      |      |      |      |      |      |      |  IR  |      |      |
|                             27.x                             | 4.27.x |      |      |      |      |      |      |      |      |  IR  |      |

| **Legend 图例**                    |
| ---------------------------------- |
| Initial release (IR) 初始发布 (IR) |
| Public support (PS) 公开支持 (PS)  |
| Support ends (SE) 支持结束 (SE)    |

## Objective-C

The Objective-C 3.25.x runtime was first released in 2023 Q4.

​	Objective-C 3.25.x 运行时首次发布在 2023 年第四季度。

|                            protoc                            |  ObjC  | 23Q1 | 23Q2 | 23Q3 | 23Q4 | 24Q1 | 24Q2 |
| :----------------------------------------------------------: | :----: | :--: | :--: | :--: | :--: | :--: | :--: |
|                             22.x                             | 3.22.x |  IR  |      |      |      |      |      |
|                             23.x                             | 3.23.x |      |  IR  |      |      |      |      |
|                             24.x                             | 3.24.x |      |      |  IR  |      |      |      |
|                             25.x                             | 3.25.x |      |      |      |  IR  |  PS  |  PS  |
| The cells below are projections of future releases, but are not guarantees 下方的单元格是未来版本的预测，但并非保证 that those releases will happen, or that they will happen on that schedule. 这些版本会发布，或者会按照该时间表发布。 |        |      |      |      |      |      |      |
|                             26.x                             |        |      |      |      |      |  IR  |      |
|                             27.x                             |        |      |      |      |      |      |  IR  |

| **Legend 图例**                    |
| ---------------------------------- |
| Initial release (IR) 初始发布 (IR) |
| Public support (PS) 公开支持 (PS)  |
| Support ends (SE) 支持结束 (SE)    |

## PHP

The PHP 3.25.x runtime was first released in 2023 Q4.

​	PHP 3.25.x 运行时首次发布在 2023 年第四季度。

|                            protoc                            |  PHP   | 23Q1 | 23Q2 | 23Q3 | 23Q4 | 24Q1 | 24Q2 |
| :----------------------------------------------------------: | :----: | :--: | :--: | :--: | :--: | :--: | :--: |
|                             22.x                             | 3.22.x |  IR  |      |      |      |      |      |
|                             23.x                             | 3.23.x |      |  IR  |      |      |      |      |
|                             24.x                             | 3.24.x |      |      |  IR  |      |      |      |
|                             25.x                             | 3.25.x |      |      |      |  IR  |  PS  |  PS  |
| The cells below are projections of future releases, but are not guarantees 下方的单元格是未来版本的预测，但不能保证 that those releases will happen, or that they will happen on that schedule. 这些版本会发布，或者会按照该时间表发布。 |        |      |      |      |      |      |      |
|                             26.x                             |        |      |      |      |      |  IR  |      |
|                             27.x                             |        |      |      |      |      |      |  IR  |

| **Legend 图例**                    |
| ---------------------------------- |
| Initial release (IR) 初始版本 (IR) |
| Public support (PS) 公开支持 (PS)  |
| Support ends (SE) 支持结束 (SE)    |

### PHP 平台和库支持 PHP Platform and Library Support 

Protobuf is committed to following the platform and library support policy described in [PHP Support Policy](https://cloud.google.com/php/getting-started/supported-php-versions). For specific versions supported, see [Foundational PHP Support Matrix](https://github.com/google/oss-policies-info/blob/main/foundational-php-support-matrix.md).

​	Protobuf 致力于遵循 PHP 支持政策中描述的平台和库支持政策。有关受支持的特定版本，请参阅基础 PHP 支持矩阵。

## Python

The Python 4.25.x runtime was first released in 2023 Q4.

​	Python 4.25.x 运行时首次发布在 2023 年第四季度。

The Python 3.x runtime went out of support on July 1, 2023.

​	Python 3.x 运行时于 2023 年 7 月 1 日停止支持。

|                            protoc                            | Python | 22Q3 | 22Q4 | 23Q1 | 23Q2 | 23Q3 | 23Q4 | 24Q1 | 24Q2 |
| :----------------------------------------------------------: | :----: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: |
|                             20.x                             | 3.20.x |  PS  |  PS  |  PS  |  SE  |      |      |      |      |
|                             21.x                             | 4.21.x |  PS  |  PS  |      |      |      |      |      |      |
|                             22.x                             | 4.22.x |      |      |  IR  |      |      |      |      |      |
|                             23.x                             | 4.23.x |      |      |      |  IR  |      |      |      |      |
|                             24.x                             | 4.24.x |      |      |      |      |  IR  |      |      |      |
|                             25.x                             | 4.25.x |      |      |      |      |      |  IR  |  PS  |  PS  |
| The cells below are projections of future releases, but are not guarantees 下表是未来版本的预测，但不能保证 that those releases will happen, or that they will happen on that schedule. 这些版本会发布，或者会按照该时间表发布。 |        |      |      |      |      |      |      |      |      |
|                             26.x                             |        |      |      |      |      |      |      |  IR  |      |
|                             27.x                             |        |      |      |      |      |      |      |      |  IR  |

| **Legend 图例**                    |
| ---------------------------------- |
| Initial release (IR) 初始版本 (IR) |
| Public support (PS) 公开支持 (PS)  |
| Support ends (SE) 支持结束 (SE)    |

### Python 平台和库支持 Python Platform and Library Support 

Protobuf is committed to following the platform and library support policy described in [Python Support Policy](https://cloud.google.com/python/docs/supported-python-versions). For specific versions supported, see [Foundational Python Support Matrix](https://github.com/google/oss-policies-info/blob/main/foundational-python-support-matrix.md).

​	Protobuf 致力于遵循 Python 支持政策中描述的平台和库支持政策。有关受支持的特定版本，请参阅基础 Python 支持矩阵。

## Ruby

The Ruby 3.25.x runtime was first released in 2023 Q4.

​	Ruby 3.25.x 运行时首次发布在 2023 年第四季度。

|                            protoc                            |  Ruby  | 22Q2 | 22Q3 | 22Q4 | 23Q1 | 23Q2 | 23Q3 | 23Q4 | 24Q1 | 24Q2 |
| :----------------------------------------------------------: | :----: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: |
|                             21.x                             | 3.21.x |  IR  |  PS  |  PS  |      |      |      |      |      |      |
|                             22.x                             | 3.22.x |      |      |      |  IR  |      |      |      |      |      |
|                             23.x                             | 3.23.x |      |      |      |      |  IR  |      |      |      |      |
|                             24.x                             | 3.24.x |      |      |      |      |      |  IR  |      |      |      |
|                             25.x                             | 3.25.x |      |      |      |      |      |      |  IR  |  PS  |  PS  |
| The cells below are projections of future releases, but are not guarantees 以下单元格是未来版本的预测，但不能保证 that those releases will happen, or that they will happen on that schedule. 这些版本会发布，或者会按照该时间表发布。 |        |      |      |      |      |      |      |      |      |      |
|                             26.x                             |        |      |      |      |      |      |      |      |  IR  |      |
|                             27.x                             |        |      |      |      |      |      |      |      |      |  IR  |

| **Legend 图例**                    |
| ---------------------------------- |
| Initial release (IR) 初始版本 (IR) |
| Public support (PS) 公开支持 (PS)  |
| Support ends (SE) 支持结束 (SE)    |

### Ruby 平台和库支持 Ruby Platform and Library Support 

Protobuf is committed to following the platform and library support policy described in [Ruby Support Policy](https://cloud.google.com/ruby/getting-started/supported-ruby-versions). For specific versions supported, see [Foundational Ruby Support Matrix](https://github.com/google/oss-policies-info/blob/main/foundational-ruby-support-matrix.md).

​	Protobuf 致力于遵循 Ruby 支持政策中描述的平台和库支持政策。有关受支持的特定版本，请参阅基础 Ruby 支持矩阵。

