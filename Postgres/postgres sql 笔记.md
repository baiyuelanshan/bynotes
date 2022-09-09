## 简介
本文记录了 postgreSQL 日常使用的一些命令和技巧，包含
- 查看表定义
- 查看自定义函数定义
- 添加字段注释
- 添加字段
- 授权

#### 查看表定义
pg_get_tabledef('[schema].[table]')

```sql
select pg_get_tabledef('public.order_users')
```

#### 查看自定义函数定义
select prosrc from pg_proc where proname = '[functionname]'
```sql
select prosrc from pg_proc where proname = 'test_function_1'
```

#### 添加字段注释
COMMENT ON COLUMN [schema].[table].[field] IS '[comment]';
```sql
COMMENT ON COLUMN dtl.test.city IS '城市';
```

#### 添加字段
alter table [schema].[table] add column [fieldname]  [datatype]
```sql
alter table dtl.test add column create_time timestamp default current_timestamp;
```

#### 授权
grant [privillege] on [schema].[table] to [user]
```sql
grant select,INSERT,UPDATE,DELETE,TRUNCATE,REFERENCES,TRIGGER on dtl.dtl_supply_reach_summary to test_user
```

#### 查看当前活跃的SQL 
```
select pid,coorname, usename, client_addr, sysdate-query_start as dur, enqueue, query_id,query from pgxc_stat_activity where usename != 'Ruby' and state = 'active' 
```