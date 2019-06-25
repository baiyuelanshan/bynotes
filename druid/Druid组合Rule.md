在历史节点分组（hot, _default_tier）的基础之上，通过设置组合Rule来实现数据分层：

样例一：

```
curl -X POST 'http://localhost:8888/druid/coordinator/v1/rules/DRUID-TOPIC-ROLLUP-F-02' -H 'Content-Type:application/json' -H 'Accept:application/json' -d @composeRule.json
```

```
[
{
	"type": "loadByPeriod",
	"tieredReplicants": {
		"hot": 1
	},
	"period": "P1D"
},
{
	"type": "loadByPeriod",
	"tieredReplicants": {
		"_default_tier": 1
	},
	"period": "P2D"
},
{
	"type": "dropForever"
}
]
```

样例二：
```
curl -X POST 'http://localhost:8888/druid/coordinator/v1/rules/DRUID-TOPIC-ROLLUP-F-02' -H 'Content-Type:application/json' -H 'Accept:application/json' -d @composeRule.01.json
```

```
[
{
	"type": "loadByPeriod",
	"tieredReplicants": {
		"hot": 1
	},
	"period": "PT1H"
},
{
	"type": "loadByPeriod",
	"tieredReplicants": {
		"_default_tier": 1
	},
	"period": "PT24H"
},
{
	"type": "dropForever"
}
]
```
