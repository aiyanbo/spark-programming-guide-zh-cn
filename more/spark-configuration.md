# Spark配置

Spark提供三个位置用来配置系统：

- Spark properties控制大部分的应用程序参数，可以用SparkConf对象或者java系统属性设置
- Environment variables可以通过每个节点的` conf/spark-env.sh`脚本设置每台机器的设置。例如IP地址
- Logging可以通过log4j.properties配置

## Spark属性

