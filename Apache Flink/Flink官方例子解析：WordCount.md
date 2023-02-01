
## 1. 简介

今天介绍的是官方子项目flink-examples-streaming里面的WordCount例子。

![[Pasted image 20230131101106.png]]

WordCount ，中文：单词统计，是大数据计算常用的例子。

## 2. WordCount需要实现的功能

- 监听指定目录下的文件，读取文件的文本内容；如果未指定监听路径，则读取静态的字符串变量
- 分词
- 统计每个单词的出现次数
- 把单词统计的结果输出到指定的文件中；如果未指定输出路径，则把结果打印输出

参数说明：
`--input` 指定监听目录， 非必填
`--output` 指定结果输出的文件路径， 非必填
`--discovery-interval` 指定监听的间隔时间， 非必填
`--execution-mode`  指定Flink的执行模式，非必填，默认为STREAMING模式


## 3. 代码实现

#### 3.1 指定监听目录

```java
//使用工具类CLI的fromArgs方法解析参数
final CLI params = CLI.fromArgs(args);

//setGlobalJobParameters(params),可以在Flink Web UI 中查看到传入的参数
env.getConfig().setGlobalJobParameters(params);

if (params.getInputs().isPresent()) {  
    // 如果指定了--input参数，则创建file source, 从指定的路径读取文件
   
        FileSource.FileSourceBuilder<String> builder =  
            FileSource.forRecordStreamFormat(  
                    new TextLineInputFormat(), params.getInputs().get());  

    // 如果指定了--discovery-interval参数，file source 会持续监听指定的目录的新文件      
    params.getDiscoveryInterval().ifPresent(builder::monitorContinuously);  
  
    // 指定算子名称为"file-input"
    text = env.fromSource(builder.build(), WatermarkStrategy.noWatermarks(), "file-input");  
} else {
    // 没有指定--input参数， 从静态变量WordCountData.WORDS读取数据
    // 指定算子名称为"in-memory-input"
    text = env.fromElements(WordCountData.WORDS).name("in-memory-input");  
}

```


#### 3.2 分词

要统计一段的文章每个单词的词频，分词是重要的一步，例如，这一段：
`"To be, or not to be,--that is the question:--"`
我们需要忽略掉空格，以及逗号等特殊字符，只保留单词

```java
public static final class Tokenizer  
        implements FlatMapFunction<String, Tuple2<String, Integer>> {  
  
    @Override  
    public void flatMap(String value, Collector<Tuple2<String, Integer>> out) {  
        // 把字符串转为小写后按单词分隔，存入到数组
        String[] tokens = value.toLowerCase().split("\\W+");  
  
        // 输出每个单词
        for (String token : tokens) {  
            if (token.length() > 0) {  
                out.collect(new Tuple2<>(token, 1));  
            }  
        }  
    }  
}
```

#### 3.3 统计每个单词的出现次数

```java
DataStream<Tuple2<String, Integer>> counts =  
      text
        // 分词处理，等到二元组 (word, 1) , 
        .flatMap(new Tokenizer()).name("tokenizer")   
        // 按单词分组，f0指的是二元组中的第一个字段        
        .keyBy(value -> value.f0)  
        //对二元组的第二个字段累加                      
        .sum(1)  
        .name("counter");
```


#### 3.4 指定输出路径
```java
if (params.getOutput().isPresent()) {  
    // 如果指定了输出的目录，则创建一个FileSink, 并把算子命名为file-sink  
    // 设置输出到文件的策略，1.内存的数据大于1M； 2.每隔10秒输出 。满足其中一个条件即输出
        counts.sinkTo(  
                    FileSink.<Tuple2<String, Integer>>forRowFormat(  
                                    params.getOutput().get(), new SimpleStringEncoder<>())  
                            .withRollingPolicy(  
                                    DefaultRollingPolicy.builder()  
                                        .withMaxPartSize(MemorySize.ofMebiBytes(1))  
                                        .withRolloverInterval(Duration.ofSeconds(10))  
                                        .build())  
                            .build())  
            .name("file-sink");  
} else { 
    // 没有指定输出的目录则打印， 算子名字为print-sink
    counts.print().name("print-sink");  
}
```


获取WordCount完整代码请参考文章: [Flink官方例子解析：Flink源码子项目flink-examples](!https://blog.csdn.net/weixin_47298890/article/details/128802299)
代码可在IDEA IntelliJ运行。

## 4. 执行效果

#### 4.1 在IDEA IntelliJ中配置程序的参数

![[Pasted image 20230131173427.png]]

```
--input D:\project\source_code\flink\flink-examples\flink-examples-streaming\src\main\java\org\apache\flink\streaming\examples\wordcount\input --discovery-interval 20 --output D:\project\source_code\flink\flink-examples\flink-examples-streaming\src\main\java\org\apache\flink\streaming\examples\wordcount\output
```


#### 4.2 启动程序

程序启动后，可在Flink WebUI看到以下DAG图

![[Pasted image 20230131173325.png]]




## 5. 结语

本篇到此结束，欢迎订阅Flink专栏，学习更多Flink的相关知识。