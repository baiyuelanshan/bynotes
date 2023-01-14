## 1. 简述

精准去重的常用方式是使用SQL函数`COUNT(DISTINCT)`，这种方法最简单，但是要求所有数据都汇聚在一个节点上计算，在数据量大的情况下，需要等待比较常的时间。
例如一个6000000行数据的表，执行以下SQL，需要等待12秒左右的时间
```sql
select 
lo_shipmode
,bitmap_count(bitmap_union(to_bitmap(lo_orderkey))) cnt
from ssb.lineorder 
group by 
lo_shipmode
```

![[Pasted image 20230113180503.png]]

那么还有没有快一点的方式呢？接下来介绍BitMap函数的方式。


## 2. BitMap函数

#### 2.1 BitMap函数的执行效果

```sql
select 
lo_shipmode
,bitmap_count(bitmap_union(to_bitmap(lo_orderkey))) cnt
from ssb.lineorder 
group by 
lo_shipmode
```

`使用BitMap函数耗时6秒，相比COUNT(DISTINCT)提速1倍左右`

![[Pasted image 20230113182520.png]]


#### 2.2 例子中用到的BitMap函数

`to_bitmap()` 将输入值转换了BitMap, BitMap在SQL客户端无法正常显示，输出为NULL，实际是正常的
`bitmap_union()` 聚合函数，用于计算分组后的BitMap并集，有去重效果
`bitmap_count()` 返回bitmap的个数

#### 2.3 BitMap函数的局限性

BitMap函数在Apache Doris中仍然需要先把数据汇聚到一个FE节点才能执行计算，并不能充分发挥分布式计算的优势，在数据量大到一定的情况下，BitMap函数并不能获得比COUNT(DISTINCT)更好的性能，例如， 我们把分组去掉：

```sql
select count(DISTINCT lo_orderkey) from ssb.lineorder
```
COUNT(DISTINCT)耗时2.15秒

```sql
select bitmap_count(bitmap_union(to_bitmap(lo_orderkey))) from ssb.lineorder
```
BitMap函数耗时3.26秒， 比COUNT(DISTINCT)要慢

下一篇文章，笔者将介绍如何优化以充分利用分布式计算的优势。



