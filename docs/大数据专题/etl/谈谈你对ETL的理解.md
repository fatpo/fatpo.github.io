<!-- TOC -->

* [1. 谈到ETL](#1-谈到etl)
* [2. ETL的挑战](#2-etl的挑战)
    * [2.1. 异常处理和故障恢复机制](#21-异常处理和故障恢复机制)
    * [2.2. 自动化重试与幂等性设计](#22-自动化重试与幂等性设计)
    * [2.3. 数据质量保障](#23-数据质量保障)
    * [2.4. 事务管理与回滚](#24-事务管理与回滚)
        * [2.4.1. clickhouse怎么保证事务](#241-clickhouse怎么保证事务)
            * [2.4.1.1. 使用适当的表引擎](#2411-使用适当的表引擎)
            * [2.4.1.2. 数据去重](#2412-数据去重)
            * [2.4.1.3. 分区处理](#2413-分区处理)
        * [2.4.2. impala 怎么保证事务](#242-impala-怎么保证事务)
            * [2.4.2.1. 幂等性操作](#2421-幂等性操作)
            * [2.4.2.2. 分区处理](#2422-分区处理)
            * [2.4.2.3. 临时表](#2423-临时表)
            * [2.4.2.4. 事务性操作模拟](#2424-事务性操作模拟)
    * [2.5. 容错性测试和优化](#25-容错性测试和优化)
    * [2.6. 实时监控和报警](#26-实时监控和报警)

<!-- TOC -->

## 1. 谈到ETL

你马上得想到这张图：
![etc.png](img/etl.png)

本质就是：extract -> transform -> load ，抽取数据，清洗数据，转换数据，保存数据。

## 2. ETL的挑战

### 2.1. 异常处理和故障恢复机制

一般跑一次task要N个小时，中间发生了什么事儿，你永远无法预测，比如timeout了，比如服务器断电了，比如某个数据突然不按照格式组织了，index
out of range了。

各种异常，要做好异常处理，python中要把`try-catch`玩的飞起，尽可能把异常情况和场景考虑周全。如果可以的话，能够恢复就自动恢复。

比如：timeout了，捕获异常，再增加3次retry，让task继续顺序往下走。

### 2.2. 自动化重试与幂等性设计

* 自动化重试是处理临时错误的有效方法，例如暂时的网络波动或系统负载过高。通过在失败时自动重试作业，可以在问题解决后自动恢复作业的执行。
* 此外，在设计ETL过程时，考虑采用幂等性设计，即使在多次执行时，数据结果也是一致的。这样即使重试发生，不会对数据产生重复或冲突。

这里我说说自己的想法：

* 毕竟一次task要N个小时，你还是得有自动化重试，尽可能让task顺序往下走。
    * 幂等性非常关键：
        * 要有唯一ID + CONFLICT + UPDATE SET，保证落库数据只会插入一条
            * ```text
              INSERT INTO target_table (id, data)
              SELECT id, data
              FROM source_table
              ON CONFLICT (id) DO UPDATE SET data = EXCLUDED.data;
                ```
        * 如果没有唯一ID，那么在处理数据的时候，可以自动开发一些unique_func()，比如把query_经纬度当成一个唯一标识符。

### 2.3. 数据质量保障

这个我没啥想法，因为都是按照产品PM要求给的数据，这些数据质量高不高。。我说了不算。

### 2.4. 事务管理与回滚

* 为确保数据处理的一致性，对于需要跨多个步骤或多个数据源的复杂作业，建议引入事务管理机制。
* 通过将相关操作放入事务中，并且遵循ACID特性，可以确保数据在处理过程中的`原子性`、`一致性`、`隔离性`和`持久性`。
* 当出现错误或失败时，事务会自动回滚，将所有已完成的操作撤销，保持数据的一致性，避免数据损坏和不一致。

#### 2.4.1. clickhouse怎么保证事务

谈到事务，要保证数据干净，clickhouse也是提供了一些保证：

##### 2.4.1.1. 使用适当的表引擎

选择适合事务需求的表引擎，例如 MergeTree 引擎系列（如 MergeTree、ReplacingMergeTree、SummingMergeTree 等）支持数据的原子性和一致性。

##### 2.4.1.2. 数据去重

使用 ReplacingMergeTree 表引擎，通过唯一标识符去重，实现幂等性。此方法可用于消除重复记录，确保数据的一致性。

```sql
CREATE TABLE my_table
(
    id      UInt64,
    value   String,
    version UInt64
) ENGINE = ReplacingMergeTree(version)
ORDER BY id;
```

##### 2.4.1.3. 分区处理

对数据进行分区处理，保证每次ETL作业处理的数据分区是独立的，避免跨分区操作，减少事务冲突。

```sql
CREATE TABLE my_table
(
    event_date Date,
    id         UInt64,
    value      String
) ENGINE = MergeTree()
PARTITION BY toYYYYMM(event_date)
ORDER BY id;
```

#### 2.4.2. impala 怎么保证事务

据我所知，impala 不支持事务，但我们不是毫无办法。

##### 2.4.2.1. 幂等性操作

设计幂等性的 ETL 作业，确保同一数据处理任务无论执行多少次，结果都是一致的。使用唯一键或主键来防止重复插入。

```sql
INSERT
OVERWRITE TABLE target_table
SELECT DISTINCT *
FROM source_table;
```

##### 2.4.2.2. 分区处理

利用分区表进行增量数据处理，确保每个分区的数据独立且完整，避免跨分区操作。

```sql
INSERT
OVERWRITE TABLE target_table PARTITION (year, month)
SELECT *
FROM source_table
WHERE year = 2023 AND month = 07;
```

##### 2.4.2.3. 临时表

在执行 ETL 作业时，先将数据加载到临时表中，数据验证通过后再将其插入到目标表，确保数据一致性。

```sql
CREATE TABLE temp_table AS
SELECT *
FROM source_table;

-- 数据验证和清洗

INSERT INTO target_table
SELECT *
FROM temp_table;

DROP TABLE temp_table;
```

##### 2.4.2.4. 事务性操作模拟

虽然 Impala 不支持 ACID 事务，但可以通过脚本或程序模拟事务操作。例如，使用外部脚本控制数据插入、更新和删除，确保整个过程的原子性。

```bash
# Bash 脚本示例
impala-shell -q "CREATE TABLE temp_table AS SELECT * FROM source_table;"

# 数据验证和清洗

impala-shell -q "INSERT INTO target_table SELECT * FROM temp_table;"
impala-shell -q "DROP TABLE temp_table;"
```

### 2.5. 容错性测试和优化

* 为了进一步提高ETL过程的容错性，推荐定期进行容错性测试。
* 通过模拟异常情况和故障，测试ETL过程的反应和恢复能力，发现并解决潜在的问题。
* 在测试中，可以模拟网络中断、源数据丢失、转换错误等情况。在发现问题后，及时优化代码和流程，增强系统的容错性和稳定性。

这里说说自己的想法：

* 一般整个ETL服务都有专业的运维老师在负责，我们也就不要在上面乱测试了，给别人添堵。

### 2.6. 实时监控和报警

* ETL过程通常会长时间运行，因此建议实施实时监控和报警系统。
* 监控可以检测作业的运行状态、性能指标以及数据流向，帮助快速发现异常和瓶颈。
* 当作业失败或出现异常时，报警系统能够及时通知相关人员，使其能够迅速响应并采取必要的措施。

这里说说自己的想法：

* 飞书的告警链路，发生问题了，能通知到人，这非常关键，正反馈，不然你要等9小时后task跑完看日志？还是过程中一直盯着日志？不现实的。
