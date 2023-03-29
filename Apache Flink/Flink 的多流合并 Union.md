
## 1. 简介

当Flink多条流的下游需要使用相同的逻辑处理数据时，可以把多流合并起来，以简化代码，更优雅。

需要注意的事项：
* 多条流必须使用的相同的Schema，譬如相同的Pojo类，或者通用的JSONObject


## 2. 使用案例

商城的流量数据有曝光、点击、加购，都是以Json格式存储，需要计算曝光人数、点击人数、加购人数。

在不使用多流合并的情况下，需要使用代码分别解析Json和计算：
```java
//伪代码
DataStream<Result> exposureUserCount = env.addSource()
.map() //解析Json
.process() //计算曝光人数
;

DataStream<Result> clickUserCount = env.addSource()
.map() //解析Json
.process() //计算点击人数
;

DataStream<Result> addCartUserCount = env.addSource()
.map() //解析Json
.process() //计算加购人数
;
```

使用多流合并：
```java
//伪代码
DataStream<Result> count = env.addSource()//曝光
.union(env.addSource()//点击,
	   env.addSource()//加购
	   )
.map() //解析Json
.keyBy() //按流量类型曝光、点击、加购分组
.process() //计算各类型的人数
;
```

## 3. 代码实现

#### 3.1 Source类
```java
package com.test.realtime.dw.dtl.app.test;  
  
import org.apache.flink.streaming.api.functions.source.SourceFunction;  
  
public class LogSource implements SourceFunction<String> {  
  
    private boolean running = true;  
    private String template = "{\"event\":\"event_type\",\"user_id\":\"user_id_num\"}";  
  
    private String eventType = null;  
  
    public LogSource(String eventType) {  
        this.eventType = eventType;  
    }  
  
    @Override  
    public void run(SourceContext<String> sourceContext) throws Exception {  
        long userId = 10000L;  
        while (running){  
            userId +=1;  
            Thread.sleep(1000);  
            sourceContext.collect(template.replace("event_type", this.eventType).replace("user_id_num", String.valueOf(userId)));  
        }  
    }  
    @Override  
    public void cancel() {  
        running = false;  
    }  
}
```


#### 3.2 主类

```java
package com.test.realtime.dw.dtl.app.test;  
  
import com.alibaba.fastjson.JSON;  
import com.alibaba.fastjson.JSONObject;  
import org.apache.flink.api.common.functions.MapFunction;  
import org.apache.flink.api.java.tuple.Tuple2;  
import org.apache.flink.configuration.Configuration;  
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;  
  
  
public class UserBehaviorCount {  
    public static void main(String[] args) throws Exception {  
        Configuration configuration = new Configuration();  
        configuration.setString("rest.port","9091"); //指定 Flink Web UI 端口为9091  
        StreamExecutionEnvironment env = StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(configuration);  
        env.enableCheckpointing(10000);  
          
        env.addSource(new LogSource("exposure"))  
                .union(env.addSource(new LogSource("click")))  
                .union(env.addSource(new LogSource("add_cart")))  
                .map(new MapFunction<String, Tuple2<String,Integer>>() {  
                    @Override  
                    public Tuple2<String,Integer> map(String value) throws Exception {  
                        JSONObject jsonObject = JSON.parseObject(value);  
                        return Tuple2.of(jsonObject.getString("event"), 1);  
                    }  
                })  
                .keyBy(0)  
                .sum(1)  
                .print();  
  
        env.execute();  
    }  
  
}
```


#### 3.3 执行效果

![[Pasted image 20230311181112.png]]

#### 3.4 Flink作业DAG图对比


未使用Union的DAG图


![[Pasted image 20230311181650.png]]

使用Union后的DAG图
![[Pasted image 20230311181300.png]]


