例子:

#### 只在8点的时候执行下游节点
```
#{DateUtil.format(Job.planTime,"HH") == '08' ? 'true' : 'false'}
```