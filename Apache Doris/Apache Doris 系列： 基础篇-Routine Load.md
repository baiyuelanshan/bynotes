简介
----
Routine Load 支持用户提交一个常驻的导入任务，通过不断的从指定的数据源读取数据，将数据导入到 Doris 中。
目前仅支持通过无认证或者 SSL 认证方式，从 Kakfa 导入 CSV 或 Json 格式的数据。

接下来通过一个案例介绍 Routine Load 的使用。
案例内容：
1. 部署单节点Kafka
2. 准备测试数据并导入kafka
3. 导入数据到 Doris

部署单节点Kafka
----

#### 下载
https://archive.apache.org/dist/kafka/2.4.1/kafka_2.11-2.4.1.tgz

#### 部署

1) 解压
```
tar xvf  kafka_2.11-2.4.1.tgz
# 创建软连接
ln -s kafka_2.11-2.4.1 kafka
cd kafka
```

2）启动zookeeper和kafka
```
# 启动 zookeeper
bin/zookeeper-server-start.sh -daemon config/zookeeper.properties  

# 启动 kafka server
bin/kafka-server-start.sh -daemon config/server.properties
```

3) 查看日志
```
tail -f logs/server.log
```
出现一下日志说明Kafka server已经启动成功了。

>
`[2022-09-13 06:38:03,596] INFO [KafkaServer id=0] started (kafka.server.KafkaServer)`




准备测试数据并导入kafka
----
```
{"order_date":"2022-09-07","order_id":"10001","buy_num":"2","user_id":"30001","create_time":"2022-09-07 09:00:00","update_time":"2022-09-07 09:00:00"}
{"order_date":"2022-09-07","order_id":"10002","buy_num":"2","user_id":"30001","create_time":"2022-09-07 09:00:01","update_time":"2022-09-07 09:00:01"}
{"order_date":"2022-09-07","order_id":"10003","buy_num":"2","user_id":"30001","create_time":"2022-09-07 09:00:02","update_time":"2022-09-07 09:00:02"}
{"order_date":"2022-09-07","order_id":"10004","buy_num":"2","user_id":"30001","create_time":"2022-09-07 09:00:03","update_time":"2022-09-07 09:00:03"}
{"order_date":"2022-09-07","order_id":"10005","buy_num":"2","user_id":"30001","create_time":"2022-09-07 09:00:04","update_time":"2022-09-07 09:00:04"}
{"order_date":"2022-09-07","order_id":"10006","buy_num":"2","user_id":"30001","create_time":"2022-09-07 09:00:05","update_time":"2022-09-07 09:00:05"}
```

#### 创建kafka topic 

创建名叫 order_info 的kafka topic, 3个分区，1个副本
```
 bin/kafka-topics.sh --create --topic order_info --bootstrap-server localhost:9092 --partitions 3 --replication-factor 1
```

#### 导入数据到 Kafka topic
```
 bin/kafka-console-producer.sh --topic order_info --broker-list localhost:9092 < /tmp/data.txt 
```

#### 查看导入的数据
```
bin/kafka-console-consumer.sh --topic order_info --bootstrap-server localhost:9092 --from-beginning

{"order_date":"2022-09-07","order_id":"10001","buy_num":"2","user_id":"30001","create_time":"2022-09-07 09:00:00","update_time":"2022-09-07 09:00:00"}
{"order_date":"2022-09-07","order_id":"10002","buy_num":"2","user_id":"30001","create_time":"2022-09-07 09:00:01","update_time":"2022-09-07 09:00:01"}
{"order_date":"2022-09-07","order_id":"10003","buy_num":"2","user_id":"30001","create_time":"2022-09-07 09:00:02","update_time":"2022-09-07 09:00:02"}
{"order_date":"2022-09-07","order_id":"10004","buy_num":"2","user_id":"30001","create_time":"2022-09-07 09:00:03","update_time":"2022-09-07 09:00:03"}
{"order_date":"2022-09-07","order_id":"10005","buy_num":"2","user_id":"30001","create_time":"2022-09-07 09:00:04","update_time":"2022-09-07 09:00:04"}
{"order_date":"2022-09-07","order_id":"10006","buy_num":"2","user_id":"30001","create_time":"2022-09-07 09:00:05","update_time":"2022-09-07 09:00:05"}
```


导入数据到 Doris 
----

`登录mysql客户端`

#### 创建 Doris 数据表

```
CREATE TABLE IF NOT EXISTS test.order_info_example
(  
    `order_date` DATE NOT NULL COMMENT "下单日期",  
    `order_id` INT NOT NULL COMMENT "订单id",  
    `buy_num` TINYINT COMMENT "购买件数",  
    `user_id` INT COMMENT "[-9223372036854775808, 9223372036854775807]",  
    `create_time` DATETIME COMMENT "创建时间",  
    `update_time` DATETIME COMMENT "更新时间"
)  
ENGINE=olap  
DUPLICATE KEY(`order_date`, `order_id`)  
DISTRIBUTED BY HASH(`order_id`) BUCKETS 32  
PROPERTIES (  
 "replication_num" = "1"); 
```

#### 创建 Doris ROUTINE LOAD

语法：
```
CREATE ROUTINE LOAD [db.]job_name ON tbl_name  
[merge_type]  
[load_properties]  
[job_properties]  
FROM data_source [data_source_properties]  

```


```
CREATE ROUTINE LOAD test.order_info_routine_load_example ON order_info_example  
WITH APPEND
COLUMNS(order_date,order_id,buy_num,user_id,create_time,update_time)  
PROPERTIES  
(  
    "desired_concurrent_number"="3",    "max_batch_interval" = "20",    "max_batch_rows" = "300000",    "max_batch_size" = "209715200",    "strict_mode" = "false",    "format" = "json")  
FROM KAFKA  
(  
    "kafka_broker_list" = "localhost:9092",    "kafka_topic" = "order_info",   "property.group.id" = "order_info_routine_load_example",  "property.kafka_default_offsets" = "OFFSET_BEGINNING");  

```

`merge_type`: 数据合并类型，默认为 WITH APPEND，表示导入的数据都是普通的追加写操作。MERGE 和 DELETE 类型仅适用于 Unique Key 模型表。
`strict_mode`: 是否开启严格模式，默认为关闭。如果开启后，非空原始数据的列类型变换如果结果为 NULL，则会被过滤
`format`: 指定导入数据格式，默认是csv，支持json格式。
更多参数请参考[ROUTINE LOAD](https://doris.apache.org/zh-CN/docs/sql-manual/sql-reference/Data-Manipulation-Statements/Load/CREATE-ROUTINE-LOAD)

可通过`show routine load`查看任务的运行情况


#### 查看数据

mysql> select * from order_info_example;
+------------+----------+---------+---------+---------------------+---------------------+
| order_date | order_id | buy_num | user_id | create_time         | update_time         |
+------------+----------+---------+---------+---------------------+---------------------+
| 2022-09-07 |    10006 |       2 |   30001 | 2022-09-07 09:00:05 | 2022-09-07 09:00:05 |
| 2022-09-07 |    10005 |       2 |   30001 | 2022-09-07 09:00:04 | 2022-09-07 09:00:04 |
| 2022-09-07 |    10004 |       2 |   30001 | 2022-09-07 09:00:03 | 2022-09-07 09:00:03 |
| 2022-09-07 |    10002 |       2 |   30001 | 2022-09-07 09:00:01 | 2022-09-07 09:00:01 |
| 2022-09-07 |    10003 |       2 |   30001 | 2022-09-07 09:00:02 | 2022-09-07 09:00:02 |
| 2022-09-07 |    10001 |       2 |   30001 | 2022-09-07 09:00:00 | 2022-09-07 09:00:00 |
+------------+----------+---------+---------+---------------------+---------------------+
6 rows in set (0.05 sec)

至此，数据已经成功写入到Doris。
