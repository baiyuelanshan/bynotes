介绍
-- 

一组数据DataSet，有多个维度A,B，我们需要对这组数据进行一下聚合：
* 全量数据的Min,Max,Avg
* 按维度A聚合，计算 Min,Max,Avg
* 按维度B聚合，计算 Min,Max,Avg
* 按维度AB聚合，计算 Min,Max,Avg


实现这个需求的直觉是根据不同的维度分别聚合计算出Min,Max,Avg结果，然后再用UNION ALL把所有结果连接起来：
```sql
-- 全量数据的Min,Max,Avg
SELECT
NULL, 
NULL,
Min,
Max,
Avg
from
DataSet

UNION ALL 
-- 按维度A聚合，计算 Min,Max,Avg
SELECT 
A,
NUll,
Min,
Max,
Avg
from
DataSet
GROUP BY 
A

UNION ALL 
-- 按维度B聚合，计算 Min,Max,Avg
SELECT 
NULL,
B,
Min,
Max,
Avg
from
DataSet
GROUP BY 
B 

UNION ALL 
--按维度AB聚合，计算 Min,Max,Avg
SELECT 
A,
B,
Min,
Max,
Avg
from
DataSet
GROUP BY 
A,
B

```


这种方式能达到效果，但是实现起来比较繁琐，如果维度比较多，就要一长串的SQL代码。
那么有没有比较简单且优雅的实现方式呢？
使用 `GROUPING SETS` 替代


```sql
SELECT 
A,
B,
Min,
Max,
Avg
from
DataSet
GROUP BY 
A,
B
GROUPING SETS(
        (A),
        (B),
        (A,B),
        ()
    )

```
这个写法也太简洁了吧！！

`当数据集中有多个维度，并且需要实现维度的多组合聚合， 使用GROUPING SETS可以完美解决`


使用案例
--
有一组薪酬明细数据，维度有部门，职级，需要计算一下结果：
- 全公司薪酬的Min,Max,Avg
- 部门薪酬的Min,Max,Avg
- 职级薪酬的Min,Max,Avg
- 部门内职级薪酬的Min,Max,Avg


1）准备数据
```txt
李明,一班,语文,80
韩梅梅,一班,语文,90
李明,一班,数学,90
韩梅梅,一班,数学,90
露西,二班,语文,90
莉莉,二班,语文,100
露西,二班,数学,100
莉莉,二班,数学,90
```


```
CREATE TABLE yishou_daily_test.score_detail (
    name string,
    class string,
    course string,
    score int
) 
...
```

2) 多维度分组聚合计算

```sql
SELECT 
coalesce(class,'全部') as class,
coalesce(course,'全部') as course,
Min(score) as min_score,
Max(score) as max_score,
Avg(score) as avg_score
from
yishou_daily_test.score_detail
GROUP BY 
class,
course
GROUPING SETS(
        (class),
        (course),
        (class,course),
        ()
    )

```

执行结果：

|department |level|min_salary|max_salary|avg_salary|
|-----------|-----|----------|----------|----------|
|财务部	|P6	|16000	|21000	|18500|
|财务部	|P7	|31000	|36000	|33500|
|财务部	|全部	|16000	|36000	|26000|
|技术部	|P7	|30000	|35000	|32500|
|技术部	|全部	|15000	|35000	|25000|
|技术部	|P6	|15000	|20000	|17500|
|全部	|全部	|15000	|36000	|25500|
|全部	|P6	|15000	|21000	|18000|
|全部	|P7	|30000	|36000	|33000|



|class|	course|	min_score|	max_score|	avg_score|
|-----|-------|----------|-----------|-----------|
|一班|	全部	|   80|	90	|87.5|
|一班|	语文|	80|	90	|85|
|一班|	数学|	90|	90	|90|
|全部|	全部|	80|	100	|91.25|
|全部|	数学|	90|	100	|92.5|
|全部|	语文|	80|	100	|90|
|二班|	数学|	90|	100	|95|
|二班|	全部|	90|	100	|95|
|二班|	语文|	90|	100	|95|
