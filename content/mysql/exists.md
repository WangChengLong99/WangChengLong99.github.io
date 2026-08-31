在 MySQL 中，`EXISTS` 是一个用于判断子查询是否返回任何行的逻辑运算符。它通常用在 `WHERE` 子句中，返回**真 (TRUE)** 或 **假 (FALSE)**，非常适合做“是否存在”的判断。

---

## 1. 基本语法

```sql
SELECT columns
FROM outer_table o
WHERE EXISTS (
    SELECT 1
    FROM inner_table i
    WHERE i.column = o.column
);
```

- `EXISTS` 后的子查询如果**至少返回一行**，则条件成立，返回 `TRUE`。
- 如果子查询**没有任何结果**，则返回 `FALSE`。
- 子查询中的 `SELECT` 列表不重要，通常写成 `SELECT 1` 或 `SELECT *`，因为 `EXISTS` 只关心行是否存在。

---

## 2. 工作原理（相关子查询）

`EXISTS` 往往与**相关子查询**一起使用，即子查询引用了外部查询的列。MySQL 会对外部查询的每一行，执行一次子查询，一旦找到匹配的行就立即停止当前行的判断（类似于短路求值）。

**执行过程示意：**
1. 取出外部表的第一行。
2. 将这行的相关值代入子查询中执行。
3. 如果子查询返回至少一行，`EXISTS` 为 `TRUE`，该外部行被选中；否则跳过。
4. 继续处理外部表的下一行，重复步骤 2-3。

正因为是逐行关联执行，**`EXISTS` 特别适合处理“外表小、内表大”的场景**，尤其是内表有合适的索引时。

---

## 3. 与关键词判断的结合（你之前的场景）

回到你最初的需求：判断某个字符串是否包含关键词表中的任意词。

假设关键词表 `keywords`：

| id | keyword |
|----|---------|
| 1  | 苹果    |
| 2  | 橙子    |
| 3  | 香蕉    |

字符串 `'我喜欢吃苹果和橙子'`，检查它是否包含任意关键词。

```sql
SELECT CASE 
         WHEN EXISTS (
           SELECT 1 FROM keywords
           WHERE '我喜欢吃苹果和橙子' LIKE CONCAT('%', keyword, '%')
         )
         THEN '是'
         ELSE '否'
       END AS result;
```

- 子查询会扫描 `keywords` 表，依次检查 `'%苹果%'`、`'%橙子%'`、`'%香蕉%'`。
- 当检查到 `'%苹果%'` 时，`LIKE` 匹配成功，子查询返回一行。
- 此时 `EXISTS` 立即得到 `TRUE`，外部 `CASE` 返回 `'是'`，后续的关键词不再检查（短路）。
- 如果整个关键词表都没有匹配，子查询无结果，`EXISTS` 为 `FALSE`。

**为什么要用 EXISTS？**
- 只需要知道“是否至少存在一个”，而不需要返回具体关键词。
- 能够利用到短路特性，一旦找到匹配即停止，比用 `IN` 或 `JOIN` 可能更高效（尤其在关键词很多时）。

---

## 4. EXISTS 与 IN 的区别

很多人会问 `EXISTS` 和 `IN` 能否互换，它们在某些情况下可以，但有重要差异。

**示例：** 查询有订单的所有客户。
```sql
-- 使用 IN
SELECT * FROM customers WHERE id IN (SELECT customer_id FROM orders);

-- 使用 EXISTS
SELECT * FROM customers c WHERE EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.id);
```

| 特性 | EXISTS | IN |
|------|--------|----|
| **核心逻辑** | 判断子查询是否“存在”行 | 判断值是否在子查询返回的列表中 |
| **子查询结果** | 不关心具体值，只看有无返回行 | 需要获取一列具体值进行比较 |
| **NULL 处理** | 不影响，直接判断行存在 | 如果子查询结果含 NULL，可能返回未知（UNKNOWN） |
| **性能** | 通常外表小、内表有索引时更快 | 子查询结果集较小且无 NULL 时较快 |

**NULL 处理的经典陷阱**：
```sql
SELECT * FROM t WHERE id NOT IN (SELECT id FROM t2 WHERE condition);
```
如果 `t2` 的子查询结果中包含 `NULL`，整个 `NOT IN` 条件会变成 `UNKNOWN`，导致最终没有任何行返回。  
而 `NOT EXISTS` 不会出现这个问题：
```sql
SELECT * FROM t WHERE NOT EXISTS (SELECT 1 FROM t2 WHERE t2.id = t.id);
```

---

## 5. NOT EXISTS

`NOT EXISTS` 表示子查询**没有返回任何行**时为真，常用于查询“不存在于另一表中的数据”。

**示例：** 查询没有下过任何订单的客户。
```sql
SELECT * FROM customers c
WHERE NOT EXISTS (
    SELECT 1 FROM orders o WHERE o.customer_id = c.id
);
```

这也是 `LEFT JOIN ... IS NULL` 的一种替代写法，多数时候二者性能相近，但 `NOT EXISTS` 语义更直观。

---

## 6. 在 UPDATE / DELETE 中使用

`EXISTS` 同样可用于 `UPDATE` 和 `DELETE` 的条件，实现关联操作。

**示例：** 删除没有任何订单的客户。
```sql
DELETE FROM customers c
WHERE NOT EXISTS (
    SELECT 1 FROM orders o WHERE o.customer_id = c.id
);
```

**示例：** 将存在订单的客户状态标记为“活跃”。
```sql
UPDATE customers c
SET status = 'ACTIVE'
WHERE EXISTS (
    SELECT 1 FROM orders o WHERE o.customer_id = c.id
);
```

---

## 7. 性能考量与索引

- **对内表的关联列建立索引**：由于 `EXISTS` 子查询每行都会去内表查找，关联列上必须有索引才能避免全表扫描。例如上面例子中 `orders(customer_id)` 上应该有索引。
- **小表驱动大表**：`EXISTS` 适合外部表较小的情况。如果外部表很大而内表相对较小，可以考虑用 `IN` 或 `JOIN`，但必须根据执行计划验证。
- **避免子查询中包含不可优化的条件**：比如在子查询中使用 `LIKE '%keyword%'` 无法使用普通索引，此时如果关键词表大，性能会较差。这种情况下可考虑全文索引或应用层匹配，但 `EXISTS` 的短路特性依然能带来一定优化——找到第一个匹配就停止。

**查看执行计划**：通过 `EXPLAIN` 观察 `DEPENDENT SUBQUERY` 的扫描行数，确保索引有效。

---

## 8. 综合示例：多条件 EXISTS

判断某个字符串同时包含“苹果”和“香蕉”（两者都属于关键词表）：
```sql
SET @str = '我喜欢吃苹果和香蕉';

SELECT CASE 
         WHEN EXISTS (SELECT 1 FROM keywords WHERE @str LIKE CONCAT('%', keyword, '%') AND keyword = '苹果')
          AND EXISTS (SELECT 1 FROM keywords WHERE @str LIKE CONCAT('%', keyword, '%') AND keyword = '香蕉')
         THEN '包含两者'
         ELSE '否'
       END;
```
这里用两个 `EXISTS` 组合完成 AND 逻辑。

---

## 总结

- `EXISTS` 是存在性检查的利器，一旦找到匹配立即返回，无需扫描所有数据。
- 配合相关子查询，逐行关联判断，灵活性高。
- 处理 `NULL` 安全，不会出现 `NOT IN` 的意外空结果。
- 合理使用索引可大幅提升性能，常用于 `SELECT`、`UPDATE`、`DELETE` 中。

在你的关键词匹配场景中，`EXISTS` + `LIKE` 是最简单直接的实现方式，能够快速给出“是否包含”的答案。