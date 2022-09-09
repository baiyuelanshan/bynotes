#### 下载 Flink 二进制包 并 解压
```shell
axel -n 20 https://downloads.apache.org/flink/flink-1.14.3/flink-1.14.3-bin-scala_2.11.tgz

tar xvf flink-1.14.3-bin-scala_2.11.tgz

```

#### 下载 JDK 8 并解压
```
axel -n 20 https://mirrors.huaweicloud.com/java/jdk/8u202-b08/jdk-8u202-linux-x64.tar.gz

tar xvf jdk-8u202-linux-x64.tar.gz

```

#### 配置环境变量

在文件 /home/ubuntu/.profile 最末尾添加以下内容：
```
JAVA_HOME=/opt/jdk1.8.0_202
PATH=$PATH:${JAVA_HOME}/bin
```

添加完之后，执行：
```shell
$ source /home/ubuntu/.profile
$ java -version
java version "1.8.0_202"
Java(TM) SE Runtime Environment (build 1.8.0_202-b08)
Java HotSpot(TM) 64-Bit Server VM (build 25.202-b08, mixed mode)

```

#### 启动单机集群
```shell
$ bin/start-cluster.sh
Starting cluster.
Starting standalonesession daemon on host ubuntu.
Starting taskexecutor daemon on host ubuntu.
```

http://localhost:8081 可访问 Flink Web UI。

#### 执行自带的例子
```shell
$ bin/flink run ./examples/streaming/TopSpeedWindowing.jar
Executing TopSpeedWindowing example with default input data set.
Use --input to specify file input.
Printing result to stdout. Use --output to specify output path.
Job has been submitted with JobID 012a761874d47d6b8f40c04c95c657e4
```

查看 Flink Web UI:

![[Pasted image 20220218172848.png]]

查看 taskmanager 日志:

![[Pasted image 20220218172929.png]]

#### 停止集群
```shell
$ bin/stop-cluster.sh
Stopping taskexecutor daemon (pid: 24741) on host ubuntu.
Stopping standalonesession daemon (pid: 24481) on host ubuntu.
```

