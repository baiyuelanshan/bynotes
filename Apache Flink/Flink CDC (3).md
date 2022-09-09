## 系统环境
Ubuntu 20.04
JDK 1.8
Maven 3.6.3


## MySQL 测试数据准备
```sql

mysql> CREATE DATABASE mydb;

mysql> USE mydb;

mysql> CREATE TABLE products (
       id INTEGER NOT NULL AUTO_INCREMENT PRIMARY KEY,
       name VARCHAR(255) NOT NULL,
       description VARCHAR(512)
     );

mysql> ALTER TABLE products AUTO_INCREMENT = 101;

mysql> INSERT INTO products
     VALUES (default,"scooter","Small 2-wheel scooter"),
            (default,"car battery","12V car battery"),
            (default,"12-pack drill bits","12-pack of drill bits with sizes ranging from #40 to #3"),
            (default,"hammer","12oz carpenter's hammer"),
            (default,"hammer","14oz carpenter's hammer"),
            (default,"hammer","16oz carpenter's hammer"),
            (default,"rocks","box of assorted rocks"),
            (default,"jacket","water resistent black wind breaker"),
            (default,"spare tire","24 inch spare tire");
```

## Flink CDC 编译
参考文章《Flink CDC 系列（2）—— Flink CDC 源码编译》
https://blog.csdn.net/weixin_47298890/article/details/123029835

## Flink 集群准备
```shell
##  下载 flink 1.13.6 的二进制安装包
axel -n 20 https://archive.apache.org/dist/flink/flink-1.13.6/flink-1.13.6-bin-scala_2.11.tgz

## 解压
tar xvf flink-1.13.6-bin-scala_2.11.tgz

## 将 flink-sql-connector-mysql-cdc-2.2-SNAPSHOT.jar 拷贝到 flink lib 目录下
cp /opt/flink-cdc-connectors/flink-sql-connector-mysql-cdc/target/flink-sql-connector-mysql-cdc-2.2-SNAPSHOT.jar /opt/flink-1.13.6/lib

## 启动单机集群
cd flink-1.13.6
bin/start-cluster.sh

## 查看 jobmanager 和 taskmanager 的进程是否存活
jps -m 
## 正常情况会出现两个进程，如下：
$ jps -m
67440 StandaloneSessionClusterEntrypoint --configDir /opt/flink-1.13.6/conf --executionMode cluster -D jobmanager.memory.off-heap.size=134217728b -D jobmanager.memory.jvm-overhead.min=201326592b -D jobmanager.memory.jvm-metaspace.size=268435456b -D jobmanager.memory.heap.size=1073741824b -D jobmanager.memory.jvm-overhead.max=201326592b
68054 Jps -m
67705 TaskManagerRunner --configDir /opt/flink-1.13.6/conf -D taskmanager.memory.network.min=134217730b -D taskmanager.cpu.cores=1.0 -D taskmanager.memory.task.off-heap.size=0b -D taskmanager.memory.jvm-metaspace.size=268435456b -D external-resources=none -D taskmanager.memory.jvm-overhead.min=201326592b -D taskmanager.memory.framework.off-heap.size=134217728b -D taskmanager.memory.network.max=134217730b -D taskmanager.memory.framework.heap.size=134217728b -D taskmanager.memory.managed.size=536870920b -D taskmanager.memory.task.heap.size=402653174b -D taskmanager.numberOfTaskSlots=1 -D taskmanager.memory.jvm-overhead.max=201326592b

```

## 测试 
建议启动两个命令行窗口，一个运行 Flink SQL Client , 另一个运行 MySQL Client。

#### 1.  启动 Flink SQL Client  
```
cd /opt/flink-1.13.6
bin/sql-client.sh
```

#### 2.  在 Flink SQL Client  中执行


```sql 
    CREATE TABLE products (
     id INT,
     name STRING,
     description STRING,
     PRIMARY KEY (id) NOT ENFORCED
   ) WITH (
     'connector' = 'mysql-cdc',
     'hostname' = '192.168.64.6',
     'port' = '3306',
     'username' = 'test',
     'password' = 'test',
     'database-name' = 'mydb',
     'table-name' = 'products'
   );

select count(1) from products;
-- 结果为9
-- 不退出，继续下一步
```

#### 3. 在 MySQL 客户端继续插入数据
```
INSERT INTO products VALUES (default,"scooter1","Small 2-wheel scooter");
INSERT INTO products VALUES (default,"scooter2","Small 2-wheel scooter");
INSERT INTO products VALUES (default,"scooter3","Small 2-wheel scooter");
INSERT INTO products VALUES (default,"scooter4","Small 2-wheel scooter");
```
#### 4. Flink SQL Client
观察 Flink SQL Client 窗口的数值变化，此时数值应为 13。



