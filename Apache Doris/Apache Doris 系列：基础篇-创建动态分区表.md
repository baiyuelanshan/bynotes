## 动态分区的重要参数

`dynamic_partition.enable` 是否启用动态分区  true|false, 默认为true, 即启用动态分区
`dynamic_partition.time_unit` 动态分区的时间单位, HOUR|DAY|WEEK|MONTH, 可根据配置的时间单位创建或者删除时间分区
`dynamic_partition.replication_num` 动态分区的副本数，默认为表的副本数，注意：如果只有一个be节点，该值应设置为1
`dynamic_partition.start` 一个负的偏移量，通过该偏移量计算出数据过期的时间清理历史数据
`dynamic_partition.end` 一个正的偏移量，通过该偏移量计算出需要预先创建的时间分区
`dynamic_partition.prefix` 动态分区的名称前缀
`dynamic_partition.buckets` 动态分区的分桶数量


## 使用案例

当前只有一个BE节点，有以下建表要求

* 以日期为分区的表
* 7天前的数据需清理
* 提前创建三天后的分区
 

```
CREATE TABLE user_goods_click
(
    click_date DATE,
    user_id int,
    click_count int
)
ENGINE=olap  
DUPLICATE KEY(`click_date`, `user_id`)  
PARTITION BY RANGE(click_date) ()
DISTRIBUTED BY HASH(user_id)
PROPERTIES
(
    "dynamic_partition.enable" = "true",
    "dynamic_partition.time_unit" = "DAY",
    "dynamic_partition.replication_num" = "1",
    "dynamic_partition.start" = "-7",
    "dynamic_partition.end" = "3",
    "dynamic_partition.prefix" = "p",
    "dynamic_partition.buckets" = "32",
    "replication_num" = "1"
);
```


#### 查看分区
```sql
mysql> show partitions from user_goods_click;


+-------------+---------------+----------------+---------------------+--------+--------------+----------------------------------------------------------------------------+-----------------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+-------------------------+
| PartitionId | PartitionName | VisibleVersion | VisibleVersionTime  | State  | PartitionKey | Range                                                                      | DistributionKey       | Buckets | ReplicationNum | StorageMedium | CooldownTime        | LastConsistencyCheckTime | DataSize | IsInMemory | ReplicaAllocation       |
+-------------+---------------+----------------+---------------------+--------+--------------+----------------------------------------------------------------------------+-----------------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+-------------------------+
| 63292       | p20221114     | 1              | 2022-11-14 06:50:08 | NORMAL | click_date   | [types: [DATE]; keys: [2022-11-14]; ..types: [DATE]; keys: [2022-11-15]; ) | click_date, user_name | 32      | 1              | HDD           | 9999-12-31 15:59:59 | NULL                     | 0.000    | false      | tag.location.default: 1 |
| 63357       | p20221115     | 1              | 2022-11-14 06:50:08 | NORMAL | click_date   | [types: [DATE]; keys: [2022-11-15]; ..types: [DATE]; keys: [2022-11-16]; ) | click_date, user_name | 32      | 1              | HDD           | 9999-12-31 15:59:59 | NULL                     | 0.000    | false      | tag.location.default: 1 |
| 63422       | p20221116     | 1              | 2022-11-14 06:50:08 | NORMAL | click_date   | [types: [DATE]; keys: [2022-11-16]; ..types: [DATE]; keys: [2022-11-17]; ) | click_date, user_name | 32      | 1              | HDD           | 9999-12-31 15:59:59 | NULL                     | 0.000    | false      | tag.location.default: 1 |
| 63487       | p20221117     | 1              | 2022-11-14 06:50:08 | NORMAL | click_date   | [types: [DATE]; keys: [2022-11-17]; ..types: [DATE]; keys: [2022-11-18]; ) | click_date, user_name | 32      | 1              | HDD           | 9999-12-31 15:59:59 | NULL                     | 0.000    | false      | tag.location.default: 1 |
+-------------+---------------+----------------+---------------------+--------+--------------+----------------------------------------------------------------------------+-----------------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+-------------------------+
4 rows in set (0.00 sec)


```


可以看到当前有4个分区，p20221114是当天的分区， p20221115/p20221116/p20221117 是预先创建的分区。


#### 插入和查看数据
```sql
mysql> insert into user_goods_click values('2022-11-14',10001,20)
    -> ;
Query OK, 1 row affected (0.07 sec)
{'label':'insert_4124088bb17f4a5c-b0872d2dd3b63022', 'status':'VISIBLE', 'txnId':'3038'}

mysql> select * from user_goods_click
    -> ;
+------------+---------+-------------+
| click_date | user_id | click_count |
+------------+---------+-------------+
| 2022-11-14 |   10001 |          20 |
+------------+---------+-------------+
1 row in set (0.07 sec)

```
 