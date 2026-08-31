MySQL 支持多种表连接方式，不仅包括 SQL 标准中的连接类型，还包含 MySQL 特有的实现算法和优化手段。下面从**连接语法分类**、**底层执行算法**和**相关优化概念**三个层面进行总结。

---

## 一、连接语法分类（逻辑上的连接方式）

### 1. 交叉连接（CROSS JOIN）

- 返回左表每一行与右表每一行的组合，即笛卡尔积。
- 可以不写 `ON` 条件，若写了条件则效果等价于内连接。
```sql
SELECT * FROM A CROSS JOIN B;
-- 等价于
SELECT * FROM A, B;
```

### 2. 内连接（INNER JOIN）

- 只返回两表中满足连接条件的行。
- 可使用 `ON` 或 `USING` 指定条件。
```sql
SELECT * FROM A INNER JOIN B ON A.id = B.a_id;
-- 或者省略 INNER
SELECT * FROM A JOIN B USING(id);  -- 要求两表有同名列
```

### 3. 外连接（OUTER JOIN）

- **左外连接（LEFT JOIN）**：保留左表所有行，右表无匹配时填充 NULL。
- **右外连接（RIGHT JOIN）**：保留右表所有行，左表无匹配时填充 NULL。
- `OUTER` 关键字可省略。
```sql
SELECT * FROM A LEFT JOIN B ON A.id = B.a_id;
SELECT * FROM A RIGHT JOIN B ON A.id = B.a_id;
```
MySQL 不支持全外连接，但可通过 `LEFT JOIN UNION RIGHT JOIN` 模拟。

### 4. 自然连接（NATURAL JOIN）

- 自动寻找两表中**所有同名列**进行等值连接，无需写 `ON`。
- 若同名列较多，容易产生非预期结果，慎用。
```sql
-- NATURAL JOIN 会自动连接所有同名的列
SELECT * FROM A NATURAL JOIN B;
-- NATURAL LEFT JOIN 等同样支持
```

### 5. 自连接（SELF JOIN）

- 同一个表通过不同别名与自己连接，用于处理层级或对比数据。
```sql
SELECT e.name, m.name AS manager
FROM emp e LEFT JOIN emp m ON e.mgr_id = m.id;
```

### 6. STRAIGHT_JOIN

- 功能与 `INNER JOIN` 完全相同，但强制优化器**按书写的表顺序连接**（左表驱动右表）。
- 适用于优化器选错驱动表时的强制干预。
```sql
SELECT * FROM A STRAIGHT_JOIN B ON A.id = B.a_id;
```

---

## 二、底层连接算法（执行引擎如何实现连接）

MySQL 优化器会根据表结构、索引、数据量等选择不同的算法执行连接。

## 1. 嵌套循环连接（Nested-Loop Join，NLJ）
- 最简单的连接方式：外层循环遍历驱动表，内层循环在被驱动表中查找匹配行。
- 若被驱动表的连接列有索引，则称为**索引嵌套循环连接（Index NLJ）**，内层可通过索引快速定位，效率很高；否则会退化为全表扫描嵌套。

## 2. 块嵌套循环连接（Block Nested-Loop Join，BNL）
- 用于被驱动表连接列**无索引**时。
- 把驱动表的数据分批读入 `join_buffer`，然后扫描被驱动表，在内存中做匹配，减少被驱动表的全表扫描次数。
- **MySQL 8.0.20 起该算法被移除**，完全由哈希连接替代。

## 3. 哈希连接（Hash Join）
- MySQL **8.0.18** 引入，**8.0.20** 开始作为等值连接且无可用索引时的默认算法。
- 将驱动表加载到内存构建哈希表，再扫描被驱动表，利用哈希表快速匹配，替换了原来的 BNL 算法，性能大幅提升。
- 仅适用于等值连接条件。

---

## 三、特殊连接概念（优化器中使用的连接方式）

## 半连接（Semi-join）
- 用于优化 `IN`、`EXISTS` 等子查询。
- 逻辑上只关心驱动表中**是否存在**匹配行，不关心匹配次数或需要返回被驱动表的列。
- 优化器会将子查询转换为半连接进行优化（如表拉平、FirstMatch 等策略）。

## 反连接（Anti-join）
- 用于优化 `NOT IN`、`NOT EXISTS` 等子查询。
- 只返回驱动表中**没有**匹配的行。
- 也是一种内部优化策略，不提供显式关键字。

---

## 四、最佳实践与注意事项

1. **优先为连接列建索引**  
   利用 Index NLJ，避免全表扫描和大量内存消耗。

2. **小表驱动大表**  
   优化器通常会自动选择数据量小的表作为驱动表。使用 `STRAIGHT_JOIN` 可强制指定。

3. **避免无条件的笛卡尔积**  
   `CROSS JOIN` 或忘记 `ON` 条件会导致结果集爆炸。

4. **慎用 NATURAL JOIN 和 USING**  
   列名变化或增加列可能导致连接逻辑隐式改变，建议显式写 `ON` 条件。

5. **版本差异关注**  
   MySQL 8.0.18+ 支持 Hash Join，8.0.20+ 彻底移除 BNL，升级后可获得更好的连接性能。

# 全外连接


在 MySQL 中，没有原生的 `FULL OUTER JOIN` 语法，但可以通过 **`LEFT JOIN` + `RIGHT JOIN` + `UNION`** 来实现“保留两边无连接的行”（即全外连接的效果）。

---

## 原理
- `LEFT JOIN`：保留左表**所有行**，右表无匹配的字段为 `NULL`。  
- `RIGHT JOIN`：保留右表**所有行**，左表无匹配的字段为 `NULL`。  
- 将上述两个结果用 `UNION` 合并，并自动**去掉完全重复的行**（左右都有匹配时，`LEFT JOIN` 和 `RIGHT JOIN` 会产出相同的行，`UNION` 会去重，正符合全连接语义）。

---

## 示例

假设有两张表：

**students**（学生）  
| id | name |
|----|------|
| 1  | 张三 |
| 2  | 李四 |
| 3  | 王五 |

**scores**（成绩）  
| student_id | score |
|------------|-------|
| 1          | 90    |
| 3          | 85    |
| 4          | 88    |

想连接学生和成绩，并且**保留所有学生和所有成绩信息**（即使学生没成绩、成绩没对应学生也要显示）。

```sql
-- 左连接 + 右连接 + UNION
SELECT s.id, s.name, sc.student_id, sc.score
FROM students s
LEFT JOIN scores sc ON s.id = sc.student_id

UNION

SELECT s.id, s.name, sc.student_id, sc.score
FROM students s
RIGHT JOIN scores sc ON s.id = sc.student_id;
```

### 结果：
| id   | name | student_id | score |
|------|------|------------|-------|
| 1    | 张三 | 1          | 90    |
| 2    | 李四 | NULL       | NULL  |  ← 左表有，右表无
| 3    | 王五 | 3          | 85    |
| NULL | NULL | 4          | 88    |  ← 右表有，左表无

> ✅ 完美保留了两边无匹配的行，未匹配的字段用 `NULL` 填充。

---

## 注意事项

1. **`UNION` 的去重特性**：如果左右表完全匹配的行在两边都出现，`UNION` 会自动去重，保留一行。这符合标准 `FULL JOIN` 的定义。  
2. **如果两表中有重复数据**，并且你希望保留所有重复行，可以用 `UNION ALL` 并额外添加条件排除交集，但一般情况下 `UNION` 已足够。  
3. **列的顺序和类型**：`UNION` 要求两个查询的列数、顺序和数据类型兼容，所以两边的 `SELECT` 列要对应一致。

---

## 另一种写法（利用 `LEFT JOIN` + 用 `NOT EXISTS` 补上右表独有行）
```sql
SELECT s.id, s.name, sc.student_id, sc.score
FROM students s LEFT JOIN scores sc ON s.id = sc.student_id
UNION ALL
SELECT NULL, NULL, sc.student_id, sc.score
FROM scores sc
WHERE NOT EXISTS (SELECT 1 FROM students s WHERE s.id = sc.student_id);
```
这种方法用 `UNION ALL` 且不需要去重，但需要额外子查询，通常直接 `LEFT JOIN` + `RIGHT JOIN` + `UNION` 更简洁。

---

总结：**MySQL 里实现“全外连接”就用 `LEFT JOIN ... UNION ... RIGHT JOIN` 即可保留两边所有无匹配的行。**

