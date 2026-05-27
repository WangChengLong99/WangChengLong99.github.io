---
tags:
  - mysql
title: mysql知识点
---
# 表结构的增删改（DDL）

| 操作         | 常用语句                                                                                                                          | 说明                                                                                                                                                                                                                    |
| ---------- | ----------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **增**（创建表） | `CREATE TABLE [IF NOT EXISTS] 表名 ( id INT PRIMARY KEY AUTO_INCREMENT, 列名 类型 约束, ... ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;` | 可指定存储引擎、字符集、自增列、索引等。也可用 <br>•  `CREATE TABLE ... AS SELECT ...` ：根据查询出来的数据创建新表，包含数据。<br>•  `CREATE TABLE ... LIKE 旧表` ：根据另一个表的定义创建一个空表，包括原始表中定义的任何列属性和索引，但没有数据。                                                       |
| **删**（删除表） | `DROP TABLE [IF EXISTS] 表名;`                                                                                                  | 彻底删除表结构和数据，不可回滚（除非在事务中且适用引擎）。若只想清空数据保留结构，用 `TRUNCATE TABLE 表名;`。                                                                                                                                                      |
| **改**（修改表） | `ALTER TABLE 表名 ...`                                                                                                          | 常用子句：  <br>• 加列：`ADD 列名 类型 [约束];`  <br>• 删列：`DROP COLUMN 列名;`  <br>• 修改列类型：`MODIFY 列名 新类型;`  <br>• 重命名列：`CHANGE 旧名 新名 类型;`  <br>• 重命名表：`RENAME TABLE 旧名 TO 新名;` 或 `ALTER TABLE 旧名 RENAME TO 新名;`  <br>• 加/删索引、主键、外键等。 |
## sql示例

```sql
-- 创建用户表
CREATE TABLE IF NOT EXISTS users (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) DEFAULT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 新增一列
ALTER TABLE users ADD COLUMN age TINYINT UNSIGNED;

-- 删除列
ALTER TABLE users DROP COLUMN email;

-- 重命名表
RENAME TABLE users TO accounts;

-- 通过表查询创建表
DROP TABLE IF EXISTS company_in_out_label_tmp;
create table company_in_out_label_tmp as 
select *
from data_center.t_company_changes
WHERE project_name = "地址变更（住所地址、经营场所、驻在地址等变更）" and credit_code != ''
```

- [ ] 如何为导入数据创建表格

# 表中数据的增删改（DML）

| 操作         | 常用语句                                                                                                                 | 关键点                                                                                                                                                                                                                                                                  |
| ---------- | -------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **增**（插入行） | `INSERT INTO 表名 (列1, 列2) VALUES (值1, 值2);`  <br>`INSERT INTO 表名 SET 列1=值1, 列2=值2;`  <br>`INSERT INTO 表名 SELECT ...;` | 单行插入或多行：`VALUES (...), (...), (...);`  `REPLACE INTO` 会在主键/唯一键冲突时先删除旧行再插入。  <br>`INSERT ... ON DUPLICATE KEY UPDATE` 冲突时执行更新。<br>**`insert into ... select ...`** 利用查询数据进行插入数据。                                                                                      |
| **删**（删除行） | `DELETE FROM 表名 WHERE 条件;`                                                                                           | **必须慎用**：不加 `WHERE` 会删除全部行！ 删除全表用 `TRUNCATE TABLE 表名;` 更快，且重置自增计数器（不可回滚）。<br>- **TRUNCATE 与 DELETE 区别**：`TRUNCATE` 不记日志、不可回滚、**重置自增列**，`DELETE` 可加条件、触发触发器、逐行记录日志。<br>**有表连接时需要指明要删除的表**：<br>DELETE 表1, 表2   -- 删除哪几张表的记录<br>FROM 表1<br>JOIN 表2 ON 条件<br>[WHERE 条件]; |
| **改**（更新行） | `UPDATE 表名 SET 列1 = 值1, 列2 = 值2 WHERE 条件;`                                                                           | **必须带 WHERE**，否则全表更新。 可结合 `ORDER BY` 和 `LIMIT` 控制更新行数。<br>**更新单表：**（左连接时没有匹配的字段会视为Null）<br>UPDATE <br>表1 JOIN 表2 ON 表1.关联字段 = 表2.关联字段<br>SET 表1.要更新的字段 = 新值<br>WHERE 条件;<br>**更新多表：**<br>UPDATE <br>表1 JOIN 表2 ON 条件<br>SET 表1.字段 = 值1, 表2.字段 = 值2<br>WHERE ...;     |

对于excel中的导入数据，可以先利用concat函数将数据相关的语句拼凑好，再合并成sql语句。
对于数据库中的数据，可以通过表连接用其他表的数据作为条件。

> [!attention]+
> 删除和更新时，注意一定要有where条件，否则会全表删除和更新

## sql示例

```sql
-- 插入单行
INSERT INTO users (username, age) VALUES ('张三', 28);

-- 批量插入
INSERT INTO users (username, age) VALUES 
('李四', 25),
('王五', 30);

-- 插入查询结果
INSERT INTO vip_users SELECT * FROM users WHERE age > 28;

INSERT INTO hxnew_center.com_base (unifiedSocialCreditCode,companyName,listImportDate)
select t1.credit_code,t1.comp_name,CURDATE()
from (select * from data_center.t_company_base WHERE biz_date = (select max(biz_date) from data_center.t_company_base WHERE credit_code != '') and credit_code != '') t1
left join hxnew_center.com_base t2
WHERE t2.unifiedSocialCreditCode is NULL;

-- 删除一条记录
DELETE FROM users WHERE id = 10;

-- 更新指定用户的年龄
UPDATE users SET age = 29 WHERE username = '张三';

-- 如果希望保留左表所有行，即使右表没有匹配（没有匹配的字段会视为 NULL）
UPDATE orders o
LEFT JOIN customers c ON o.customer_id = c.id
SET o.discount = IF(c.level = 'VIP', 0.1, 0);

-- 删除没有订单的客户
DELETE c
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
WHERE o.id IS NULL;
```

# 日期时间处理

MySQL 提供了丰富的日期时间函数，可以灵活地进行获取、提取、运算、格式化和转换。下面按功能分类总结，并附上常用的计算场景和示例。

---

## 1. 获取当前日期和时间
| 函数                                            | 说明              |
| --------------------------------------------- | --------------- |
| `NOW()`                                       | 当前日期时间（语句开始时确定） |
| `CURDATE()`                                   | 当前日期            |
| `CURTIME()`                                   | 当前时间            |
| `SYSDATE()`                                   | 执行时的实时日期时间      |
| `UTC_DATE()`, `UTC_TIME()`, `UTC_TIMESTAMP()` | 返回 UTC 时间       |

```sql
SELECT NOW();                     -- 2026-05-26 14:30:00
SELECT CURDATE();                 -- 2026-05-26
SELECT CURTIME();                 -- 14:30:00
SELECT UTC_TIMESTAMP();           -- 2026-05-26 06:30:00
```

---

## 2. 提取日期/时间分量
| 函数 | 说明 |
|------|------|
| `YEAR(date)`, `MONTH(date)`, `DAY(date)` | 年、月、日 |
| `HOUR(time)`, `MINUTE(time)`, `SECOND(time)` | 时、分、秒 |
| `QUARTER(date)` | 季度（1-4） |
| `MONTHNAME(date)`, `DAYNAME(date)` | 月份名、星期名 |
| `DAYOFWEEK(date)` | 星期几（1=周日,7=周六） |
| `WEEKDAY(date)` | 星期几（0=周一,6=周日） |
| `DAYOFYEAR(date)` | 一年中的第几天 |
| `WEEK(date[,mode])` | 周数 |
| `EXTRACT(unit FROM date)` | 灵活提取，如 `YEAR_MONTH` |

```sql
SELECT YEAR('2026-05-26');         -- 2026
SELECT MONTHNAME('2026-05-26');    -- May
SELECT DAYOFWEEK('2026-05-26');    -- 3 (周二)
SELECT WEEKDAY('2026-05-26');      -- 1 (周二)
SELECT EXTRACT(YEAR_MONTH FROM '2026-05-26'); -- 202605
```

---

## 3. 日期/时间运算
| 函数 | 说明 |
|------|------|
| `DATE_ADD(date, INTERVAL expr unit)` | 日期加 |
| `DATE_SUB(date, INTERVAL expr unit)` | 日期减 |
| `ADDDATE(date, INTERVAL expr unit)` 或 `ADDDATE(date, days)` | 与上面相同（两用法） |
| `SUBDATE(date, INTERVAL expr unit)` 或 `SUBDATE(date, days)` | 相减 |
| `date + INTERVAL expr unit` / `date - INTERVAL expr unit` | 直接运算符 |
| `DATEDIFF(date1, date2)` | 日期差（天数） |
| `TIMEDIFF(time1, time2)` | 时间差 |
| `TIMESTAMPDIFF(unit, dt1, dt2)` | 按单位返回差值 |
| `PERIOD_ADD(period, n)` | 给 YYYYMM 加 n 个月 |
| `PERIOD_DIFF(p1, p2)` | 两个 YYYYMM 相差月数 |

常用 unit：`MICROSECOND, SECOND, MINUTE, HOUR, DAY, WEEK, MONTH, QUARTER, YEAR`  
复合 unit：`YEAR_MONTH, DAY_HOUR, DAY_MINUTE, DAY_SECOND, HOUR_MINUTE, HOUR_SECOND, MINUTE_SECOND` 等。

```sql
SELECT DATE_ADD('2026-05-26', INTERVAL 1 MONTH);   -- 2026-06-26
SELECT DATE_SUB('2026-05-26', INTERVAL 10 DAY);    -- 2026-05-16
SELECT '2026-05-26' + INTERVAL 1 YEAR;             -- 2027-05-26
SELECT DATEDIFF('2026-06-01', '2026-05-26');       -- 6
SELECT TIMESTAMPDIFF(MONTH, '2025-01-15', '2026-05-26'); -- 16
SELECT PERIOD_DIFF(202605, 202412);                -- 5
```

---

## 4. 格式化与解析
| 函数 | 说明 |
|------|------|
| `DATE_FORMAT(date, format)` | 日期/时间按格式输出 |
| `TIME_FORMAT(time, format)` | 仅格式化时间 |
| `STR_TO_DATE(str, format)` | 字符串 → 日期时间 |
| `GET_FORMAT(type, region)` | 预定义格式 |

常用格式符：`%Y`(4位年), `%y`(2位年), `%m`(月), `%d`(日), `%H`(24小时), `%i`(分), `%s`(秒), `%W`(星期名), `%M`(月份名) 等。

```sql
SELECT DATE_FORMAT(NOW(), '%Y年%m月%d日 %H:%i:%s'); -- 2026年05月26日 14:30:00
SELECT STR_TO_DATE('26/05/2026', '%d/%m/%Y');        -- 2026-05-26
```

---

## 5. 构造与类型转换
| 函数 | 说明 |
|------|------|
| `MAKEDATE(year, dayofyear)` | 由年和第几天生成日期 |
| `MAKETIME(h, m, s)` | 生成时间 |
| `DATE(expr)` | 截取日期部分 |
| `TIME(expr)` | 截取时间部分 |
| `TIMESTAMP(expr)` / `TIMESTAMP(d, t)` | 返回 datetime |
| `FROM_UNIXTIME(ts[, format])` | Unix 时间戳 → 日期 |
| `UNIX_TIMESTAMP([date])` | 日期 → Unix 时间戳 |
| `CONVERT_TZ(dt, from_tz, to_tz)` | 时区转换 |
| `CAST(expr AS DATETIME)` | 类型转换 |

```sql
SELECT MAKEDATE(2026, 146);           -- 2026-05-26
SELECT DATE('2026-05-26 14:30:00');  -- 2026-05-26
SELECT FROM_UNIXTIME(1760000000);     -- 转换时间戳
SELECT UNIX_TIMESTAMP('2026-05-26'); -- 对应时间戳
SELECT CONVERT_TZ('2026-05-26 12:00:00','+00:00','+08:00'); -- 时区转换（需时区表）
```

---

## 6. 其他实用函数
| 函数 | 说明 |
|------|------|
| `LAST_DAY(date)` | 该月最后一天 |
| `TO_DAYS(date)` | 从公元0年算起的天数 |
| `FROM_DAYS(n)` | 天数 → 日期 |
| `TIME_TO_SEC(time)` | 时间 → 秒数 |
| `SEC_TO_TIME(seconds)` | 秒数 → 时间 |
| `TO_SECONDS(expr)` | 从公元0年算起的秒数 |

```sql
SELECT LAST_DAY('2026-05-26');             -- 2026-05-31
SELECT TO_DAYS('2026-05-26');              -- 739759
SELECT SEC_TO_TIME(3661);                  -- 01:01:01
```

---

## 常用日期时间计算方法

### 1. 计算两个日期相差天数/月数/年数
```sql
-- 相差天数
SELECT DATEDIFF('2026-06-01', '2026-05-26');   -- 6

-- 相差月数（忽略日差）
SELECT TIMESTAMPDIFF(MONTH, '2025-01-15', '2026-05-26'); -- 16

-- 相差年数（常用于年龄）
SELECT TIMESTAMPDIFF(YEAR, '1990-05-15', CURDATE());     -- 36
```

### 2. 计算精确年龄（过完生日才算一岁）
```sql
SELECT TIMESTAMPDIFF(YEAR, birth, CURDATE()) 
FROM users;
-- 或使用公式
SELECT (YEAR(CURDATE())-YEAR(birth)) - 
       (DATE_FORMAT(CURDATE(),'%m%d') < DATE_FORMAT(birth,'%m%d'))
FROM users;
```

### 3. 本月第一天、最后一天
```sql
-- 月初
SELECT DATE_SUB(CURDATE(), INTERVAL DAYOFMONTH(CURDATE())-1 DAY);
SELECT DATE_FORMAT(CURDATE(), '%Y-%m-01');                     -- 字符串形式

-- 月末
SELECT LAST_DAY(CURDATE());                                     -- 2026-05-31
```

### 4. 上月最后一天、下月第一天
```sql
-- 上月最后一天
SELECT LAST_DAY(DATE_SUB(CURDATE(), INTERVAL 1 MONTH));

-- 下月第一天
SELECT DATE_ADD(LAST_DAY(CURDATE()), INTERVAL 1 DAY);
```

### 5. 本周一（以周一为起始）
```sql
SELECT DATE_SUB(CURDATE(), INTERVAL WEEKDAY(CURDATE()) DAY);
-- 若当前 2026-05-26 是周二，得 2026-05-25（周一）
```

### 6. 本周日、下周一
```sql
-- 本周日
SELECT DATE_ADD(CURDATE(), INTERVAL (6 - WEEKDAY(CURDATE())) DAY);

-- 下一个周一（如果今天是周一，则返回下周一的日期）
SELECT DATE_ADD(CURDATE(), INTERVAL (7 - WEEKDAY(CURDATE())) DAY);
```

### 7. 简单判断工作日（跳过周六日）
```sql
-- 给定日期 @d 的下一个工作日（不考虑节假日）
SELECT CASE DAYOFWEEK(@d)
         WHEN 7 THEN DATE_ADD(@d, INTERVAL 2 DAY)   -- 周六 -> 周一
         WHEN 1 THEN DATE_ADD(@d, INTERVAL 1 DAY)   -- 周日 -> 周一
         ELSE DATE_ADD(@d, INTERVAL 1 DAY)
       END AS next_workday;
```

### 8. 季度第一天、季度最后一天
```sql
-- 当前季度的第一天
SELECT MAKEDATE(YEAR(CURDATE()), 1) + INTERVAL (QUARTER(CURDATE())-1)*3 MONTH;

-- 当前季度最后一天
SELECT LAST_DAY(MAKEDATE(YEAR(CURDATE()), 1) + INTERVAL QUARTER(CURDATE())*3 MONTH - INTERVAL 1 MONTH);
```

### 9. 判断闰年（某年2月有29天）
```sql
SELECT DAYOFMONTH(LAST_DAY(CONCAT(@year,'-02-01'))) = 29 AS is_leap;
```

### 10. 计算两个时间点的小时/分钟差（如工作时长）
```sql
SELECT TIMESTAMPDIFF(MINUTE, '2026-05-26 09:00:00', '2026-05-26 18:30:00') / 60 AS work_hours;
-- 9.5 小时
SELECT SEC_TO_TIME(TIMESTAMPDIFF(SECOND, start_time, end_time)) AS duration;
```

---

**提示**  
- 进行月份加减时，若目标月没有该日（如1月31日加1个月），MySQL 会自动调整到该月最后一天（结果为2月28/29日）。  
- 时区转换函数 `CONVERT_TZ` 依赖系统时区表，使用时需确保 `mysql.time_zone` 表已填充数据（可用 `mysql_tzinfo_to_sql` 加载），否则会返回 `NULL`。  
- 直接使用 `+ INTERVAL` 语法与 `DATE_ADD` 功能等价，写法更简洁。  

掌握上述函数与计算方法，基本能覆盖绝大部分业务场景中的日期时间处理需求。

# 字符串处理

下面按功能类别总结 MySQL 常用字符串函数，然后结合业务场景说明如何灵活运用。

---

## 1. 字符串连接与拼接
| 函数 | 说明 |
|------|------|
| `CONCAT(str1, str2, ...)` | 拼接多个字符串，遇 NULL 返回 NULL |
| `CONCAT_WS(sep, str1, str2, ...)` | 用分隔符拼接，自动忽略 NULL |
| `GROUP_CONCAT([DISTINCT] expr ORDER BY … SEPARATOR …)` | 行转列，将分组内的值拼接成一个字符串 |

```sql
SELECT CONCAT('Hello', ' ', 'World');         -- Hello World
SELECT CONCAT_WS('-', '2026', '05', '26');    -- 2026-05-26
SELECT GROUP_CONCAT(DISTINCT city ORDER BY city SEPARATOR '/') FROM users;
```

---

## 2. 截取子串
| 函数 | 说明 |
|------|------|
| `LEFT(str, len)` | 从左边截取 len 个**字符** |
| `RIGHT(str, len)` | 从右边截取 len 个字符 |
| `SUBSTRING(str, pos [, len])` 或 `SUBSTR()`、`MID()` | 从位置 pos 开始截取 len 个字符（pos 从 1 开始） |
| `SUBSTRING_INDEX(str, delim, count)` | 按分隔符截取，count 为正数从左数，负数从右数 |

```sql
SELECT LEFT('abcdef', 3);                    -- abc
SELECT RIGHT('abcdef', 2);                   -- ef
SELECT SUBSTRING('abcdef', 2, 3);            -- bcd
SELECT SUBSTRING_INDEX('www.mysql.com', '.', 2);  -- www.mysql
SELECT SUBSTRING_INDEX('www.mysql.com', '.', -1); -- com
```

---

## 3. 字符串长度
| 函数 | 说明 |
|------|------|
| `CHAR_LENGTH(str)` 或 `CHARACTER_LENGTH()` | 字符个数（多字节算 1 个） |
| `LENGTH(str)` | 字节数（UTF-8 中一个汉字通常 3 字节） |
| `BIT_LENGTH(str)` | 比特长度（字节数*8） |

```sql
SELECT CHAR_LENGTH('你好');   -- 2
SELECT LENGTH('你好');        -- 6 （UTF-8 下）
```

---

## 4. 查找与定位
| 函数 | 说明 |
|------|------|
| `LOCATE(substr, str [, pos])` | 返回子串第一次出现的位置（从 1 开始），找不到返回 0 |
| `POSITION(substr IN str)` | 同 `LOCATE` 语法不同 |
| `INSTR(str, substr)` | 参数顺序与 `LOCATE` 相反 |
| `FIND_IN_SET(str, strlist)` | 在逗号分隔的列表中查找，返回位置（从 1 开始） |
| `FIELD(str, str1, str2, ...)` | 在参数列表中查找，返回位置（0 表示找不到） |

```sql
SELECT LOCATE('b', 'abcabc');            -- 2
SELECT INSTR('abcabc', 'b');             -- 2
SELECT FIND_IN_SET('a', 'a,b,c');        -- 1
SELECT FIELD('b', 'a', 'b', 'c');        -- 2
```

---

## 5. 替换与插入
| 函数 | 说明 |
|------|------|
| `REPLACE(str, from_str, to_str)` | 全局替换 |
| `INSERT(str, pos, len, newstr)` | 从 pos 开始删除 len 个字符，再插入 newstr |
| `REPEAT(str, count)` | 重复字符串 count 次 |
| `REVERSE(str)` | 反转字符串 |

```sql
SELECT REPLACE('ab12cd12ef', '12', 'XX');  -- abXXcdXXef
SELECT INSERT('abcdef', 3, 2, 'XX');       -- abXXef
SELECT REPEAT('ab', 3);                    -- ababab
```

---

## 6. 大小写转换
| 函数 | 说明 |
|------|------|
| `UPPER(str)` 或 `UCASE(str)` | 转大写 |
| `LOWER(str)` 或 `LCASE(str)` | 转小写 |

```sql
SELECT UPPER('Hello');   -- HELLO
SELECT LOWER('Hello');   -- hello
```

---

## 7. 去除空格与填充
| 函数 | 说明 |
|------|------|
| `TRIM([{BOTH｜LEADING｜TRAILING} [remstr] FROM] str)` | 移除两端/前导/末尾的指定字符（默认空格） |
| `LTRIM(str)` | 去除左边空格 |
| `RTRIM(str)` | 去除右边空格 |
| `LPAD(str, len, padstr)` | 左侧填充到指定字符长度 |
| `RPAD(str, len, padstr)` | 右侧填充到指定字符长度 |

```sql
SELECT TRIM('  hello  ');                -- hello
SELECT TRIM(LEADING '0' FROM '00123');   -- 123
SELECT LPAD('5', 3, '0');                -- 005
SELECT RPAD('ab', 5, '0');               -- ab000
```

---

## 8. 正则表达式（MySQL 8.0+）
| 函数 | 说明 |
|------|------|
| `REGEXP_LIKE(expr, pat [, match_type])` | 返回是否匹配（1/0） |
| `REGEXP_INSTR(expr, pat [, pos …])` | 返回匹配子串的位置 |
| `REGEXP_SUBSTR(expr, pat [, pos …])` | 返回匹配的子串 |
| `REGEXP_REPLACE(expr, pat, repl [, pos …])` | 正则替换 |

```sql
SELECT REGEXP_LIKE('abc123', '[0-9]+');          -- 1
SELECT REGEXP_SUBSTR('abc123def', '[0-9]+');     -- 123
SELECT REGEXP_REPLACE('abc123def', '[0-9]+', 'X'); -- abcXdef
```

MySQL 5.7 只能用 `expr REGEXP pat` 进行条件匹配，不支持提取和替换。

---

## 9. 比较与排序
| 函数 | 说明 |
|------|------|
| `STRCMP(expr1, expr2)` | 按当前排序规则比较，返回 -1/0/1 |
| `expr1 LIKE pat` | 简单模式匹配（`%` 任意多字符，`_` 一个字符） |
| `expr NOT LIKE pat` | 不匹配 |
| 排序规则 `COLLATE` | 可用于 `ORDER BY` 或 `WHERE` 实现大小写/重音不敏感 |

```sql
SELECT STRCMP('abc', 'abd');      -- -1
SELECT 'hello' LIKE '%ello';      -- 1
SELECT 'a' COLLATE utf8mb4_general_ci = 'A';  -- 1（大小写不敏感）
```

---

## 10. 其他实用函数
| 函数 | 说明 |
|------|------|
| `FORMAT(X, D [, locale])` | 数字格式化（千分位，返回字符串） |
| `ASCII(str)` | 首字符 ASCII 码 |
| `CHAR(N, ... [USING charset])` | 将整数转为对应字符 |
| `HEX(str)`, `UNHEX(str)` | 字符串与十六进制互转 |
| `BIN(N)`, `OCT(N)` | 十进制转二进制、八进制字符串 |
| `QUOTE(str)` | 产生带单引号的 SQL 转义字符串 |
| `SOUNDEX(str)` | 模糊读音匹配 |
| `ELT(N, str1, str2, ...)` | 返回第 N 个字符串 |
| `MAKE_SET(bits, str1, str2, ...)` | 根据二进制位选择字符串子集 |
| `WEIGHT_STRING(str [AS …])` | 排序权重值（调试用） |

```sql
SELECT FORMAT(1234567.89, 2);         -- 1,234,567.89
SELECT CHAR(65 USING utf8mb4);        -- A
SELECT HEX('abc');                    -- 616263
SELECT ELT(2, 'a', 'b', 'c');         -- b
```

---

## 常用业务场景与处理方案

### 1. 拼接用户姓名（处理 NULL）
- 直接 `CONCAT(last_name, first_name)` 若 last_name 为 NULL，结果全 NULL。
```sql
SELECT CONCAT_WS(' ', last_name, first_name) AS full_name FROM users;
```

### 2. 手机号/身份证脱敏
```sql
-- 手机号中间四位变 ****
SELECT CONCAT(LEFT(phone, 3), '****', RIGHT(phone, 4)) AS masked_phone
FROM users;

-- 身份证后四位保留，其余用*覆盖（假设18位）
SELECT CONCAT(REPEAT('*', 14), RIGHT(id_card, 4)) AS masked_id
FROM users;
```

### 3. 提取邮箱域名/用户名
```sql
SELECT SUBSTRING_INDEX(email, '@', -1) AS domain FROM users;
SELECT SUBSTRING_INDEX(email, '@', 1) AS username FROM users;
```

### 4. 从地址中提取数字（如楼栋号）
- 适合用正则（8.0+），否则可结合 `SUBSTRING` 和 `LOCATE`。假设楼栋号为“XX号楼”前的数字。
```sql
-- MySQL 8.0+
SELECT REGEXP_SUBSTR(address, '[0-9]+(?=号楼)') AS building_no FROM houses;

-- 5.7 简易方式：截取“号楼”前的部分再处理（受限）
SELECT SUBSTRING_INDEX(address, '号楼', 1) AS prefix FROM houses;
-- 再提取末尾数字较为复杂
```

### 5. 判断字符串是否包含某子串
```sql
-- 返回 1 表示包含
SELECT LOCATE('admin', user_name) > 0 AS is_admin FROM users;
-- 或用 REGEXP
SELECT user_name REGEXP 'admin' FROM users;
```

### 6. 逗号分隔字段查询（多值匹配）
```sql
-- 表中 tags 列存储 'a,b,c'，查询包含 'b' 的记录
SELECT * FROM articles WHERE FIND_IN_SET('b', tags);
```

### 7. 格式化输出（生成订单号）
```sql
SELECT CONCAT('ORD', DATE_FORMAT(NOW(), '%Y%m%d'), LPAD(id, 6, '0')) AS order_no
FROM orders;
-- 结果类似 ORD20260526000123
```

### 8. 清除收件地址中的多余空格/特殊字符
```sql
-- 去首尾空格和连续空格
SELECT REPLACE(TRIM(address), '  ', ' ') AS clean_address FROM orders;
-- 更彻底的去除所有空白字符 (MySQL 8.0+)
SELECT REGEXP_REPLACE(address, '[[:space:]]+', ' ') AS clean_address FROM orders;
```

### 9. 大小写不敏感的精确查询
```sql
SELECT * FROM users WHERE UPPER(username) = UPPER('Admin');
-- 或使用 COLLATE
SELECT * FROM users WHERE username COLLATE utf8mb4_general_ci = 'Admin';
```

### 10. 按首字母分类（中文拼音需自定义函数，英文简单）
```sql
SELECT UPPER(LEFT(name, 1)) AS initial, COUNT(*) FROM users
GROUP BY initial;
```

### 11. 计算字段显示长度，避免前端溢出
```sql
SELECT title, CHAR_LENGTH(title) AS title_len FROM articles
WHERE CHAR_LENGTH(title) > 30;
```

### 12. 生成随机验证码（字母+数字）
```sql
-- 6位随机码
SELECT UPPER(SUBSTRING(MD5(RAND()), 1, 6));
-- 或自选字符集
SELECT SUBSTRING('23456789ABCDEFGHJKLMNPQRSTUVWXYZ', FLOOR(1+RAND()*32), 6);
```

### 13. 简单 JSON 字段提取（MySQL 5.7+ 支持 JSON 类型更好）
```sql
-- 假设 extra 为 JSON 字符串，提取 name
SELECT JSON_UNQUOTE(JSON_EXTRACT(extra, '$.name')) AS name FROM profiles;
-- 若单纯字符串模拟，可用 SUBSTRING_INDEX 等但不够健壮
```

---

**注意**  
- `SUBSTRING`、`LEFT`、`RIGHT` 等函数基于**字符**计算，多字节字符如中文不会截断乱码。  
- 涉及性能时，对大量数据使用 `LIKE '%xxx%'` 或正则表达式可能较慢，尽量考虑前缀索引或全文索引。  
- 正则函数 `REGEXP_REPLACE`、`REGEXP_SUBSTR` 是 MySQL 8.0 新特性，5.7 环境需用其他方式或应用程序处理。  

熟练掌握这些函数和场景，可以高效解决绝大多数 SQL 中的字符串处理需求。

# 数据类型与转换

MySQL 中并没有直接返回“数据类型”的函数，但可以通过一些函数判断值的特征（是否为空、是否为数字等），同时提供了强大的显式类型转换机制，以及在运算和比较中会自动发生的隐式转换。

---

## 1. 数据类型判断

虽然没有 `TYPEOF()` 这样的函数，但以下方法可以帮助判断值的数据特征：

| 函数/表达式 | 说明 |
|-------------|------|
| `ISNULL(expr)` | 判断是否为 `NULL`，是返回 1，否则 0 |
| `IFNULL(expr1, expr2)` | 若 `expr1` 为 `NULL` 则返回 `expr2` |
| `COALESCE(expr1, ...)` | 返回第一个非 `NULL` 值 |
| `NULLIF(expr1, expr2)` | 若两值相等返回 `NULL`，否则返回 `expr1` |
| `JSON_TYPE(json_val)` | 返回 JSON 值的类型（`OBJECT`、`ARRAY`、`STRING` 等） |
| `expr REGEXP '正则'` | 可用于判断字符串是否全为数字、日期格式等 |
| `expr BETWEEN ...` 或 `CASE` 结合算术比较 | 间接判断字符串能否转为数字（MySQL 会将无法转的字符串当作 0） |

**示例**
```sql
SELECT ISNULL(NULL);              -- 1
SELECT COALESCE(NULL, 'a', 'b');  -- a
SELECT NULLIF(10, 10);            -- NULL

-- 判断是否全为数字（纯整数）
SELECT '12345' REGEXP '^[0-9]+$';   -- 1
SELECT '12a'   REGEXP '^[0-9]+$';   -- 0

-- 判断是否可转为日期
SELECT STR_TO_DATE('2026-05-26', '%Y-%m-%d') IS NOT NULL;  -- 1
```

---

## 2. 显式类型转换：`CAST` 与 `CONVERT`

这两个函数可以将一个值强制转换为指定类型。

### CAST

```sql
CAST(expr AS type)
```
支持类型：
- `BINARY[(N)]`
- `CHAR[(N)]` / `CHARACTER[(N)]` / `NCHAR[(N)]`
- `DATE`
- `DATETIME[(fsp)]`
- `DECIMAL[(M[,D])]`
- `DOUBLE`
- `FLOAT[(p)]`
- `SIGNED [INTEGER]`   （有符号整数）
- `UNSIGNED [INTEGER]` （无符号整数）
- `TIME[(fsp)]`
- `YEAR`
- `JSON`（MySQL 5.7+）

### CONVERT

两种用法：
1. 类型转换（与 `CAST` 类似）：
   ```sql
   CONVERT(expr, type)
   ```
   例如 `CONVERT('123', SIGNED)`
2. 字符集转换：
   ```sql
   CONVERT(expr USING charset_name)
   ```

```sql
SELECT CAST('123' AS UNSIGNED) + 1;          -- 124
SELECT CAST(123.456 AS DECIMAL(5,2));        -- 123.46
SELECT CONVERT('2026-05-26', DATE);          -- 2026-05-26
SELECT CONVERT('Hello' USING utf8mb4);       -- 修改字符集
```

## 常见转换组合

| 源类型 | 目标类型 | 示例 |
|--------|----------|------|
| 字符串 → 整数 | `SIGNED` / `UNSIGNED` | `CAST('007' AS UNSIGNED)` → 7 |
| 字符串 → 小数 | `DECIMAL(M,D)` | `CAST('12.34' AS DECIMAL(5,2))` |
| 数字 → 字符串 | `CHAR` | `CAST(123 AS CHAR)` → '123' |
| 日期/时间 → 字符串 | `DATE` / `DATETIME` | `CAST(NOW() AS CHAR)` |
| 字符串 → 日期 | `DATE` / `DATETIME` | `CAST('2026-05-26' AS DATE)` |
| 二进制 → 字符串 | `CHAR` | `CAST(binary_data AS CHAR)` |

---

## 3. 隐式类型转换及注意事项

在以下场景中，MySQL 会自动转换数据类型，**可能产生意想不到的结果**：

### 3.1 字符串与数字比较 / 运算

- 字符串会被转换为数字，如果字符串不以数字开头，则转换为 **0**。
```sql
SELECT 'a' = 0;          -- 1（true！）
SELECT '123abc' = 123;   -- 1，因为 '123abc' 转成数字 123
SELECT '123abc' + 1;     -- 124
SELECT 'abc' + 1;        -- 1
```

### 3.2 日期时间比较

- 字符串与 `DATE` / `DATETIME` 比较时，字符串会被当作日期解析。

```sql
SELECT '2026-05-26' > '2026-05-25';          -- 1（字符串按字典序比较）
SELECT '2026-05-26' > DATE '2026-05-25';    -- 1（字符串先转成日期再比）
```

直接用字符串比较日期格式，如果没有分隔符或格式不一致可能出错，**建议显式转换**。

### 3.3 与 `NULL` 比较

- 任何值与 `NULL` 比较（除了 `IS NULL`）都返回 `NULL`。
- `SELECT 10 = NULL;` → `NULL`，不是 0。

### 3.4 排序时的隐式转换

- 如果 `ORDER BY` 的列是字符串类型（如 `VARCHAR`）但存的是数字，排序会按**字典序**而不是数值大小。
```sql
-- num_str 列值：'1','10','2','20'
SELECT num_str FROM t ORDER BY num_str;        -- '1','10','2','20' (字典序)
SELECT num_str FROM t ORDER BY CAST(num_str AS UNSIGNED); -- '1','2','10','20'
```

---

## 4. 常用业务场景中的类型转换

### 4.1 将存储为字符串的数字用于数值排序或计算
```sql
-- 正确排序
SELECT * FROM products ORDER BY CAST(price_str AS DECIMAL(10,2));

-- 计算总和
SELECT SUM(CAST(amount_str AS UNSIGNED)) FROM payments;
```

### 4.2 数字补零 / 格式化

```sql
-- 订单编号：'ORD' + 6位流水号
SELECT CONCAT('ORD', LPAD(CAST(id AS CHAR), 6, '0')) AS order_no
FROM orders;
-- 如果 id 是数字，LPAD 会隐式转为字符串
```

### 4.3 提取字符串中的数字并转换为数值

```sql
-- MySQL 8.0 正则提取并转换
SELECT CAST(REGEXP_SUBSTR('楼层15-16室', '[0-9]+') AS UNSIGNED) AS floor;
-- 结果为 15
```

### 4.4 安全处理空值或无效值

```sql
-- 如果字段可能为 NULL 或非数字，计算时给默认值
SELECT IFNULL(CAST(score AS SIGNED), 0) FROM exams;

-- 或者用 CASE 判断
SELECT CASE WHEN score REGEXP '^[0-9]+$' THEN CAST(score AS SIGNED) ELSE 0 END
FROM exams;
```

### 4.5 判断字符串能否转为日期，并转换

```sql
-- 尝试转为日期，如果失败则返回 NULL
SELECT IF(STR_TO_DATE(date_str, '%Y-%m-%d') IS NULL, NULL, CAST(date_str AS DATE))
FROM raw_data;
-- 或者直接用 STR_TO_DATE
SELECT STR_TO_DATE(date_str, '%Y-%m-%d') FROM raw_data;  -- 无法转换返回 NULL
```

### 4.6 JSON 字段值提取并转换类型

```sql
SELECT JSON_UNQUOTE(JSON_EXTRACT(extra, '$.age')) AS age_str,
       CAST(JSON_UNQUOTE(JSON_EXTRACT(extra, '$.age')) AS UNSIGNED) AS age_int
FROM profiles;
-- MySQL 5.7.8+ 也可使用 ->> 操作符
SELECT extra->>'$.age' AS age_str, CAST(extra->>'$.age' AS UNSIGNED) AS age_int
FROM profiles;
```

### 4.7 字符串比较忽略大小写（通过字符集转换实现）

```sql
SELECT * FROM users WHERE CONVERT(username USING utf8mb4) = 'admin';
-- 或者使用 COLLATE，更规范
```

### 4.8 二进制与字符串互转
```sql
-- 字符串转二进制（如存储哈希值）
SELECT UNHEX(SHA2('hello', 256));
-- 二进制转可读字符串
SELECT HEX(binary_field) FROM t;
```

### 4.9 避免隐式转换带来的"诡异"结果
```sql
-- 错误：'a' 被转为 0，导致 WHERE 条件为真
SELECT * FROM t WHERE 0 = 'a';  -- 返回所有行！
-- 正确做法：使用 CAST 或显式类型确保类型一致
SELECT * FROM t WHERE CAST(col AS UNSIGNED) = 0;
```

---

**总结要点**
- 类型判断多借助 `REGEXP`、`ISNULL`、`STR_TO_DATE` 等函数来间接实现。
- 显式转换使用 `CAST(expr AS type)` 或 `CONVERT(expr, type)`，可使代码清晰、避免隐式陷阱。
- 注意 **字符串转数字时非数字开头会变成 0**，这种隐式行为可能导致逻辑错误和性能问题（索引失效）。
- 处理用户输入或不确定格式的数据时，**务必显式转换并配合判空/默认值**，提高 SQL 的健壮性。

# 变量与函数

MySQL 没有严格命名为“变量函数”的内置函数分类，但围绕**变量的定义、赋值、读取和系统变量获取**，有一套丰富的语法和机制。下面从变量类型、使用方式、常用技巧以及相关“函数式”操作进行详解。

---

## 1. MySQL 变量的三种类型

| 类型 | 声明方式 | 作用域 | 前缀 |
|------|----------|--------|------|
| **用户变量** | `SET @var = value` 或 `SELECT @var := expr` | 当前会话（连接） | `@` |
| **系统变量** | 预先定义，通过 `SET` 修改 | 全局（GLOBAL）或会话（SESSION） | `@@` |
| **局部变量** | 存储过程/函数内 `DECLARE var TYPE` | 存储程序内部 | 无前缀 |

---

## 2. 用户变量（@var）详解

用户变量无需声明类型，MySQL 根据赋值自动推断，且在整个会话期间有效。

### 2.1 赋值方式
```sql
-- 方式一：SET（可同时赋值多个，使用 = 或 :=）
SET @name = 'Alice';
SET @age := 30, @city := 'Hefei';

-- 方式二：SELECT ... INTO（必须返回单行，或使用 LIMIT 1）
SELECT phone INTO @phone FROM users WHERE id = 1;

-- 方式三：在 SELECT 语句中使用 := 进行赋值（边查询边赋值）
SELECT @num := COUNT(*) FROM orders;
```

### 2.2 读取
```sql
SELECT @name, @age, @phone;
-- 或用于表达式
SELECT * FROM products WHERE price > @avg_price;
```

### 2.3 未初始化的变量
未赋值的用户变量值为 `NULL`，类型也为 `NULL`。在运算中可能产生意外结果，例如 `@undefined + 10` 结果为 `NULL`。

```sql
SELECT @unknown;   -- NULL
SELECT @unknown + 1;  -- NULL
```

---

## 3. 用户变量在查询中的实战技巧

### 3.1 生成行号（MySQL 8.0 之前常用，8.0+ 建议用 `ROW_NUMBER()`）
```sql
SET @rownum = 0;
SELECT @rownum := @rownum + 1 AS row_num, name, score
FROM students
ORDER BY score DESC;
```

### 3.2 计算累计总和
```sql
SET @cum := 0;
SELECT month, amount, @cum := @cum + amount AS cumulative
FROM monthly_sales
ORDER BY month;
```

### 3.3 分组内排名（模拟 `RANK()`）
```sql
SET @rank = 0, @prev_score = NULL;
SELECT name, score,
       @rank := IF(@prev_score = score, @rank, @rank + 1) AS rank,
       @prev_score := score
FROM students
ORDER BY score DESC;
```
注意：MySQL 对于 `SELECT` 中赋值表达式的求值顺序没有严格保证，上述写法在大多数情况下能正常运行，但官方文档提醒这种用法在涉及复杂查询时可能结果不确定。8.0 版本后推荐直接使用窗口函数。

### 3.4 行间比较（计算差值）
```sql
SET @prev = NULL;
SELECT date, close,
       IF(@prev IS NULL, NULL, close - @prev) AS diff,
       @prev := close
FROM stock_prices
ORDER BY date;
```

---

## 4. 系统变量（@@）详解

系统变量控制 MySQL 的行为，分为 **GLOBAL**（影响所有新连接）和 **SESSION**（仅当前连接）。

### 4.1 查看系统变量
```sql
-- 查看所有系统变量
SHOW VARIABLES;
SHOW GLOBAL VARIABLES LIKE '%timeout%';

-- 用 SELECT @@ 读取
SELECT @@global.max_connections;   -- 全局值
SELECT @@session.sql_mode;         -- 会话值（@@local 同义）
SELECT @@sql_mode;                 -- 优先返回会话值
```

也可以从 `information_schema` 查询：
```sql
SELECT * FROM INFORMATION_SCHEMA.GLOBAL_VARIABLES WHERE VARIABLE_NAME = 'max_connections';
SELECT * FROM INFORMATION_SCHEMA.SESSION_VARIABLES;
```

### 4.2 设置系统变量
```sql
-- 设置全局变量（需要 SUPER 权限）
SET GLOBAL max_connections = 200;
SET @@global.max_connections = 200;

-- 设置会话变量
SET SESSION sql_mode = 'STRICT_TRANS_TABLES';
SET @@session.sql_mode = 'STRICT_TRANS_TABLES';
SET sql_mode = 'STRICT_TRANS_TABLES';   -- 默认就是 SESSION
```

### 4.3 常用系统变量示例
| 变量 | 说明 |
|------|------|
| `autocommit` | 自动提交 |
| `max_connections` | 最大连接数 |
| `sql_mode` | SQL 模式（严格模式等） |
| `character_set_client` | 客户端字符集 |
| `time_zone` | 时区 |
| `version` | 只读，MySQL 版本 |

版本变量其实可以通过 `SELECT VERSION();` 函数获取，但 `@@version` 也可。

---

## 5. 局部变量（DECLARE ...）简介

仅用于存储过程、函数、触发器和事件中，语法严格。

```sql
CREATE PROCEDURE demo()
BEGIN
    DECLARE total INT DEFAULT 0;
    DECLARE done INT DEFAULT FALSE;
    DECLARE cur CURSOR FOR SELECT amount FROM sales;
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;

    OPEN cur;
    read_loop: LOOP
        FETCH cur INTO total;
        IF done THEN
            LEAVE read_loop;
        END IF;
    END LOOP;
    CLOSE cur;
    SELECT total;
END;
```

局部变量的作用域在 `BEGIN...END` 块内，不能带有 `@` 前缀，且必须声明类型。

---

## 6. 与变量相关的“函数式”语法

严格来说并没有函数叫“变量函数”，但以下几个常与变量配合使用，或能产生类似动态变量的效果：

### 6.1 `SELECT ... INTO` 语句
虽然不是函数，但可以把查询结果赋值给变量：
```sql
SELECT COUNT(*), AVG(score) INTO @cnt, @avg FROM exams;
```

### 6.2 `IFNULL()` / `COALESCE()` 处理变量为 NULL 的情况
```sql
SELECT @count := IFNULL(@count, 0) + 1;
```

### 6.3 `FIELD()` 和 `ELT()` —— 列表选择
可实现类似于“变量值对应选项”的逻辑：
```sql
SET @color = 2;
SELECT ELT(@color, 'Red', 'Green', 'Blue');   -- Green
SELECT FIELD('Green', 'Red', 'Green', 'Blue'); -- 2
```

### 6.4 用户级锁函数（可视为一种全局命名变量）
```sql
SELECT GET_LOCK('my_lock', 10);   -- 获取锁，成功返回 1
SELECT RELEASE_LOCK('my_lock');   -- 释放锁
SELECT IS_FREE_LOCK('my_lock');   -- 检查锁是否空闲
```

### 6.5 `FOUND_ROWS()` / `ROW_COUNT()` 获取上一条语句影响行数
```sql
SELECT SQL_CALC_FOUND_ROWS * FROM users LIMIT 10;
SELECT FOUND_ROWS();   -- 返回不带 LIMIT 的总行数
SELECT ROW_COUNT();    -- 返回上条 UPDATE/DELETE/INSERT 影响行数
```

### 6.6 动态 SQL（借助变量构建语句）
用户变量可以存储拼接的 SQL 字符串，然后通过预处理执行，实现动态“变量函数”功能：
```sql
SET @table = 'users';
SET @query = CONCAT('SELECT * FROM ', @table);
PREPARE stmt FROM @query;
EXECUTE stmt;
DEALLOCATE PREPARE stmt;
```

---

## 7. 使用变量的注意要点

- **赋值与读取的顺序**：`SELECT` 语句中同时赋值和读取用户变量时，结果可能受优化器影响。避免在同一个非聚合查询的 `SELECT` 列表中既读取又修改同一个变量，除非你非常清楚执行顺序（MySQL 8.0 文档称此类操作不保证求值顺序）。
- **类型推断**：用户变量的类型来自第一次赋值的值，后续赋值可能改变类型。如从字符串变为整数。
- **事务与回滚**：用户变量的值不受 `ROLLBACK` 影响，因为变量操作不是事务性的。
- **并发**：每个连接拥有独立的用户变量，不会被其他连接干扰。

---

## 8. 业务场景示例

- **生成唯一序列号**：
  ```sql
  UPDATE serial_table SET current = LAST_INSERT_ID(current + 1) WHERE name = 'invoice';
  SELECT LAST_INSERT_ID();   -- 获得下一个编号
  ```
  （这里使用了 `LAST_INSERT_ID` 函数，虽然不是直接变量，但可当作无锁序列号生成器）

- **缓存中间结果**：
  ```sql
  SET @total = (SELECT SUM(amount) FROM orders WHERE year = 2026);
  SELECT product, SUM(amount), SUM(amount)/@total AS percent
  FROM orders WHERE year = 2026
  GROUP BY product;
  ```

- **存储过滤条件**：
  ```sql
  SET @min_price = 100;
  SELECT * FROM products WHERE price >= @min_price;
  ```

---

**小结**  
MySQL 虽然没有叫“变量函数”的独立分类，但通过**用户变量**、**系统变量**和**局部变量**，配合 `SET`、`SELECT ... INTO`、`:=` 等赋值语法，以及 `ELT()`、`FIELD()`、`GET_LOCK()` 等辅助函数，完全可以实现类似“变量函数”的动态效果。在处理行号、累积统计、动态 SQL 等场景中，这些技巧非常实用。