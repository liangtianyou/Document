## 需求
数据库中字段 ```name``` 的数据格式为 ```SANLun111```，数字 ```111``` 的含义：
数字第一位 ```1```，表示控制器编号。
后面的数字 ```11``` 表示在该控制器上的 ```Lun``` 的编号。
创建新的 ```Lun``` 时，需要获取该控制器上最大的 ```Lun``` 编号。

## 处理
通过 ```substring_index(str,delim,count)``` 函数。

说明：```substring_index(被截取字段/字符串,关键字,按关键字切分后的索引)```

**注意**：索引从 ```1``` 开始，如果索引为负数，则是从后倒数。

## 示例
比如要截取的字符串为 ```aaa bbb ccc```，通过空格截取。
```sql
mysql> select substring_index("aaa bbb ccc"," ",1);
+--------------------------------------+
| substring_index("aaa bbb ccc"," ",1) |
+--------------------------------------+
| aaa                                  |
+--------------------------------------+
1 row in set (0.00 sec)

mysql> select substring_index("aaa bbb ccc"," ",-1);
+---------------------------------------+
| substring_index("aaa bbb ccc"," ",-1) |
+---------------------------------------+
| ccc                                   |
+---------------------------------------+
1 row in set (0.00 sec)
```
从上面示例可以看到，具体的使用方式。

## 最终解决
数据库表名：lun
字段名：name
```python
controller = '1'
sqlcmd = "select max(convert(substring_index(name, 'SANLun{0}', -1), SIGNED)) as name from lun where name like 'SANLun{0}%'".format(controller)
```
