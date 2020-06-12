# MySQL CAST与CONVERT 函数的用法

[TOC]

## 1 语法
MySQL 的 ```CAST``` 和 ```CONVERT``` 函数可用来转换字段的类型。两者具体的语法如下：
```sql
CAST(value as type);
CONVERT(value, type);
```
也就是
```sql
CAST(xxx AS 类型)
CONVERT(xxx,类型)
```

可以转换的类型是有限制的。这个类型可以是以下值其中的一个：
1. 二进制，同带binary前缀的效果 : BINARY    
2. 字符型，可带参数 : CHAR()     
3. 日期 : DATE     
4. 时间: TIME     
5. 日期时间型 : DATETIME     
6. 浮点数 : DECIMAL      
7. 整数 : SIGNED     
8. 无符号整数 : UNSIGNED 

## 2 示例
### 2.1 示例一
```sql
mysql> SELECT CONVERT('23',SIGNED);
+----------------------+
| CONVERT('23',SIGNED) |
+----------------------+
|                   23 |
+----------------------+
1 row in set
```

### 2.2 示例二
```sql
mysql> SELECT CAST('125e342.83' AS signed);
+------------------------------+
| CAST('125e342.83' AS signed) |
+------------------------------+
|                          125 |
+------------------------------+
1 row in set
```
### 2.3 示例三
```sql
mysql> SELECT CAST('3.35' AS signed);
+------------------------+
| CAST('3.35' AS signed) |
+------------------------+
|                      3 |
+------------------------+
1 row in set
```

像上面例子一样，将 ```varchar``` 转为 ```int``` 用 ```cast(a as signed)```，其中 ```a``` 为 ```varchar``` 类型的字符串。
