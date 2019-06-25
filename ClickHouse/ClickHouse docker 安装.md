参考：  
https://hub.docker.com/r/yandex/clickhouse-server/

拉取容器镜像：  
```
docker pull yandex/clickhouse-server
```

启动容器clickhouse-server：  
```
docker run -d -p 9000:9000 -p 8123:8123 --name some-clickhouse-server --ulimit nofile=262144:262144 --volume=/Users/mac/Documents/ClickHouse/docker/some_clickhouse_database:/var/lib/clickhouse yandex/clickhouse-server 
```

启动容器clickhouse-client：  
```
docker run -it --rm --link some-clickhouse-server:clickhouse-server yandex/clickhouse-client --host clickhouse-server
```

进入容器：  
```
docker exec -it e10e27dc7634 /bin/bash
```

