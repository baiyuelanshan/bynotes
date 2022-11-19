#### 建表和数据准备
```sql
CREATE TABLE IF NOT EXISTS test.user_order_agg
(    
    `order_id` INT NOT NULL COMMENT "订单id",  
    `user_id` INT COMMENT "[-9223372036854775808, 9223372036854775807]",  
    `buy_num` int sum COMMENT "总购买件数",  
    `order_time` DATE MAX NOT NULL COMMENT "最后下单时间",
    `create_time` DATETIME MIN COMMENT "最早创建时间"
)  
ENGINE=olap  
AGGREGATE KEY (`order_id`,`user_id`)  
DISTRIBUTED BY HASH(`order_id`) BUCKETS 32  
PROPERTIES (  
 "replication_num" = "1"); 

insert into test.user_order_agg values(10000,9000,2,'2022-11-15 09:00:00','2022-11-15 09:00:00');
insert into test.user_order_agg values(10001,9000,2,'2022-11-15 09:00:00','2022-11-15 09:00:00');
insert into test.user_order_agg values(10000,9000,2,'2022-11-15 09:00:00','2022-11-15 09:00:00');
insert into test.user_order_agg values(10001,9000,2,'2022-11-15 09:00:00','2022-11-15 09:00:00');
```


#### 没有创建Rollup的执行计划
`explain select sum(buy_num),user_id from user_order_agg group by user_id`

```sql
PLAN FRAGMENT 0
  OUTPUT EXPRS:<slot 3> sum(`buy_num`) | <slot 2> `user_id`
  PARTITION: UNPARTITIONED

  VRESULT SINK

  4:VEXCHANGE

PLAN FRAGMENT 1

  PARTITION: HASH_PARTITIONED: <slot 2> `user_id`

  STREAM DATA SINK
    EXCHANGE ID: 04
    UNPARTITIONED

  3:VAGGREGATE (merge finalize)
  |  output: sum(<slot 3> sum(`buy_num`))
  |  group by: <slot 2> `user_id`
  |  cardinality=-1
  |  
  2:VEXCHANGE

PLAN FRAGMENT 2

  PARTITION: HASH_PARTITIONED: `default_cluster:test`.`user_order_agg`.`order_id`

  STREAM DATA SINK
    EXCHANGE ID: 02
    HASH_PARTITIONED: <slot 2> `user_id`

  1:VAGGREGATE (update serialize)
  |  STREAMING
  |  output: sum(`buy_num`)
  |  group by: `user_id`
  |  cardinality=-1
  |  
  0:VOlapScanNode
     TABLE: user_order_agg(user_order_agg), PREAGGREGATION: ON
     partitions=1/1, tablets=32/32, tabletList=70080,70082,70084 ...
     cardinality=3, avgRowSize=4810.0, numNodes=1

```

#### 创建Rollup
```sql
alter table user_order_agg add ROLLUP rollup_user_id(user_id,buy_num)

```

#### 创建Rollup后的执行计划
`explain select sum(buy_num),user_id from user_order_agg group by user_id`

```sql
PLAN FRAGMENT 0
  OUTPUT EXPRS:<slot 3> sum(`buy_num`) | <slot 2> `user_id`
  PARTITION: UNPARTITIONED

  VRESULT SINK

  4:VEXCHANGE

PLAN FRAGMENT 1

  PARTITION: HASH_PARTITIONED: <slot 2> `user_id`

  STREAM DATA SINK
    EXCHANGE ID: 04
    UNPARTITIONED

  3:VAGGREGATE (merge finalize)
  |  output: sum(<slot 3> sum(`buy_num`))
  |  group by: <slot 2> `user_id`
  |  cardinality=-1
  |  
  2:VEXCHANGE

PLAN FRAGMENT 2

  PARTITION: HASH_PARTITIONED: `default_cluster:test`.`user_order_agg`.`order_id`

  STREAM DATA SINK
    EXCHANGE ID: 02
    HASH_PARTITIONED: <slot 2> `user_id`

  1:VAGGREGATE (update serialize)
  |  STREAMING
  |  output: sum(`buy_num`)
  |  group by: `user_id`
  |  cardinality=-1
  |  
  0:VOlapScanNode
     TABLE: user_order_agg(rollup_user_id), PREAGGREGATION: ON
     partitions=1/1, tablets=32/32, tabletList=70005,70007,70009 ...
     cardinality=1, avgRowSize=3220.0, numNodes=1
```

创建Rollup之后VOlapScanNode这一步从`rollup_user_id`取数据

