简介
----
本文介绍 Flink 如何流式写入 Apache Doris，分为一下几个部分：
* Flink Doris connector
* Doris FE 节点配置
* Flink DataStream 写 Doris



Flink Doris connector
----
Flink Doris connector 本质是通过Stream Load来时实现数据的查询和写入功能。
支持二阶段提交，可实现Exatly Once的写入。


Doris FE 节点配置
----
1）需在 apache-doris/fe/fe.conf 配置文件添加如下配置：
```
enable_http_server_v2 = true
```

2) 重启 FE 节点
```
apache-doris/fe/bin/stop_fe.sh
apache-doris/fe/bin/start_fe.sh --daemon

```

Flink DataStream 读写 Doris
----

1) Flink DataStream 读取 Doris

```
        //Doris Source
        DorisOptions.Builder sourceBuilder =
                DorisOptions.builder()
                        .setFenodes("192.168.56.104:8030")  //FE节点IP和端口
                        .setTableIdentifier("test.order_info_example")
                        .setUsername("test")
                        .setPassword("password123");


        DorisSource<List<?>> dorisSource = DorisSourceBuilder.<List<?>>builder()
                .setDorisOptions(sourceBuilder.build())
                .setDorisReadOptions(DorisReadOptions.builder().build())
                .setDeserializer(new SimpleListDeserializationSchema())
                .build();

        DataStreamSource<List<?>> source = env.fromSource(dorisSource, WatermarkStrategy.noWatermarks(), "doris source");

```

2) Flink DataStream 写入 Doris

```
        //Doris Sink
        DorisSink.Builder<String> sinkBuilder = DorisSink.builder();
        DorisOptions.Builder dorisBuilder = DorisOptions.builder();
        dorisBuilder.setFenodes("192.168.56.104:8030")
                .setTableIdentifier("test.order_info_output")
                .setUsername("test")
                .setPassword("password123");

        Properties properties = new Properties();
        properties.setProperty("column_separator", ",");
        DorisExecutionOptions.Builder  executionBuilder = DorisExecutionOptions.builder();
        executionBuilder.setLabelPrefix("label-doris-20") //streamload label prefix
                .setStreamLoadProp(properties);
        sinkBuilder.setDorisReadOptions(DorisReadOptions.builder().build())
                .setDorisExecutionOptions(executionBuilder.build())
                .setDorisOptions(dorisBuilder.build())
                .setSerializer(new SimpleStringSerializer()) //serialize according to string
                 ;


        DataStream<String> transform = source.flatMap(new FlatMapFunction<List<?>, String>() {
            @Override
            public void flatMap(List<?> element, Collector<String> collector) throws Exception {

                //collector.collect();
                StringBuffer stringBuffer = new StringBuffer();
                stringBuffer.append(element.get(0))
                        .append(",")
                        .append(element.get(1))
                        .append(",")
                        .append(element.get(2))
                        .append(",")
                        .append(element.get(3))
                        .append(",")
                        .append(element.get(4))
                        .append(",")
                        .append(element.get(5))
                        ;
                    collector.collect(stringBuffer.toString());

            }
        });


        transform.print();
        transform.sinkTo(sinkBuilder.build());

```

[Github完整代码](https://github.com/baiyuelanshan/apache-doris-example/blob/main/src/main/java/FlinkDataStreamExample.java)