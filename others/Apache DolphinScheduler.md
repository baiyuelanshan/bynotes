## 单机安装
```shell
axel -n 20 https://downloads.apache.org/dolphinscheduler/3.1.2/apache-dolphinscheduler-3.1.2-bin.tar.gz
tar xvf apache-dolphinscheduler-3.1.2-bin.tar.gz
export JAVA_HOME=/home/ubuntu/jdk1.8.0_202

# 启动
bash ./bin/dolphinscheduler-daemon.sh start standalone-server
```

## Web UI 
```shell
http://192.168.56.104:12345/dolphinscheduler/ui/login
# 若在本地安装，ip也可以用127.0.0.1

```

默认的用户名和密码： **admin/dolphinscheduler123**

创建租户 -> 创建用户 -> 创建项目 -> 创建工作流 -> 上线工作流 -> 配置定时调度 -> 定时器上线

