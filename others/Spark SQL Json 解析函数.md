## 函数
get_json_object
使用方法： get_json_object(<表字段>, "$.<json中的字段>")

## 举例

test_json 表的 content 字段有如下 Json 数据，
>{"tableUseAmount":0,"table_name":"yishou_data.add_cart_not_order"}

使用  get_json_object 函数解析：
```sql
select 
get_json_object(content, "$.table_name") as table_name
,get_json_object(content, "$.tableUseAmount") as tableUseAmount
from yishou_data.ods_hw_mea_dict_dt 
```

输出结果：
> table_name                                       tableUseAmount 
> yishou_data.add_cart_not_order           0


