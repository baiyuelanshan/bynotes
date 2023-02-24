## 1. 背景

Flink SQL1.10 没有collect_list函数，可以通过自定义函数的方式实现。
文章最后介绍自定义函数的泛型化。

## 2. 实现CollectList函数

```java
package com.yishou.realtime.dw.dtl.operator;   
import org.apache.flink.table.functions.AggregateFunction;  
import java.util.ArrayList;  
import java.util.List;  
  
public class CollectList extends AggregateFunction<String, List<String>> {  
  
    private static String separator = "_";  
    public CollectList(String separator) {  
        this.separator = separator;  
    }  
  
    public void retract(List acc , String column){  
        acc.remove(column);  
    }  
  
    public void accumulate(List acc, String column){  
        acc.add(column);  
    }  
  
    @Override  
    public String getValue(List<String> list) {  
        return String.join(this.separator, list);  
    }  
  
    @Override  
    public List<String> createAccumulator() {  
        List list = new ArrayList();  
        return list;  
    }  
  
    public void resetAccumulator(List list) {  
        list.clear();  
    }  
  
}

```

函数的功能：
- 创建累加器， 对应代码`createAccumulator`方法
- 接收字符串类型的数据，存到list中，对应代码中的`accumulate`方法
- 把list中的元素转换为字符串输出，元素以`"_"`分隔，用户可以指定其它字符作为分隔符，对应代码中的`getValue`方法
- 当发生回撤时，需删除回撤的元素，对应代码中的`retract`方法
- 重置累加器，对应代码`resetAccumulator`方法

## 3. 注册CollectList函数

在Flink环境中注册CollectList函数
```java
tenv.registerFunction("collect_list", new CollectList("_"));
```


## 4. 完整的测试代码

```java
package com.yishou.realtime.dw.dtl.app;  
  
import com.yishou.realtime.dw.dtl.operator.CollectList;  
import org.apache.flink.api.java.tuple.Tuple3;  
import org.apache.flink.configuration.Configuration;  
import org.apache.flink.streaming.api.datastream.DataStream;  
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;  
import org.apache.flink.streaming.api.functions.source.SourceFunction;  
import org.apache.flink.table.api.EnvironmentSettings;  
import org.apache.flink.table.api.Table;  
import org.apache.flink.table.api.java.StreamTableEnvironment;  
import org.apache.flink.table.functions.AggregateFunction;  
import org.apache.flink.types.Row;  
  
  
public class TestSQLCustomUDF {  
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
                    code = code + 1;  
                    if (code >= 510003){  
                        code = 510000;  
                    }  
                    Thread.sleep(1000);  
                    sourceContext.collect(Tuple3.of(city, code, System.currentTimeMillis()));  
                }  
            }  
  
            @Override  
            public void cancel() {  
                isRunning = false;  
  
            }  
        });  
  
        source.print();  
  
  
        tenv.createTemporaryView("source", source, "city,code,create_time");  
        tenv.registerFunction("collect_list", new CollectList("_"));  
        Table table = tenv.sqlQuery("select \n" +  
                "    city,\n" +  
                "    collect_list(cast(code as String)) as code_list,\n" +  
                "    collect_list(cast(create_time as String)) as create_time_list\n" +  
                "from \n" +  
                "(\n" +  
                "    select \n" +  
                "    city,\n" +  
                "    code,\n" +  
                "    create_time,\n" +  
                "    row_number() over(partition by city order by create_time desc) as ranking\n" +  
                "    from \n" +  
                "    (\n" +  
                "        select \n" +  
                "        city,\n" +  
                "        code,\n" +  
                "        create_time,\n" +  
                "        row_number() over(partition by city,code order by create_time desc) as aranking\n" +  
                "        from source\n" +  
                "    ) r\n" +  
                "    where aranking = 1 \n" +  
                ")\n" +  
                "where ranking <= 3\n" +  
                "group by \n" +  
                "city\n");  
        tenv.toRetractStream(table, Row.class).print();  
        env.execute();  
    }  
  
}
```


![[Pasted image 20230223184611.png]]


## 5. 如何泛化CollectList函数

在本文第二节中实现的CollectList函数只能处理String类型的数据，要处理Integer，Long等类型时需要先转换为String类型才能执行。为了使CollectList函数能够处理更多数据类型的数据，需要实现泛型化，代码如下：
```java
package com.yishou.bigdata.realtime.dw.udf;  
  
import org.apache.flink.api.common.typeinfo.TypeInformation;  
import org.apache.flink.api.common.typeinfo.Types;  
import org.apache.flink.table.functions.AggregateFunction;  
  
import java.util.ArrayList;  
import java.util.List;  
  
public class CollectList<OUT,IN> extends AggregateFunction<OUT, List<IN>> {  
  
    private static String separator = null;  
    public CollectList(String separator) {  
        this.separator = separator;  
  
    }  
  
    public void retract(List acc , IN column){  
        acc.remove(column);  
    }  
  
    public void accumulate(List acc, IN column){  
        acc.add(column);  
    }  
  
    @Override  
    public OUT getValue(List<IN> list) {  
        List<String> result = new ArrayList<>();  
        for(IN item:list){  
            result.add(String.valueOf(item));  
        }  
        return (OUT) String.join(this.separator, result);  
    }  
  
    @Override  
    public List<IN> createAccumulator() {  
        List list = new ArrayList();  
        return list;  
    }  
  
    public void resetAccumulator(List list) {  
        list.clear();  
    }  
  
    @Override  
    public TypeInformation<OUT> getResultType() {  
        return (TypeInformation<OUT>) Types.STRING;  
    }  
  
}
```

