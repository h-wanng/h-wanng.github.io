+++
author = "H. Wang"
title = "MySQL 特定值在结果中的位置"
date = "2023-03-03"
description = "MySQL 特定值在结果中的位置 使用@rn 和 order by"
tags = [
    "MySQL"
]
categories = [
    "MySQL"
]

image = ""
+++

## MySQL 特定值在结果中的位置（序号）

```sql
select tt.rn from (
  select t.id, (@rn := @rn + 1) as rn from (
    select * from t_example te where te.a = 'a' and te.b = 'b' order by te.id
  ) t cross join (select @rn := 0) params
) tt where tt.id = '123'
```

```sql
select t.rn from (
  select te.id, (@rn := @rn + 1) as rn from t_example te cross join (select @rn := 0) params where te.a = 'a' and te.b = 'b' order by te.id
) t where t.id = '123'
```

可先在内层select子句中将满足指定条件的数据查询出来，并按特定字段进行排序。然后为查询结果增加行号，最后即可查询出特定值在结果中的位置。

也可直接为查询结果加上行号，再查询特定值的序号。

其中，`@xx` 为用户变量，`@`符号后加上变量名，用于定义一个变量。为变量赋值使用的符号为`:=`，如：

```sql
select @i := 1, @i := @i + 1, @i := @i + 1
```

```sql
@i := 1|@i := @i + 1|@i := @i + 1|
-------+------------+------------+
      1|         2.0|         3.0|
```

而`(select @rn := 0) params ` 结果只有一行，与其进行笛卡尔乘积等于在结果集中增加一行，以此实现查询行号。

```sql
select * from (select 1, 2, 3) t1 cross join (select 4) t2
```

```sql
1|2|3|4|
-+-+-+-+
1|2|3|4|
```

但在使用中发现MySQL提示这种用户变量定义方式已经弃用：

`Setting user variables within expressions is deprecated and will be removed in a future release. Consider alternatives: 'SET variable=expression, ...', or 'SELECT expression(s) INTO variables(s)'.`

查询发现还有另一种实现行号的方法，即通过`row_number() over(order by )`，因此上文中的查询语句可改为：

```sql
select t.rn from (select id, row_number() over(order by id) as rn from t_example where a = 'a' and b = 'b') t where t.id = '123'
```

