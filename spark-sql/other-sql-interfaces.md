# 其它SQL接口

Spark SQL也支持直接运行SQL查询的接口，不用写任何代码。

## 运行Thrift JDBC/ODBC服务器

这里实现的Thrift JDBC/ODBC服务器与Hive 0.12中的[HiveServer2](https://cwiki.apache.org/confluence/display/Hive/Setting+Up+HiveServer2)相一致。你可以用在Spark
或者Hive 0.12附带的beeline脚本测试JDBC服务器。

在Spark目录中，运行下面的命令启动JDBC/ODBC服务器。

```shell
./sbin/start-thriftserver.sh
```

这个脚本接受任何的`bin/spark-submit`命令行参数，加上一个`--hiveconf`参数用来指明Hive属性。你可以运行`./sbin/start-thriftserver.sh --help`来获得所有可用选项的完整
列表。默认情况下，服务器监听`localhost:10000`。你可以用环境变量覆盖这些变量。

```shell
export HIVE_SERVER2_THRIFT_PORT=<listening-port>
export HIVE_SERVER2_THRIFT_BIND_HOST=<listening-host>
./sbin/start-thriftserver.sh \
  --master <master-uri> \
  ...
```
或者通过系统变量覆盖。

```shell
./sbin/start-thriftserver.sh \
  --hiveconf hive.server2.thrift.port=<listening-port> \
  --hiveconf hive.server2.thrift.bind.host=<listening-host> \
  --master <master-uri>
  ...
```
现在你可以用beeline测试Thrift JDBC/ODBC服务器。

```shell
./bin/beeline
```
连接到Thrift JDBC/ODBC服务器的方式如下：

```shell
beeline> !connect jdbc:hive2://localhost:10000
```

Beeline将会询问你用户名和密码。在非安全的模式，简单地输入你机器的用户名和空密码就行了。对于安全模式，你可以按照[Beeline文档](https://cwiki.apache.org/confluence/display/Hive/HiveServer2+Clients)的说明来执行。

## 运行Spark SQL CLI

Spark SQL CLI是一个便利的工具，它可以在本地运行Hive元存储服务、执行命令行输入的查询。注意，Spark SQL CLI不能与Thrift JDBC服务器通信。

在Spark目录运行下面的命令可以启动Spark SQL CLI。

```shell
./bin/spark-sql
```

