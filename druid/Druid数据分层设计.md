参考文档：  
https://www.infoq.cn/article/fXl2QXxCQ-AEx14Ee8X5  
http://www.sohu.com/a/122746096_465959




#### 思路
通过配置Historical的Tier分组机制和数据加载的Rule机制实现冷热数据的分层，保证数据的查询效率（前提是大多数的查询都是基于热数据的）


如下图所示，
- Historical节点分为两组：hot和default_tier
- 通过配置数据加载的Rule，把最近30天的数据加载到标记为hot的Historical节点，最近180天的数据加载到default_tier的Historical节点

![avatar](https://static001.infoq.cn/resource/image/7b/86/7b0edabfa6db713585f5a573a5429486.jpg)


#### 如何配置Historical节点的tier
Historical节点的tier默认是default_tier,要把节点的tier更改为hot,可以修改配置文件historical/runtime.properties，加入一下参数：
```
druid.server.tier=hot
druid.server.priority=10
```
提高 "hot" 分组集群的 druid.server.priority 值（默认是 0），热数据的查询都会落到 “hot” 分组。



#### 数据加载的Rule配置

通过Druid提供的HTTP API提交Rule配置，
如：  
curl -X POST 'http://localhost:8888/druid/coordinator/v1/rules/DRUID-TOPIC-ROLLUP-F-02' -H 'Content-Type:application/json' -H 'Accept:application/json' -d @composeRule.json


composeRule.json文件内容：
```
[
{
	"type": "loadByPeriod",
	"tieredReplicants": {
		"hot": 1
	},
	"period": "P30D"
},
{
	"type": "loadByPeriod",
	"tieredReplicants": {
		"_default_tier": 1
	},
	"period": "P180D"
},
{
	"type": "dropForever"
}
]
```

标记为dropForever的数据会在内存和本地缓存文件中被删除，在deep storage 中仍然保留。


#### 如何把DropForever（deep storage）的数据重新加载到Historical节点
DropForever的数据无法被查询到，如果再次使用这部分数据，可以通过以下步骤将其重新加载入Historical节点：
1. 增加一个loadByInterval的Rule,加载指定时间段的数据，或者修改loadByPeriod的Rule, 如：P180D -> P360D

2. 调用Druid HTTP API,通知Historical节点从deep storage加载数据文件，如：  
 curl -X POST 'http://druid:8081/druid/coordinator/v1/datasources/DRUID-TOPIC-ROLLUP-F-02'




