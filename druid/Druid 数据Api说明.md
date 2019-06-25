[toc]
# Druid Api 说明


## kafka数据摄入

接口地址：http://druid_router_host:8090/druid/indexer/v1/supervisor  
示例： http://10.92.208.219:8888/druid/indexer/v1/supervisor  
请求方式：POST  
请求的主要参数：

参数名 | 类型 | 是否必须 | 说明
---|---|---|---
type | String | 是 | 数据源类型
dataSource | String | 是 | 数据源的名字
dimensions | List | 是 | 维度列表
metricsSpec | List | 否 | 指标列表
granularitySpec | Map | 否 | 粒度设置,all,none,second,minute,hour,day,week,month,quarter,year
ioconfig | String | 是 | 数据源相关配置

传入json的示例：  
```
{
  "type": "kafka",
  "dataSchema": {
    "dataSource": "DRUID-TOPIC-DL-F-101",
    "parser": {
      "type": "string",
      "parseSpec": {
        "format": "json",
        "timestampSpec": {
          "column": "timestamp",
          "format": "auto"
        },
        "dimensionsSpec": {
          "dimensions": [
			"timestamp",
			"province",
			"city",
			"datasource",
			"jldbh",
			"yhbh",
			"bjzcbh"
          ]
        }
      }
    },
    "metricsSpec" : [
		{ "type" : "floatSum", "name" : "dl_pos_p_e_total", "fieldName" : "dl_pos_p_e_total" },
		{ "type" : "floatSum", "name" : "dl_pos_p_e_peak", "fieldName" : "dl_pos_p_e_peak" },
		{ "type" : "floatSum", "name" : "dl_pos_p_e_flat", "fieldName" : "dl_pos_p_e_flat" },
		{ "type" : "floatSum", "name" : "dl_pos_p_e_valley", "fieldName" : "dl_pos_p_e_valley" },
		{ "type" : "floatSum", "name" : "dl_pos_p_e_tine", "fieldName" : "dl_pos_p_e_tine" },
		{ "type" : "floatSum", "name" : "dl_pos_q_e_total", "fieldName" : "dl_pos_q_e_total" },
		{ "type" : "floatSum", "name" : "dl_pos_q_e_peak", "fieldName" : "dl_pos_q_e_peak" },
		{ "type" : "floatSum", "name" : "dl_pos_q_e_flat", "fieldName" : "dl_pos_q_e_flat" },
		{ "type" : "floatSum", "name" : "dl_pos_q_e_valley", "fieldName" : "dl_pos_q_e_valley" },
		{ "type" : "floatSum", "name" : "dl_pos_q_e_tine", "fieldName" : "dl_pos_q_e_tine" },
		{ "type" : "floatSum", "name" : "dl_pos_p_e_a", "fieldName" : "dl_pos_p_e_a" },
		{ "type" : "floatSum", "name" : "dl_pos_p_e_b", "fieldName" : "dl_pos_p_e_b" },
		{ "type" : "floatSum", "name" : "dl_pos_p_e_c", "fieldName" : "dl_pos_p_e_c" },
		{ "type" : "floatSum", "name" : "dl_rev_p_e_total", "fieldName" : "dl_rev_p_e_total" },
		{ "type" : "floatSum", "name" : "dl_rev_p_e_peak", "fieldName" : "dl_rev_p_e_peak" },
		{ "type" : "floatSum", "name" : "dl_rev_p_e_flat", "fieldName" : "dl_rev_p_e_flat" },
		{ "type" : "floatSum", "name" : "dl_rev_p_e_valley", "fieldName" : "dl_rev_p_e_valley" },
		{ "type" : "floatSum", "name" : "dl_rev_p_e_tine", "fieldName" : "dl_rev_p_e_tine" },
		{ "type" : "floatSum", "name" : "dl_rev_q_e_total", "fieldName" : "dl_rev_q_e_total" },
		{ "type" : "floatSum", "name" : "dl_rev_q_e_peak", "fieldName" : "dl_rev_q_e_peak" },
		{ "type" : "floatSum", "name" : "dl_rev_q_e_flat", "fieldName" : "dl_rev_q_e_flat" },
		{ "type" : "floatSum", "name" : "dl_rev_q_e_valley", "fieldName" : "dl_rev_q_e_valley" },
		{ "type" : "floatSum", "name" : "dl_rev_q_e_tine", "fieldName" : "dl_rev_q_e_tine" }
	],
    "granularitySpec": {
      "type": "uniform",
      "segmentGranularity": "HOUR",
      "queryGranularity": "NONE",
      "rollup": false
    }
  },
  "tuningConfig": {
    "type": "kafka",
    "reportParseExceptions": false
  },
  "ioConfig": {
    "topic": "DRUID-TOPIC-DL-F-101",
    "replicas": 1,
    "taskDuration": "PT10M",
    "completionTimeout": "PT20M",
	"useEarliestOffset": true,
    "consumerProperties": {
      "bootstrap.servers": "cdh4:9092"
    }
  }
}
```

正常返回值：{"id":"DRUID-TOPIC-ROLLUP-F"}




## hdfs数据文件摄入
接口地址：http://druid_router_host:8888/druid/indexer/v1/task  
示例：http://10.92.208.219:8888/druid/indexer/v1/task  
请求方式：POST  
请求的主要参数：

参数名 | 类型 | 是否必须 | 说明
---|---|---|---
type | String | 是 | 数据源类型
dataSource | String | 是 | 数据源的名字
dimensions | List | 是 | 维度列表
metricsSpec | List | 否 | 指标列表
granularitySpec | Map | 否 | 粒度设置,all,none,second,minute,hour,day,week,month,quarter,year
ioconfig | Map | 是 | 数据源相关配置
jobProperties | Map | 是 | hdfs相关配置
  
传入json的相关配置：  
```
{
  "type" : "index_hadoop",
  "spec" : {
    "dataSchema" : {
      "dataSource" : "wikipedia_hadoop_02",
      "parser" : {
        "type" : "hadoopyString",
        "parseSpec" : {
          "format" : "json",
          "dimensionsSpec" : {
            "dimensions" : [
              "channel",
              "cityName",
              "comment",
              "countryIsoCode",
              "countryName",
              "isAnonymous",
              "isMinor",
              "isNew",
              "isRobot",
              "isUnpatrolled",
              "metroCode",
              "namespace",
              "page",
              "regionIsoCode",
              "regionName",
              "user",
              { "name": "added", "type": "long" },
              { "name": "deleted", "type": "long" },
              { "name": "delta", "type": "long" }
            ]
          },
          "timestampSpec" : {
            "format" : "auto",
            "column" : "time"
          }
        }
      },
      "metricsSpec" : [],
      "granularitySpec" : {
        "type" : "uniform",
        "segmentGranularity" : "day",
        "queryGranularity" : "none",
        "intervals" : ["2015-09-12/2015-09-13"],
        "rollup" : false
      }
    },
    "ioConfig" : {
      "type" : "hadoop",
      "inputSpec" : {
        "type" : "static",
        "paths" : "/quickstart/wikiticker-2015-09-13-sampled.json.gz"
      }
    },
    "tuningConfig" : {
      "type" : "hadoop",
      "partitionsSpec" : {
        "type" : "hashed",
        "targetPartitionSize" : 5000000
      },
      "forceExtendableShardSpecs" : true,
      "jobProperties" : {
        "fs.default.name" : "hdfs://druid-hadoop-demo:9000",
        "fs.defaultFS" : "hdfs://druid-hadoop-demo:9000",
        "dfs.datanode.address" : "druid-hadoop-demo",
        "dfs.client.use.datanode.hostname" : "true",
        "dfs.datanode.use.datanode.hostname" : "true",
        "yarn.resourcemanager.hostname" : "druid-hadoop-demo",
        "yarn.nodemanager.vmem-check-enabled" : "false",
        "mapreduce.map.java.opts" : "-Duser.timezone=UTC -Dfile.encoding=UTF-8",
        "mapreduce.job.user.classpath.first" : "true",
        "mapreduce.reduce.java.opts" : "-Duser.timezone=UTC -Dfile.encoding=UTF-8",
        "mapreduce.map.memory.mb" : 1024,
        "mapreduce.reduce.memory.mb" : 1024
      }
    }
  },
  "hadoopDependencyCoordinates": ["org.apache.hadoop:hadoop-client:2.8.3"]
}
```

正常返回值：{"task":"index_hadoop_wikipedia_hadoop_02_2019-05-10T08:06:37.738Z"}


## 数据查询
接口地址：http://druid_router_host:8888/druid/v2/?pretty  
示例：http://10.92.208.219:8888/druid/v2/?pretty  
请求方式：POST  
请求的主要参数：

参数名 | 类型 | 是否必须 | 说明
---|---|---|---
queryType | String | 是 | 查询类型
dataSource | String | 是 | 数据源的名字
granularity | String | 是 | 时间粒度，all,none,second,minute,hour,day,week,month,quarter,year
descending | String | 否 | 查询结果是否降序
filter | Map | 否 | 过滤条件
aggregations | Map | 是 | 聚合函数,count,longSum,doubleSum,floatSum,doubleMin,doubleMax,floatMin,floatMax,longMin,longMax
intervals | List | 是 | 时间范围

### 查询示例一：设置filter过滤条件

provice="GD" and city="DG"
```
{
  "queryType": "timeseries",
  "dataSource": "DRUID-TOPIC-DL-F-101",
  "granularity": "hour",
  "descending": "true",
  "filter": {
    "type": "and",
    "fields": [
      { "type": "selector", "dimension": "province", "value": "GD" },
      { "type": "selector", "dimension": "city", "value": "DG" }
    ]
  },
  "aggregations": [
    { "type": "floatSum", "name": "dl_pos_p_e_total", "fieldName": "dl_pos_p_e_total" }
  ],
  "intervals": [ "2019-05-06T03:00:00.000/2019-05-16T04:00:00.000Z" ]
}

```

正常返回值：
```
[ {
  "timestamp" : "2019-05-06T03:00:00.000Z",
  "result" : {
    "dl_pos_p_e_total" : 7725.82373046875
  }
} ]
```

### 查询示例二：设置filter条，and 和 or同时使用
provice="GD" and city = "DG" or city = "YF"

```
{
  "queryType": "timeseries",
  "dataSource": "DRUID-TOPIC-DL-F-102",
  "granularity": "hour",
  "descending": "true",
  "filter": {
    "type": "and",
    "fields": [
      { "type": "selector", "dimension": "province", "value": "GD" },
      { "type": "or",
         "fields": [
           { "type": "selector", "dimension": "city", "value": "YF" },
           { "type": "selector", "dimension": "city", "value": "DG" }
         ]
      }
    ]
  },
  "aggregations": [
    { "type": "floatSum", "name": "dl_pos_p_e_total", "fieldName": "dl_pos_p_e_total" }
  ],
  "intervals": [ "2019-05-13T08:00:00.000/2019-05-13T17:00:00.000Z" ]
}
```

正常返回值：
```
[ {
  "timestamp" : "2019-05-13T16:00:00.000Z",
  "result" : {
    "dl_pos_p_e_total" : 21905.958984375
  }
}, {
  "timestamp" : "2019-05-13T15:00:00.000Z",
  "result" : {
    "dl_pos_p_e_total" : 25809.890625
  }
}, {
  "timestamp" : "2019-05-13T14:00:00.000Z",
  "result" : {
    "dl_pos_p_e_total" : 0.0
  }
}, {
  "timestamp" : "2019-05-13T13:00:00.000Z",
  "result" : {
    "dl_pos_p_e_total" : 0.0
  }
}, {
  "timestamp" : "2019-05-13T12:00:00.000Z",
  "result" : {
    "dl_pos_p_e_total" : 0.0
  }
}, {
  "timestamp" : "2019-05-13T11:00:00.000Z",
  "result" : {
    "dl_pos_p_e_total" : 27603.525390625
  }
}, {
  "timestamp" : "2019-05-13T10:00:00.000Z",
  "result" : {
    "dl_pos_p_e_total" : 28640.46875
  }
}, {
  "timestamp" : "2019-05-13T09:00:00.000Z",
  "result" : {
    "dl_pos_p_e_total" : 29702.15625
  }
}, {
  "timestamp" : "2019-05-13T08:00:00.000Z",
  "result" : {
    "dl_pos_p_e_total" : 7057.0126953125
  }
} ]
```
### 查询示例三：查询平均值
```
{
  "queryType": "timeseries",
  "dataSource": "DRUID-TOPIC-ROLLUP-F-PAR-01",
  "granularity": "hour",
  "descending": "true",
  "filter": {
    "type": "and",
    "fields": [
      { "type": "selector", "dimension": "province", "value": "GD" },
      { "type": "or",
        "fields": [
          { "type": "selector", "dimension": "city", "value": "YF" },
          { "type": "selector", "dimension": "city", "value": "DG" }
        ]
      }
    ]
  },
  "aggregations": [
    { "type": "floatSum", "name": "dl_pos_p_e_total_sum", "fieldName": "dl_pos_p_e_total" },
    { "type": "count", "name": "dl_pos_p_e_total_count", "fieldName": "dl_pos_p_e_total" }
  ],
  "postAggregations": [
    { "type": "arithmetic",
      "name": "avg",
      "fn": "/",
      "fields": [
        { "type": "fieldAccess", "name": "dl_pos_p_e_total_sum", "fieldName": "dl_pos_p_e_total_sum" },
        { "type": "fieldAccess", "name": "dl_pos_p_e_total_count", "fieldName": "dl_pos_p_e_total_count" }
      ]
    }
  ],
  "intervals": [ "2019-05-06T03:00:00.000/2019-05-06T04:00:00.000Z" ]
}
```
正常返回值：
```
[ {
  "timestamp" : "2019-05-06T03:00:00.000Z",
  "result" : {
    "dl_pos_p_e_total_count" : 325,
    "dl_pos_p_e_total_sum" : 7725.82373046875,
    "avg" : 23.77176532451923
  }
} ]
```

## SQL查询
接口地址：http://druid_router_host:8888/druid/v2/sql  
示例：http://10.92.208.219:8888/druid/v2/sql  
请求方式：POST

```
curl -X POST 'http://10.92.208.219:8888/druid/v2/sql' -H 'Content-Type:application/json' -H 'Accept:application/json' -d @sql-query.json -u druid_system:changeme
```
sql-query.json:
```
{
    "query": "select * from \"DRUID-TOPIC-DL-F-102\" LIMIT 10"
}
```

返回值：
```
[{"__time":"2019-05-13T08:45:21.515Z","bjzcbh":"000000004","city":"DG","datasource":"TMR","dl_pos_p_e_a":13.5,"dl_pos_p_e_b":14.2,"dl_pos_p_e_c":36.97,"dl_pos_p_e_flat":9.49,"dl_pos_p_e_peak":28.97,"dl_pos_p_e_tine":8.58,"dl_pos_p_e_total":16.23,"dl_pos_p_e_valley":28.23,"dl_pos_q_e_flat":5.69,"dl_pos_q_e_peak":0.51,"dl_pos_q_e_tine":46.27,"dl_pos_q_e_total":22.26,"dl_pos_q_e_valley":3.71,"dl_rev_p_e_flat":2.14,"dl_rev_p_e_peak":41.78,"dl_rev_p_e_tine":22.77,"dl_rev_p_e_total":42.64,"dl_rev_p_e_valley":12.39,"dl_rev_q_e_flat":16.48,"dl_rev_q_e_peak":19.16,"dl_rev_q_e_tine":8.22,"dl_rev_q_e_total":12.81,"dl_rev_q_e_valley":21.53,"jldbh":"718000000000002000000004","province":"GD","timestamp":"1557737121515","yhbh":"718000000000002"},{"__time":"2019-05-13T08:45:24.517Z","bjzcbh":"000000005","city":"DG","datasource":"TMR","dl_pos_p_e_a":27.94,"dl_pos_p_e_b":0.81,"dl_pos_p_e_c":33.99,"dl_pos_p_e_flat":14.07,"dl_pos_p_e_peak":18.91,"dl_pos_p_e_tine":25.04,"dl_pos_p_e_total":49.27,"dl_pos_p_e_valley":38.2,"dl_pos_q_e_flat":35.39,"dl_pos_q_e_peak":3.94,"dl_pos_q_e_tine":48.77,"dl_pos_q_e_total":34.47,"dl_pos_q_e_valley":24.52,"dl_rev_p_e_flat":15.47,"dl_rev_p_e_peak":29.32,"dl_rev_p_e_tine":8.5,"dl_rev_p_e_total":43.22,"dl_rev_p_e_valley":44.68,"dl_rev_q_e_flat":44.83,"dl_rev_q_e_peak":20.67,"dl_rev_q_e_tine":16.44,"dl_rev_q_e_total":19.27,"dl_rev_q_e_valley":33.33,"jldbh":"718000000000003000000005","province":"GD","timestamp":"1557737124517","yhbh":"718000000000003"},{"__time":"2019-05-13T08:45:27.519Z","bjzcbh":"000000006","city":"DG","datasource":"TMR","dl_pos_p_e_a":15.05,"dl_pos_p_e_b":34.64,"dl_pos_p_e_c":19.13,"dl_pos_p_e_flat":9.07,"dl_pos_p_e_peak":48.24,"dl_pos_p_e_tine":19.69,"dl_pos_p_e_total":30.34,"dl_pos_p_e_valley":30.68,"dl_pos_q_e_flat":34.21,"dl_pos_q_e_peak":41.44,"dl_pos_q_e_tine":11.81,"dl_pos_q_e_total":47.48,"dl_pos_q_e_valley":30.2,"dl_rev_p_e_flat":1.87,"dl_rev_p_e_peak":27.79,"dl_rev_p_e_tine":6.43,"dl_rev_p_e_total":22.08,"dl_rev_p_e_valley":8.51,"dl_rev_q_e_flat":10.73,"dl_rev_q_e_peak":38.51,"dl_rev_q_e_tine":14.67,"dl_rev_q_e_total":4.17,"dl_rev_q_e_valley":39.48,"jldbh":"718000000000003000000006","province":"GD","timestamp":"1557737127519","yhbh":"718000000000003"},{"__time":"2019-05-13T08:45:30.520Z","bjzcbh":"000000007","city":"DG","datasource":"TMR","dl_pos_p_e_a":45.41,"dl_pos_p_e_b":45.83,"dl_pos_p_e_c":2.07,"dl_pos_p_e_flat":40.86,"dl_pos_p_e_peak":28.16,"dl_pos_p_e_tine":35.66,"dl_pos_p_e_total":29.23,"dl_pos_p_e_valley":33.88,"dl_pos_q_e_flat":5.21,"dl_pos_q_e_peak":29.76,"dl_pos_q_e_tine":24.43,"dl_pos_q_e_total":23.78,"dl_pos_q_e_valley":9.52,"dl_rev_p_e_flat":33.42,"dl_rev_p_e_peak":18.22,"dl_rev_p_e_tine":42.24,"dl_rev_p_e_total":7.51,"dl_rev_p_e_valley":46.0,"dl_rev_q_e_flat":33.5,"dl_rev_q_e_peak":37.52,"dl_rev_q_e_tine":2.66,"dl_rev_q_e_total":5.09,"dl_rev_q_e_valley":1.66,"jldbh":"718000000000003000000007","province":"GD","timestamp":"1557737130520","yhbh":"718000000000003"},{"__time":"2019-05-13T08:45:33.521Z","bjzcbh":"000000008","city":"DG","datasource":"TMR","dl_pos_p_e_a":14.52,"dl_pos_p_e_b":6.41,"dl_pos_p_e_c":11.73,"dl_pos_p_e_flat":4.45,"dl_pos_p_e_peak":16.31,"dl_pos_p_e_tine":44.36,"dl_pos_p_e_total":22.66,"dl_pos_p_e_valley":43.07,"dl_pos_q_e_flat":37.22,"dl_pos_q_e_peak":39.28,"dl_pos_q_e_tine":19.82,"dl_pos_q_e_total":36.94,"dl_pos_q_e_valley":43.55,"dl_rev_p_e_flat":14.62,"dl_rev_p_e_peak":45.06,"dl_rev_p_e_tine":9.61,"dl_rev_p_e_total":21.75,"dl_rev_p_e_valley":16.75,"dl_rev_q_e_flat":46.33,"dl_rev_q_e_peak":17.55,"dl_rev_q_e_tine":27.54,"dl_rev_q_e_total":5.85,"dl_rev_q_e_valley":20.25,"jldbh":"718000000000003000000008","province":"GD","timestamp":"1557737133521","yhbh":"718000000000003"},{"__time":"2019-05-13T08:45:36.522Z","bjzcbh":"000000009","city":"DG","datasource":"TMR","dl_pos_p_e_a":26.73,"dl_pos_p_e_b":33.52,"dl_pos_p_e_c":26.5,"dl_pos_p_e_flat":6.62,"dl_pos_p_e_peak":46.96,"dl_pos_p_e_tine":45.4,"dl_pos_p_e_total":11.8,"dl_pos_p_e_valley":43.8,"dl_pos_q_e_flat":45.35,"dl_pos_q_e_peak":20.03,"dl_pos_q_e_tine":48.74,"dl_pos_q_e_total":39.06,"dl_pos_q_e_valley":15.02,"dl_rev_p_e_flat":42.76,"dl_rev_p_e_peak":29.94,"dl_rev_p_e_tine":15.43,"dl_rev_p_e_total":36.41,"dl_rev_p_e_valley":25.84,"dl_rev_q_e_flat":10.99,"dl_rev_q_e_peak":23.06,"dl_rev_q_e_tine":37.11,"dl_rev_q_e_total":1.41,"dl_rev_q_e_valley":16.67,"jldbh":"718000000000004000000009","province":"GD","timestamp":"1557737136522","yhbh":"718000000000004"},{"__time":"2019-05-13T08:45:39.523Z","bjzcbh":"000000010","city":"DG","datasource":"TMR","dl_pos_p_e_a":33.61,"dl_pos_p_e_b":20.56,"dl_pos_p_e_c":34.24,"dl_pos_p_e_flat":5.19,"dl_pos_p_e_peak":26.01,"dl_pos_p_e_tine":36.87,"dl_pos_p_e_total":22.9,"dl_pos_p_e_valley":1.05,"dl_pos_q_e_flat":34.77,"dl_pos_q_e_peak":42.39,"dl_pos_q_e_tine":29.74,"dl_pos_q_e_total":45.62,"dl_pos_q_e_valley":42.34,"dl_rev_p_e_flat":8.61,"dl_rev_p_e_peak":37.86,"dl_rev_p_e_tine":16.36,"dl_rev_p_e_total":13.63,"dl_rev_p_e_valley":7.32,"dl_rev_q_e_flat":38.61,"dl_rev_q_e_peak":23.27,"dl_rev_q_e_tine":7.12,"dl_rev_q_e_total":21.65,"dl_rev_q_e_valley":35.53,"jldbh":"718000000000004000000010","province":"GD","timestamp":"1557737139523","yhbh":"718000000000004"},{"__time":"2019-05-13T08:45:42.525Z","bjzcbh":"000000011","city":"DG","datasource":"TMR","dl_pos_p_e_a":21.86,"dl_pos_p_e_b":36.58,"dl_pos_p_e_c":13.4,"dl_pos_p_e_flat":17.33,"dl_pos_p_e_peak":27.45,"dl_pos_p_e_tine":22.6,"dl_pos_p_e_total":11.54,"dl_pos_p_e_valley":28.48,"dl_pos_q_e_flat":34.67,"dl_pos_q_e_peak":30.96,"dl_pos_q_e_tine":3.62,"dl_pos_q_e_total":22.93,"dl_pos_q_e_valley":18.6,"dl_rev_p_e_flat":33.38,"dl_rev_p_e_peak":8.09,"dl_rev_p_e_tine":32.44,"dl_rev_p_e_total":21.0,"dl_rev_p_e_valley":25.13,"dl_rev_q_e_flat":1.24,"dl_rev_q_e_peak":3.33,"dl_rev_q_e_tine":13.44,"dl_rev_q_e_total":36.23,"dl_rev_q_e_valley":18.01,"jldbh":"718000000000004000000011","province":"GD","timestamp":"1557737142525","yhbh":"718000000000004"},{"__time":"2019-05-13T08:45:45.526Z","bjzcbh":"000000012","city":"DG","datasource":"TMR","dl_pos_p_e_a":11.76,"dl_pos_p_e_b":18.38,"dl_pos_p_e_c":8.15,"dl_pos_p_e_flat":17.55,"dl_pos_p_e_peak":22.12,"dl_pos_p_e_tine":48.76,"dl_pos_p_e_total":20.94,"dl_pos_p_e_valley":10.08,"dl_pos_q_e_flat":31.92,"dl_pos_q_e_peak":7.16,"dl_pos_q_e_tine":16.3,"dl_pos_q_e_total":48.3,"dl_pos_q_e_valley":2.72,"dl_rev_p_e_flat":27.99,"dl_rev_p_e_peak":21.57,"dl_rev_p_e_tine":4.36,"dl_rev_p_e_total":34.33,"dl_rev_p_e_valley":32.27,"dl_rev_q_e_flat":48.36,"dl_rev_q_e_peak":47.34,"dl_rev_q_e_tine":32.75,"dl_rev_q_e_total":16.13,"dl_rev_q_e_valley":19.08,"jldbh":"718000000000004000000012","province":"GD","timestamp":"1557737145526","yhbh":"718000000000004"},{"__time":"2019-05-13T08:45:48.527Z","bjzcbh":"000000013","city":"DG","datasource":"TMR","dl_pos_p_e_a":25.22,"dl_pos_p_e_b":5.48,"dl_pos_p_e_c":38.63,"dl_pos_p_e_flat":43.25,"dl_pos_p_e_peak":25.61,"dl_pos_p_e_tine":6.67,"dl_pos_p_e_total":36.62,"dl_pos_p_e_valley":49.6,"dl_pos_q_e_flat":40.42,"dl_pos_q_e_peak":42.48,"dl_pos_q_e_tine":23.69,"dl_pos_q_e_total":43.85,"dl_pos_q_e_valley":19.68,"dl_rev_p_e_flat":8.14,"dl_rev_p_e_peak":19.91,"dl_rev_p_e_tine":27.04,"dl_rev_p_e_total":9.6,"dl_rev_p_e_valley":47.62,"dl_rev_q_e_flat":12.12,"dl_rev_q_e_peak":8.02,"dl_rev_q_e_tine":45.05,"dl_rev_q_e_total":7.4,"dl_rev_q_e_valley":31.18,"jldbh":"718000000000004000000013","province":"GD","timestamp":"1557737148527","yhbh":"718000000000004"}]
```





## 数据源列表获取
接口地址：http://druid_router_host:8888/druid/coordinator/v1/metadata/datasources
请求方式：GET  

示例：
http://10.92.208.219:8888/druid/coordinator/v1/metadata/datasources

正常返回值：
```
["DRUID-TOPIC-ROLLUP-F-PAR","DRUID-TOPIC-ROLLUP-F-PAR-01","retention-tutorial","updates-tutorial","wikipedia","wikipedia_hadoop","wikipedia_hadoop_01"]
```

## 数据源信息获取（包含维度和度量）
接口地址：http://druid_router_host:8888/druid/coordinator/v1/metadata/datasources/[dataSource]  
请求方式：GET  

示例：  
http://10.92.208.219:8888/druid/coordinator/v1/metadata/datasources/++DRUID-TOPIC-ROLLUP-F++

正常返回值：
 ```
 {
	"name": "DRUID-TOPIC-ROLLUP-F",
	"properties": {
		"created": "2019-05-06T06:27:27.142Z"
	},
	"segments": [{
		"dataSource": "DRUID-TOPIC-ROLLUP-F",
		"interval": "2019-05-06T03:00:00.000Z/2019-05-06T04:00:00.000Z",
		"version": "2019-05-06T04:58:47.871Z",
		"loadSpec": {
			"type": "hdfs",
			"path": "hdfs://druid-hadoop-demo:9000/Users/mac/druid/apache-druid-0.14.0-incubating/data/DRUID-TOPIC-ROLLUP-F/20190506T030000.000Z_20190506T040000.000Z/2019-05-06T04_58_47.871Z/0_3cfce931-6b6d-4d96-afa4-d238a5d2a356_index.zip"
		},
		"dimensions": "timestamp,province,city,datasource,jldbh,yhbh,bjzcbh",
		"metrics": "dl_pos_p_e_total,dl_pos_p_e_peak,dl_pos_p_e_flat,dl_pos_p_e_valley,dl_pos_p_e_tine,dl_pos_q_e_total,dl_pos_q_e_peak,dl_pos_q_e_flat,dl_pos_q_e_valley,dl_pos_q_e_tine,dl_pos_p_e_a,dl_pos_p_e_b,dl_pos_p_e_c,dl_rev_p_e_total,dl_rev_p_e_peak,dl_rev_p_e_flat,dl_rev_p_e_valley,dl_rev_p_e_tine,dl_rev_q_e_total,dl_rev_q_e_peak,dl_rev_q_e_flat,dl_rev_q_e_valley,dl_rev_q_e_tine",
		"shardSpec": {
			"type": "numbered",
			"partitionNum": 0,
			"partitions": 0
		},
		"binaryVersion": 9,
		"size": 82693,
		"identifier": "DRUID-TOPIC-ROLLUP-F_2019-05-06T03:00:00.000Z_2019-05-06T04:00:00.000Z_2019-05-06T04:58:47.871Z"
	}, {
		"dataSource": "DRUID-TOPIC-ROLLUP-F",
		"interval": "2019-05-06T04:00:00.000Z/2019-05-06T05:00:00.000Z",
		"version": "2019-05-06T04:58:48.868Z",
		"loadSpec": {
			"type": "hdfs",
			"path": "hdfs://druid-hadoop-demo:9000/Users/mac/druid/apache-druid-0.14.0-incubating/data/DRUID-TOPIC-ROLLUP-F/20190506T040000.000Z_20190506T050000.000Z/2019-05-06T04_58_48.868Z/0_c7df05b0-e6cd-4dc2-b1d3-92863212e00e_index.zip"
		},
		"dimensions": "timestamp,province,city,datasource,jldbh,yhbh,bjzcbh",
		"metrics": "dl_pos_p_e_total,dl_pos_p_e_peak,dl_pos_p_e_flat,dl_pos_p_e_valley,dl_pos_p_e_tine,dl_pos_q_e_total,dl_pos_q_e_peak,dl_pos_q_e_flat,dl_pos_q_e_valley,dl_pos_q_e_tine,dl_pos_p_e_a,dl_pos_p_e_b,dl_pos_p_e_c,dl_rev_p_e_total,dl_rev_p_e_peak,dl_rev_p_e_flat,dl_rev_p_e_valley,dl_rev_p_e_tine,dl_rev_q_e_total,dl_rev_q_e_peak,dl_rev_q_e_flat,dl_rev_q_e_valley,dl_rev_q_e_tine",
		"shardSpec": {
			"type": "numbered",
			"partitionNum": 0,
			"partitions": 0
		},
		"binaryVersion": 9,
		"size": 278234,
		"identifier": "DRUID-TOPIC-ROLLUP-F_2019-05-06T04:00:00.000Z_2019-05-06T05:00:00.000Z_2019-05-06T04:58:48.868Z"
	}, {
		"dataSource": "DRUID-TOPIC-ROLLUP-F",
		"interval": "2019-05-06T05:00:00.000Z/2019-05-06T06:00:00.000Z",
		"version": "2019-05-06T05:00:01.561Z",
		"loadSpec": {
			"type": "hdfs",
			"path": "hdfs://druid-hadoop-demo:9000/Users/mac/druid/apache-druid-0.14.0-incubating/data/DRUID-TOPIC-ROLLUP-F/20190506T050000.000Z_20190506T060000.000Z/2019-05-06T05_00_01.561Z/0_b8f98efc-d571-4064-8413-331302f3b709_index.zip"
		},
		"dimensions": "timestamp,province,city,datasource,jldbh,yhbh,bjzcbh",
		"metrics": "dl_pos_p_e_total,dl_pos_p_e_peak,dl_pos_p_e_flat,dl_pos_p_e_valley,dl_pos_p_e_tine,dl_pos_q_e_total,dl_pos_q_e_peak,dl_pos_q_e_flat,dl_pos_q_e_valley,dl_pos_q_e_tine,dl_pos_p_e_a,dl_pos_p_e_b,dl_pos_p_e_c,dl_rev_p_e_total,dl_rev_p_e_peak,dl_rev_p_e_flat,dl_rev_p_e_valley,dl_rev_p_e_tine,dl_rev_q_e_total,dl_rev_q_e_peak,dl_rev_q_e_flat,dl_rev_q_e_valley,dl_rev_q_e_tine",
		"shardSpec": {
			"type": "numbered",
			"partitionNum": 0,
			"partitions": 0
		},
		"binaryVersion": 9,
		"size": 278379,
		"identifier": "DRUID-TOPIC-ROLLUP-F_2019-05-06T05:00:00.000Z_2019-05-06T06:00:00.000Z_2019-05-06T05:00:01.561Z"
	}, {
		"dataSource": "DRUID-TOPIC-ROLLUP-F",
		"interval": "2019-05-06T06:00:00.000Z/2019-05-06T07:00:00.000Z",
		"version": "2019-05-06T06:00:02.548Z",
		"loadSpec": {
			"type": "hdfs",
			"path": "hdfs://druid-hadoop-demo:9000/Users/mac/druid/apache-druid-0.14.0-incubating/data/DRUID-TOPIC-ROLLUP-F/20190506T060000.000Z_20190506T070000.000Z/2019-05-06T06_00_02.548Z/0_a3af2764-65b0-4b18-bac6-5b557d189633_index.zip"
		},
		"dimensions": "timestamp,province,city,datasource,jldbh,yhbh,bjzcbh",
		"metrics": "dl_pos_p_e_total,dl_pos_p_e_peak,dl_pos_p_e_flat,dl_pos_p_e_valley,dl_pos_p_e_tine,dl_pos_q_e_total,dl_pos_q_e_peak,dl_pos_q_e_flat,dl_pos_q_e_valley,dl_pos_q_e_tine,dl_pos_p_e_a,dl_pos_p_e_b,dl_pos_p_e_c,dl_rev_p_e_total,dl_rev_p_e_peak,dl_rev_p_e_flat,dl_rev_p_e_valley,dl_rev_p_e_tine,dl_rev_q_e_total,dl_rev_q_e_peak,dl_rev_q_e_flat,dl_rev_q_e_valley,dl_rev_q_e_tine",
		"shardSpec": {
			"type": "numbered",
			"partitionNum": 0,
			"partitions": 0
		},
		"binaryVersion": 9,
		"size": 112627,
		"identifier": "DRUID-TOPIC-ROLLUP-F_2019-05-06T06:00:00.000Z_2019-05-06T07:00:00.000Z_2019-05-06T06:00:02.548Z"
	}]
}

 ```


## 设置数据时效规则
接口地址：http://druid_router_host:8888/druid/coordinator/v1/rules/[dataSource]  
示例：http://10.92.208.219:8888/druid/coordinator/v1/rules/DRUID-TOPIC-ROLLUP-F  
请求方式：POST  
请求的主要参数：

参数名 | 类型 | 是否必须 | 说明
---|---|---|---
type | String | 是 | 数据时效类型
tieredReplicants | Map | 是 | 数据源的名字
period | String | 是 | 时间粒度


示例：
curl -X POST 'http://localhost:8888/druid/coordinator/v1/rules/++DRUID-TOPIC-ROLLUP-F++' -H 'Content-Type:application/json' -H 'Accept:application/json' -d @composeRule.json


composeRule.json:
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

正常返回值：  
无数据返回  
HTTP code: 200


## 获取数据存放和时效规则
接口地址：http://druid_router_host:8888/druid/coordinator/v1/rule  
示例： http://10.92.208.219:8888/druid/coordinator/v1/rules  
请求方式：GET  

示例：  
1. 获取所有数据源的存放和时效规则  
curl -X GET 'http://10.92.208.219:8888/druid/coordinator/v1/rules'
返回值：  
```
{"_default":[{"tieredReplicants":{"_default_tier":2},"type":"loadForever"}],"wikipedia":[{"tieredReplicants":{"_default_tier":1},"type":"loadForever"}],"DRUID-TOPIC-DL-F-101":[{"period":"P30D","includeFuture":true,"tieredReplicants":{"hot":1},"type":"loadByPeriod"},{"period":"P180D","includeFuture":true,"tieredReplicants":{"_default_tier":1},"type":"loadByPeriod"},{"type":"dropForever"}]}
```
2. 获取单个数据源的存放和时效规则  
curl -X GET 'http://10.92.208.219:8888/druid/coordinator/v1/rules/wikipedia'
返回值：
```
[{"tieredReplicants":{"_default_tier":1},"type":"loadForever"}]
```

注意：  
当所有datasource都没有设置时效规则的时候，返回值是：
```
{"_default":[{"tieredReplicants":{"_default_tier":2},"type":"loadForever"}]}
```


## 获取payload(datasource 配置信息)
接口地址：http://druid_router_host:8888/druid/indexer/v1/supervisor/[datasource]   
示例：http://10.92.208.219:8888/druid/indexer/v1/supervisor/DRUID-TOPIC-DL-F-102  
请求方式：GET

示例：
curl -X GET  http://10.92.208.219:8888/druid/indexer/v1/supervisor/DRUID-TOPIC-DL-F-102

返回值：

```
{
	"type": "kafka",
	"dataSchema": {
		"dataSource": "DRUID-TOPIC-DL-F-102",
		"parser": {
			"type": "string",
			"parseSpec": {
				"format": "json",
				"timestampSpec": {
					"column": "timestamp",
					"format": "auto"
				},
				"dimensionsSpec": {
					"dimensions": ["timestamp", "province", "city", "datasource", "jldbh", "yhbh", "bjzcbh"]
				}
			}
		},
		"metricsSpec": [{
			"type": "floatSum",
			"name": "dl_pos_p_e_total",
			"fieldName": "dl_pos_p_e_total",
			"expression": null
		}, {
			"type": "floatSum",
			"name": "dl_pos_p_e_peak",
			"fieldName": "dl_pos_p_e_peak",
			"expression": null
		}, {
			"type": "floatSum",
			"name": "dl_pos_p_e_flat",
			"fieldName": "dl_pos_p_e_flat",
			"expression": null
		}, {
			"type": "floatSum",
			"name": "dl_pos_p_e_valley",
			"fieldName": "dl_pos_p_e_valley",
			"expression": null
		}, {
			"type": "floatSum",
			"name": "dl_pos_p_e_tine",
			"fieldName": "dl_pos_p_e_tine",
			"expression": null
		}, {
			"type": "floatSum",
			"name": "dl_pos_q_e_total",
			"fieldName": "dl_pos_q_e_total",
			"expression": null
		}, {
			"type": "floatSum",
			"name": "dl_pos_q_e_peak",
			"fieldName": "dl_pos_q_e_peak",
			"expression": null
		}, {
			"type": "floatSum",
			"name": "dl_pos_q_e_flat",
			"fieldName": "dl_pos_q_e_flat",
			"expression": null
		}, {
			"type": "floatSum",
			"name": "dl_pos_q_e_valley",
			"fieldName": "dl_pos_q_e_valley",
			"expression": null
		}, {
			"type": "floatSum",
			"name": "dl_pos_q_e_tine",
			"fieldName": "dl_pos_q_e_tine",
			"expression": null
		}, {
			"type": "floatSum",
			"name": "dl_pos_p_e_a",
			"fieldName": "dl_pos_p_e_a",
			"expression": null
		}, {
			"type": "floatSum",
			"name": "dl_pos_p_e_b",
			"fieldName": "dl_pos_p_e_b",
			"expression": null
		}, {
			"type": "floatSum",
			"name": "dl_pos_p_e_c",
			"fieldName": "dl_pos_p_e_c",
			"expression": null
		}, {
			"type": "floatSum",
			"name": "dl_rev_p_e_total",
			"fieldName": "dl_rev_p_e_total",
			"expression": null
		}, {
			"type": "floatSum",
			"name": "dl_rev_p_e_peak",
			"fieldName": "dl_rev_p_e_peak",
			"expression": null
		}, {
			"type": "floatSum",
			"name": "dl_rev_p_e_flat",
			"fieldName": "dl_rev_p_e_flat",
			"expression": null
		}, {
			"type": "floatSum",
			"name": "dl_rev_p_e_valley",
			"fieldName": "dl_rev_p_e_valley",
			"expression": null
		}, {
			"type": "floatSum",
			"name": "dl_rev_p_e_tine",
			"fieldName": "dl_rev_p_e_tine",
			"expression": null
		}, {
			"type": "floatSum",
			"name": "dl_rev_q_e_total",
			"fieldName": "dl_rev_q_e_total",
			"expression": null
		}, {
			"type": "floatSum",
			"name": "dl_rev_q_e_peak",
			"fieldName": "dl_rev_q_e_peak",
			"expression": null
		}, {
			"type": "floatSum",
			"name": "dl_rev_q_e_flat",
			"fieldName": "dl_rev_q_e_flat",
			"expression": null
		}, {
			"type": "floatSum",
			"name": "dl_rev_q_e_valley",
			"fieldName": "dl_rev_q_e_valley",
			"expression": null
		}, {
			"type": "floatSum",
			"name": "dl_rev_q_e_tine",
			"fieldName": "dl_rev_q_e_tine",
			"expression": null
		}],
		"granularitySpec": {
			"type": "uniform",
			"segmentGranularity": "HOUR",
			"queryGranularity": {
				"type": "none"
			},
			"rollup": false,
			"intervals": null
		},
		"transformSpec": {
			"filter": null,
			"transforms": []
		}
	},
	"tuningConfig": {
		"type": "kafka",
		"maxRowsInMemory": 1000000,
		"maxBytesInMemory": 0,
		"maxRowsPerSegment": 5000000,
		"maxTotalRows": null,
		"intermediatePersistPeriod": "PT10M",
		"basePersistDirectory": "/opt/druid-0.14.0/var/tmp/1557739167741-0",
		"maxPendingPersists": 0,
		"indexSpec": {
			"bitmap": {
				"type": "concise"
			},
			"dimensionCompression": "lz4",
			"metricCompression": "lz4",
			"longEncoding": "longs"
		},
		"buildV9Directly": true,
		"reportParseExceptions": false,
		"handoffConditionTimeout": 0,
		"resetOffsetAutomatically": false,
		"segmentWriteOutMediumFactory": null,
		"workerThreads": null,
		"chatThreads": null,
		"chatRetries": 8,
		"httpTimeout": "PT10S",
		"shutdownTimeout": "PT80S",
		"offsetFetchPeriod": "PT30S",
		"intermediateHandoffPeriod": "P2147483647D",
		"logParseExceptions": false,
		"maxParseExceptions": 2147483647,
		"maxSavedParseExceptions": 0,
		"skipSequenceNumberAvailabilityCheck": false
	},
	"ioConfig": {
		"topic": "DRUID-TOPIC-DL-F-101",
		"replicas": 1,
		"taskCount": 1,
		"taskDuration": "PT600S",
		"consumerProperties": {
			"bootstrap.servers": "cdh4:9092"
		},
		"pollTimeout": 100,
		"startDelay": "PT5S",
		"period": "PT30S",
		"useEarliestOffset": true,
		"completionTimeout": "PT1200S",
		"lateMessageRejectionPeriod": null,
		"earlyMessageRejectionPeriod": null,
		"skipOffsetGaps": false,
		"stream": "DRUID-TOPIC-DL-F-101",
		"useEarliestSequenceNumber": true
	},
	"context": null,
	"suspended": false
}
```

## 获取数据源的元数据（数据源的大小、条数）
接口地址：http://druid_router_host:8888/druid/v2/sql  
示例地址：http://10.92.208.219:8888/druid/v2/sql  
请求方式：POST 

示例：
curl -X POST 'http://10.92.208.219:8888/druid/v2/sql' -H 'Content-Type:application/json' -H 'Accept:application/json' -d @metadata.json

metadata.json：
```
{
	"query": "SELECT\n  datasource,\n  COUNT(*) AS num_segments,\n  SUM(is_available) AS num_available_segments,\n  SUM(\"size\") AS size,\n  SUM(\"num_rows\") AS num_rows\nFROM sys.segments\nGROUP BY 1"
}
```

返回值：
```
[{"datasource":"DRUID-TOPIC-DL-F-103","num_segments":64,"num_available_segments":64,"size":7041977,"num_rows":29052},{"datasource":"wikipedia","num_segments":1,"num_available_segments":1,"size":4821529,"num_rows":39244},{"datasource":"DRUID-TOPIC-DL-F-102","num_segments":162,"num_available_segments":162,"size":7886089,"num_rows":29052}]
```

## 获取标签的值
接口地址：http://druid_router_host:8888/druid/v2/sql  
示例地址：http://10.92.208.219:8888/druid/v2/sql  
请求方式：POST 

示例：当标签是city时，获取city的可选值  
curl -X POST 'http://10.92.208.219:8888/druid/v2/sql' -H 'Content-Type:application/json' -H 'Accept:application/json' -d @tags_value.json  


tags_value.json:  
```
{
	"query": "select distinct city from \"DRUID-TOPIC-DL-F-102\""
}
```

返回值：
```
[{"city":"CZ"},{"city":"DG"},{"city":"FS"},{"city":"GZ"},{"city":"HY"},{"city":"HZ"},{"city":"JM"},{"city":"JY"},{"city":"MM"},{"city":"MZ"},{"city":"QY"},{"city":"SG"},{"city":"ST"},{"city":"SW"},{"city":"YF"},{"city":"YJ"},{"city":"ZH"},{"city":"ZJ"},{"city":"ZQ"},{"city":"ZS"}]
```

## 获取tier的可选值
接口地址：http://druid_router_host:8888/druid/v2/sql  
示例地址：http://10.92.208.219:8888/druid/v2/sql  
请求方式：POST 

示例:  
curl -X POST 'http://10.92.208.219:8888/druid/v2/sql' -H 'Content-Type:application/json' -H 'Accept:application/json' -d @get_tier.json -u druid_system:changeme

get_tier.json:

```
{
    "query": "select tier from sys.servers where server_type='historical'"
}
```

返回值：
```
[{"tier":"_default_tier"}]
```


## 数据时效中文说明
Retention: 数据时效策略  
loadByPeriod: 按时间期限加载  
loadByInteval: 按时间范围加载  
loadForever: 永久加载  
dropByPeriod: 按时间期限删除  
dropByInteval: 按时间范围删除  
dropForever: 永久删除  

默认策略：loadForever（永久加载）

Period格式：  
P[n]Y[n]M[n]D,  年月日  
PT[n]H[n]M[n]S，时分秒
            
            
            
## 获取活动的supervisor列表
接口地址：http://druid_router_host:8888/druid/indexer/v1/supervisor  
请求方式：GET  
示例：curl -X GET http://localhost:8888/druid/indexer/v1/supervisor
返回值：
```
["DRUID-TOPIC-ROLLUP-T-G","DRUID-TOPIC-ROLLUP-T-G-A","DRUID-TOPIC-ROLLUP-F-01","Test-BaseDataTable-Day03-1","DRUID-TOPIC-DL-F-101","DRUID-TOPIC-ROLLUP-T","DRUID-TOPIC-ROLLUP-F"]
```



## 数据源失效(disable datasource)

1. 判断是否是流式数据源（通过获取活动的supervisor列表，数据源在此列表中则代表该数据源是流式数据源），如果是则执行以下操作终止supervisor，如果不是则跳到步骤2  
**终止supervior**:   
接口地址：http://druid_router_host:8888/druid/indexer/v1/supervisor/[datasource]/terminate  
请求方式：POST  
示例：  
curl -X POST http://localhost:8888/druid/indexer/v1/supervisor/++DRUID-TOPIC-ROLLUP-F-PAR-01++/terminate  
正常返回值：  
```
{"id":"DRUID-TOPIC-ROLLUP-F-PAR-01"}
```
终止supervisor,会把磁盘缓存文件删除，保留深度存储的数据文件

2. Disable datasource:  
接口地址：http://druid_router_host:8888/druid/coordinator/v1/datasources/{datasource}  
请求方式：DELETE  
示例：  
curl -X DELETE http://localhost:8888/druid/coordinator/v1/datasources/++DRUID-TOPIC-ROLLUP-F-PAR-01++


## 数据源生效（enable datasource）
接口地址：http://druid_router_host:8888/druid/coordinator/v1/datasources/[datasource]
请求方式：POST
示例：  
curl -X POST http://localhost:8888/druid/coordinator/v1/datasources/++DRUID-TOPIC-ROLLUP-F-02++

## 删除数据源
1. 先判断数据源是否已失效，如果还是生效状态，数据源不能执行删除操作。  
如何判断数据源是否已失效：首先获取生效的数据源列表，如果数据源不在此列表中代表该数据源已失效。（获取生效的数据源列表方法请在本篇文章中搜索关键字“数据源列表获取”）


2. 执行以下操作删除数据源  
接口地址：http://druid_router_host:8888/druid/indexer/v1/task  
请求方式：POST  
示例：  
curl -X POST -H 'Content-Type:application/json' -d @deletion-kill-DRUID-TOPIC-ROLLUP-F-PAR.json http://localhost:8888/druid/indexer/v1/task

deletion-kill-DRUID-TOPIC-ROLLUP-F-PAR.json文件内容：
```
{
	"type": "kill",
	"id": "kill_DRUID-TOPIC-ROLLUP-F-PAR",
	"dataSource": "DRUID-TOPIC-ROLLUP-F-PAR",
	"interval": "1000-01-01T00:00:00.000Z/3000-01-01T00:00:00.000Z"
}
```
正常返回值：
```
{"task":"kill_DRUID-TOPIC-ROLLUP-F-PAR"}
```

## supervisor管理
接口地址：http://druid_router_host:8888/druid/indexer/v1/supervisor  
示例：
### 暂停supervisor
请求方式：POST  
curl -X POST http://localhost:8888/druid/indexer/v1/supervisor/++DRUID-TOPIC-ROLLUP-F-PAR++/suspend

### 恢复supervisor
请求方式：POST  
curl -X POST http://localhost:8888/druid/indexer/v1/supervisor/++DRUID-TOPIC-ROLLUP-F-PAR++/resume

### 重置supervisor
请求方式：POST  
curl -X POST http://localhost:8888/druid/indexer/v1/supervisor/++DRUID-TOPIC-ROLLUP-F-PAR++/reset

### 终止supervisor
请求方式：POST  
curl -X POST http://localhost:8888/druid/indexer/v1/supervisor/++DRUID-TOPIC-ROLLUP-F-PAR++/terminate


### supervisor的状态
请求方式：GET  
curl -X GET http://localhost:8888/druid/indexer/v1/supervisor/++DRUID-TOPIC-ROLLUP-F-PAR++/status

