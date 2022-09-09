## 下载 Flink 1.15.0
https://dlcdn.apache.org/flink/flink-1.15.0/flink-1.15.0-bin-scala_2.12.tgz
```bash
cd /opt
# 解压
tar xvf flink-1.15.0-bin-scala_2.12.tgz
FLINK_HOME=/opt/flink-1.15.0

```

## 拉取 Flink table store 代码并编译
```bash
cd /opt
# 拉取代码
git clone https://github.com/apache/flink-table-store.git

# 编译
cd flink-table-store
mvn clean install -DskipTests

# 把编译好的jar包拷贝到flink lib目录下
cp ./flink-table-store-dist/target/flink-table-store-dist-*.jar ${FLINK_HOME}/lib/

```

## 下载 Hadoop Bundle Jar
下载链接：
https://repo.maven.apache.org/maven2/org/apache/flink/flink-shaded-hadoop-2-uber/2.8.3-10.0/flink-shaded-hadoop-2-uber-2.8.3-10.0.jar

```bash
# 把下载好的hadoop jar包拷贝到flink lib目录下
cp flink-shaded-hadoop-2-uber-2.8.3-10.0.jar ${FLINK_HOME}/lib/
```

## 启动 Flink 本地集群
修改以下配置，使得集群可以启动多个作业
```bash
vi $FLINK_HOME/conf/flink-conf.yaml
taskmanager.numberOfTaskSlots: 2
```

启动命令
```bash
$FLINK_HOME/bin/start-cluster.sh
```
浏览器打开 localhost:8081 访问Flink UI 

启动SQL命令行客户端
```bash
$FLINK_HOME/bin/sql-client.sh embedded
```


## 创建 Dynamic Table
```sql

-- set root path to session config
SET 'table-store.root-path' = '/tmp/table_store';

-- create a word count dynamic table without 'connector' option
CREATE TABLE word_count (
    word STRING PRIMARY KEY NOT ENFORCED,
    cnt BIGINT
);

```


## 写数据
```sql
-- create a word data generator table
CREATE TABLE word_table (
    word STRING
) WITH (
    'connector' = 'datagen',
    'fields.word.length' = '1'
);

-- table store requires checkpoint interval in streaming mode
SET 'execution.checkpointing.interval' = '10 s';

-- write streaming data to dynamic table
INSERT INTO word_count SELECT word, COUNT(*) FROM word_table GROUP BY word;
```

## OLAP 查询
```sql
-- use tableau result mode
SET 'sql-client.execution.result-mode' = 'tableau';

-- switch to batch mode
RESET 'execution.checkpointing.interval';
SET 'execution.runtime-mode' = 'batch';

-- olap query the table
SELECT * FROM word_count;

+------+-------+
| word |   cnt |
+------+-------+
|    0 | 90920 |
|    1 | 91072 |
|    2 | 91028 |
|    3 | 91144 |
|    4 | 91428 |
|    5 | 91586 |
|    6 | 91768 |
|    7 | 91273 |
|    8 | 90926 |
|    9 | 91277 |
|    a | 91064 |
|    b | 91427 |
|    c | 91319 |
|    d | 91163 |
|    e | 91162 |
|    f | 91443 |
+------+-------+
16 rows in set

```

## Streaming 查询

```sql
-- switch to streaming mode
SET 'execution.runtime-mode' = 'streaming';

-- track the changes of table 
Flink SQL> select * from word_count where word = 'a';
+----+--------------------------------+----------------------+
| op |                           word |                  cnt |
+----+--------------------------------+----------------------+
| +I |                              a |               403513 |
| -U |                              a |               403513 |
| +U |                              a |               409738 |
| -U |                              a |               409738 |
| +U |                              a |               416128 |

```