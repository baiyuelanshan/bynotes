简介
----
本文介绍 Flink 如何流式写入 Apache Doris，分为一下几个部分：
* Flink Doris connector
* Doris FE 节点配置
* Flink SQL 写 Doris



Flink Doris connector
----
Flink Doris connector 本质是通过Stream Load来时实现数据的查询和写入功能。
支持二阶段提交，可实现Exatly Once的写入。


Doris FE 节点配置
----
1）需在 apache-doris/fe/fe.conf 配置文件添加如下配置：
```
enable_http_server_v2 = true
```

2) 重启 FE 节点
```
apache-doris/fe/bin/stop_fe.sh
apache-doris/fe/bin/start_fe.sh --daemon
```

Doris BE 节点配置
----
1）需在 apache-doris/be/be.conf 配置文件添加如下配置：
```
enable_stream_load_record = true
```

2) 重启 BE 节点
```
apache-doris/be/bin/stop_be.sh
apache-doris/be/bin/start_be.sh --daemon
```

生成测试数据
---


通过Flink SQL自带的datagen生成测试数据:

参考官网[DataGen SQL Connector](https://nightlies.apache.org/flink/flink-docs-release-1.15/docs/connectors/table/datagen/)

```sql
CREATE TABLE order_info_source (
    order_date DATE,
    order_id     INT,
    buy_num      INT,
    user_id      INT,
    create_time  TIMESTAMP(3),
    update_time   TIMESTAMP(3)
) WITH (
  'connector' = 'datagen',
  'rows-per-second' =  '10',
  'fields.order_id.min' = '30001',
  'fields.order_id.max' = '30500',
  'fields.user_id.min' = '10001',
  'fields.user_id.max' = '20001',
  'fields.buy_num.min' = '10',
  'fields.buy_num.max' = '20',
  'number-of-rows' = '100'
)

```

datagen参数:
`'rows-per-second' =  '10'` : 每秒发送10条数据
`'fields.order_id.min' = '30001'`: order_id最小值为30001
`'fields.order_id.max' = '30500'`: order_id最大值为30500
`'fields.user_id.min' = '10001'`: user_id最小值为10001
`'fields.user_id.max' = '20001'`: user_id最大值为20001
`'fields.buy_num.min' = '10'`: buy_num最小值为10
`'fields.buy_num.max' = '20'`: buy_num最大值为20
`'number-of-rows' = '100'`: 共发送100条数据, 不设置的话会无限量发送数据



注册Doris Sink表
---


```sql
CREATE TABLE order_info_sink (  
order_date DATE,  
order_id INT,  
buy_num INT,
user_id INT,
create_time TIMESTAMP(3),
update_time TIMESTAMP(3)
)  
WITH (
'connector' = 'doris',   
'fenodes' = '192.168.56.104:8030',   
'table.identifier' = 'test.order_info_example',   
'username' = 'test',   
'password' = 'password123',   
'sink.label-prefix' = 'sink_doris_label_8'
)
```

写入Doris Sink表
---

```sql
insert into order_info_sink select * from order_info_source
```


通过Mysql客户端查看Doris表的数据

```sql
mysql> select * from  test.order_info_example limit 10;
+------------+----------+---------+---------+---------------------+---------------------+
| order_date | order_id | buy_num | user_id | create_time         | update_time         |
+------------+----------+---------+---------+---------------------+---------------------+
| 2022-09-22 |    30007 |      10 |   10560 | 2022-09-22 07:42:21 | 2022-09-22 07:42:21 |
| 2022-09-22 |    30125 |      16 |   17591 | 2022-09-22 07:42:26 | 2022-09-22 07:42:26 |
| 2022-09-22 |    30176 |      17 |   10871 | 2022-09-22 07:42:24 | 2022-09-22 07:42:24 |
| 2022-09-22 |    30479 |      16 |   19847 | 2022-09-22 07:42:25 | 2022-09-22 07:42:25 |
| 2022-09-22 |    30128 |      16 |   19807 | 2022-09-22 07:42:24 | 2022-09-22 07:42:24 |
| 2022-09-22 |    30039 |      13 |   18237 | 2022-09-22 07:42:28 | 2022-09-22 07:42:28 |
| 2022-09-22 |    30060 |      10 |   18309 | 2022-09-22 07:42:24 | 2022-09-22 07:42:24 |
| 2022-09-22 |    30246 |      18 |   10855 | 2022-09-22 07:42:24 | 2022-09-22 07:42:24 |
| 2022-09-22 |    30288 |      19 |   12347 | 2022-09-22 07:42:26 | 2022-09-22 07:42:26 |
| 2022-09-22 |    30449 |      17 |   11488 | 2022-09-22 07:42:23 | 2022-09-22 07:42:23 |
+------------+----------+---------+---------+---------------------+---------------------+
10 rows in set (0.05 sec)

```


完整代码
---

[src/main/java/FlinkSQLSinkExample.java](https://github.com/baiyuelanshan/apache-doris-example/blob/main/src/main/java/FlinkSQLSinkExample.java)

