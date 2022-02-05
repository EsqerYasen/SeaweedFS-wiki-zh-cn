# SeaweedFS

[![Slack](https://img.shields.io/badge/slack-purple)](https://join.slack.com/t/seaweedfs/shared_invite/enQtMzI4MTMwMjU2MzA3LTEyYzZmZWYzOGQ3MDJlZWMzYmI0OTE4OTJiZjJjODBmMzUxNmYwODg0YjY3MTNlMjBmZDQ1NzQ5NDJhZWI2ZmY) [![Twitter](https://img.shields.io/twitter/follow/seaweedfs.svg?style=social&label=Follow)](https://twitter.com/intent/follow?screen_name=seaweedfs) [![Build Status](https://img.shields.io/github/workflow/status/chrislusf/seaweedfs/Go)](https://github.com/chrislusf/seaweedfs/actions/workflows/go.yml) [![GoDoc](https://godoc.org/github.com/chrislusf/seaweedfs/weed?status.svg)](https://godoc.org/github.com/chrislusf/seaweedfs/weed) [![Wiki](https://img.shields.io/badge/docs-wiki-blue.svg)](https://github.com/chrislusf/seaweedfs/wiki) [![Docker Pulls](https://img.shields.io/docker/pulls/chrislusf/seaweedfs?maxAge=4800)](https://hub.docker.com/r/chrislusf/seaweedfs/) [![SeaweedFS on Maven Central](https://img.shields.io/maven-central/v/com.github.chrislusf/seaweedfs-client)](https://search.maven.org/search?q=g:com.github.chrislusf)

![SeaweedFS Logo](https://raw.githubusercontent.com/chrislusf/seaweedfs/master/note/seaweedfs.png)

<h2 align="center"><a href="https://www.patreon.com/seaweedfs"> Patreon 上赞助 SeaweedFS</a></h2>

  SeaweedFS是一个独立的Apache许可的开源项目，其正在进行的开发完全归功于[这些赞助者](https://github.com/chrislusf/seaweedfs/blob/master/backers.md "赞助者清单")的支持。如果您希望将SeaweedFS变得更强壮，请考虑加入我们[的Patreon赞助商](https://www.patreon.com/seaweedfs)。

您的支持将受到我和其他支持者的衷心感谢！

<!--
<h4 align="center">Platinum</h4>

<p align="center">
  <a href="" target="_blank">
    Add your name or icon here
  </a>
</p>
-->

### 金牌赞助商

![shuguang](https://raw.githubusercontent.com/chrislusf/seaweedfs/master/note/shuguang.png)

---

* [下载适用于不同平台的二进制文](https://github.com/chrislusf/seaweedfs/releases/latest)

- [SeaweedFS on Slack](https://join.slack.com/t/seaweedfs/shared_invite/enQtMzI4MTMwMjU2MzA3LTEyYzZmZWYzOGQ3MDJlZWMzYmI0OTE4OTJiZjJjODBmMzUxNmYwODg0YjY3MTNlMjBmZDQ1NzQ5NDJhZWI2ZmY)
- [SeaweedFS on Twitter](https://twitter.com/SeaweedFS)
- [SeaweedFS on Telegram](https://t.me/Seaweedfs)
- [SeaweedFS Mailing List](https://groups.google.com/d/forum/seaweedfs)
- [Wiki 文档](https://github.com/chrislusf/seaweedfs/wiki)
- [SeaweedFS 白皮书](https://github.com/chrislusf/seaweedfs/wiki/SeaweedFS_Architecture.pdf)
- [SeaweedFS 幻灯片介绍 - 2021.5](https://docs.google.com/presentation/d/1DcxKWlINc-HNCjhYeERkpGXXm6nTCES8mi2W5G0Z4Ts/edit?usp=sharing)
- [SeaweedFS 幻灯片介绍 - 2019.3](https://www.slideshare.net/chrislusf/seaweedfs-introduction)

目录
====

* [快速入门](#quick-start)
* [介绍](#introduction)
* [功能](#features)
  * [附加功能](#additional-features)
  * [文件管理器功能](#filer-features)
* [实例: 用SeaweedFS做对象存储](#example-Using-Seaweed-Object-Store)
* [架构](#architecture)
* [跟其他文件系统对比](#compared-to-other-file-systems)
  * [跟HDFS对比](#compared-to-hdfs)
  * [跟GlusterFS, Ceph对比](#compared-to-glusterfs-ceph)
  * [跟 GlusterFS对比](#compared-to-glusterfs)
  * [跟Ceph比较](#compared-to-ceph)
* [开发者计划](#dev-plan)
* [安装指南](#installation-guide)
* [跟磁盘相关的话题](#disk-related-topics)
* [基准](#Benchmark)
* [License - 许可证](#license)

## S3 API  Docker 快速启动

`docker run -p 8333:8333 chrislusf/seaweedfs server -s3`

## 二进制文件快速启动

* 从[https://github.com/chrislusf/seaweedfs/releases](https://github.com/chrislusf/seaweedfs/releases)下载最新的二进制文件并解压的到二进制文件 `weed` 或 `weed.exe`
* 执行 `weed server -dir=/some/data/dir -s3` 启动一个主服务 ，一个卷服务, 一个文件管理器和一个S3 gateway.

此外，要增加容量，只需通过在本地或在其他计算机上或数千台计算机上运行来添加更多卷服务器即可。就是这样！`weed volume -dir="/some/data/dir2" -mserver="<master_host>:9333" -port=8081`

## 介绍

SeaweedFS是一个简单且高度可扩展的分布式文件系统。我们有两个目标：

1. 存储数十亿个文件！
2. 快速处理文件

  SeaweedFS最初是作为对象存储的形式以有效地处理小文件开始的。主服务器不管理服务器上的所有文件元数据，而是仅管理卷服务器上的卷，卷服务器管理文件及其元数据。这减轻了主服务器的并发压力，并将文件元数据扩散到卷服务器中，从而允许更快的文件访问（O（1），通常只有一个磁盘读取操作）。

  每个文件的元数据只有 40 字节的磁盘存储开销。O（1） 磁盘读取非常简单，欢迎您通过实际用例来挑战性能。

SeaweedFS从[Facebook的Haystack设计论文](http://www.usenix.org/event/osdi10/tech/full_papers/Beaver.pdf) 实现开始的。此外，SeaweedFS使用f4的想法实现了擦除编码[：Facebook的Warm BLOB存储系统](https://www.usenix.org/system/files/conference/osdi14/osdi14-paper-muralidhar.pdf)，并且与[Facebook的Tectonic文件系统](https://www.usenix.org/system/files/fast21-pan.pdf)有很多相似之处。

  在对象存储之上，可选的[Filer](https://github.com/chrislusf/seaweedfs/wiki/Directories-and-Files)可以支持目录和 POSIX 属性。Filer是一个单独的线性可扩展无状态服务器，具有可自定义的元数据存储，例如MySql，Postgres，Redis，Cassandra，HBase，Mongodb，Elastic Search，LevelDB，RocksDB，Sqlite，MemSql，TiDB，Etcd，CockroachDB等。

  对于任何分布式键值存储，可以将大值卸载到 SeaweedFS。凭借快速的访问速度和线性可扩展的容量，SeaweedFS可以作为分布式[的Key-Big-Value存储](https://github.com/chrislusf/seaweedfs/wiki/Filer-as-a-Key-Large-Value-Store)工作。

  SeaweedFS可以透明地与云集成。凭借本地集群上的热数据，以及具有O（1）访问时间的云上的暖数据，SeaweedFS可以实现快速的本地访问时间和弹性云存储容量。更重要的是，云存储访问API成本最小化。比直接云存储更快，更便宜！

[回到目录](#目录)

## 附加功

* 可以选择无副本或不同的副本级别，机架和数据中心感知。
* 自动主服务器故障切换 - 无单点故障 （SPOF）。
* 自动 Gzip 压缩取决于文件 MIME 类型。
* 自动压缩，可在删除或更新后回收磁盘空间。
* 自动输入 TTL 过期。
* 任何具有一些磁盘空间的服务器都可以添加到总存储空间中。
* 添加/删除服务器**不会**导致任何数据重新平衡，除非由管理命令触发。
* 可选的图片大小调整。
* 支持电子标签，接受范围，上次修改时间等
* 支持in-memory/leveldb/readonly 模式调整，以实现内存/性能平衡。
* 支持重新平衡可写卷和只读卷。
* [可自定义的多个存储层](https://github.com/chrislusf/seaweedfs/wiki/Tiered-Storage)：可自定义的存储磁盘类型，以平衡性能和成本。
* [透明的云集成](https://github.com/chrislusf/seaweedfs/wiki/Cloud-Tier)：通过分层云存储无限容量，用于热数据。
* [用于热存储的擦除编码](https://github.com/chrislusf/seaweedfs/wiki/Erasure-coding-for-warm-storage)机架感知 10.4 擦除编码可降低存储成本并提高可用性。

[回到目录](#目录)

## 文件管理器功

* [文件管理器服务器](https://github.com/chrislusf/seaweedfs/wiki/Directories-and-Files)通过 http 提供"正常"目录和文件。
* [文件 TTL](https://github.com/chrislusf/seaweedfs/wiki/Filer-Stores) 会自动使文件元数据和实际文件数据过期。
* [挂载文件管理器](https://github.com/chrislusf/seaweedfs/wiki/FUSE-Mount)通过 FUSE 直接作为本地目录读取和写入文件。
* [文件管理器存储复制](https://github.com/chrislusf/seaweedfs/wiki/Filer-Store-Replication)为文件管理器元数据数据存储启用 HA。
* [主动-主动复制](https://github.com/chrislusf/seaweedfs/wiki/Filer-Active-Active-cross-cluster-continuous-synchronization)支持异步单向或双向跨群集连续复制。
* [与 Amazon S3 兼容的 API](https://github.com/chrislusf/seaweedfs/wiki/Amazon-S3-API) 使用 S3 工具访问文件。
* [Hadoop 兼容文件系统](https://github.com/chrislusf/seaweedfs/wiki/Hadoop-Compatible-File-System)从 Hadoop/Spark/Flink/etc 访问文件，甚至运行 HBase。
* [Async Replication To Cloud](https://github.com/chrislusf/seaweedfs/wiki/Async-Replication-to-Cloud)具有极快的本地访问和备份到Amazon S3，Google Cloud Storage，Azure，BackBlaze的速度。
* [WebDAV](https://github.com/chrislusf/seaweedfs/wiki/WebDAV) 在 Mac 和 Windows 上作为映射驱动器访问，或从移动设备访问。
* [AES256-GCM加密存储](https://github.com/chrislusf/seaweedfs/wiki/Filer-Data-Encryption)安全地存储加密数据。
* [超大型文件](https://github.com/chrislusf/seaweedfs/wiki/Data-Structure-for-Large-Files)以数十 TB 为单位存储大型或超大型文件。
* [云盘](https://github.com/chrislusf/seaweedfs/wiki/Cloud-Drive-Architecture)将云存储挂载到本地集群，缓存后可通过异步回写功能实现快速读写。
* [远程对象存储](https://github.com/chrislusf/seaweedfs/wiki/Gateway-to-Remote-Object-Storage)网关将存储桶操作镜像到远程对象存储，以及 [Cloud Drive](https://github.com/chrislusf/seaweedfs/wiki/Cloud-Drive-Architecture)

## Kubernetes

* [Kubernetes CSI Driver][SeaweedFsCsiDriver] 容器存储接口 （CSI） 驱动程序. [![Docker Pulls](https://img.shields.io/docker/pulls/chrislusf/seaweedfs-csi-driver.svg?maxAge=4800)](https://hub.docker.com/r/chrislusf/seaweedfs-csi-driver/)
* [SeaweedFS Operator](https://github.com/seaweedfs/seaweedfs-operator)

[回到目录](#目录)

## 示例: 用SeaweedFS 做对象存储

默认情况下，主节点在端口 9333 上运行，卷节点在端口 8080 上运行。让我们在端口 8080 和 8081 上启动一个主节点和两个卷节点。理想情况下，它们应该从不同的机器启动。我们将以 localhost 为例。

SeaweedFS 使用 HTTP REST 操作来读取、写入和删除。响应采用 JSON 或 JSONP 格式。

### 启动主服务器

```shell
> ./weed master
```

### 启动卷服务器

```shell
> weed volume -dir="/tmp/data1" -max=5  -mserver="localhost:9333" -port=8080 &
> weed volume -dir="/tmp/data2" -max=10 -mserver="localhost:9333" -port=8081 &
```

### 写入文

要上传文件：首先，发送 HTTP POST、PUT 或 GET 请求以获取卷服务器 URL：`/dir/assign `fid

```shell
> curl http://localhost:9333/dir/assign  
{"count":1,"fid":"3,01637037d6","url":"127.0.0.1:8080","publicUrl":"localhost:8080"}
```

其次，要存储文件内容，请从响应发送HTTP多部分POST请求：`url + '/' + fid`

```shell
> curl -F file=@/home/chris/myphoto.jpg http://127.0.0.1:8080/3,01637037d6
{"name":"myphoto.jpg","size":43234,"eTag":"1cc0118e"}
```

要进行更新，请发送另一个包含更新文件内容的 POST 请求。

要删除，请向同一 URL 发送 HTTP 删除请求：`url + '/' + fid`

```shell
> curl -X DELETE http://127.0.0.1:8080/3,01637037d6   
```

### 保存文件 ID

现在，您可以将 ，3，01637037d6（在本例中）保存到数据库字段 `fid`

开头的数字 3 表示卷 ID。逗号后是一个文件密钥 01 和一个文件 cookie 637037d6。

卷 ID 是无符号的 32 位整数。文件密钥是无符号的 64 位整数。文件 Cookie 是一个无符号的 32 位整数，用于防止 URL 猜测。

文件密钥和文件 Cookie 都以十六进制编码。您可以将<卷 ID、文件密钥、文件 cookie>元组存储在您自己的格式中，或者只是将 存储为字符串。`fid`

如果存储为字符串，理论上需要 8+1+16+8=33 个字节。一个 char（33） 就足够了，如果不是绰绰有余的话，因为大多数使用不需要 2^32 卷。

如果空间确实是一个问题，则可以以自己的格式存储文件 ID。卷 ID 需要一个 4 字节的整数，文件密钥需要一个 8 字节长的数字，文件 Cookie 需要一个 4 字节的整数。因此，16个字节绰绰有余。

### 读取文件

下面是如何呈现 URL 的示例。

首先按文件的 volumeId 查找卷服务器的 URL：

```
> curl http://localhost:9333/dir/lookup?volumeId=3
{"volumeId":"3","locations":[{"publicUrl":"localhost:8080","url":"localhost:8080"}]}
```

由于（通常）没有太多的卷服务器，并且卷不经常移动，因此大多数时间都可以缓存结果。根据复制类型，一个卷可以有多个副本位置。只需随机选择一个位置进行阅读即可。

现在，您可以获取公共 URL、呈现 URL 或通过 url 直接从卷服务器读取：

```
 http://localhost:8080/3,01637037d6.jpg
```

请注意，我们在这里添加了文件扩展名".jpg"。它是可选的，只是客户端指定文件内容类型的一种方式。

如果您想要更好的 URL，可以使用以下替代 URL 格式之一：

```
 http://localhost:8080/3/01637037d6/my_preferred_name.jpg
 http://localhost:8080/3/01637037d6.jpg
 http://localhost:8080/3,01637037d6.jpg
 http://localhost:8080/3/01637037d6
 http://localhost:8080/3,01637037d6
```

如果要获取图像的缩放版本，可以添加一些参数：

```
http://localhost:8080/3/01637037d6.jpg?height=200&width=200
http://localhost:8080/3/01637037d6.jpg?height=200&width=200&mode=fit
http://localhost:8080/3/01637037d6.jpg?height=200&width=200&mode=fill
```

### 机架感知和数据中心感知复制

SeaweedFS 在卷级别应用复制策略。因此，当您获取文件 ID 时，可以指定复制策略。例如：

```
curl http://localhost:9333/dir/assign?replication=001
```

复制参数选项包括：

```
000：无复制
001：在同一个机架上复制一次
010：在不同的机架上复制一次，但在同一个数据中心
100：在不同的数据中心复制一次
200：在两个不同的数据中心复制两次
110：在不同的机架上复制一次，在不同的数据中心复制一次
```

有关复制的更多详细信息，请访问wiki。

您还可以在启动主服务器时设置默认复制策略。

### 在文件保存在特定的数据中心

可以使用特定的数据中心名称启动卷服务器：

```
 weed volume -dir=/tmp/1 -port=8080 -dataCenter=dc1
 weed volume -dir=/tmp/2 -port=8081 -dataCenter=dc2
```

请求文件密钥时，可选的"dataCenter"参数可以将分配的卷限制为特定数据中心。例如，这指定分配的卷应限制为"dc1"：

```
 http://localhost:9333/dir/assign?dataCenter=dc1
```

### 其他特点

* [无单点故障][feat-1]
* [使用您自己的密钥插入][feat-2]
* [对大文件进行分块][feat-3]
* [集合作为简单命名空间][feat-4]

[回到目录](#目录)

## 对象存储体系结构

通常分布式文件系统将每个文件分成块，中央主机保存文件名的映射，块索引到块句柄，以及每个块服务器具有哪些块。

主要缺点是中央 master 不能有效地处理许多小文件，并且由于所有读取请求都需要通过 chunk master，因此对于许多并发用户来说可能无法很好地扩展。

SeaweedFS 不是管理块，而是管理主服务器中的数据卷。每个数据卷大小为 32GB，可容纳大量文件。每个存储节点可以有很多数据卷。所以主节点只需要存储有关卷的元数据，这是一个相当少量的数据，通常是稳定的。

实际文件元数据存储在卷服务器上的每个卷中。由于每个卷服务器只管理自己磁盘上文件的元数据，每个文件只有 16 个字节，因此所有文件访问都可以仅从内存中读取文件元数据，只需要一次磁盘操作即可真正读取文件数据。

作为比较，考虑 Linux 中的 xfs inode 结构为 536 字节。

### 主服务器和卷服务

架构相当简单。 实际数据存储在存储节点上的卷中。 一台卷服务器可以有多个卷，并且可以通过基本身份验证同时支持读写访问。

所有卷都由主服务器管理。 主服务器包含卷 ID 到卷服务器的映射。 这是相当静态的信息，可以轻松缓存。

在每次写入请求时，主服务器还会生成一个文件密钥，它是一个不断增长的 64 位无符号整数。 由于写请求通常不像读请求那样频繁，因此一台主服务器应该能够很好地处理并发。

### 写入和读取文件

当客户端发送写入请求时，主服务器返回文件的（卷 id、文件密钥、文件 cookie、卷节点 url）。 然后客户端联系卷节点并发布文件内容。

当客户端需要根据（卷 id、文件密钥、文件 cookie）读取文件时，它会通过卷 id 向主服务器询问（卷节点 url，卷节点公共 url），或者从缓存中检索。 然后客户端可以获取内容，或者只是在网页上呈现 URL 并让浏览器获取内容。

有关写入-读取过程的详细信息，请参阅示例。

### 存储大小

在当前的实现中，每个卷可以容纳 32 G字节（32GiB 或 8x2^32 字节）。这是因为我们将内容对齐为 8 个字节。我们可以通过更改2行代码轻松地将其增加到64GiB或128GiB或更多，但代价是由于对齐而浪费了一些填充空间。

可以有 4 GB（4GiB 或 2^32 字节）的卷。因此，总系统大小为 8 x 4GiB x 4GiB，即 128 EB（128EiB 或 2^67 字节）。

每个单独的文件大小都受卷大小限制。

### 节省内存

存储在卷服务器上的所有文件元信息都可以从内存中读取，而无需磁盘访问。每个文件只需要一个 16 字节的映射条目，其中包含 <64 位密钥、32 位偏移量、32 位大小>。当然，每个地图条目都有自己的地图空间成本。但通常磁盘空间在内存耗尽之前耗尽。

### 分层存储到云

本地卷服务器速度更快，而云存储具有弹性容量，如果不经常访问（通常免费上传，但访问成本相对较高），实际上更具成本效益。 凭借仅附加结构和 O(1) 访问时间，SeaweedFS 可以通过将暖数据卸载到云来利用本地和云存储。

通常热数据是新鲜的，而暖数据是旧的。 SeaweedFS 将新创建的卷放在本地服务器上，并可选择将旧卷上传到云端。 如果旧数据的访问频率较低，这实际上为您提供了有限本地服务器的无限容量，并且对于新数据仍然快速。

使用 O(1) 访问时间，网络延迟成本保持在最低限度。

如果热/暖数据按照20/80拆分，20台服务器可以达到100台服务器的存储容量。 这节省了 80% 的成本！ 或者，您也可以重新利用 80 台服务器来存储新数据，并获得 5 倍的存储吞吐量。

[回到目录](#目录)

## 与其他文件系统相比

大多数其他分布式文件系统比较复杂，这个是没有必要的。

SeaweedFS 在设置和操作方面都意味着快速和简单。 如果你到达这里时不明白它是如何工作的，我们就失败了！ 如有任何问题，请提出问题或更新此文件并进行说明。

SeaweedFS 不断向前发展。 与其他系统相同。 这些比较很快就会过时。 请帮助他们保持更新。

[回到目录](#目录)

### 与 HDFS 相

HDFS 对每个文件使用块方法，非常适合存储大文件。

SeaweedFS 非常适合快速并发地提供相对较小的文件。

SeaweedFS 还可以通过将超大文件拆分为可管理的数据块来存储超大文件，并将数据块的文件 id 存储到元块中。 这是由“weed上传/下载”工具管理的，主服务器或卷服务器对此是不可知的。

[Back to TOC](#table-of-contents)

### 与 GlusterFS ，Ceph相比

体系结构大多相同。SeaweedFS旨在通过简单而扁平的架构快速存储和读取文件。主要区别在于

* SeaweedFS 针对小文件进行优化，确保 O（1） 磁盘寻道操作，还可以处理大文件。
* SeaweedFS 静态分配文件的卷 ID。查找文件内容只是对卷 ID 的查找，可以轻松缓存。
* SeaweedFS Filer元数据存储可以是任何知名且经过验证的数据存储，例如Redis，Cassandra，HBase，Mongodb，Elastic Search，MySql，Postgres，Sqlite，MemSql，TiDB，CockroachDB，Etcd等，并且易于定制。
* SeaweedFS Volume服务器还通过HTTP直接与客户端通信，支持范围查询，直接上传等。

| 系统            | 文件元数据               | 读取文件内容   | POSIX     | REST API | 针对大量小文件进行了优化 |
| --------------- | ------------------------ | -------------- | --------- | -------- | ------------------------ |
| SeaweedFS       | 查找卷 id，可缓存        | O(1) disk seek |           | yes      | Yes                      |
| SeaweedFS Filer | 线性可扩展，可定制       | O(1) disk seek | FUSE      | Yes      | Yes                      |
| GlusterFS       | 哈希                     |                | FUSE, NFS |          |                          |
| Ceph            | 哈希+规则                |                | FUSE      | Yes      |                          |
| MooseFS         | 在内存中                 |                | FUSE      |          | No                       |
| MinIO           | 每个文件都有单独的元文件 |                |           | Yes      | No                       |

[回到目录](#目录)

### 与GlustFS相比

GlusterFS 将文件（包括目录和内容）存储在称为“砖”的可配置卷中。

GlusterFS 将路径和文件名散列成 id，并分配给虚拟卷，然后映射到“砖块”。

[回到目录](#目录)

### 与MooseFS相

MooseFS 选择忽略小文件问题。 从 moosefs 3.0 手册中，“即使是一个小文件也会占用 64KiB 加上另外 4KiB 的校验和和 1KiB 的标题”，因为它“最初是为保存大量（如数千个）非常大的文件而设计的”

MooseFS 主服务器将所有元数据保存在内存中。 与 HDFS 名称节点相同的问题。

[回到目录](#目录)

### 与 Ceph 相

Ceph 可以像 SeaweedFS 一样设置为 key->blob 存储。它要复杂得多，需要在其上支持层。这是更详细的比较

SeaweedFS 有一个集中的主组来查找空闲卷，而 Ceph 使用哈希和元数据服务器来定位其对象。拥有一个集中的 master 使得编码和管理变得容易。

与 SeaweedFS 一样，Ceph 也是基于对象存储 RADOS。 Ceph 相当复杂，评论不一。

Ceph 使用 CRUSH 哈希来自动管理数据放置，这样可以高效地定位数据。但是数据必须按照 CRUSH 算法放置。任何错误的配置都会导致数据丢失。拓扑变化，比如增加新的服务器来增加容量，会导致高IO成本的数据迁移来适应CRUSH算法。 SeaweedFS 通过将数据分配给任何可写卷来放置数据。如果写入一个卷失败，只需选择另一个卷进行写入。添加更多卷也尽可能简单。

SeaweedFS 针对小文件进行了优化。小文件存储为一个连续的内容块，文件之间最多有 8 个未使用的字节。小文件访问是 O(1) 磁盘读取。

SeaweedFS Filer 使用现成的存储，如 MySql、Postgres、Sqlite、Mongodb、Redis、Elastic Search、Cassandra、HBase、MemSql、TiDB、CockroachCB、Etcd 来管理文件目录。这些存储经过验证、可扩展且更易于管理。

| SeaweedFS | Ceph    | 优点                                |
| --------- | ------- | ----------------------------------- |
| Master    | MDS     | 相似                                |
| Volume    | OSD     | 针对小文件进行了优化                |
| Filer     | Ceph FS | 线性可扩展、可定制、O(1) 或 O(logN) |

[回到目录](#目录)

### 与 MinIO 相比

MinIO 紧跟 AWS S3，是测试 S3 API 的理想选择。 它具有良好的 UI、策略、版本控制等。SeaweedFS 正试图在这方面迎头赶上。 以后也可以把MinIO作为网关放在SeaweedFS前面。

MinIO 元数据位于简单文件中。 每个文件写入都会导致对相应元文件的额外写入。

MinIO 没有针对大量小文件进行优化。 这些文件只是按原样存储到本地磁盘。 加上用于擦除编码的额外元文件和分片，它只会放大 LOSF 问题。

MinIO 有多个磁盘 IO 来读取一个文件。 SeaweedFS 具有 O(1) 磁盘读取，即使对于纠删码文件也是如此。

MinIO 具有全时擦除编码。 SeaweedFS 对热数据使用复制以提高速度，并可选择对热数据应用纠删码。

MinIO 没有类似 POSIX 的 API 支持。

MinIO 对存储布局有特定的要求。 容量调整不灵活。 在 SeaweedFS 中，只需启动一个指向主服务器的卷服务器。 就这样。

## 开发者计划

* 有关如何管理和扩展系统的更多工具和文档。
* 读取和写入流数据。
* 支持结构化数据

这是一个超级令人兴奋的项目！我们需要帮助和[赞助](https://www.patreon.com/seaweedfs)!

[回到目录](#目录)

## 安装指南

> 针对不熟悉 golang 的用户的安装指南

第 1 步：在您的机器上安装 go 并按照以下说明设置环境：

https://golang.org/doc/install

确保您设置了$GOPATH

第 2 步：签出（下载）此仓库：

```bash
git clone https://github.com/chrislusf/seaweedfs.git
```

步骤 3：通过执行以下命令下载、编译和安装项目

```bash
cd seaweedfs/weed && make install
```

完成此操作后，您将在  `$GOPATH/bin` 目录中找到可执行文件"weed".

[回到目录](#目录)

## 磁盘相关的话题

### 硬盘性能

在SeaweedFS上测试读取性能时，基本上就变成了对硬盘随机读取速度的性能测试。 硬盘驱动器通常获得 100MB/s~200MB/s。

[返回目录](#目录)

### 固态硬盘-SSD

要修改或删除小文件，SSD 必须一次删除整个块，并将现有块中的内容移动到新块。 SSD 全新时速度很快，但随着时间的推移会变得碎片化，您必须进行垃圾收集，压缩块。 SeaweedFS 对 SSD 很友好，因为它是仅附加的。 删除和压缩是在后台的卷级别上完成的，不会减慢读取速度，也不会造成碎片。

[返回目录](#目录)

## 性能测试

我自己的不太严谨的实验环境：Mac Book with Solid State Disk, CPU: 1 Intel Core i7 2.6GHz

写入 100 万个 1KB 文件：

```
Concurrency Level:      16
Time taken for tests:   66.753 seconds
Complete requests:      1048576
Failed requests:        0
Total transferred:      1106789009 bytes
Requests per second:    15708.23 [#/sec]
Transfer rate:          16191.69 [Kbytes/sec]

Connection Times (ms)
              min      avg        max      std
Total:        0.3      1.0       84.3      0.9

Percentage of the requests served within a certain time (ms)
   50%      0.8 ms
   66%      1.0 ms
   75%      1.1 ms
   80%      1.2 ms
   90%      1.4 ms
   95%      1.7 ms
   98%      2.1 ms
   99%      2.6 ms
  100%     84.3 ms
```

随机读取100万个文件：

```
Concurrency Level:      16
Time taken for tests:   22.301 seconds
Complete requests:      1048576
Failed requests:        0
Total transferred:      1106812873 bytes
Requests per second:    47019.38 [#/sec]
Transfer rate:          48467.57 [Kbytes/sec]

Connection Times (ms)
              min      avg        max      std
Total:        0.0      0.3       54.1      0.2

Percentage of the requests served within a certain time (ms)
   50%      0.3 ms
   90%      0.4 ms
   98%      0.6 ms
   99%      0.7 ms
  100%     54.1 ms
```

[返回目录](#目录)

## 许可证

根据 Apache 许可证 2.0 版（“许可证”）获得许可； 除非遵守许可，否则您不得使此软件。 您可以在以下网址获取许可证的副本

    http://www.apache.org/licenses/LICENSE-2.0

除非适用法律要求或书面同意，否则根据许可分发的软件将按“原样”分发，没有任何明示或暗示的保证或条件。 有关许可下的特定语言管理权限和限制，请参阅许可。

此页面的文本可根据知识共享署名-相同方式共享 3.0 未移植许可和 GNU 自由文档许可（无版本，没有不变的部分、封面文本或封底文本）的条款进行修改和重复使用。

[返回目录](#目录)

## Github Star数量

[![Stargazers over time](https://starchart.cc/chrislusf/seaweedfs.svg)](https://starchart.cc/chrislusf/seaweedfs)

[Filer]: https://github.com/chrislusf/seaweedfs/wiki/Directories-and-Files
[SuperLargeFiles]: https://github.com/chrislusf/seaweedfs/wiki/Data-Structure-for-Large-Files
[Mount]: https://github.com/chrislusf/seaweedfs/wiki/FUSE-Mount
[AmazonS3API]: https://github.com/chrislusf/seaweedfs/wiki/Amazon-S3-API
[BackupToCloud]: https://github.com/chrislusf/seaweedfs/wiki/Async-Replication-to-Cloud
[Hadoop]: https://github.com/chrislusf/seaweedfs/wiki/Hadoop-Compatible-File-System
[WebDAV]: https://github.com/chrislusf/seaweedfs/wiki/WebDAV
[ErasureCoding]: https://github.com/chrislusf/seaweedfs/wiki/Erasure-coding-for-warm-storage
[TieredStorage]: https://github.com/chrislusf/seaweedfs/wiki/Tiered-Storage
[CloudTier]: https://github.com/chrislusf/seaweedfs/wiki/Cloud-Tier
[FilerDataEncryption]: https://github.com/chrislusf/seaweedfs/wiki/Filer-Data-Encryption
[FilerTTL]: https://github.com/chrislusf/seaweedfs/wiki/Filer-Stores
[VolumeServerTTL]: https://github.com/chrislusf/seaweedfs/wiki/Store-file-with-a-Time-To-Live
[SeaweedFsCsiDriver]: https://github.com/seaweedfs/seaweedfs-csi-driver
[ActiveActiveAsyncReplication]: https://github.com/chrislusf/seaweedfs/wiki/Filer-Active-Active-cross-cluster-continuous-synchronization
[FilerStoreReplication]: https://github.com/chrislusf/seaweedfs/wiki/Filer-Store-Replication
[KeyLargeValueStore]: https://github.com/chrislusf/seaweedfs/wiki/Filer-as-a-Key-Large-Value-Store
[CloudDrive]: https://github.com/chrislusf/seaweedfs/wiki/Cloud-Drive-Architecture
[GatewayToRemoteObjectStore]: https://github.com/chrislusf/seaweedfs/wiki/Gateway-to-Remote-Object-Storage
[Replication]: https://github.com/chrislusf/seaweedfs/wiki/Replication
[feat-1]: https://github.com/chrislusf/seaweedfs/wiki/Failover-Master-Server
[feat-2]: https://github.com/chrislusf/seaweedfs/wiki/Optimization#insert-with-your-own-keys
[feat-3]: https://github.com/chrislusf/seaweedfs/wiki/Optimization#upload-large-files
[feat-4]: https://github.com/chrislusf/seaweedfs/wiki/Optimization#collection-as-a-simple-name-space
