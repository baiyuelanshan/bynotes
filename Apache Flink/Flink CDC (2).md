## 什么时候需要源码编译
一般来说，源码编译是不需要的，用户可以直接在 Flink CDC 官网下载官方编译好的二进制包或者在 pom.xml 文件中添加相关依赖即可。
以下几种情况需要进行源码编译：

 1. 用户对Flink CDC 源码进行了修改    
 2. Flink CDC 某依赖项的版本与运行环境不一致
 3. 官方未提供最新版本 Flink CDC 二进制安装包

比如，官方最新的 Flink CDC 二进制安装包是2.1版本的，而源代码已经到2.2版本了，如果想要使用2.2版本的 Flink CDC， 那么就需要自行编译了。

下面将介绍 Flink CDC  2.2 版本的编译。

## 系统环境
Ubuntu 20.04
JDK 1.8
Maven 3.6.3


## 下载源码
```shell
$ git clone https://github.com/ververica/flink-cdc-connectors.git
Cloning into 'flink-cdc-connectors'...
remote: Enumerating objects: 7187, done.
remote: Counting objects: 100% (7186/7186), done.
remote: Compressing objects: 100% (2504/2504), done.
remote: Total 7187 (delta 2777), reused 7001 (delta 2690), pack-reused 1
Receiving objects: 100% (7187/7187), 10.47 MiB | 9.16 MiB/s, done.
Resolving deltas: 100% (2777/2777), done.
```

## 修改 pom.xml
在 pom.xml 中找到这一项：flink.version
修改 flink 版本号为：
```xml
<flink.version>1.13.6</flink.version>
```

## 编译
```shell
cd /opt/flink-cdc-connectors
mvn clean package -DskipTests

[INFO] Reactor Summary for flink-cdc-connectors 2.2-SNAPSHOT:
[INFO]
[INFO] flink-cdc-connectors ............................... SUCCESS [  1.666 s]
[INFO] flink-connector-debezium ........................... SUCCESS [  2.733 s]
[INFO] flink-connector-test-util .......................... SUCCESS [  0.623 s]
[INFO] flink-connector-mysql-cdc .......................... SUCCESS [  3.039 s]
[INFO] flink-connector-postgres-cdc ....................... SUCCESS [  0.718 s]
[INFO] flink-connector-oracle-cdc ......................... SUCCESS [  0.889 s]
[INFO] flink-connector-mongodb-cdc ........................ SUCCESS [  0.785 s]
[INFO] flink-connector-sqlserver-cdc ...................... SUCCESS [  0.576 s]
[INFO] flink-sql-connector-mysql-cdc ...................... SUCCESS [ 32.567 s]
[INFO] flink-sql-connector-postgres-cdc ................... SUCCESS [  2.368 s]
[INFO] flink-sql-connector-mongodb-cdc .................... SUCCESS [  3.245 s]
[INFO] flink-sql-connector-oracle-cdc ..................... SUCCESS [  3.290 s]
[INFO] flink-sql-connector-sqlserver-cdc .................. SUCCESS [  2.087 s]
[INFO] flink-format-changelog-json ........................ SUCCESS [  1.696 s]
[INFO] flink-cdc-e2e-tests ................................ SUCCESS [ 15.605 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  01:12 min
[INFO] Finished at: 2022-02-20T22:00:35+08:00
[INFO] ------------------------------------------------------------------------
```

如果 maven 下载速度慢，可以在 pom.xml 文件加入这一段
```xml
    <repositories>
        <repository>
            <id>tbds</id>
            <url>https://maven.aliyun.com/repository/public</url>
            <snapshots>
                <enabled>true</enabled>
                <updatePolicy>always</updatePolicy>
            </snapshots>
            <releases>
                <enabled>true</enabled>
                <updatePolicy>always</updatePolicy>
            </releases>
        </repository>
    </repositories>
```
