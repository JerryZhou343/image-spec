# Open Container Initiative
## Image Format Specification

This specification defines an OCI Image, consisting of a [manifest](manifest.md), an [image index](image-index.md) (optional), a set of [filesystem layers](layer.md), and a [configuration](config.md).

这个规范定义了 OCI 镜像， 包含[清单](manifest.md),[镜像索引](image-index.md),一套[文件系统层](layer.md)和[配置](config.md)

The goal of this specification is to enable the creation of interoperable tools for building, transporting, and preparing a container image to run.
本规范的目标是实现创建可互操作的工具，用于构建、运输和准备运行容器镜像。

### Table of Contents

- [Introduction](spec.md)介绍
- [Notational Conventions](#notational-conventions)符号约定
- [Overview](#overview) 预览
    - [Understanding the Specification](#understanding-the-specification)理解规格
    - [Media Types](media-types.md)媒介类型
- [Content Descriptors](descriptor.md) 内容描述
- [Image Layout](image-layout.md) 镜像布局
- [Image Manifest](manifest.md)镜像清单
- [Image Index](image-index.md)镜像索引
- [Filesystem Layers](layer.md)文件系统分层
- [Image Configuration](config.md)镜像配置
- [Annotations](annotations.md)注解
- [Conversion](conversion.md)转换
- [Considerations](considerations.md)考量
    - [Extensibility](considerations.md#extensibility)扩展性
    - [Canonicalization](considerations.md#canonicalization) 规范化

## Notational Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119) (Bradner, S., "Key words for use in RFCs to Indicate Requirement Levels", BCP 14, RFC 2119, March 1997).

关键词 "MUST"、"MUST NOT"、"REQUIRED"、"SHALL"、"SHALL NOT"、"SHOULD"、"SHOULD NOT"、"RECOMMENDED"、"NOT RECOMMENDED"、"MAY "和 "OPTIONAL "应按照[RFC 2119](http://tools.ietf.org/html/rfc2119)中的描述进行解释(Bradner, S., "Key words for use in RFCs to Indicate Requirement Levels", BCP 14, RFC 2119, March 1997)。

The key words "unspecified", "undefined", and "implementation-defined" are to be interpreted as described in the [rationale for the C99 standard][c99-unspecified].
"未说明"、"未定义 "和 "实施定义 "等关键词应按照[C99标准的理由][c99-未说明]中的说明来解释。

An implementation is not compliant if it fails to satisfy one or more of the MUST, MUST NOT, REQUIRED, SHALL, or SHALL NOT requirements for the protocols it implements.
如果一个实现不能满足它所实现的协议的MUST、MUST NOT、REQUIRED、SHALL或SHALL NOT中的一个或多个要求，那么它就不符合要求。

An implementation is compliant if it satisfies all the MUST, MUST NOT, REQUIRED, SHALL, and SHALL NOT requirements for the protocols it implements.
如果一个实现满足它所实现的协议的所有MUST、MUST NOT、REQUIRED、SHALL和SHALL NOT要求，那么这个实现就是合规的。

## Overview

At a high level the image manifest contains metadata about the contents and dependencies of the image including the content-addressable identity of one or more [filesystem layer changeset](layer.md) archives that will be unpacked to make up the final runnable filesystem.
在高层次上，镜像清单包含了关于镜像的内容和依赖性的元数据，包括一个或多个[文件系统层变化集](layer.md)存档的内容可寻址标识，这些存档将被解压以组成最终的可运行文件系统。

The image configuration includes information such as application arguments, environments, etc.
镜像配置包括应用程序参数、环境变量等信息。

The image index is a higher-level manifest which points to a list of manifests and descriptors.
镜像索引是一个更高级的清单，它指向一个清单和描述符的列表
Typically, these manifests may provide different implementations of the image, possibly varying by platform or other attributes.
通常情况下，这些清单可以提供镜像的不同实现，可能因平台或其他属性而不同。
![](img/build-diagram.png)

Once built the OCI Image can then be discovered by name, downloaded, verified by hash, trusted through a signature, and unpacked into an [OCI Runtime Bundle](https://github.com/opencontainers/runtime-spec/blob/master/bundle.md).
一旦建立了OCI镜像，就可以通过名称发现、下载、通过哈希验证、通过签名信任，并解压成[OCI运行时捆绑包]（https://github.com/opencontainers/runtime-spec/blob/master/bundle.md）。
![](img/run-diagram.png)

### Understanding the Specification

The [OCI Image Media Types](media-types.md) document is a starting point to understanding the overall structure of the specification.
OCI镜像媒介类型](media-types.md)文档是理解该规范整体结构的起点。
The high-level components of the spec include:
该规格的高级组成部分包括：

* [Image Manifest](manifest.md) - a document describing the components that make up a container image
* [镜像清单](manifest.md) - 描述构成容器镜像的组件的文件。
* [Image Index](image-index.md) - an annotated index of image manifests
* [镜像索引](image-index.md) - 镜像清单的注释索引。
* [Image Layout](image-layout.md) - a filesystem layout representing the contents of an image
* [镜像布局](image-layout.md) - 表示镜像内容的文件系统布局。
* [Filesystem Layer](layer.md) - a changeset that describes a container's filesystem
* [文件系统层](layer.md) - 描述容器的文件系统的变化集
* [Image Configuration](config.md) - a document determining layer ordering and configuration of the image suitable for translation into a [runtime bundle][runtime-spec]
* [镜像配置](config.md) - 确定如何转换成[运行时捆绑][运行时规格]的镜像的层序和配置的文档；
* [Conversion](conversion.md) - a document describing how this translation should occur
* [转换](conversion.md) - 说明应如何进行转换的文档。
* [Descriptor](descriptor.md) - a reference that describes the type, metadata and content address of referenced content
* [描述符](descriptor.md) - 描述被引用内容的类型、元数据和内容地址的参考。

Future versions of this specification may include the following OPTIONAL features:
本规范的未来版本可能包括以下OPTIONAL功能。

* Signatures that are based on signing image content address
* Naming that is federated based on DNS and can be delegated

[c99-unspecified]: http://www.open-std.org/jtc1/sc22/wg14/www/C99RationaleV5.10.pdf#page=18
[runtime-spec]: https://github.com/opencontainers/runtime-spec
