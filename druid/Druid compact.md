compact.json
```
{
  "type": "compact",
  "dataSource": "DRUID-TOPIC-ROLLUP-F-PAR-01",
  "interval": "2019-05-07T03:00:00.000/2019-05-07T04:00:00.000",
  "tuningConfig" : {
    "type" : "index",
    "targetPartitionSize" : 500000,
    "maxRowsInMemory" : 25000,
    "forceExtendableShardSpecs" : true
  }
}
```

curl -X 'POST' -H 'Content-Type:application/json' -d @compact-01.json http://localhost:8090/druid/indexer/v1/task



In this example, the compaction task reads all segments of the interval 2018-06-01/2018-12-01 and merges them into a single interval. To control the number of result segments, you can set targetPartitionSize or numShards. targetPartitionSize specifies total number of rows in segments. numShards directly specifies the number of shards to create. numShards cannot be specified if targetPartitionSize is set. Here, I have set targetPartitionSize to 500000, so that if there is less than 500000 rows in existing data segments, only 1 new segment will be created. Otherwise segments number will grow as multiples of 500000. Alternatively, if you set numShard to 1, only one segment will be created. If it is set to 2, data will be compacted into 2 segments.

