
## 1. 简介

很多朋友在学习Flink的时候都希望能有个demo代码来参考实践，于是在百度或者Google一顿搜索，虽然能找到很多demo代码，但是这些demo要么版本比较旧，要么代码不全，根本跑不通。

其实，Flink官网就提供了很多可供参考的demo代码，只需要拉取Flink源码子项目flink-examples即可。

## 2. 获取flink-examples子项目

#### 2.1 拉取 flink 源代码
```shell
git clone https://github.com/apache/flink.git
```

#### 2.2 切换到1.16版本的代码

```shell
git checkout remotes/origin/release-1.16
```

demo代码就在flink/flink-examples目录下

![[Pasted image 20230130143746.png]]

## 3. 使用IntelliJ IDEA 打开flink-examples项目

![[Pasted image 20230130143920.png]]

![[Pasted image 20230130144008.png]]

flink-examples下还有三个子项目：
`flink-examples-batch`: 批处理demo
`flink-exampls-streaming`: 流处理demo
`flink-exmaples-table`: table demo


以`flink-exampls-streaming`为例， 包含了flink流式处理丰富的例子，如窗口，状态，join，测流输出，异步IO等。

![[Pasted image 20230130144749.png]]

## 4. 注意事项

有部分JAR包下载失败，原因是从github拉取下来的Flink版本是1.16-SNAPSHOT，而maven库的版本号却是1.16.0， 因此，建议把flink-examples项目中的所有pom.xml中的Flink版本号统一修改为1.16.0

```xml
<parent>  
   <artifactId>flink-examples-build-helper</artifactId>  
   <groupId>org.apache.flink</groupId>  
   <!--<version>1.16-SNAPSHOT</version>-->  
   <version>1.16.0</version>  
</parent>
```


## 5. 结语

本篇到此结束，欢迎订阅Flink专栏，学习更多Flink的相关知识。