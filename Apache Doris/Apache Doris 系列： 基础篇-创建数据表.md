
本文介绍 Doris 怎么创建表及其他的相关操作。

连接 Doris
--------

部署完成 Doris 之后，可以通过任意 MySQL 客户端来连接 Doris。

```
 mysql -u<username> -P<query_port> -h<FE_IP>
```

> 注意：  
> 这里连接 Doris ，指的是连接 Doris FE，  
> 连接的 IP 地址就是 FE 节点 IP 地址，端口是 FE 的 `query_port` 默认是9030



创建数据库
-----

使用 root 用户登录，创建一个 test 的数据库

```
mysql -uroot -P9030 -h127.0.0.1  
  
mysql> create database test;
Query OK, 0 rows affected (0.05 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| test               |
+--------------------+
2 rows in set (0.01 sec)
```

创建新用户
----------
创建 test 用户
```
mysql> CREATE USER test@'%' IDENTIFIED BY "password123";
Query OK, 0 rows affected (0.11 sec)
```

给用户授予权限
--------
```
mysql> GRANT SELECT_PRIV,Load_priv,Alter_priv,Create_priv,Drop_priv ON *.* to test@'%';
Query OK, 0 rows affected (0.01 sec)
```

使用test用户重新登录mysql客户端
```
mysql -utest -P9030 -h192.168.56.104 -p
```


> 权限说明：
> SELECT_PRIV: 数据库和表的只读权限
> Load_priv: 数据库和表的写权限，包括Load, Insert, Delete等
> Alter_priv: 数据库和表的修改权限，包括重命名表， 列的添加/删除/修改, 分区的添加和删除
> Create_priv: 数据库，表和视图的创建权限
> Drop_priv: 数据库，表和视图的删除权限
> Node_priv: 节点的变更权限，包括FE,BE,BROKER节点的添加/删除/下线
> Grant_priv: 权限管理的权限， 包括授权，用户或者角色的添加/修改/删除
> Node_priv 和 Grant_priv一般只授予管理员角色的用户





创建表
---

Doris 提供了数据模型，来满足不同用户场景的使用，
- 明细模型（默认），需指定排序的列， 如： DUPLICATE KEY(col1, col2)
- 聚合模型， 需指定维度列， 如：AGGREGATE KEY(k1, k2, k3)
- 主键模型， 需指定主键， 如：UNIQUE KEY(k1, k2)  

以明细模型为例：


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

```
mysql> desc test.order_info;
+-------------+----------+------+-------+---------+-------+
| Field       | Type     | Null | Key   | Default | Extra |
+-------------+----------+------+-------+---------+-------+
| order_date  | DATE     | No   | true  | NULL    |       |
| order_id    | INT      | No   | true  | NULL    |       |
| buy_num     | TINYINT  | Yes  | false | NULL    | NONE  |
| user_id     | INT      | Yes  | false | NULL    | NONE  |
| create_time | DATETIME | Yes  | false | NULL    | NONE  |
| update_time | DATETIME | Yes  | false | NULL    | NONE  |
+-------------+----------+------+-------+---------+-------+
6 rows in set (0.02 sec)
```


> 建表语句说明：  
>1.  Doris 对字段名称是不区分大小写的，对表名是区分大小写的，如果想忽略大小写，请参照Doris变量配置中的`lower_case_table_names`说明。
>2.  DISTRIBUTED BY 这个是必选项，而且分桶字段必须在 key 里定义
>3.  当创建Doris内部表时， ENGINE=olap, 外部表可支持MYSQL, BROKER, HIVE, ICEBERG, HUDI
>4.  通过replication_num指定副本数



插入数据
----
```
mysql>
mysql> insert into test.order_info values ('2022-09-07', 10001,2,30001,'2022-09-07 09:00:00','2022-09-07 09:00:00')
    -> ;
Query OK, 1 row affected (0.04 sec)
{'label':'insert_948c2a679c8b4e6f-82ef92dd1f39ebcc', 'status':'VISIBLE', 'txnId':'9'}

mysql> select * from test.order_info
    -> ;
+------------+----------+---------+---------+---------------------+---------------------+
| order_date | order_id | buy_num | user_id | create_time         | update_time         |
+------------+----------+---------+---------+---------------------+---------------------+
| 2022-09-07 |    10001 |       2 |   30001 | 2022-09-07 09:00:00 | 2022-09-07 09:00:00 |
+------------+----------+---------+---------+---------------------+---------------------+
1 row in set (0.05 sec)
```

`不建议在生产环境中使用insert into values()语句`
