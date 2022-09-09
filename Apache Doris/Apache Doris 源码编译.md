
拉取 docker 镜像
```shell
sudo docker pull apache/incubator-doris:build-env-ldb-toolchain-latest
```

进入 docker 容器
```shell
sudo docker run -it apache/incubator-doris:build-env-ldb-toolchain-latest
```

选择容器的 Java 版本，本文选用 JDK 1.8
```shell
[root@c3050d8400d8 output]# alternatives --set java java-1.8.0-openjdk.x86_64
[root@c3050d8400d8 output]# alternatives --set javac java-1.8.0-openjdk.x86_64
[root@c3050d8400d8 output]# export JAVA_HOME=/usr/lib/jvm/java-1.8.0
[root@c3050d8400d8 output]#
[root@c3050d8400d8 output]# java -version
openjdk version "1.8.0_322"
OpenJDK Runtime Environment (build 1.8.0_322-b06)
OpenJDK 64-Bit Server VM (build 25.322-b06, mixed mode)
[root@c3050d8400d8 output]#
```

下载 Doris 的源码
```shell
git clone https://github.com/apache/incubator-doris.git
```

开始编译
```shell
cd incubator-doris
sh build.sh --clean --be --fe --ui
```

编译完成，在 output 目录下生成以下目录： 
```shell
[root@c3050d8400d8 incubator-doris]# cd output
[root@c3050d8400d8 output]# ls -l
total 12
drwxr-xr-x 8 root root 4096 Feb 18 04:22 be
drwxr-xr-x 9 root root 4096 Feb 18 04:21 fe
drwxr-xr-x 4 root root 4096 Feb 18 04:21 udf
[root@c3050d8400d8 output]# ls be
bin  conf  lib  log  storage  www
[root@c3050d8400d8 output]# ls fe
bin  conf  doris-meta  lib  log  spark-dpp  webroot
[root@c3050d8400d8 output]# ls udf
include  lib
[root@c3050d8400d8 output]#
```

Docker 容器内的系统版本是 Centos 7.9, JDK 1.8， 可以将 output 目录拷贝到相同环境使用。