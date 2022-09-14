简介
-----
Stream Load 提供 HTTP API 提交数据导入任务，支持本地数据文件(csv,json)的导入。
特点：
- 同步导入
- 保证数据原子性，导入一批数据时，要么全部成功，要么全部失败



Stream Load 的API地址
----

```
http://fe_host:http_port/api/{db}/{table}/_stream_load 

```
 `fe_host`      FE节点的IP地址
 `http_port`  FE节点的webserver_port， 默认为8030
 `db`                数据库名
 `table`          数据表名


例子
---- 

1) 建表
```
CREATE TABLE IF NOT EXISTS test.order_info  
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

2) 准备一批测试数据

数据文件/home/ubuntu/apache-doris/test_data.txt
文件内容：

```
2022-09-07, 10001,2,30001,2022-09-07 09:00:00,2022-09-07 09:00:00
2022-09-07, 10002,2,30001,2022-09-07 09:00:01,2022-09-07 09:00:01
2022-09-07, 10003,2,30001,2022-09-07 09:00:02,2022-09-07 09:00:02
2022-09-07, 10004,2,30001,2022-09-07 09:00:03,2022-09-07 09:00:03
2022-09-07, 10005,2,30001,2022-09-07 09:00:04,2022-09-07 09:00:04
2022-09-07, 10006,2,30001,2022-09-07 09:00:05,2022-09-07 09:00:05
```

3) 提交 Stream Load 任务

```
curl --location-trusted -u test:password123 -H "label:load_data_order_info" -H "column_separator:," -T /home/ubuntu/apache-doris/test_data.txt http://192.168.56.104:8030/api/test/order_info/_stream_load
```


参数说明：
`-u` 用户名:密码
`-T` 本地文件路径
`column_separator` 字段的分隔符
`label`   导入标识, 相同标识的数据不能重复导入多次，用于避免重复导入数据

更多参数可参考官网[STREAM-LOAD](https://doris.apache.org/docs/dev/sql-manual/sql-reference/Data-Manipulation-Statements/Load/STREAM-LOAD/)


执行返回json：
```
ubuntu@ubuntu:~/apache-doris$ curl --location-trusted -u test:password123 -H "label:load_data_order_info" -H "column_separator:," -T /home/ubuntu/apache-doris/test_data.txt http://192.168.56.104:8030/api/test/order_info/_stream_load
{
    "TxnId": 20,
    "Label": "load_data_order_info",
    "TwoPhaseCommit": "false",
    "Status": "Success",
    "Message": "OK",
    "NumberTotalRows": 6,
    "NumberLoadedRows": 6,
    "NumberFilteredRows": 0,
    "NumberUnselectedRows": 0,
    "LoadBytes": 396,
    "LoadTimeMs": 77,
    "BeginTxnTimeMs": 0,
    "StreamLoadPutTimeMs": 2,
    "ReadDataTimeMs": 0,
    "WriteDataTimeMs": 44,
    "CommitAndPublishTimeMs": 29
}
ubuntu@ubuntu:~/apache-doris$
```
`"Status": "Success"` 说明执行成功了。

4) 查看数据

```
mysql> select * from test.order_info;
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
6 rows in set (0.06 sec)

mysql>

```


5) 查看出错信息

如果导入失败(`"Status": "Fail"`)，可以根据返回的json中的`ErrorURL`查看错误信息

```json
{
    "TxnId": 19,
    "Label": "load_data_order_info",
    "TwoPhaseCommit": "false",
    "Status": "Fail",
    "Message": "too many filtered rows",
    "NumberTotalRows": 6,
    "NumberLoadedRows": 0,
    "NumberFilteredRows": 6,
    "NumberUnselectedRows": 0,
    "LoadBytes": 396,
    "LoadTimeMs": 82,
    "BeginTxnTimeMs": 13,
    "StreamLoadPutTimeMs": 37,
    "ReadDataTimeMs": 0,
    "WriteDataTimeMs": 22,
    "CommitAndPublishTimeMs": 0,
    "ErrorURL": "http://192.168.56.104:8030/api/_load_error_log?file=__shard_12/error_log_insert_stmt_fc406c9ec2ab000e-9b2ae69ce54e9c8a_fc406c9ec2ab000e_9b2ae69ce54e9c8a"
}
```