## 1. 简介

在处理流式数据时， Flink SQL的ROW_NUMBER，Group by等操作会产生大量的回撤数据，对下游的算子产生巨大的压力，下游算子处理不过来便会产生反压，造成延迟。

如下图，前面两个SQL算子产生的回撤流，给下游的KeyedProcess和Sink算子带来的巨大的压力。


![[Pasted image 20230313141318.png]]


## 2. 解决

开启微批处理。流处理的机制是每来一条数据便会触发一次算子计算，微批处理则是攒够一批数据后触发算子计算，能有效减少ROW_NUMBER，Group by等操作产生的回撤数据，有效提高计算效率。



相关参数：
`table.exec.mini-batch.enabled` : 开启微批处理模式，默认值为false
`table.exec.mini-batch.allow-latency`:  攒批时间，达到该时间则完成攒批触发算子计算
`table.exec.mini-batch.size`: 攒批条数，达到该数值则完成攒批算子计算


```java
// instantiate table environment
TableEnvironment tEnv = ...

// access flink configuration
Configuration configuration = tEnv.getConfig().getConfiguration();
// set low-level key-value options
configuration.setString("table.exec.mini-batch.enabled", "true");
configuration.setString("table.exec.mini-batch.allow-latency", "5 s");
configuration.setString("table.exec.mini-batch.size", "5000");
```

## 3. 开启微批模式后


![[Pasted image 20230313145250.png]]

下游算子的压力降低，反压的情况消失。

![[Pasted image 20230313145459.png]]

