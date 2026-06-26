---
tags:
  - 首页
title: mysql实际问题
---
#  `''`和`null`的显示区别

在 MySQL 中，空字符串 (`''`) 和 `NULL` 的显示方式**完全不同**。

**空字符串在 MySQL 中显示为空白（或一个看似空白的单元格，没有任何字符），而 `NULL` 值则明确显示为 `NULL` 字样。**

# null

和null的任何运算结果都是null

判断是否是null，用`is null` 不能用`=`

可以用`ifnull`函数判断一个值是否为null，如果为null，另赋一个值。如`a.mainBusinessRevenue - ifnull(b.mainBusinessRevenue, 0)`

# 对累增数据求每期数据

求差分，自联结本期和上期连接，比如按季度累增求每季度数据

```sql
select 
	a.taxpayerId,
	DATE_FORMAT(a.taxPeriodEnd,"%Y%m") `统计周期`,
	(a.mainBusinessRevenue - ifnull(b.mainBusinessRevenue, 0)) as `主营业务收入`
from 
	(select * from hxnew_center.hx_gxqy_cwbb  WHERE 
									(MONTH(taxPeriodStart)=1 and MONTH(taxPeriodEnd) = 3) or
									(MONTH(taxPeriodStart)=4 and MONTH(taxPeriodEnd) = 6) or
									(MONTH(taxPeriodStart)=7 and MONTH(taxPeriodEnd) = 9) or
									(MONTH(taxPeriodStart)=10 and MONTH(taxPeriodEnd) =12) 
) a
left join 
	(select * from hxnew_center.hx_gxqy_cwbb WHERE
						(MONTH(taxPeriodStart)=1 and MONTH(taxPeriodEnd) = 3) or
						(MONTH(taxPeriodStart)=4 and MONTH(taxPeriodEnd) = 6) or
						(MONTH(taxPeriodStart)=7 and MONTH(taxPeriodEnd) = 9) or
						(MONTH(taxPeriodStart)=10 and MONTH(taxPeriodEnd) =12) 
) b
on 
	quarter(a.taxPeriodEnd) = quarter(b.taxPeriodEnd) + 1 
	and a.year = b.year
	and a.taxpayerId = b.taxpayerId
```


# 构造一列数据，用于左连接

利用union将多个单值连成一列

```sql
select 2024
union all
select 2025
union all
select 2026
```

这样就可以取这三年的数据，并且当数据中没有该年份时显示为空

