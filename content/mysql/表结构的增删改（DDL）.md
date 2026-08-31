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
