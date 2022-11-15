
Doris 三种数据模型
* 聚合模型
* 唯一模型
* 明细模型

#### 聚合模型
根据是否设置了AggregationType来分辨维度列和指标列，
维度列不设置AggregationType
指标列设置AggregationType


AggregationType有以下四种：
1. SUM
2. REPLACE
3. MAX
4. MIN

关键字`AGGREGATE KEY` 用于指定聚合的维度列，按维度列排序


```sql
CREATE TABLE IF NOT EXISTS test.user_order_agg 
(    
    `order_id` INT NOT NULL COMMENT "订单id",  
    `user_id` INT COMMENT "[-9223372036854775808, 9223372036854775807]",  
    `buy_num` int sum COMMENT "总购买件数",  
    `order_time` DATE MAX NOT NULL COMMENT "最后下单时间",
    `create_time` DATETIME MIN COMMENT "最早创建时间"
)  
ENGINE=olap  
AGGREGATE KEY (`order_id`,`user_id`)  
DISTRIBUTED BY HASH(`order_id`) BUCKETS 32  
PROPERTIES (  
 "replication_num" = "1"); 
```


```sql
insert into test.user_order_agg values(10000,9000,2,'2022-11-15 09:00:00','2022-11-15 09:00:00');
insert into test.user_order_agg values(10001,9000,2,'2022-11-15 09:00:00','2022-11-15 09:00:00');


insert into test.user_order_agg values(10000,9000,2,'2022-11-15 09:00:00','2022-11-15 09:00:00');
insert into test.user_order_agg values(10001,9000,2,'2022-11-15 09:00:00','2022-11-15 09:00:00');
```

```sql
mysql> insert into test.user_order_agg values(10000,9000,2,'2022-11-15 09:00:00','2022-11-15 09:00:00');
Query OK, 1 row affected (0.08 sec)
{'label':'insert_56d5cf2ae9764b61-9be8593aa8282015', 'status':'VISIBLE', 'txnId':'3047'}

mysql> insert into test.user_order_agg values(10001,9000,2,'2022-11-15 09:00:00','2022-11-15 09:00:00');
Query OK, 1 row affected (0.04 sec)
{'label':'insert_ddd9548464fb4204-a3c7d7fbdce69b59', 'status':'VISIBLE', 'txnId':'3048'}

mysql> insert into test.user_order_agg values(10000,9000,2,'2022-11-15 09:00:00','2022-11-15 09:00:00');
Query OK, 1 row affected (0.05 sec)
{'label':'insert_dce8bc22a55748b9-babd3402a75939ad', 'status':'VISIBLE', 'txnId':'3049'}

mysql> insert into test.user_order_agg values(10001,9000,2,'2022-11-15 09:00:00','2022-11-15 09:00:00');
Query OK, 1 row affected (0.06 sec)
{'label':'insert_f2c267be9942452b-ad30a4accda9f51d', 'status':'VISIBLE', 'txnId':'3050'}

mysql>
mysql> select * from test.user_order_agg;
+----------+---------+---------+------------+---------------------+
| order_id | user_id | buy_num | order_time | create_time         |
+----------+---------+---------+------------+---------------------+
|    10001 |    9000 |       4 | 2022-11-15 | 2022-11-15 09:00:00 |
|    10000 |    9000 |       4 | 2022-11-15 | 2022-11-15 09:00:00 |
+----------+---------+---------+------------+---------------------+
2 rows in set (0.02 sec)

mysql> select count(*) from test.user_order_agg
    -> ;
+----------+
| count(*) |
+----------+
|        2 |
+----------+
1 row in set (0.01 sec)


```


#### 唯一模型
唯一键约束，保证指定列的唯一性，是一种特殊的聚合模型
关键字`Unique Key` 指定唯一键，并根据唯一键排序


```sql
CREATE TABLE IF NOT EXISTS test.user_order_uniq
(  
    `order_id` INT NOT NULL COMMENT "订单id",  
    `user_id` INT COMMENT "[-9223372036854775808, 9223372036854775807]",  
    `buy_num` TINYINT COMMENT "购买件数",  
    `order_time` DATETIME NOT NULL COMMENT "下单时间",  
    `create_time` DATETIME COMMENT "创建时间"
)  
ENGINE=olap  
Unique Key(`order_id`)  
DISTRIBUTED BY HASH(`order_id`) BUCKETS 32  
PROPERTIES (  
 "replication_num" = "1"); 
```


```sql
insert into test.user_order_uniq values(10000,9000,2,'2022-11-15 09:00:00','2022-11-15 09:00:00');
insert into test.user_order_uniq values(10001,9000,2,'2022-11-15 09:00:00','2022-11-15 09:00:00');


insert into test.user_order_uniq values(10000,9000,2,'2022-11-15 09:00:00','2022-11-15 09:00:00');
insert into test.user_order_uniq values(10001,9000,2,'2022-11-15 09:00:00','2022-11-15 09:00:00');
```

```sql
mysql> insert into test.user_order_uniq values(10000,9000,2,'2022-11-15 09:00:00','2022-11-15 09:00:00');
Query OK, 1 row affected (0.07 sec)
{'label':'insert_1646833cb6e44202-a086fe38e1d65476', 'status':'VISIBLE', 'txnId':'3043'}

mysql> insert into test.user_order_uniq values(10001,9000,2,'2022-11-15 09:00:00','2022-11-15 09:00:00');
Query OK, 1 row affected (0.06 sec)
{'label':'insert_8bb4e5785b624c19-8ce040c495886f7d', 'status':'VISIBLE', 'txnId':'3044'}

mysql> insert into test.user_order_uniq values(10000,9000,2,'2022-11-15 09:00:00','2022-11-15 09:00:00');
Query OK, 1 row affected (0.05 sec)
{'label':'insert_54652d9f54d148bf-98a0e98690207ae2', 'status':'VISIBLE', 'txnId':'3045'}

mysql> insert into test.user_order_uniq values(10001,9000,2,'2022-11-15 09:00:00','2022-11-15 09:00:00');
Query OK, 1 row affected (0.08 sec)
{'label':'insert_8f564c7f8ef455b-ac7a8779aa96d2e9', 'status':'VISIBLE', 'txnId':'3046'}

mysql> select * from test.user_order_uniq;
+----------+---------+---------+---------------------+---------------------+
| order_id | user_id | buy_num | order_time          | create_time         |
+----------+---------+---------+---------------------+---------------------+
|    10000 |    9000 |       2 | 2022-11-15 09:00:00 | 2022-11-15 09:00:00 |
|    10001 |    9000 |       2 | 2022-11-15 09:00:00 | 2022-11-15 09:00:00 |
+----------+---------+---------+---------------------+---------------------+
2 rows in set (0.04 sec)

mysql> select count(*) from test.user_order_uniq
    -> ;
+----------+
| count(*) |
+----------+
|        2 |
+----------+
1 row in set (0.01 sec)

```


#### 明细模型
关键字`DUPLICATE KEY` 仅用于排序
保留明细数据，没有聚合操作

```sql
CREATE TABLE IF NOT EXISTS test.user_order_detail  
(  
    `order_id` INT NOT NULL COMMENT "订单id",  
    `user_id` INT COMMENT "[-9223372036854775808, 9223372036854775807]",  
    `buy_num` TINYINT COMMENT "购买件数",  
    `order_time` DATETIME NOT NULL COMMENT "下单时间",  
    `create_time` DATETIME COMMENT "创建时间"
)  
ENGINE=olap  
DUPLICATE KEY(`order_id`, `user_id`)  
DISTRIBUTED BY HASH(`order_id`) BUCKETS 32  
PROPERTIES (  
 "replication_num" = "1"); 
```



```sql
insert into test.user_order_detail values(10000,9000,2,'2022-11-15 09:00:00','2022-11-15 09:00:00');
insert into test.user_order_detail values(10001,9000,2,'2022-11-15 09:00:00','2022-11-15 09:00:00');


insert into test.user_order_detail values(10000,9000,2,'2022-11-15 09:00:00','2022-11-15 09:00:00');
insert into test.user_order_detail values(10001,9000,2,'2022-11-15 09:00:00','2022-11-15 09:00:00');
```

```sql

mysql> insert into test.user_order_detail values(10000,9000,2,'2022-11-15 09:00:00','2022-11-15 09:00:00');
Query OK, 1 row affected (0.05 sec)
{'label':'insert_edbf77fbc34f4ebb-94d0448b6c9b3ee3', 'status':'VISIBLE', 'txnId':'3039'}

mysql> insert into test.user_order_detail values(10001,9000,2,'2022-11-15 09:00:00','2022-11-15 09:00:00');
Query OK, 1 row affected (0.07 sec)
{'label':'insert_a1a156a742ed47fa-a17402d0db29e862', 'status':'VISIBLE', 'txnId':'3040'}
mysql>
mysql> insert into test.user_order_detail values(10000,9000,2,'2022-11-15 09:00:00','2022-11-15 09:00:00');
Query OK, 1 row affected (0.07 sec)
{'label':'insert_25ce47d5b5cf48c9-8ce591dfe090ed84', 'status':'VISIBLE', 'txnId':'3041'}

mysql> insert into test.user_order_detail values(10001,9000,2,'2022-11-15 09:00:00','2022-11-15 09:00:00');
Query OK, 1 row affected (0.04 sec)
{'label':'insert_b101f7e7bd0b4229-8d0636524d4bd9e8', 'status':'VISIBLE', 'txnId':'3042'}

mysql>
mysql> select * from test.user_order_detail;
+----------+---------+---------+---------------------+---------------------+
| order_id | user_id | buy_num | order_time          | create_time         |
+----------+---------+---------+---------------------+---------------------+
|    10000 |    9000 |       2 | 2022-11-15 09:00:00 | 2022-11-15 09:00:00 |
|    10000 |    9000 |       2 | 2022-11-15 09:00:00 | 2022-11-15 09:00:00 |
|    10001 |    9000 |       2 | 2022-11-15 09:00:00 | 2022-11-15 09:00:00 |
|    10001 |    9000 |       2 | 2022-11-15 09:00:00 | 2022-11-15 09:00:00 |
+----------+---------+---------+---------------------+---------------------+
4 rows in set (0.03 sec)

mysql>
mysql> select count(1) from test.user_order_detail;
+----------+
| count(1) |
+----------+
|        4 |
+----------+
1 row in set (0.05 sec)

```