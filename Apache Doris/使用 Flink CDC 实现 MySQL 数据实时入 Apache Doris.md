## 简介

主要内容如下：
1. MySQL 安装和开启binog
2. Flink环境准备
3. Apache Doris 环境准备
4. 启动Flink CDC作业


## 1. MySQL 安装和开启binog

参考文章：[Ubuntu 安装 Mysql server](https://blog.csdn.net/weixin_47298890/article/details/122714055), 这篇文章介绍了MySQL的安装，用户创建，Binlog开启等内容。

MySQL安装后之后创建表，插入测试数据
```sql
CREATE TABLE `test`.`products` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) NOT NULL,
  `description` varchar(512) DEFAULT NULL,
  `address` varchar(50) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=153;

INSERT INTO products
     VALUES (default,"scooter","Small 2-wheel scooter"),
            (default,"car battery","12V car battery"),
            (default,"12-pack drill bits","12-pack of drill bits with sizes ranging from #40 to #3"),
            (default,"hammer","12oz carpenter's hammer"),
            (default,"hammer","14oz carpenter's hammer"),
            (default,"hammer","16oz carpenter's hammer"),
            (default,"rocks","box of assorted rocks"),
            (default,"jacket","water resistent black wind breaker"),
            (default,"spare tire","24 inch spare tire");

```



## 2. Flink 环境准备

```shell
##  下载 flink 1.15.3 的二进制安装包
axel -n 20 https://dlcdn.apache.org/flink/flink-1.15.3/flink-1.15.3-bin-scala_2.12.tgz

## 解压flink 1.15.3 的二进制安装包
tar xvf flink-1.15.3-bin-scala_2.12.tgz

## 下载 flink sql connector mysql cdc jar包
axel -n 20 https://repo1.maven.org/maven2/com/ververica/flink-sql-connector-mysql-cdc/2.3.0/flink-sql-connector-mysql-cdc-2.3.0.jar

## 把flink-sql-connector-mysql-cdc-2.3.0.jar拷贝到flink的lib目录
cp flink-sql-connector-mysql-cdc-2.3.0.jar flink-1.15.3/lib


## 下载 flink doris connector jar包
axel -n 20  https://repo.maven.apache.org/maven2/org/apache/doris/flink-doris-connector-1.15/1.2.1/flink-doris-connector-1.15-1.2.1.jar

## 把flink-doris-connector-1.15-1.2.1.jar拷贝到拷贝到flink的lib目录
cp flink-doris-connector-1.15-1.2.1.jar flink-1.15.3/lib

## 启动单机集群
cd flink-1.15.3
bin/start-cluster.sh

## 查看 jobmanager 和 taskmanager 的进程是否存活
jps -m 
## 正常情况会出现两个进程，如下：
27353 TaskManagerRunner --configDir /home/ubuntu/installer/flink-1.15.3/conf -D taskmanager.memory.network.min=67108864b -D taskmanager.cpu.cores=6.0 -D taskmanager.memory.task.off-heap.size=0b -D taskmanager.memory.jvm-metaspace.size=268435456b -D external-resources=none -D taskmanager.memory.jvm-overhead.min=201326592b -D taskmanager.memory.framework.off-heap.size=134217728b -D taskmanager.memory.network.max=67108864b -D taskmanager.memory.framework.heap.size=134217728b -D taskmanager.memory.managed.size=241591914b -D taskmanager.memory.task.heap.size=26843542b -D taskmanager.numberOfTaskSlots=6 -D taskmanager.memory.jvm-overhead.max=201326592b
27086 StandaloneSessionClusterEntrypoint -D jobmanager.memory.off-heap.size=134217728b -D jobmanager.memory.jvm-overhead.min=201326592b -D jobmanager.memory.jvm-metaspace.size=268435456b -D jobmanager.memory.heap.size=469762048b -D jobmanager.memory.jvm-overhead.max=201326592b --configDir /home/ubuntu/installer/flink-1.15.3/conf --executionMode cluster

```


## 3. Apache Doris 环境准备
参考文章：[Apache Doris 安装](https://blog.csdn.net/weixin_47298890/article/details/126749389), 
注意配置以下参数:
```
## FE节点
enable_http_server_v2 = true

## BE节点
enable_stream_load_record = true
```

Apache Doris环境准备后之后创建表：
```sql
CREATE TABLE IF NOT EXISTS test.products_doris
(  
     id INT,
     name varchar(20),
     description varchar(100)
)  
ENGINE=olap  
Unique Key(`id`)  
DISTRIBUTED BY HASH(`id`) BUCKETS 2
PROPERTIES (  
 "replication_num" = "1"); 
--使用唯一模型，以支持Flink CDC update/delete操作
--只有一个BE节点，因此replication_num设置为1
```

## 4. 启动Flink SQL作业

使用Flink SQL命令行客户端bin/sql-client.sh执行以下SQL:
```sql
--设置checkpoint间隔
SET execution.checkpointing.interval = 3s;

-- source表
CREATE TABLE products_source (
     id INT,
     name STRING,
     description STRING,
     PRIMARY KEY (id) NOT ENFORCED
   ) WITH (
     'connector' = 'mysql-cdc',
     'hostname' = '192.168.56.104',
     'port' = '3306',
     'username' = 'test',
     'password' = 'test',
     'database-name' = 'test',
     'table-name' = 'products'
   );
   
-- sink表
CREATE TABLE products_flink_doris_sink (
    id INT,
    name STRING,
    description STRING
    ) 
    WITH (
      'connector' = 'doris',
      'fenodes' = '192.168.56.104:8030',
      'table.identifier' = 'test.products_doris',
      'username' = 'test',
      'password' = 'password123',
      'sink.label-prefix' = 'doris_label'
);

--执行insert语句
INSERT INTO products_flink_doris_sink select id,name,description from products_source
```

Flink SQL提交完成后，查询Doris客户端可以看到数据过来了
![[Pasted image 20230106165333.png]]