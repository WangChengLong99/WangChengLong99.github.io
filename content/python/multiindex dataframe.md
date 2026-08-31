在处理包含 `MultiIndex`（多层索引）的 DataFrame 时，既要能灵活操作索引本身，又要能对数据进行常规的增删改查。下面从**索引操作**和**数据操作**两大方面，结合 Pandas 常用方法进行详细总结。

---

## 一、索引的增删改查

假设我们有一个带行 MultiIndex 的 DataFrame：
```python
import pandas as pd
import numpy as np

# 创建示例 MultiIndex DataFrame
index = pd.MultiIndex.from_tuples([
    ('A', 'x'), ('A', 'y'), ('B', 'x'), ('B', 'y')
], names=['group', 'subgroup'])
df = pd.DataFrame({'val1': [1, 2, 3, 4], 'val2': [5, 6, 7, 8]}, index=index)
# df:
#               val1  val2
# group subgroup          
# A     x          1     5
#       y          2     6
# B     x          3     7
#       y          4     8
```

### 1. 查（查询索引）

- **查看索引对象**：
  `df.index` 返回 MultiIndex 对象。  
  `df.index.levels` 查看各级别的唯一值列表；`df.index.codes` 查看每个标签对应的整数编码；`df.index.names` 查看各级别名称。
- **获取某一级别的所有值**：`df.index.get_level_values('group')` 返回 Index 对象。
- **基于索引标签选择行**：使用 `.loc[]`。
  ```python
  df.loc['A']                 # 选择第一级为 'A' 的所有行
  df.loc[('A', 'x')]          # 选择具体组合
  df.loc[('A', slice(None))]  # 等价于 df.loc['A']
  df.loc[(slice(None), 'x')]  # 选择第二级为 'x' 的所有行
  ```
- **使用 `xs()` 进行横截面选取**（可以不显示指定所有层级）：
  ```python
  df.xs('x', level='subgroup')   # 选取 subgroup 级别为 'x' 的所有行
  df.xs('A', level='group')      # 选取 group 级别为 'A' 的行
  # drop_level=False 可保留被切级别
  df.xs('x', level='subgroup', drop_level=False)
  ```
- **使用 `IndexSlice` 进行复杂切片**（与 `.loc[]` 配合）：
  ```python
  idx = pd.IndexSlice
  df.loc[idx['A':'B', 'x':], :]   # 第一级从 'A' 到 'B'，第二级从 'x' 到最后
  ```
- **布尔索引**：
  ```python
  mask = df.index.get_level_values('group') == 'A'
  df[mask]
  ```
- **使用 `query()` 方法**（需要列名与索引名不冲突）：
  ```python
  df.query('group == "A" and subgroup == "x"')
  ```

### 2. 增（增加索引层级或索引条目）
增加索引操作主要分为两类：添加新的索引级别，或往现有索引中添加新的标签（行）。

**添加索引级别：**
- **将现有列提升为索引**：`set_index()` 可以直接添加列作为新的索引级别，若 DataFrame 已有 MultiIndex，会追加层级。
  ```python
  df['category'] = ['cat1', 'cat2', 'cat1', 'cat2']
  df.set_index('category', append=True)  # append=True 保留原索引，新列追加为最内层
  ```
- **直接构造新级别后赋予**：  
  ```python
  new_idx = pd.MultiIndex.from_tuples([...], names=[...])
  df.index = new_idx   # 整体替换
  # 或使用 pd.concat 拼接索引，但更常见的是 set_index 或重新构建
  ```
- **插入临时级别**：可以使用 `assign` 生成临时列再 `set_index`，或使用 `pd.MultiIndex.from_product` 等。

**增加索引标签（添加行）：**
使用 `.loc[]` 赋值新行即可自动扩展索引。
```python
df.loc[('C', 'z')] = [9, 10]    # 添加新行
df.loc['D'] = [11, 12]           # 如果第二级缺失，会变成 NaN（需注意）
# 更安全的方式：
df.loc[('D', 'w'), :] = [11, 12]
```
也可用 `pd.concat()` 追加一个带有新索引的 DataFrame。

### 3. 删（删除索引级别或索引条目）
- **删除指定索引级别的行**：`df.drop(index=..., level=...)`。
  ```python
  df.drop('x', level='subgroup')        # 删除 subgroup 级别中值为 'x' 的所有行
  df.drop(('A', 'x'))                   # 删除特定的组合
  ```
- **删除整个索引级别**（将 MultiIndex 降级）：
  - `reset_index(level=..., drop=True)`：将指定级别移除（不保留为列）。
    ```python
    df.reset_index(level='subgroup', drop=True)   # 只保留 group 作索引
    ```
  - `droplevel()`：返回移除了指定级别的新 DataFrame。
    ```python
    df.droplevel('subgroup')
    ```
  - 若要完全删除索引（变成默认整数索引）：`df.reset_index(drop=True)`。
- **删除列索引中的级别**：若列也是 MultiIndex，可用 `df.columns.droplevel()`。

### 4. 改（修改索引名称、排序、值）
- **修改索引级别名称**：
  ```python
  df.index.names = ['g', 'sg']   # 直接赋值
  # 或使用 rename_axis
  df.rename_axis(index={'group': 'g', 'subgroup': 'sg'})
  ```
- **修改索引的值（标签重命名）**：
  - 使用 `rename()` 方法：
    ```python
    df.rename(index={'x': 'xx'}, level='subgroup')
    ```
  - 使用 `set_levels()` 整体替换某一级别的类别值：
    ```python
    new_levels = df.index.levels[0].str.lower()   # 将 group 级别全变为小写
    df.index = df.index.set_levels(new_levels, level=0)
    ```
  - 通过映射修改：`df.index = df.index.map(lambda t: (t[0].lower(), t[1]))`。
- **排序索引**：`sort_index(level=..., ascending=...)`。
  ```python
  df.sort_index(level='subgroup', ascending=False)
  ```
- **调整索引级别的顺序**：
  - `reorder_levels()`：交换级别顺序。
    ```python
    df.reorder_levels(['subgroup', 'group'])
    ```
  - `swaplevel()`：交换两个级别。

---

## 二、数据的增删改查

这部分操作与普通 DataFrame 类似，但需要特别注意**多层索引下的对齐和广播规则**。

### 1. 查（选择数据）
**选择列**：直接 `df['val1']` 或 `df[['val1', 'val2']]`（返回 DataFrame 或 Series）。  
**选择行与列结合**：主要用 `.loc[]` 和 `.iloc[]`。
- 基于标签：`df.loc[('A', 'x'), 'val1']` 选取单个值。
- 使用切片：
  ```python
  df.loc['A':'B']          # 包括从 'A' 到 'B' 的所有行（按默认排序）
  df.loc[:, 'val1':'val2'] # 列切片
  ```
- 使用布尔条件：
  ```python
  df[df['val1'] > 2]               # 行过滤
  df.loc[df['val1'] > 2, 'val2']   # 过滤后选列
  ```
- 使用 `query()`（列名不与索引名冲突时）：
  ```python
  df.query('val1 > 2 and group == "B"')
  ```

**多层列索引的查询**：
```python
df_col = pd.DataFrame(np.random.randn(4,4),
                      index=index,
                      columns=pd.MultiIndex.from_tuples([('a','x'),('a','y'),('b','x'),('b','y')]))
df_col.loc[:, ('a', 'x')]          # 选取单个列
df_col.loc[:, (slice(None), 'x')]  # 所有第一层，第二层为 'x' 的列
```

### 2. 增（增加数据）
**增加列**：
- 直接赋值：`df['new_col'] = values`（values 长度需匹配，或可广播）。
- `df.assign(new_col=lambda x: x['val1']*2)`。
- 插入指定位置：`df.insert(loc, column, value)`。

**增加行**：
- `.loc[]` 赋值新索引标签，如前所述。
- `pd.concat([df, new_df])`，要求索引对齐。
- `df.append()`（已弃用，建议用 concat）。

### 3. 删（删除数据）
**删除列**：
- `del df['val2']`
- `df.drop(columns=['val1'])` 或 `df.drop('val1', axis=1)`

**删除行**：
- `df.drop(('A', 'x'))` 删除特定索引标签的行。
- 结合布尔条件删除：`df.drop(df[df['val1']<2].index)`
- 使用 `drop` 的 `level` 参数删除某级别下特定值的所有行（前面索引部分已提）。

### 4. 改（修改数据值）
- 直接基于标签或位置修改：
  ```python
  df.loc[('A', 'x'), 'val1'] = 100
  df.iloc[0, 0] = 100
  ```
- 批量修改：
  ```python
  df['val1'] = df['val1'] * 10          # 整列修改
  df.loc['A', 'val2'] = [50, 60]        # 对第一级索引为 'A' 的行的 val2 列赋值
  ```
- 使用 `where`/`mask` 条件替换：
  ```python
  df['val1'] = df['val1'].where(df['val1']>2, other=0)   # 小于等于2的置0
  ```
- 替换特定值：`df.replace({1: 100, 2: 200})`
- 修改列的数据类型：`df.astype({'val1': float})`
- 处理缺失值：`fillna()` 等。

**特别注意多层索引的自动对齐**：
当给一个 Series 赋值时，Pandas 会按索引自动对齐。例如：
```python
s = pd.Series([100, 200], index=pd.MultiIndex.from_tuples([('A','x'), ('B','y')]))
df['val1'] = s   # 只有 ('A','x') 和 ('B','y') 行会被更新，其余变 NaN
```
因此进行局部更新时要注意索引的一致性，否则可能引入意外缺失值。

---

## 小结

| 操作 | 索引相关 | 数据相关 |
|------|---------|---------|
| **查** | `.index`、`.get_level_values()`、`.xs()`、`IndexSlice` | `.loc[]`、`.iloc[]`、`query()`、布尔索引 |
| **增** | `.set_index(append=True)` 追加层级；`.loc[new_label]` 新增行 | 直接赋值新列、`.assign()`、`concat()` 添加行 |
| **删** | `.droplevel()`、`reset_index(level=, drop=True)` 删除级别；`drop(level=)` 删除条目 | `drop(columns=)` 删除列；`drop(index=)` 删除行 |
| **改** | `rename_axis`、`rename(level=)`、`set_levels()`、`reorder_levels()` | 直接赋值修改值、`replace()`、`where()`、`astype()` |

掌握这些方法，就能灵活、高效地处理带有 MultiIndex 的 DataFrame。

下面是一份适合直接使用的“行索引 + 列索引 都是 MultiIndex 时的综合操作速查表”。它把“行、列、筛选、增删改查”放到一起，便于你在实际代码里快速对照。

---

# 列MultiIndex

## 1. 先构造一个同时有行和列 MultiIndex 的 DataFrame

```python
import pandas as pd

df = pd.DataFrame(
    [[1, 2, 3, 4],
     [5, 6, 7, 8]],
    index=pd.MultiIndex.from_tuples(
        [("A", 1), ("B", 2)],
        names=["R1", "R2"]
    ),
    columns=pd.MultiIndex.from_tuples(
        [("X", "u"), ("X", "v"), ("Y", "u"), ("Y", "v")],
        names=["C1", "C2"]
    )
)

print(df)
```

这时：

- 行索引是 `R1/R2`
- 列索引是 `C1/C2`

---

## 2. 行索引和列索引的基础查看

### 查看行索引
```python
df.index
df.index.names
df.index.get_level_values("R1")
```

### 查看列索引
```python
df.columns
df.columns.names
df.columns.get_level_values("C1")
```

---

## 3. 行索引操作速查

### 3.1 查某一行
```python
df.loc[("A", 1)]
```

### 3.2 按某一层查行
```python
df.xs("A", level="R1")
```

### 3.3 删除某一行
```python
df = df.drop(index=("A", 1))
```

### 3.4 修改某一行的索引
```python
df = df.rename(index={("A", 1): ("X", 1)})
```

### 3.5 删除某一层
```python
df = df.droplevel("R2", axis=0)
```

### 3.6 交换行层级顺序
```python
df = df.swaplevel(0, 1, axis=0)
```

---

## 4. 列索引操作速查

### 4.1 查某一列
```python
df[("X", "u")]
df.loc[:, ("X", "u")]
```

### 4.2 按某一层查列
```python
df.xs("X", axis=1, level="C1")
```

### 4.3 删除某一列
```python
df = df.drop(columns=[("X", "u")])
```

### 4.4 修改某一列名
```python
df = df.rename(columns={("X", "u"): ("M", "u")})
```

### 4.5 删除某一层
```python
df = df.droplevel("C2", axis=1)
```

### 4.6 交换列层级顺序
```python
df = df.swaplevel(0, 1, axis=1)
```

---

## 5. 组合查询：行 + 列一起查

### 5.1 查某个单元格
```python
df.loc[("A", 1), ("X", "u")]
```

### 5.2 查某一行中的某一类列
```python
df.loc[("A", 1), "X"]
```

### 5.3 查某一列中的某一类行
```python
df.loc["A", ("X", "u")]
```

### 5.4 取一个子矩阵
```python
df.loc[[("A", 1), ("B", 2)], [("X", "u"), ("Y", "v")]]
```

---

## 6. 增删改数据速查

### 6.1 增加一个单元格值
```python
df.loc[("A", 1), ("X", "u")] = 100
```

### 6.2 增加一行
```python
df.loc[("C", 3), :] = [10, 11, 12, 13]
```

### 6.3 增加一列
```python
df[("Z", "w")] = [9, 10]
```

### 6.4 删除某一行
```python
df = df.drop(index=("A", 1))
```

### 6.5 删除某一列
```python
df = df.drop(columns=[("X", "u")])
```

### 6.6 修改某一列全部值
```python
df[("X", "u")] = df[("X", "u")] * 2
```

### 6.7 按条件修改
```python
df.loc[df[("X", "u")] > 2, ("X", "u")] = 0
```

---

## 7. 条件筛选速查

### 7.1 按行条件筛选
```python
df[df[("X", "u")] > 2]
```

### 7.2 按列条件筛选
```python
df.loc[:, df.columns.get_level_values("C1") == "X"]
```

### 7.3 按行和列一起筛选
```python
df.loc[df[("X", "u")] > 2, [("X", "u"), ("Y", "v")]]
```

### 7.4 用 query 进行筛选
```python
df.query("(`X`,`u`) > 2")
```

> 注意：如果列名是 MultiIndex，`query` 的写法会稍微复杂，常见还是用 `loc` + 条件表达式更稳。

---

## 8. 索引层级重组速查

### 8.1 重命名层级名
```python
df = df.rename_axis(index=["R1_new", "R2_new"], columns=["C1_new", "C2_new"])
```

### 8.2 交换行层级
```python
df = df.swaplevel(0, 1, axis=0)
```

### 8.3 交换列层级
```python
df = df.swaplevel(0, 1, axis=1)
```

### 8.4 重排行层级顺序
```python
df = df.reorder_levels(["R2", "R1"], axis=0)
```

### 8.5 重排列层级顺序
```python
df = df.reorder_levels(["C2", "C1"], axis=1)
```

---

## 9. 最常用的 15 个核心方法

```python
loc
xs
drop
rename
swaplevel
reorder_levels
droplevel
reset_index
set_index
query
get_level_values
columns
index
iloc
rename_axis
```

---

## 10. 一句话记忆法

当行和列都是 MultiIndex 时，记住这三类核心思维：

- 行操作：看 `index` / `loc` / `xs`
- 列操作：看 `columns` / `loc` / `xs`
- 组合操作：用 `loc[行条件, 列条件]`

---

## 11. 最实用的口诀

- 想查“某个单元格”用 `loc[行标签, 列标签]`
- 想查“某一层的所有行/列”用 `xs`
- 想删“某行/某列”用 `drop`
- 想改“某个值”用 `loc[...] = ...`
- 想筛“满足条件的子表”用 `loc` + 条件

如果你愿意，我可以继续把这份内容再压缩成“面试版一页速查表”或者“代码示例版速查表”。

在 pandas 中，`MultiIndex`（多级索引）是用来表示**分层索引**的数据结构，可以让你在 `Series`、`DataFrame` 的行或列上拥有多个级别。以下是对 `MultiIndex` 最常用功能的系统总结。

---

# multiIndex

## 1. 什么是 MultiIndex
- 让一个轴（通常是行，也可以是列）拥有**多个层级**的索引，形如 `(level_0, level_1, ...)`。
- 非常适合表示**分组、面板数据、时间序列在不同类别下的结构**。

```python
import pandas as pd
import numpy as np

index = pd.MultiIndex.from_tuples([('A', 'x'), ('A', 'y'), ('B', 'x'), ('B', 'y')],
                                  names=['group', 'subgroup'])
df = pd.DataFrame(np.random.randn(4, 2), index=index, columns=['val1', 'val2'])
print(df)
#                    val1      val2
# group subgroup                    
# A     x        1.2345   -0.5678
#       y        0.9876    1.2345
# B     x       -0.4321   -0.8765
#       y        0.1234    0.3456
```

---

## 2. 创建 MultiIndex
| 方式 | 函数 | 说明 |
|------|------|------|
| 由数组列表 | `pd.MultiIndex.from_arrays([...])` | 每个列表是一个层级 |
| 由元组列表 | `pd.MultiIndex.from_tuples([...])` | 每个元组是一行多级索引 |
| 笛卡尔积 | `pd.MultiIndex.from_product([...])` | 生成所有组合 |
| 由 DataFrame 构造 | `pd.MultiIndex.from_frame(df)` | 将 DataFrame 的多列转为索引 |
| 直接通过 `groupby` / `pivot_table` 等产生 | 自动生成 | 常用 |

```python
arrays = [['A','A','B','B'], ['x','y','x','y']]
mi = pd.MultiIndex.from_arrays(arrays, names=['first', 'second'])
```

---

## 3. 基础索引与切片
- **`df.loc[(level0_val, level1_val)]`** — 精确匹配某一组合。
- **`df.loc['A']`** — 只给出第一级的值，返回该级别下所有子级数据（部分索引）。
- **`df.loc[('A', slice(None)), :]`** 或 **`df.loc[pd.IndexSlice['A', :], :]`** — 用切片灵活选择。
- **`xs()`** — 跨层级取值，`df.xs('x', level='subgroup')` 直接提取所有 subgroup 为 x 的行，结果不再保留该级索引。

```python
# 取 group=='A' 且 subgroup=='y' 的行
df.loc[('A', 'y')]

# 取所有 group 为 'A' 的行（保留 subgroup 索引）
df.loc['A']

# 取 subgroup=='x' 的所有行，去掉 subgroup 级别
df.xs('x', level='subgroup')
```

---

## 4. 层级操作
| 操作 | 方法 | 示例 |
|------|------|------|
| 提取某一级的值 | `get_level_values(level)` | `df.index.get_level_values('group')` |
| 交换层级顺序 | `swaplevel(i, j)` | `df.swaplevel('group','subgroup')` |
| 重新排序层级 | `reorder_levels(order)` | `df.reorder_levels(['subgroup','group'])` |
| 删除某一级 | `droplevel(level)` | `df.droplevel('subgroup')` |
| 重命名级别名字 | `rename(names, level=...)` | `df.index.rename(['cat','sub'], level=[0,1])` |

---

## 5. 排序
- 对 MultiIndex **排序**很重要，否则许多操作（如切片、`xs`）会很慢，并报 `PerformanceWarning`。
- 使用 `df.sort_index()` 按所有级别排序；可指定 `level` 参数针对特定级别排序。

```python
df.sort_index(level='group', ascending=False)
df.sort_values(by='val1')
```

---

## 6. stack / unstack
这是多层索引**最强大的变换工具**：
- `stack()` — 将列上的某个层级“压”到行索引，返回 Series 或 DataFrame。
- `unstack()` — 将行索引的某个层级“展开”到列上，常用于从长格式变宽格式。

```python
# 假设有行多级索引和普通列
# 把 subgroup 从行展开到列
df.unstack('subgroup')      # 列变成多级 ('val1','x') ...
# 或者堆叠列
df.stack()                  # 把列变成行索引的最后一层
```

---

## 7. 多级列索引
列也可以是 `MultiIndex`，操作类似：
- 使用 `.loc[:, (level0_col, level1_col)]` 选取。
- 使用 `df.columns.get_level_values()`、`swaplevel`、`droplevel` 等同样方法操作。

```python
columns = pd.MultiIndex.from_product([['score', 'rank'], ['math', 'english']])
df = pd.DataFrame(np.random.randn(4,4), columns=columns)
df.loc[:, ('score', 'math')]   # 选出 score 下的 math 列
```

---

## 8. 使用 IndexSlice 进行复杂选择
```python
idx = pd.IndexSlice
df.loc[idx['A':'B', 'x'], 'val1']        # group 在 A:B 且 subgroup='x'
df.loc[idx[:, 'y'], :]                   # 所有第一级，第二级为 y
```

---

## 9. 与 groupby 结合
`groupby` 可以直接接受 `level` 参数，按照某个索引层级聚合：
```python
df.groupby(level='group').sum()          # 按第一级求和
df.groupby(level=[0,1]).mean()           # 按两个级别
```

---

## 10. 注意事项
- **尽量排序**：使用 `sort_index()` 后性能显著提升。
- 避免层级太多（>3）导致索引混乱，可考虑用列存储部分维度。
- `reset_index()` 可将所有（或指定）索引层级转回普通列。
- `set_index()` 可将普通列设为多层索引。

```python
df.reset_index()                  # 全部变成列
df.reset_index(level='subgroup')  # 只把 subgroup 变成列
```

---

掌握上述 `MultiIndex` 操作后，处理分层数据、面板数据、多维度聚合与重塑会非常高效。如果还有更具体的场景（如时间序列、交叉表等），可以继续深入。