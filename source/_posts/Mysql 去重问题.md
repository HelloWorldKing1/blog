---
title: Mysql 去重问题
tags:
  - Mysql
  - SQL
categories: 问题
abbrlink: 3916608150
date: 2022-08-01 11:21:51
---

今早收到一条BUG；
我接手的别人的项目，SQL查询后数据没有对上；问题：关联查询出来的数据和单表查询的数据相比多出了几条；

SQL：

```sql
SELECT
	hsc.customer_simple_name AS customerSimpleName,
	hsc.telehone,
	so.high_seas_code AS highSeasCode,
	so.user_name AS userName,
	hsd.address,
	so.create_time AS createTime,
	hsd.region_code AS regionCode
FROM
	sales_order so
LEFT JOIN reform_customer_price rcp ON (so.high_seas_code = rcp.high_seas_code  AND  rcp.high_seas_code IS NULL)
LEFT JOIN high_seas_customer hsc ON hsc.high_seas_code = so.high_seas_code
LEFT JOIN high_seas_detail hsd ON hsd.high_seas_code = so.high_seas_code
WHERE
	so.shop_code = 'GC'
AND so.state = 2
AND rcp.high_seas_code IS NULL
ORDER BY
	createTime DESC
```

查询结果数量为`496`条；

单表查询`sales_order`这张表为`493`条

<font color="RED">分析问题；应该是关联的表字段有多条重复数据；导致关联查询的数据会增加</font>

解决：找到对应产生重复数据的表

```sql
select
  *
from
  high_seas_detail
where
  high_seas_code in (
    select
      high_seas_code
    from
      high_seas_detail
    group by
      high_seas_code
    having
      count(high_seas_code) > 1
  )
  order by high_seas_code
```

查询结果发现有不少重复的`high_seas_code`;

<font color="RED">找到罪魁祸首了；</font>

现在要去重处理 (去重要求：保留最新时间的那条数据)

```sql
SELECT high_seas_code,address,region_code FROM high_seas_detail GROUP BY create_time ORDER BY create_time DESC 
```

问题：虽然去重成功了，但是拿出来的数据似乎不是最新时间的那条

> 根据MySQL执行顺序
>
> 1. from 阶段
>
> 2. where 阶段
>
> 3. group by 阶段
>
> 4. having 阶段
>
> 5. select 阶段
>
> 6. order by 阶段
>
> 7. limit 阶段

所以，group by 在 order by 之前。所以这条sql的逻辑是先去重了，再按时间排序，不符合要求

解决：先排序后去重

注：5.7以后对[子查询](https://so.csdn.net/so/search?q=子查询&spm=1001.2101.3001.7020)排序做了优化，子查询全表排序失效，网上很多是在排序后面加limit的,这样感觉很不好,数据量大会丢数据,建议可以这样：

```sql
select
  t.high_seas_code,t.address,t.region_code
from
  (
    select
      high_seas_code,address,region_code
    from
      high_seas_detail
    where
      state = 1
    having
      1
    order by
      create_time desc
  ) t
group by
  t.high_seas_code
```

至此；成功去重,并且数据都对应上了

最终SQL

```sql
SELECT
  so.id,
  hsc.customer_simple_name AS customerSimpleName,
  hsc.telehone,
  so.high_seas_code AS highSeasCode,
  so.user_name AS userName,
  
  so.create_time AS createTime,
  hsd.address,
  hsd.region_code AS regionCode
FROM
  sales_order so
  LEFT JOIN reform_customer_price rcp ON (so.high_seas_code = rcp.high_seas_code  AND  rcp.high_seas_code IS NULL)
  LEFT JOIN high_seas_customer hsc ON hsc.high_seas_code = so.high_seas_code
  LEFT JOIN (SELECT t.high_seas_code,t.address,t.region_code FROM ( SELECT  high_seas_code,address,region_code FROM high_seas_detail WHERE state = 1 HAVING 1 ORDER BY create_time DESC ) t GROUP BY t.high_seas_code) hsd ON hsd.high_seas_code = so.high_seas_code
WHERE
  so.shop_code = 'GC'
  AND so.state = 2
ORDER BY
  createTime DESC;
```



---

经测试反馈；sql查询速度较慢;

将SQL改造如下：

```sql
select
      max(create_time) as create_time,
      high_seas_code,
      address,
      region_code
    from
      high_seas_detail
    group by
      high_seas_code
```

问题出现：

<font  color="RED">**由于mysql的表中一行只能对应一行数据，所以才会造成和max函数混合使用的错误。当使用max函数之后，一个group中的多行数据，只会显示第一行数据**</font>

我们可以了解到在使用**max（create_time）和group by(high_seas_code)函数时**只有**high_seas_code,create_time**正确，**其他的列均不正确**

所以可以将这两列作为子查询结果:

```sql
SELECT
	ss.high_seas_code,
	ss.address,
	ss.region_code,
	ss.create_time
FROM
	(
		SELECT
			max(create_time) AS create_time,
			high_seas_code
		FROM
			high_seas_detail
		GROUP BY
			high_seas_code
	) AS temp
JOIN high_seas_detail ss
WHERE
	ss.create_time = temp.create_time
AND ss.high_seas_code = temp.high_seas_code
```

结果： 可以正确的返回每组中最新时间的结果！

经测试速度也快了1.5秒左右。

最终sql：

```sql
SELECT
            hsc.customer_simple_name AS customerSimpleName,
            hsc.telehone,
            so.high_seas_code AS highSeasCode,
            so.user_name AS userName,
            so.create_time AS createTime,
            hsd.address,
            hsd.region_code AS regionCode
        FROM
            sales_order so
                LEFT JOIN reform_customer_price rcp ON (so.high_seas_code = rcp.high_seas_code AND rcp.high_seas_code IS NULL)
                LEFT JOIN high_seas_customer hsc ON hsc.high_seas_code = so.high_seas_code
                LEFT JOIN (
                SELECT ss.high_seas_code, ss.address, ss.region_code, ss.create_time FROM ( SELECT max(create_time) AS 		create_time, high_seas_code FROM high_seas_detail GROUP BY high_seas_code ) AS temp JOIN high_seas_detail ss WHERE ss.create_time = temp.create_time AND ss.high_seas_code = temp.high_seas_code
                ) hsd ON hsd.high_seas_code = so.high_seas_code
        WHERE
            so.shop_code = 'GC'
          AND so.state = 1
        ORDER BY
			createTime DESC
```

