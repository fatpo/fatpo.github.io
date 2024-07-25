<!-- TOC -->
  * [1. hdfs是在什么呀](#1-hdfs是在什么呀)
  * [2. HDFS 基础架构](#2-hdfs-基础架构)
  * [3. 写数据到 HDFS](#3-写数据到-hdfs)
  * [4. 从 HDFS 读取数据](#4-从-hdfs-读取数据)
  * [5. 数据副本和容错机制](#5-数据副本和容错机制)
    * [5.1. HDFS 通过数据副本机制确保数据的高可用性和容错性](#51-hdfs-通过数据副本机制确保数据的高可用性和容错性)
    * [5.2. 副本放置策略](#52-副本放置策略)
    * [5.3. 副本重建](#53-副本重建)
<!-- TOC -->

## 1. hdfs是在什么呀

HDFS（Hadoop Distributed File System）是 Hadoop 生态系统中的核心组件，用于存储和管理大规模数据。

简单来说，是一个文件系统，专为大数据服务的文件系统。

## 2. HDFS 基础架构

在了解 HDFS 的读写流程之前，先了解其基本架构：

* NameNode：管理文件系统的元数据（文件名、权限、位置等），是 HDFS 的核心节点。
* DataNode：存储实际的数据块，负责处理来自客户端的读写请求。
* Secondary NameNode：辅助 NameNode 进行元数据检查点操作，并不用于故障转移。

## 3. 写数据到 HDFS

当客户端将数据写入 HDFS 时，主要步骤如下：

* 客户端请求：客户端向 `NameNode` 请求将文件写入 HDFS。
* NameNode 响应：NameNode 检查文件系统命名空间是否存在该文件，并分配一个新的文件记录。如果文件已存在，则返回错误。
* 分块分配：NameNode 为文件分配数据块和 `DataNode`，默认情况下，HDFS 将文件分成 `128 MB` 的块。
* 创建输出流：客户端获得 DataNode 的地址后，开始创建输出流，将数据写入第一个 DataNode。
* 数据复制：第一个 DataNode 接收到数据块后，将数据块复制到第二个 DataNode，第二个 DataNode 再复制到第三个 DataNode（默认副本数为
  3）。
* 数据确认：每个 DataNode 接收到完整的数据块后，向上一个 DataNode 发送确认信息，最后返回给客户端，表示数据写入成功。

```text
客户端  ->  NameNode (请求写入)  ->  DataNode1 (写入)  ->  DataNode2 (复制)  ->  DataNode3 (复制)
```

* 注意1：一口气要重复写3次！！！
* 注意2：凡事要听NameNode的安排。

## 4. 从 HDFS 读取数据

当客户端从 HDFS 读取数据时，主要步骤如下：

* 客户端请求：客户端向 NameNode 请求读取文件。
* NameNode 响应：NameNode 检查文件系统命名空间，返回数据块所在的 DataNode 列表。
* 选择 DataNode：客户端根据 NameNode 返回的信息选择`最近的` DataNode 进行读取。
* 创建输入流：客户端创建输入流，从选定的 DataNode 开始读取数据块。
* 数据流动：客户端顺序读取每个数据块，直到文件读取完毕。

```markdown
客户端 ->  NameNode (请求读取)  ->  DataNode1 (读取)  ->  DataNode2 (读取)  ->  DataNode3 (读取)
```

注意：凡事都要过问NameNode，听NameNode的安排，它让你去哪里读，你就去哪里读数据。

## 5. 数据副本和容错机制

### 5.1. HDFS 通过数据副本机制确保数据的高可用性和容错性

副本存储：每个数据块默认存储三份副本，分别存储在不同的 DataNode 上。

### 5.2. 副本放置策略

* 第一个副本放置在写入客户端所在的 DataNode（或机架）上。
* 第二个副本放置在不同机架的 DataNode 上。
* 第三个副本放置在与第二个副本相同机架的不同 DataNode 上。

### 5.3. 副本重建

当某个 DataNode 故障时，NameNode 监测到数据块副本减少，会自动触发副本重建，将数据块复制到其他 DataNode 上。

```markdown
DataNode1 (副本1)  ->  DataNode2 (副本2)  ->  DataNode3 (副本3)
```
