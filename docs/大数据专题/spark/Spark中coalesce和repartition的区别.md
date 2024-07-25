<!-- TOC -->

* [1. repartition](#1-repartition)
* [2. coalesce](#2-coalesce)
* [3. 总结](#3-总结)
    * [3.1. repartition](#31-repartition)
    * [3.2. coalesce](#32-coalesce)
    * [3.3. 实际应用](#33-实际应用)

<!-- TOC -->

在 Apache Spark 中，coalesce 和 repartition 都用于调整 RDD 或 DataFrame 的分区数，但它们在实现和用途上有所不同。下面是它们的区别：

## 1. repartition

功能：
repartition 用于重新分区RDD或DataFrame，能够增加或减少分区数。它会重新分布数据到新的分区中。

实现：
repartition 会触发一次全量的洗牌（shuffle），这意味着数据会在集群中重新分配和移动，以确保数据均匀地分布到新的分区中。

用法：

```scala
val repartitionedRDD = rdd.repartition(10)  // 重新分区为10个分区
val repartitionedDF = df.repartition(10)  // DataFrame 重新分区为10个分区
```

特点：

* 增加或减少分区：repartition 可以增加或减少分区数。
* 全量洗牌：由于全量洗牌，repartition 的开销较大，特别是当分区数量变化很大时。
* 数据均匀分布：通过洗牌操作，repartition 能够实现数据的均匀分布，避免了数据倾斜的问题。

## 2. coalesce

功能：
coalesce 用于减少分区数。它将现有的分区合并为更少的分区，而不会进行全量洗牌操作。

实现：
coalesce 通常不会触发全量洗牌，而是尝试将数据从较多的分区移动到较少的分区中。这使得 coalesce 的开销相对较小，但可能会导致数据不均匀分布。

用法：

```scala
val coalescedRDD = rdd.coalesce(5)  // 减少分区数到5个
val coalescedDF = df.coalesce(5)  // DataFrame 减少分区数到5个
```

特点：

* 减少分区：coalesce 主要用于减少分区数，不能增加分区。
* 减少洗牌：通常不会进行全量洗牌，因此比 repartition 更高效，适合于减少分区时使用。
* 数据不均匀分布：由于没有全量洗牌，coalesce 可能会导致数据在减少分区后的分布不均匀。

## 3. 总结

### 3.1. repartition

* 用于增加或减少分区数。
* 触发全量洗牌，确保数据均匀分布。
* 开销较大，适合对分区数有较大变化的情况。

### 3.2. coalesce

* 用于减少分区数。
* 通常不会触发全量洗牌，效率较高。
* 适合在需要减少分区时使用，可能导致数据分布不均。

### 3.3. 实际应用

* 增加分区数：当你需要将数据重新分布到更多的分区中以提高并行度时，使用 repartition。
* 减少分区数：当你需要减少分区数以便于将数据合并或减少后续操作的开销时，使用 coalesce。
