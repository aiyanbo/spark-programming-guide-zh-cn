# Shark用户迁移指南

## 调度(Scheduling)

为JDBC客户端会话设置[Fair Scheduler](https://spark.apache.org/docs/latest/job-scheduling.html#fair-scheduler-pools)池。用户可以设置`spark.sql.thriftserver.scheduler.pool`
变量。

```shell
SET spark.sql.thriftserver.scheduler.pool=accounting;
```
## Reducer数量

在Shark中，默认的reducer数目是1，可以通过`mapred.reduce.tasks`属性来控制其多少。Spark SQL反对使用这个属性，支持`spark.sql.shuffle.partitions`属性，它的默认值是200。
用户可以自定义这个属性。

```shell
SET spark.sql.shuffle.partitions=10;
SELECT page, count(*) c
FROM logs_last_month_cached
GROUP BY page ORDER BY c DESC LIMIT 10;
```

你也可以在`hive-site.xml`中设置这个属性覆盖默认值。现在,`mapred.reduce.tasks`属性仍然可以被识别，它会自动转换为`spark.sql.shuffle.partition`。

## Caching

表属性`shark.cache`不再存在，名字以`_cached`结尾的表也不再自动缓存。作为替代的方法，我们提供`CACHE TABLE`和`UNCACHE TABLE`语句显示地控制表的缓存。

```shell
CACHE TABLE logs_last_month;
UNCACHE TABLE logs_last_month;
```
注意：`CACHE TABLE tbl`是懒加载的，类似于在RDD上使用`.cache`。这个命令仅仅标记tbl来确保计算时分区被缓存，但实际并不缓存它，直到一个查询用到tbl才缓存。

要强制表缓存，你可以简单地执行`CACHE TABLE`后，立即count表。

```shell
CACHE TABLE logs_last_month;
SELECT COUNT(1) FROM logs_last_month;
```

有几个缓存相关的特征现在还不支持：

- 用户定义分区级别的缓存逐出策略
- RDD重加载
- 内存缓存的写入策略(In-memory cache write through policy)