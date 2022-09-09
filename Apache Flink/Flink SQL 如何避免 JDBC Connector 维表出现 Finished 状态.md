## 背景
JDBC Connector 使得关系型数据库（ Mysql、PostgreSQL）可以作为维表，如下图：
![[Pasted image 20220217174606.png]]

但如果使用不当会出现 JDBC Connector Source 在运行一段时间之后出现 Finished 状态，导致 checkpoint 不能正常触发，如下日志：

>2022-02-17 16:16:15.707 INFO  [60] org.apache.flink.runtime.checkpoint.CheckpointCoordinator    - Checkpoint triggering task Source: JDBCTableSource(goods_no, supply_id) -> SourceConversion(table=[default_catalog.default_database.dim_goods_lib, source: [JDBCTableSource(goods_no, supply_id)]], fields=[goods_no, supply_id]) (1/4) of job 5e7450a604c232eb96406bd493421fe4 is not in state RUNNING but FINISHED instead. **Aborting checkpoint**.


## 解决

假如 MySQL 有这张维表 goods_lib，在 Flink SQL 创建该表的映射：

```sql
create table dim_goods_lib(goods_no bigint, supply_id bigint)
with(
    'connector.type' = 'jdbc',
    'connector.url' = 'jdbc:mysql://localhost:3306/test?serverTimezone=UTC',
    'connector.table' = 'fmys_goods_lib',
    'connector.driver' = 'com.mysql.jdbc.Driver',
    'connector.username' = 'test',
    'connector.password' = 'test',
    'connector.lookup.cache.max-rows' = '50000',
    'connector.lookup.cache.ttl' = '60s'
)

```

使用 JOIN 链接维表， SQL 需使用 **FOR SYSTEM_TIME AS OF** ： 
``
```sql
select 
distinct
b.supply_id
,1
,a.planinfoid
from
(
    select 
    goods.goods_no
    ,goods_info.planinfoid
    from
    kafka_goods goods  
    join kafka_goods_info goods_info on goods.planinfoid = goods_info.planinfoid 
    where goods_info.status  = 0
) a
join dim_goods_lib FOR SYSTEM_TIME AS OF PROCTIME() as b on a.goods_no = b.goods_no
```

加上 `FOR SYSTEM_TIME AS OF PROCTIME()`，表示JOIN维表当前时刻所看到的每条数据。

如果没有这些关键词，Flink 只会从 MySQL 中拉一次数据便把 JDBC Source 的状态置为 Finished。
