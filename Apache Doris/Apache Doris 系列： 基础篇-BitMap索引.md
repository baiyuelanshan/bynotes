## 1. 测试数据准备

本文使用SSB（Star-Schema-Benchmark）的测试数据，读者也可以自行准备测试数据

#### 1.1 编译ssb-dbgen 数据生成工具
```shell
## 拉取Apache Doris源代码
git clone https://github.com/apache/doris.git

## 编译ssb-dbgen
cd doris/tools/ssb-tools/bin
./build-ssb-dbgen.sh
```

#### 1.2 生成测试数据文件
```shell
./gen-ssb-data.sh
```

数据文件在 doris/tools/ssb-tools/bin/ssb-data 目录下。

#### 1.3 配置Doris集群的信息

修改配置文件， doris/tools/ssb-tools/conf/doris-cluster.conf
![[Pasted image 20230112145202.png]]

#### 1.4 创建Doris表
在doris/tools/ssb-tools/bin目录下，找到create-ssb-tables.sh
```shell
./create-ssb-tables.sh
```

#### 1.5 数据导入Doris
doris/tools/ssb-tools/bin目录下， 找到load-ssb-data.sh
```shell
./load-ssb-data.sh
```

执行完成后，可以在SQL客户端执行select语句查看数据。
```sql
select count(1),count(DISTINCT lo_orderkey), COUNT(DISTINCT lo_custkey), COUNT(DISTINCT lo_partkey), COUNT(DISTINCT lo_suppkey) from ssb.lineorder
```

![[Pasted image 20230112172816.png]]
总共有7000多万左右的数据。

在创建BitMAP前执行以下SQL，
```sql
select count(1) from  ssb.lineorder where lo_suppkey = 26775
```
`冷查询耗时654ms，热查询耗时272ms`

## 2. 创建BitMAP索引

我们要根据lo_suppkey字段进行等值查询，因此对该字段创建BitMAP索引。
在SQL客户端执行
```sql
--创建索引
create index index_lo_suppkey on ssb.lineorder(lo_suppkey) using bitmap;


-- 等待几分钟，查看索引是否创建完成
show index from ssb.lineorder
-- Key_name字段的值出现index_lo_suppkey时说明索引创建完成了
```

执行以下SQL，
```sql
select count(1) from  ssb.lineorder where lo_suppkey = 26775
```
`冷查询耗时256ms，热查询耗时75ms`
相比没有索引提升了2-3倍的查询效率

## 3. 适用场景

*  BitMap索引适用于列基数比较小的列，通俗来说就是值范围比较固定，重复率较高，建议基数在100到100,000之间，比如：country，city等。

* BitMAP不适用于更新频繁的列，在doris中，bitmap INDEX的创建和删除本质上就是Schema Change的过程，所以数据量较大的情况下耗时较长，相关优化后文详细介绍，如果列值更新，对象的索引值也要跟新，所以在频繁更新的列建索引会加大doris的负荷，对doris的性能大打折扣。

