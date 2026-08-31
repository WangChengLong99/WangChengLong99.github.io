在 pandas 中，`pd.concat`、`DataFrame.join` 和 `pd.merge` 是三类最常用的数据组合工具。它们各有侧重，下面为你详细梳理其用法与区别。

---

## 1. pd.concat — 轴向拼接

**核心功能**：沿着一条轴（行/列）将多个 DataFrame 或 Series 直接堆叠在一起。  
**常见场景**：将结构相同的多张表纵向堆叠，或横向拼接多个特征列。

### 常用参数
- `objs`：要拼接的对象序列（列表、字典等）。
- `axis`：拼接方向。`0`（默认）沿行纵向堆叠；`1` 沿列横向拼接。
- `join`：处理非拼接轴上的索引冲突方式。
  - `'outer'`（默认）：取并集，缺失填 NaN。
  - `'inner'`：取交集。
- `ignore_index`：`True` 时丢弃原索引，生成 0~N-1 的新索引（常用在纵向堆叠）。
- `keys`：为各片段创建多层索引的最外层标签，便于区分来源。
- `verify_integrity`：`True` 时检查拼接轴上是否有重复索引，有则报错。

### 示例
```python
import pandas as pd

df1 = pd.DataFrame({'A': [1, 2], 'B': [3, 4]}, index=[0, 1])
df2 = pd.DataFrame({'A': [5, 6], 'B': [7, 8]}, index=[2, 3])

# 纵向堆叠
pd.concat([df1, df2])
#    A  B
# 0  1  3
# 1  2  4
# 2  5  7
# 3  6  8

# 忽略原索引
pd.concat([df1, df2], ignore_index=True)

# 横向拼接（列堆叠）
df3 = pd.DataFrame({'C': [9, 10]}, index=[0, 1])
pd.concat([df1, df3], axis=1)

# 添加键，区分来源
pd.concat([df1, df2], keys=['first', 'second'])
```

---

## 2. DataFrame.join — 基于索引的合并

**核心功能**：将另一个 DataFrame 的列，按其索引（或指定列）与主表的索引（或指定列）对齐后合并。本质是使用 `merge` 的简化接口。  
**常见场景**：快速将辅助表按索引贴到主表上，或按索引做类似数据库的连接操作。

### 常用参数
- `other`：要合并的 DataFrame（或多个 DataFrame 的列表）。
- `on`：主表中作为连接键的列名。此时仍以 `other` 的索引为匹配基准。
- `how`：连接方式，默认 `'left'`（左连接）。支持 `'left'`、`'right'`、`'inner'`、`'outer'`。
- `lsuffix` / `rsuffix`：当左右表列名冲突时，分别为左、右表列名添加的后缀。
- `sort`：是否按连接键排序（默认 False）。

### 示例
```python
left = pd.DataFrame({'A': [1, 2], 'B': [3, 4]}, index=['x', 'y'])
right = pd.DataFrame({'C': [5, 6], 'D': [7, 8]}, index=['x', 'z'])

# 左连接（保留左表所有索引）
left.join(right, how='left')
#    A  B    C    D
# x  1  3  5.0  7.0
# y  2  4  NaN  NaN

# 外连接
left.join(right, how='outer')

# 使用主表的某列作为连接键（右表仍用索引）
left2 = pd.DataFrame({'key': ['x', 'y'], 'A': [1, 2]})
right2 = pd.DataFrame({'C': [5, 6]}, index=['x', 'y'])
left2.join(right2, on='key')
```

---

## 3. pd.merge — 类似数据库的通用连接

**核心功能**：根据一个或多个列（或索引）作为键，将两个 DataFrame 的行进行匹配合并，非常像 SQL 的 JOIN。  
**常见场景**：主键关联、多表查询、复杂的条件合并。

### 常用参数
- `left` / `right`：两个 DataFrame。
- `how`：连接类型，`'inner'`（默认）、`'outer'`、`'left'`、`'right'`、`'cross'`。
- `on`：两张表共有的连接列名（单列或多列）。
- `left_on` / `right_on`：当左右表连接键名称不同时指定。
- `left_index` / `right_index`：使用索引作为连接键。
- `suffixes`：重名列的后缀，默认 `('_x', '_y')`。
- `indicator`：添加来源标识列（显示 `'both'`、`'left_only'`、`'right_only'`）。
- `validate`：校验合并关系的唯一性（如 `'one_to_one'`）。

### 示例
```python
left = pd.DataFrame({'key': ['a', 'b', 'c'], 'value': [1, 2, 3]})
right = pd.DataFrame({'key': ['b', 'c', 'd'], 'value2': [4, 5, 6]})

# 内连接（只保留匹配的键）
pd.merge(left, right, on='key')  # how='inner'

# 左连接
pd.merge(left, right, on='key', how='left')

# 不同列名连接
left2 = pd.DataFrame({'lkey': ['a', 'b'], 'L': [1, 2]})
right2 = pd.DataFrame({'rkey': ['a', 'b'], 'R': [3, 4]})
pd.merge(left2, right2, left_on='lkey', right_on='rkey')

# 使用索引作为键
pd.merge(left, right, left_index=True, right_index=True)

# 多键连接
pd.merge(left, right, on=['key1', 'key2'])

# 添加来源标识
pd.merge(left, right, on='key', how='outer', indicator=True)
```

---

## 4. 三者对比总结

| 特性                 | pd.concat                     | DataFrame.join                  | pd.merge                          |
|----------------------|-------------------------------|----------------------------------|-----------------------------------|
| **合并依据**         | 沿轴堆叠，不考虑值匹配        | 基于索引（或指定列）对齐        | 基于指定列（或索引）的值相等     |
| **操作方向**         | 纵向（axis=0）或横向（axis=1） | 横向（增加列）                  | 横向（增加列）                    |
| **连接类型**         | outer / inner（处理另一轴索引）| left / right / inner / outer    | inner / left / right / outer / cross |
| **主要应用**         | 同结构数据追加、特征拼接      | 快速按索引贴表                  | 类似 SQL 的复杂关系合并           |
| **索引处理**         | 可保留、忽略或创建多层索引    | 以索引为主，也可指定列          | 可选择列或索引作为键              |
| **重复列名处理**     | 可保留，参数调整有限          | lsuffix / rsuffix               | suffixes                         |

### 内在联系
- `join` 底层可以视为 `merge` 在左右表均使用索引（或左表指定列、右表用索引）时的快捷方式。
- `concat` 也能通过 `axis=1` 实现类似按索引对齐的横向拼接（使用 `join` 参数控制索引取并集/交集），但无法依据列值进行匹配。

### 选择建议
- **简单堆叠** → `pd.concat`（纵向追加、横向拼宽）
- **按索引快速关联** → `DataFrame.join`
- **按列值灵活关联（最接近 SQL）** → `pd.merge`

掌握了这三个工具，几乎可以应对所有 DataFrame 组合的需求。根据实际数据结构与关联逻辑，选择最直接的方式即可。

## 根据某张表删除另一张表

在 pandas 中，根据一张表的某些行去删除另一张表的对应行，本质上是一种 **反连接**（Anti-Join）：保留那些 **未匹配成功** 的行。根据匹配依据的不同（列值、索引、多列组合），可以选择最便捷的方法。

---

### 1. 基于单列值删除：`isin()` + 布尔索引

最常见情况：表 A 有一列 `id`，想删除表 B 中所有 `id` 出现在表 A 里的行。

```python
import pandas as pd
# 表B：主数据
df_b = pd.DataFrame({
    'id': [1, 2, 3, 4, 5],
    'value': ['a', 'b', 'c', 'd', 'e']
})
# 表A：要删除的id列表
df_a = pd.DataFrame({'id': [2, 4, 6]})
# 删除 df_b 中 id 存在于 df_a['id'] 的行
df_result = df_b[~df_b['id'].isin(df_a['id'])]
print(df_result)
```


```python
输出：

text

   id value
0   1     a
2   3     c
4   5     e
```


- `df_b['id'].isin(df_a['id'])` 返回一个布尔序列，标记哪些行在 A 中。
    
- `~` 取反，保留不在 A 中的行，即删除匹配行。