
## 背景

在Flink SQL中使用ROW_NUMBER去重是一个比较常见的场景，那么这种去重方式是否把所有的历史数据都缓存在状态中导致状态无限增长？

接下来，通过以下实验观察状态的增长来解答以上的问题。


## 实验代码

```java
package com.yishou.realtime.dw.dtl.app;  
  
import org.apache.flink.api.common.time.Time;  
import org.apache.flink.api.java.tuple.Tuple3;  
import org.apache.flink.configuration.Configuration;  
import org.apache.flink.streaming.api.datastream.DataStream;  
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;  
import org.apache.flink.streaming.api.functions.source.SourceFunction;  
import org.apache.flink.table.api.EnvironmentSettings;  
import org.apache.flink.table.api.Table;  
import org.apache.flink.table.api.java.StreamTableEnvironment;  
import org.apache.flink.types.Row;  
  
public class TestSQL {  
  
    public static void main(String[] args) throws Exception {  
        Configuration configuration = new Configuration();  
        configuration.setString("rest.port","9091"); //指定 Flink Web UI 端口为9091  
 
        EnvironmentSettings environmentSettings = EnvironmentSettings.newInstance().useBlinkPlanner().inStreamingMode().build();  
        StreamTableEnvironment tenv = StreamTableEnvironment.create(env, environmentSettings);  
  
        env.enableCheckpointing(10000);  
  
        DataStream<Tuple3<String,Long,Long>> source = env.addSource(new SourceFunction<Tuple3<String,Long,Long>>() {  
            private boolean isRunning = true;  
  
            @Override  
            public void run(SourceContext<Tuple3<String, Long, Long>> sourceContext) throws Exception {  
                String city = "Guangzhou";  
                long code = 510000;  
                while (isRunning){  
                    Thread.sleep(100);  
                    sourceContext.collect(Tuple3.of(city, code, System.currentTimeMillis()));  
                }  
            }  
  
            @Override  
            public void cancel() {  
                isRunning = false;  
  
            }  
        });  
  
        tenv.createTemporaryView("source", source, "city,code,create_time");  
        Table table = tenv.sqlQuery("select \n" +  
                "    city,\n" +  
                "    code,\n" +  
                "    create_time\n" +  
                "from \n" +  
                "(\n" +  
                "    select \n" +  
                "    city,\n" +  
                "    code,\n" +  
                "    create_time,\n" +  
                "    row_number() over(partition by city,code order by create_time desc) as ranking\n" +  
                "    from source\n" +  
                ") r\n" +  
                "where ranking <= 2000\n");  
        tenv.toRetractStream(table, Row.class).print();  
  
        env.execute();  
  
    }  
}
```

本代码基于Flink 1.10， 在本地IDEA启动Flink Web UI添加以下依赖：
```
<dependency>  
    <groupId>org.apache.flink</groupId>  
    <artifactId>flink-runtime-web_${scala.binary.version}</artifactId>  
    <version>${flink.version}</version>  
</dependency>
```

[IDEA 启动本地 Flink Web UI](https://blog.csdn.net/weixin_47298890/article/details/122693931)

## 通过Flink Web UI 观察状态的增长情况

![[Pasted image 20230223163427.png]]


![[Pasted image 20230223163604.png]]


观察一段时间， 发现状态的大小稳定在186KB， 不再增长。状态只保留了最新的2000条数据。

## 结论

Flink SQL使用Row_number去重时，状态中只会保留最新的1或者n条数据，单个主键的状态不会无限增长。 