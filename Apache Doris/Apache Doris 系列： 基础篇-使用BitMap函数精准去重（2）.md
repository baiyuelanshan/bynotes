## 1. 背景

Apache Doris 原有的BitMap函数虽然比较通用， 但在亿级别的BitMap大基数并交计算性能较差，主要是由以下两个原因造成的：
* 当BitMap的基数过大，大小超过1GB时，网络或者磁盘的处理时间较长
* BE节点扫描完数据后传输到一个FE节点进行并交计算，给该FE节点带来压力，成为处理瓶颈


解决方案：将bitmap列的值按照范围划分，不同范围的值存储在不同的bucket上，确保在不同bucket的bitmap值是正交的。在查询的时候，先对不同bucket的bitmap值完成聚合计算，上层的FE节点只需合并聚合过的数据并输出即可。如此会极大的改善计算效率，和解决FE节点成为计算瓶颈的问题。

## 2. 使用指南

增加一列hid, 表示range的范围，作为hash分桶列

如：
```sql
CREATE TABLE ssb.`lineorder_bitmap` (
  `lo_shipmode` varchar(50) NULL COMMENT "user tag",
  `hid` smallint(6) NULL COMMENT "Bucket ID",
  `lo_orderkey` bitmap BITMAP_UNION NULL COMMENT ""
) ENGINE=OLAP
AGGREGATE KEY(`lo_shipmode`, `hid`)
COMMENT "OLAP"
DISTRIBUTED BY HASH(`hid`) BUCKETS 3
PROPERTIES (
"replication_allocation" = "tag.location.default: 1",
"in_memory" = "false",
"storage_format" = "V2",
"disable_auto_compaction" = "false"
);
```


## 3. 实际查询效果

#### 3.1 分组去重

```sql
select 
lo_shipmode
,orthogonal_bitmap_union_count(lo_orderkey) cnt
from ssb.`lineorder_bitmap` 
group by lo_shipmode
```

![[Pasted image 20230128163725.png]]
查询结果与上一篇使用COUNT(DISTINCT)的结果一致。
耗时对比：
COUNT(DISTINCT) : 12秒
bitmap_count(bitmap_union(to_bitmap(lo_orderkey)))：6秒
`本文使用的orthogonal_bitmap_union_count(lo_orderkey)：219毫秒`
计算性能得到了极大的提升。

#### 3.2 全量去重

```sql
select orthogonal_bitmap_union_count(lo_orderkey) cnt from ssb.`lineorder_bitmap`
```

COUNT(DISTINCT) : 2.15秒
bitmap_count(bitmap_union(to_bitmap(lo_orderkey)))：3.26秒
`本文使用的orthogonal_bitmap_union_count(lo_orderkey)：141毫秒`
同样，计算性能也是得到了极大的提升。






