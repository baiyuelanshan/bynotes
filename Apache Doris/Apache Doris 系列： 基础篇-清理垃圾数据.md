## 垃圾数据如何产生
* delete/drop/truncate等操作只是在逻辑上删除了数据，并没有进行物理删除
* 


## 清理所有BE节点的垃圾数据
```sql
ADMIN CLEAN TRASH;
```

## 清理指定BE节点的垃圾数据

ADMIN CLEAN TRASH [ON ("BackendHost1:BackendHeartBeatPort1", "BackendHost2:BackendHeartBeatPort2", ...)];

```sql
ADMIN CLEAN TRASH ON ("192.168.56.104:9050");
```

注意：账户需要有ADMIN权限才能执行以上命令