## 系统准备

#### 系统版本
Ubuntu系统或者Centos系统皆可， 本文采用Ubuntu 18.04.6 LTS(下载地址：https://releases.ubuntu.com/bionic/)。


#### 配置系统参数

```shell
echo "0" > /proc/sys/vm/swappiness
echo "0" > /proc/sys/vm/overcommit_memory
sysctl -p
```

修改/etc/security/limits.conf， 在文件末尾追加以下配置
```shell
*            soft     core            65535
*            hard     core            65535
*            hard     nproc           65535
*            soft     nproc           65535
*            hard     nofile          65535
*            soft     nofile          65535
```

#### 升级 Linux kernel
1）下载ubuntu-mainline-kernel.sh，用于在线升级kernel
```shell
wget https://raw.githubusercontent.com/pimlie/ubuntu-mainline-kernel.sh/master/ubuntu-mainline-kernel.sh
```
2)  把脚本放在可执行路径
```shell
sudo install ubuntu-mainline-kernel.sh /usr/local/bin/
```
3）安装5.4.120版本的kernel
```shell
sudo ubuntu-mainline-kernel.sh -i v5.4.120
```
4) 重启系统
```shell 
reboot
```
重新进入系统后，查看当前的kernel版本为 5.4.120-0504120-generic
```shell
uname -r
5.4.120-0504120-generic
```

#### 安装jdk1.8 或者 jdk11
笔者安装jdk1.8, 具体安装方法参考百度或者谷歌

#### 安装mysql客户端
```shell 
sudo apt install mysql-client
```



## 下载 Apache Doris 安装包
https://doris.apache.org/download/

1)  运行以下命令，用于查看是否支持avx2指令集，返回0，说明不支持
```
cat /proc/cpuinfo | grep avx2 | wc -l
```

2）根据JDK版本，CPU架构，是否支持avx2，下载对应的Doris安装包 
![[Pasted image 20220907154653.png]]
3) 解压缩
```shell
tar xvf apache-doris-1.1.1-bin-x86-noavx2.tar.gz
```
4) 创建软链接
```shell 
ln -s apache-doris-1.1.1-bin-x86-noavx2 apache-doris
```

## 配置和启动FE 

1) 配置fe
```shell
cd apache-doris/fe
```
在配置文件 conf/fe.conf 添加priority_networks 参数
priority_networks = 192.168.56.104/24

2) 启动fe 
```shell
./bin/start_fe.sh --daemon
```

3) 进入mysql客户端查看FE的状态
```shell
mysql -uroot -P9030 -h127.0.0.1
```

执行以下SQL命令查看FE 运行状态
```sql
show frontends\G;

```
可以看到类似结果：
```sql

mysql> show frontends\G;
*************************** 1. row ***************************
             Name: 192.168.56.104_9010_1662530252128
               IP: 192.168.56.104
      EditLogPort: 9010
         HttpPort: 8030
        QueryPort: 9030
          RpcPort: 9020
             Role: FOLLOWER
         IsMaster: true
        ClusterId: 1310365983
             Join: true
            Alive: true
ReplayedJournalId: 2622
    LastHeartbeat: 2022-09-07 08:25:25
         IsHelper: true
           ErrMsg:
          Version: 1.1.1-rc03-2dbd70bf9
 CurrentConnected: Yes
1 row in set (0.03 sec)
```

Role : 表示你的节点角色，如果你只有一个 FE 的时候，当前节点角色是 Follower，
IsMaster：该值为true，说明这个节点是 FE 的主节点
alive ：该值为 true 说明该节点运行正常



## 配置和启动BE

1) 配置be 
```shell 
cd apache-doris/be
```
在配置文件 conf/be.conf 添加以下参数：

>priority_networks = 192.168.56.104/24
   enable_stream_load_record = true

2) 启动be
```shell 
./bin/start_be.sh --daemon
```

3) 进入mysql客户端，添加BE节点并查看状态
```shell
mysql -uroot -P9030 -h127.0.0.1
```

执行以下SQL命令添加BE节点到集群
```shell
ALTER SYSTEM ADD BACKEND "be_host_ip:heartbeat_service_port";
```


be_host_ip: BE节点的IP, be.conf 配置的priority_networks 参数
heartbeat_service_port： BE 的心跳上报端口， be.conf 的heartbeat_service_port参数
如：
```sql
ALTER SYSTEM ADD BACKEND "192.168.56.104:9050"
```


查看BE节点状态
```
SHOW BACKENDS
```
可以看到类似结果：
```sql
SHOW BACKENDS\G
*************************** 1. row ***************************
            BackendId: 10002
              Cluster: default_cluster
                   IP: 192.168.56.104
        HeartbeatPort: 9050
               BePort: 9060
             HttpPort: 8040
             BrpcPort: 8060
        LastStartTime: 2022-09-07 06:38:28
        LastHeartbeat: 2022-09-07 08:29:46
                Alive: true
 SystemDecommissioned: false
ClusterDecommissioned: false
            TabletNum: 0
     DataUsedCapacity: 0.000
        AvailCapacity: 47.498 GB
        TotalCapacity: 58.316 GB
              UsedPct: 18.55 %
       MaxDiskUsedPct: 18.55 %
                  Tag: {"location" : "default"}
               ErrMsg:
              Version: 1.1.1-rc03-2dbd70bf9
               Status: {"lastSuccessReportTabletsTime":"2022-09-07 08:28:47","lastStreamLoadTime":-1,"isQueryDisabled":false,"isLoadDisabled":false}
1 row in set (0.00 sec)
```

Alive : true表示节点运行正常
SystemDecommissioned：false 表示节点没有执行下线，如果执行下线操作，这里显示的是true