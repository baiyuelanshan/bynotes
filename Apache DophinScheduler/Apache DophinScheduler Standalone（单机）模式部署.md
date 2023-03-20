
1.关于DophinScheduler
---
Apache DolphinScheduler 是一个分布式易扩展的可视化DAG工作流任务调度开源系统。适用于企业级场景，提供了一个可视化操作任务、工作流和全生命周期数据处理过程的解决方案。

Apache DolphinScheduler 旨在解决复杂的大数据任务依赖关系，并为应用程序提供数据和各种 OPS 编排中的关系。 解决数据研发ETL依赖错综复杂，无法监控任务健康状态的问题。 DolphinScheduler 以 DAG（Directed Acyclic Graph，DAG）流式方式组装任务，可以及时监控任务的执行状态，支持重试、指定节点恢复失败、暂停、恢复、终止任务等操作。


2.单机部署模式需知
---
* Standalone 仅适用于 DolphinScheduler 的快速体验
* Standalone仅建议20个以下工作流使用，因为其采用内存式的H2 Database, Zookeeper Testing Server，任务过多可能导致不稳定，并且如果重启或者停止standalone-server会导致内存中数据库里的数据清空

3.前置准备工作
---
* 下载[JDK](https://www.oracle.com/technetwork/java/javase/downloads/index.html)1.8， 并配置`JAVA_HOME` 环境变量， `PATH` 环境变量
* 下载[DophinScheduler](https://dolphinscheduler.apache.org/zh-cn/download)的二进制安装包

4.启动DolphinScheduler Standalone Server
---

#### 4.1 解压并启动DolphinScheduler

```shell
## 解压并运行, 以3.1.2版本为例
tar xvf apache-dolphinscheduler-3.1.2-bin.tar.gz
cd apache-dolphinscheduler-3.1.2-bin
bash ./bin/dolphinscheduler-daemon.sh start standalone-server
```

![[Pasted image 20230317172134.png]]

启动日志可以在这个文件查看：
standalone-server/logs/dolphinscheduler-standalone.log
![[Pasted image 20230317172406.png]]

#### 4.2 登录 DophinScheduler

浏览器访问地址 http://localhost:12345/dolphinscheduler/ui 即可登录系统UI。默认的用户名和密码是 **admin/dolphinscheduler123**

![[Pasted image 20230317172721.png]]


![[Pasted image 20230317172936.png]]

#### 4.3 服务的启停命令

```shell
# 启动 Standalone Server 服务
bash ./bin/dolphinscheduler-daemon.sh start standalone-server
# 停止 Standalone Server 服务
bash ./bin/dolphinscheduler-daemon.sh stop standalone-server
```

