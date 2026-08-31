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

# 表数据的删除

### 方法一：使用 Navicat 图形界面（最快）

1. 在 Navicat 左侧对象列表中，找到目标数据库并展开 **“表”**。
2. 右键点击你要清空的表，在弹出菜单中选择 **“清空表”**（或英文版叫 **“Empty Table”** / **“Truncate Table”**）。
3. 如果表有外键约束，可能会弹出警告，确认是否继续。
4. 点击 **“确定”**，数据立即被清空，表结构、索引、触发器等都保留。

> 这个操作本质上执行的是 `TRUNCATE TABLE`，速度极快，会重置自增 ID（AUTO_INCREMENT）计数器。

---

### 方法二：在 Navicat 的 SQL 编辑器中执行命令

点击 **“查询” → “新建查询”**，输入并执行以下任一语句：

#### 1. 使用 TRUNCATE（推荐）
```sql
TRUNCATE TABLE 表名;
```
- **优点**：删除所有数据，速度快，不记录逐行删除日志，自动重置自增 ID。
- **缺点**：无法回滚（除非在事务中且存储引擎支持，但通常视为 DDL，隐式提交），遇到外键引用时可能报错。

#### 2. 使用 DELETE（需要回滚或保留自增ID时）
```sql
DELETE FROM 表名;
```
- **优点**：逐行删除，可回滚（需事务支持，如 InnoDB），可配合 `WHERE` 条件选择性删除。
- **缺点**：速度慢（大表尤其明显），不会自动重置自增 ID（除非随后执行 `ALTER TABLE 表名 AUTO_INCREMENT = 1;`）。

---

### 常见问题与提醒

- **外键约束导致无法清空**  
  如果表被其他表的外键引用，`TRUNCATE` 会报错。可临时关闭外键检查（谨慎操作）：
  ```sql
  SET FOREIGN_KEY_CHECKS = 0;
  TRUNCATE TABLE 表名;
  SET FOREIGN_KEY_CHECKS = 1;
  ```

- **自增 ID 重置**  
  用 `TRUNCATE` 后自增 ID 从 1 开始；用 `DELETE` 则继续保持原来的最大 ID + 1。  
  如果 `DELETE` 后想重置自增值，可额外执行：
  ```sql
  ALTER TABLE 表名 AUTO_INCREMENT = 1;
  ```

- **生产环境注意备份**  
  `TRUNCATE` 和 `DELETE` 都不可逆，操作前请确认数据已备份或确实可丢弃。

在 Navicat 中，一般直接右键 **“清空表”** 最省事，它会自动处理 `TRUNCATE`，如果失败会提示原因（通常是外键导致），这时再用上面的 SQL 语句配合关闭外键检查来解决。
