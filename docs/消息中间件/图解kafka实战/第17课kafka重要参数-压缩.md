# 1、压缩的意义
消息在网络传输，流量很贵。

国哥记得当年在`小步网络`的时候，存那些图片什么的，1G好像是3毛钱吧，假设一天是1000G，那就是300元硬开销，记不大清楚了。

如果消息不是超级依赖实时，能等个几秒的吧，那么压缩也所谓。

压缩就是`时间换空间`，我花点时间压一压，空间就省出来了。

# 2、compression.type

默认是none，不压缩。

可以选择：配置为`gzip`，`snappy`和`lz4`。

## 2.1、snappy压缩
这里稍微安利下 snappy：
```text
Snappy（以前称Zippy）是Google基于LZ77的思路用C++语言编写的快速数据压缩与解压程序库，并在2011年开源。

它的目标并非最大压缩率或与其他压缩程序库的兼容性，而是非常高的速度和合理的压缩率。

使用一个运行在64位模式下的酷睿i7处理器的单个核心，压缩速度250 MB/s，解压速度500 MB/s。

压缩率比gzip低20-100%。

Snappy广泛应用在Google的项目，例如BigTable、MapReduce和Google内部RPC系统的压缩数据。

它可在开源项目中使用，例如Cassandra、Couchbase、Hadoop、LevelDB、MongoDB、RocksDB、Lucene、Spark和InfluxDB。

解压缩时会检测压缩流中是否存在错误。Snappy不使用内联汇编并且可移植。
```

## 2.2、lz4压缩
科普下 `lz4`:
```text
lz4 算法提供一个比LZO算法稍差的压缩率——这逊于gzip等算法。

但是，它的压缩速度类似LZO, 比gzip快几倍；而解压速度显著快于LZO。
```


## 2.3、gzip压缩
再看看我们最熟悉的 `gzip`:
```text
Gzip是一种压缩文件格式并且也是一个在类 Unix 上的一种文件解压缩的软件，通常指GNU计划的实现，此处的gzip代表GNU zip。

也经常用来表示gzip这种文件格式。

软件的作者是Jean-loup Gailly和Mark Adler。

在1992年10月31日第一次公开发布，版本号0.1，1993年2月，发布了1.0版本。

OpenBSD中所包含的gzip版本实际上是compress程序，其对gzip文件的支持在OpenBSD 3.4中被添加，此处的g代表免费（gratis。
```

