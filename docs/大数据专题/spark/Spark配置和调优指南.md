<!-- TOC -->

* [1. Spark是什么](#1-spark是什么)
* [2. 基础配置](#2-基础配置)
* [3. 资源分配](#3-资源分配)
* [4. 数据序列化](#4-数据序列化)
* [5. Shuffle 优化](#5-shuffle-优化)
    * [5.1. 调整并行度](#51-调整并行度)
    * [5.2. 合并小文件](#52-合并小文件)
* [6. 缓存与持久化](#6-缓存与持久化)
    * [6.1. 缓存](#61-缓存)
    * [6.2. 持久化](#62-持久化)
* [7. 广播变量](#7-广播变量)
* [8. 内存管理](#8-内存管理)
    * [8.1. 调整 Spark 内存：](#81-调整-spark-内存)
    * [8.2. GC 调优：](#82-gc-调优)
* [9. 调优技巧](#9-调优技巧)
    * [9.1. 减少数据倾斜](#91-减少数据倾斜)
    * [9.2. 避免重复计算](#92-避免重复计算)
    * [9.3. 监控和调试](#93-监控和调试)

<!-- TOC -->

## 1. Spark是什么

Apache Spark 是一个强大的开源分布式计算系统。

通过合理的配置和优化，可以显著提升 Spark 应用的性能。

## 2. 基础配置

```text
spark.master          # 设置集群的 Master URL，单机模式下可以设置为 'local' 或 'local[N]'。
spark.app.name        # Spark 应用名称。
spark.executor.memory # 每个 Executor 分配的内存大小，例如 '2g'。
spark.executor.cores  # 每个 Executor 分配的 CPU 核数。
```

举个栗子：

```text
spark.master=local[4]
spark.app.name=MySparkApp
spark.executor.memory=4g
spark.executor.cores=2
```

## 3. 资源分配

合理分配集群资源是优化 Spark 应用性能的重要步骤。考虑以下几点：

* 集群模式（Cluster Mode）：根据任务量选择适合的集群模式（Standalone、YARN、Mesos、Kubernetes）。
* Executor 个数和内存：根据集群硬件配置和任务需求分配 Executor 数量和内存。
* Cores 数量：确保每个 Executor 分配合理的 CPU 核数，以充分利用并行计算能力。

## 4. 数据序列化

Spark 支持两种主要的序列化方式：`Java 序列化（默认）`和 `Kryo 序列化`。Kryo 序列化更快且更紧凑，推荐在性能要求高的场景中使用。

```text
spark.serializer=org.apache.spark.serializer.KryoSerializer
```

## 5. Shuffle 优化

Shuffle 是 Spark 应用性能优化的关键之一。

### 5.1. 调整并行度

通过调整 `spark.sql.shuffle.partitions` 参数控制 Shuffle 操作的并行度，默认值是 `200`。

```properties
spark.sql.shuffle.partitions=400
```

### 5.2. 合并小文件

在进行 Shuffle 时，避免产生过多的小文件，可以通过调整 `spark.sql.files.maxPartitionBytes`
和 `spark.sql.files.openCostInBytes`
参数。

```properties
spark.sql.files.maxPartitionBytes=128MB
spark.sql.files.openCostInBytes=4MB
```

## 6. 缓存与持久化

当需要多次使用相同的数据集时，可以将数据集缓存或持久化到内存中，以减少重复计算。

### 6.1. 缓存

将数据集缓存到内存中，适用于快速重复访问的数据。

```scala
val df = spark.read.csv("data.csv")
df.cache()
```

### 6.2. 持久化

可以选择将数据集持久化到磁盘或内存中。

```scala
val df = spark.read.csv("data.csv")
df.persist(StorageLevel.MEMORY_AND_DISK)
```

## 7. 广播变量

在 Spark 中，如果需要在多个任务中使用相同的只读数据集，可以使用广播变量，将数据广播到每个 Executor 节点，减少网络传输开销。

```scala
val broadcastVar = sc.broadcast(Array(1, 2, 3))
```

## 8. 内存管理

合理的内存管理可以显著提高 Spark 应用的性能。以下是一些内存管理的建议：

### 8.1. 调整 Spark 内存：

根据应用需求调整 Spark 内存参数，如 spark.memory.fraction 和 spark.memory.storageFraction。

```properties
spark.memory.fraction=0.6
spark.memory.storageFraction=0.5
```

### 8.2. GC 调优：

对于内存密集型应用，可以调整 JVM 的垃圾回收参数，如 `spark.executor.extraJavaOptions`。

```properties
spark.executor.extraJavaOptions=-XX:+UseG1GC -XX:InitiatingHeapOccupancyPercent=35
```

## 9. 调优技巧

### 9.1. 减少数据倾斜

数据倾斜会导致某些节点上的任务处理时间过长，可以通过调整分区策略、增加并行度等方式减少数据倾斜。

### 9.2. 避免重复计算

尽量避免在应用中进行重复计算，可以通过缓存或持久化等方式缓存中间结果。

### 9.3. 监控和调试

使用 Spark 提供的监控工具，如 Spark UI、日志等，实时监控应用的执行情况，及时发现和解决性能瓶颈。
