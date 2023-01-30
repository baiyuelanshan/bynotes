## 1. 简介

![[Pasted image 20230130160722.png]]

CLI(org.apache.flink.streaming.examples.wordcount.util.CLI) 这个工具类在官方的大多数例子中都会使用到， 因此本文先对这个类进行介绍。

这个类比较简单，主要用于封装传入的参数，如--input, --output。

## 2. CLI预处理的参数

`input` 输入的文件路径
`output` 输出的文件路径
`discovery-interval` 发现间隔，用于STREAMING处理模式
`executionMode` Flink执行模式, 默认是STREAMING处理模式

```java
MultipleParameterTool params = MultipleParameterTool.fromArgs(args);  
Path[] inputs = null;  
if (params.has(INPUT_KEY)) {  
    inputs =  
            params.getMultiParameterRequired(INPUT_KEY).stream()  
                    .map(Path::new)  
                    .toArray(Path[]::new);  
} else {  
    System.out.println("Executing example with default input data.");  
    System.out.println("Use --input to specify file input.");  
}  
  
Path output = null;  
if (params.has(OUTPUT_KEY)) {  
    output = new Path(params.get(OUTPUT_KEY));  
} else {  
    System.out.println("Printing result to stdout. Use --output to specify output path.");  
}  
  
Duration watchInterval = null;  
if (params.has(DISCOVERY_INTERVAL)) {  
    watchInterval = TimeUtils.parseDuration(params.get(DISCOVERY_INTERVAL));  
}  
  
RuntimeExecutionMode executionMode = ExecutionOptions.RUNTIME_MODE.defaultValue();  
if (params.has(EXECUTION_MODE)) {  
    executionMode = RuntimeExecutionMode.valueOf(params.get(EXECUTION_MODE).toUpperCase());  
}  
  
return new CLI(inputs, output, watchInterval, executionMode, params);
```

另外`params`变量， 包含了所有命令行输入的参数。



在此，不得不赞一下Flink的官方例子， 即使是例子也写得相当的规范，非常适合初学者学习， 在实际生产可参考使用。


本篇到此结束，欢迎订阅Flink专栏，学习更多Flink的相关知识。


