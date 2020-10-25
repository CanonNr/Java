# where 1 = 1 是否存在性能问题

## 前言

> 如果百度搜索该问题，可能说存在性能问题的解答更多一点，但是今天通过实际操作发现这一次真理掌握在少数派手中。

测试环境：windows + mysql5.5 / 5.6

## 是什么

在动态拼接`WHERE`条件时可能出现如下情况

- 语句一

  ```sql
  SELECT * FROM Table 
  WHERE column_a = '...'
  ```

- 语句二

  ```sql
  SELECT * FROM Table 
  AND column_a = '...'
  ```

  

很显然**语句二**是有问题的，`AND`不能直接出现，面对这个问题所以出现了动态拼接`WHERE`的方法

```xml
<select id="test" resultType="Table">
  SELECT * FROM Table 
  WHERE 
  <if test="column_a != null">
    column_a = #{column_a}
  </if> 
</select>
```

但是也有这么一种

```xml
<select id="test" resultType="Table">
  SELECT * FROM Table 
  WHERE 1=1
  AND column_a = '...'
</select>
```

**因为 1=1 永真，所以不会影响后面的AND条件**

## 实测

简单创建一个测试表`goods`，包含三个字段：

- 唯一主键`id`
- 商品名 - 普通索引 `name`
- 商品数量 - 普通字段无索引 `num`



### 首先通过 EXPLAIN

返回的结果**完全相同**

```sql
mysql> EXPLAIN  SELECT id FROM goods WHERE name='手机' and 1=1;
+----+-------------+-------+------+---------------+------+---------+-------+------+----------+--------------------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref   | rows | filtered | Extra                    |
+----+-------------+-------+------+---------------+------+---------+-------+------+----------+--------------------------+
|  1 | SIMPLE      | goods | ref  | name          | name | 768     | const |    1 |   100.00 | Using where; Using index |
+----+-------------+-------+------+---------------+------+---------+-------+------+----------+--------------------------+
1 row in set, 3 warnings (0.00 sec)

mysql> EXPLAIN  SELECT id FROM goods WHERE name='手机';
+----+-------------+-------+------+---------------+------+---------+-------+------+----------+--------------------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref   | rows | filtered | Extra                    |
+----+-------------+-------+------+---------------+------+---------+-------+------+----------+--------------------------+
|  1 | SIMPLE      | goods | ref  | name          | name | 768     | const |    1 |   100.00 | Using where; Using index |
+----+-------------+-------+------+---------------+------+---------+-------+------+----------+--------------------------+
1 row in set, 3 warnings (0.00 sec)

mysql> EXPLAIN SELECT id FROM goods WHERE 1=1 AND name='手机';
+----+-------------+-------+------+---------------+------+---------+-------+------+----------+--------------------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref   | rows | filtered | Extra                    |
+----+-------------+-------+------+---------------+------+---------+-------+------+----------+--------------------------+
|  1 | SIMPLE      | goods | ref  | name          | name | 768     | const |    1 |   100.00 | Using where; Using index |
+----+-------------+-------+------+---------------+------+---------+-------+------+----------+--------------------------+
1 row in set, 3 warnings (0.00 sec)
```

### 查看语句优化信息

```sql
EXPLAIN EXTENDED SELECT id FROM goods WHERE name='手机' and 1=1
show warnings;

EXPLAIN EXTENDED SELECT id FROM goods WHERE 1=1 AND name='手机';
show warnings;

EXPLAIN  SELECT id FROM goods WHERE name='手机';
show warnings;
```

分别执行后得到结果还是相同的

```sql
+---------+------+-------------------------------------------------------------------------------------------------+
| Level   | Code | Message                                                                                         |
+---------+------+-------------------------------------------------------------------------------------------------+
| Note    | 1003 | select `test`.`goods`.`id` AS `id` from `test`.`goods` where (`test`.`goods`.`name` = '手机')   |
+---------+------+-------------------------------------------------------------------------------------------------+
```

 

可以看出最终执行的语句中压根就没有1=1，表示mysql提前将`1=1`优化掉了



## 结论

不会影响性能

