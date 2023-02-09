
## 1. 简介

本篇介绍的是带窗口的WordCount，使用窗口函数countWindow。
countWindow是一种计数窗口，有固定窗口和滑动窗口两种用法。

#### 1.1 固定窗口

countWindow(windowSize) , windowSize指的是窗口大小。
例如countWindow(5)， 说明一个窗口可以容纳5个元素（对象），当元素的个数达到5个时，触发计算。

#### 1.2 滑动窗口
countWindow(windowSize, slideSize)，  windowSize指的是窗口大小，slideSize是滑动步长。
例如countWindow(5, 2)， 说明一个窗口可以容纳5个元素（对象），窗口每进来2个元素都会触发计算，当元素的个数达到5个时，也会触发计算。


## 2. countWindow WordCount需要实现的功能

1. 监听指定目录下的文件，读取文件的文本内容；如果未指定监听路径，则读取静态的字符串变量
2. 分词
3. 每个单词每出现2次输出一次，当频率达到5次时输出一次
4.  把结果打印输出

参数说明：
`--input` 指定监听目录， 非必填
`--output` 指定结果输出的文件路径， 非必填
`--discovery-interval` 指定监听的间隔时间， 非必填
`--execution-mode`  指定Flink的执行模式，非必填，默认为STREAMING模式
`--windowSize` 窗口大小，非必填，默认250
`--slideSize` 滑动步长，非必填，默认150

## 3. 代码实现

```java
DataStream<Tuple2<String, Integer>> counts =      
        text
        // 分词处理，等到二元组 (word, 1) , 
        .flatMap(new WordCount.Tokenizer())  
        .name("tokenizer")  
        // 按单词分组，f0指的是二元组中的第一个字段       
        .keyBy(value -> value.f0)  
        // 设置滑动窗口  
        .countWindow(windowSize, slideSize)  
        //对二元组的第二个字段累加               
        .sum(1)  
        .name("counter");
```

获取完整代码请参考文章: [Flink官方例子解析：Flink源码子项目flink-examples](!https://blog.csdn.net/weixin_47298890/article/details/128802299)

## 4. 执行效果

#### 4.1 在IDEA IntelliJ中配置程序的参数
![[Pasted image 20230201180808.png]]

```
--window 5 --slide 2
```


## 5. 结语

本篇到此结束，欢迎订阅Flink专栏，学习更多Flink的相关知识。