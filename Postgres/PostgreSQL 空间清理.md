## 背景
PostgreSQL 删除、更新、覆写的历史数据不会从磁盘中清除，久而久之，磁盘的数据越来越多造成空间不足。

## 解决方案
定期找到空间占用大的表，然后执行 vacuum full 指令。

#### 1. 查找空间占用最大的100张表
```sql
SELECT table_schema || '.' || table_name AS table_full_name, pg_size_pretty(pg_total_relation_size('"' 
    || table_schema || '"."' || table_name || '"')) AS size
FROM information_schema.tables 
ORDER BY
pg_total_relation_size('"' || table_schema || '"."' || table_name || '"') DESC limit 100
```

#### 2. 执行 vacuum full 
vacuum full [schema].[table]

```sql
vacuum full schema1.test_table_1
```

注意：这里仅拿一张表为例，实际需要对第一步查询出来的所有表执行 vacuum full