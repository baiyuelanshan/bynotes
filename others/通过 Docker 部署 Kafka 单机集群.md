#### 准备 yaml 文件

docker-kafka.yaml
```yaml
version: "3.3"
services:
  zookeeper:
    image: 'bitnami/zookeeper:3.4.12-r68'
    hostname: zookeeper
    container_name: zookeeper
    ports:
      - '2181:2181'
    environment:
      - ALLOW_ANONYMOUS_LOGIN=yes
  kafka:
    image: 'bitnami/kafka:2.0.0'
    hostname: kafkabroker
    container_name: kafkabroker
    ports:
      - '9092:9092'
    environment:
      - KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181
      - ALLOW_PLAINTEXT_LISTENER=yes
networks:
  default:
```

#### 执行 docker-compose 命令
拉取 docker 镜像，创建并启动容器

```shell
$ sudo docker-compose -f docker-kafka.yaml up -d
Creating network "kafka_default" with the default driver
Pulling kafka (bitnami/kafka:2.0.0)...
2.0.0: Pulling from bitnami/kafka
476234a6f713: Pulling fs layer
2c88f470210d: Pulling fs layer
9a62d319ca4b: Pulling fs layer
e4dbc20a00f4: Pulling fs layer
476234a6f713: Pull complete
2c88f470210d: Pull complete
9a62d319ca4b: Pull complete
e4dbc20a00f4: Pull complete
dfd6244186fc: Pull complete
4955ef348e8d: Pull complete
887a9c07fc46: Pull complete
540664008e81: Pull complete
b44742fac87e: Pull complete
8e031f4a42ed: Pull complete
a1f69359766c: Pull complete
Digest: sha256:e7b90e93be338381d57991073ea5a493ba20834afa39114d64db5a601fd53827
Status: Downloaded newer image for bitnami/kafka:2.0.0
Creating kafkabroker ... done

$ sudo docker ps
CONTAINER ID   IMAGE                          COMMAND                  CREATED         STATUS         PORTS                                                           NAMES
d48c1fe8d97b   bitnami/kafka:2.0.0            "/app-entrypoint.sh …"   2 minutes ago   Up 2 minutes   0.0.0.0:9092->9092/tcp, :::9092->9092/tcp                       kafkabroker
9399eba0a867   bitnami/zookeeper:3.4.12-r68   "/app-entrypoint.sh …"   2 minutes ago   Up 2 minutes   2888/tcp, 0.0.0.0:2181->2181/tcp, :::2181->2181/tcp, 3888/tcp   zookeeper


```

####  Kafka 操作
本文使用的 kafkacat 工具，可参考：《Kafka 命令行工具 kcat/kafkacat》

**1. 查看元数据**
```shell
$ kafkacat  -b localhost:9092 -L -J | jq
{
  "originating_broker": {
    "id": -1,
    "name": "localhost:9092/bootstrap"
  },
  "query": {
    "topic": "*"
  },
  "controllerid": 1001,
  "brokers": [
    {
      "id": 1001,
      "name": "kafkabroker:9092"
    }
  ],
  "topics": []
}

```

**2. 生产数据**
```shell
$ echo "hello world" | kafkacat -C -b kafkabroker -t syslog

```


**3. 消费数据**
```shell
$ kafkacat -C -b kafkabroker -t syslog
hello world

```