## 数据模型选择

数据表使用 Aggregate 聚合模型
需要更新的字段使用关键字 `REPLACE_IF_NOT_NULL`

## 举例

#### 建表
```sql
CREATE TABLE IF NOT EXISTS test.expamle_tbl2
(
    `user_id` LARGEINT NOT NULL COMMENT "用户id",
    `date` DATE NOT NULL COMMENT "数据灌入日期时间",
    `city` VARCHAR(20) COMMENT "用户所在城市",
    `age` SMALLINT COMMENT "用户年龄",
    `sex` TINYINT COMMENT "用户性别",
    `last_visit_date` DATETIME REPLACE_IF_NOT_NULL COMMENT "用户最后一次访问时间",
    `cost` BIGINT REPLACE_IF_NOT_NULL COMMENT "用户总消费",
    `max_dwell_time` INT REPLACE_IF_NOT_NULL COMMENT "用户最大停留时间",
    `min_dwell_time` INT REPLACE_IF_NOT_NULL COMMENT "用户最小停留时间"
)
AGGREGATE KEY(`user_id`, `date`, `city`, `age`, `sex`)
DISTRIBUTED BY HASH(`user_id`) BUCKETS 1
PROPERTIES (
"replication_allocation" = "tag.location.default: 1"
);
```

#### 插入数据
```sql
mysql> insert into test.expamle_tbl2 values(10000,'2017-10-01','北京',20,0,'017-10-01 06:00:00',20,10,10);

mysql> select * from test.expamle_tbl2
    -> ;
+---------+------------+--------+------+------+---------------------+------+----------------+----------------+
| user_id | date       | city   | age  | sex  | last_visit_date     | cost | max_dwell_time | min_dwell_time |
+---------+------------+--------+------+------+---------------------+------+----------------+----------------+
| 10000   | 2017-10-01 | 北京   |   20 |    0 | 0017-10-01 06:00:00 |   20 |             10 |             10 |
+---------+------------+--------+------+------+---------------------+------+----------------+----------------+
1 row in set (0.00 sec)

```

#### 更新cost字段
```sql
mysql> insert into test.expamle_tbl2 (user_id,date,city,age,sex,cost) values(10000,'2017-10-01','北京',20,0,50);

mysql> select * from test.expamle_tbl2
    -> ;
+---------+------------+--------+------+------+---------------------+------+----------------+----------------+
| user_id | date       | city   | age  | sex  | last_visit_date     | cost | max_dwell_time | min_dwell_time |
+---------+------------+--------+------+------+---------------------+------+----------------+----------------+
| 10000   | 2017-10-01 | 北京   |   20 |    0 | 0017-10-01 06:00:00 |   50 |             10 |             10 |
+---------+------------+--------+------+------+---------------------+------+----------------+----------------+
1 row in set (0.00 sec)

```

注意：`user_id`, `date`, `city`, `age`, `sex` 这几个字段是聚合键，必须要指定