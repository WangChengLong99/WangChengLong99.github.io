---
title: 11 稀疏数据
---

> [!abstract] 本节大纲
> - SparseArray
> - SparseDtype
> - Series.sparse
> - DataFrame.sparse
> - 创建和转换
> - 密度与内存
> - 稀疏方法
> - 与其他 dtype 交互

## 1. SparseArray

`SparseArray` 是 pandas 用于存储**稀疏数据**的专用数组结构。它只存储非缺失值（或非填充值）及其位置，从而大幅节省内存。

### 1.1 概念

- 稀疏数据：大部分元素为相同值（通常为 `0` 或 `NaN`）的数组。
- `SparseArray` 内部维护两个数组：
  - `sp_values`：存储非填充值（有效值）。
  - `sp_index`：存储有效值的位置索引。
- 填充值（`fill_value`）是稀疏数组中被省略的重复值，默认为 `NaN`。

### 1.2 创建 SparseArray

- **从普通数组**：使用 `pd.arrays.SparseArray()`。

  ```python
  import pandas as pd
  import numpy as np

  arr = pd.arrays.SparseArray([0, 0, 1, 0, 2], fill_value=0)
  print(arr)
  # [0, 0, 1, 0, 2]
  # Fill: 0
  # IntIndex
  # Indices: array([2, 4], dtype=int32)
  # Values:  array([1, 2])
  ```

- **从已有数组转换**：`pd.array(..., dtype='Sparse[int]')` 或 `Series.astype(pd.SparseDtype(...))`。

### 1.3 属性

| 属性 | 说明 |
|------|------|
| `sp_values` | 返回存储的非填充值的 NumPy 数组。 |
| `sp_index` | 返回有效值的位置索引（`IntIndex`、`BlockIndex` 等）。 |
| `fill_value` | 返回填充值。 |
| `npoints` | 返回有效值（非填充值）的数量。 |
| `density` | 返回有效值占总元素的比例。 |

```python
arr = pd.arrays.SparseArray([0, 1, 0, 0, 2], fill_value=0)
arr.sp_values    # array([1, 2])
arr.sp_index     # IntIndex(indices=array([1, 4], dtype=int32))
arr.fill_value   # 0
arr.npoints      # 2
arr.density      # 0.4
```

### 1.4 方法

- `to_dense()`：转换为普通的 NumPy 数组。
- `astype()`：改变 dtype（包括稀疏 dtype）。
- `copy()`：创建副本。
- `take()`：按位置选取元素，返回 SparseArray。
- 支持大多数 NumPy 通用函数（`np.sum`、`np.mean` 等），但结果可能为普通数组或标量。

## 2. SparseDtype

`SparseDtype` 是用于描述 `SparseArray` 的数据类型，包含底层 dtype 和填充值。

### 2.1 创建 SparseDtype

- 使用字符串简写：`'Sparse[int]'`、`'Sparse[float64]'`、`'Sparse[float, 0]'`。
- 使用构造函数：`pd.SparseDtype(dtype, fill_value)`。

```python
# 字符串形式
dtype1 = pd.SparseDtype('float64', fill_value=np.nan)
dtype2 = pd.SparseDtype('int', fill_value=0)
```

- 默认填充值为 `NaN`（对于浮点和对象类型）或 `0`（对于整数类型？实际上默认填充值取决于 dtype，通常为 `np.nan`）。

### 2.2 属性

| 属性 | 说明 |
|------|------|
| `dtype` | 底层数据类型（如 `np.float64`）。 |
| `fill_value` | 填充值。 |
| `kind` | 底层数据类型的种类，如 `'f'`、`'i'`、`'O'`。 |

```python
dtype = pd.SparseDtype('int64', fill_value=0)
dtype.dtype       # dtype('int64')
dtype.fill_value  # 0
dtype.kind        # 'i'
```

### 2.3 识别 SparseDtype

- 使用 `pd.api.types.is_sparse(dtype)` 判断。
- `Series.dtype` 会返回 SparseDtype 对象。

## 3. Series.sparse

`Series.sparse` 是一个访问器，专门用于 `dtype` 为 `SparseDtype` 的 Series，提供稀疏专用的操作和属性。

### 3.1 访问条件

只有当 Series 的 dtype 是 SparseDtype 时，`.sparse` 访问器才可用。例如：

```python
s = pd.Series([0, 0, 1, 0, 2]).astype(pd.SparseDtype('int', fill_value=0))
s.sparse
```

### 3.2 属性

| 属性 | 说明 |
|------|------|
| `density` | 稀疏密度（有效值比例）。 |
| `fill_value` | 当前填充值。 |
| `npoints` | 有效值数量。 |
| `sp_values` | 有效值的数组。 |
| `sp_index` | 有效值的位置索引。 |

```python
s.sparse.density     # 0.4
s.sparse.fill_value  # 0
s.sparse.npoints     # 2
```

### 3.3 方法

- `from_spmatrix()`：从 `scipy.sparse` 矩阵创建 Series。
- `to_coo()`：将稀疏 Series 转换为 `scipy.sparse.coo_matrix`。
- `to_dense()`：转换为普通稠密数组。

```python
from scipy import sparse
coo = sparse.coo_matrix(([10, 20], ([0, 2], [0, 0])), shape=(3,1))
s_from = pd.Series.sparse.from_spmatrix(coo)
s_to_coo = s.sparse.to_coo()
```

## 4. DataFrame.sparse

`DataFrame.sparse` 是稀疏 DataFrame 的访问器，提供与稀疏 DataFrame 相关的操作。

### 4.1 访问条件

DataFrame 的列数据类型可以是 SparseDtype，从而整个 DataFrame 可视为稀疏 DataFrame。当所有列均为稀疏 dtype 时，`.sparse` 访问器可用。

```python
df = pd.DataFrame({
    'A': pd.arrays.SparseArray([0, 1, 0]),
    'B': pd.arrays.SparseArray([2, 0, 0])
})
df.sparse
```

### 4.2 属性

| 属性 | 说明 |
|------|------|
| `density` | 整个 DataFrame 的稀疏密度（有效值比例）。 |

```python
df.sparse.density   # 计算所有非填充元素占总元素比例
```

### 4.3 方法

- `from_spmatrix()`：从 `scipy.sparse` 矩阵创建 DataFrame。
- `to_dense()`：将整个 DataFrame 转换为普通稠密 DataFrame。
- `to_coo()`：将稀疏 DataFrame 转换为 `scipy.sparse.coo_matrix`。

```python
sparse_matrix = sparse.coo_matrix(([1, 2], ([0, 1], [0, 1])), shape=(2,2))
df = pd.DataFrame.sparse.from_spmatrix(sparse_matrix)
df_dense = df.sparse.to_dense()
```

## 5. 创建和转换

### 5.1 创建稀疏数据

- **通过 `astype`**：
  ```python
  s = pd.Series([0, 0, 1, 0, 2])
  s_sparse = s.astype(pd.SparseDtype('int64', fill_value=0))
  ```
- **通过 `pd.array` 指定 dtype**：
  ```python
  arr = pd.array([0, 0, 1, 0, 2], dtype='Sparse[int]')
  ```
- **通过 `SparseArray` 直接构造**（见前文）。
- **从 `scipy.sparse` 导入**：使用 `Series.sparse.from_spmatrix()` 或 `DataFrame.sparse.from_spmatrix()`。
- **通过 `pd.get_dummies(..., sparse=True)`**：将分类变量转换为稀疏独热编码。

### 5.2 转换为稠密

- 使用 `.to_dense()` 方法（适用于 Series、DataFrame、SparseArray）。
- 使用 `astype` 转换回普通 dtype：
  ```python
  s_dense = s_sparse.astype('int64')  # 或 s_sparse.sparse.to_dense()
  ```

### 5.3 修改填充值

- 使用 `SparseArray.fill_value` 属性赋值（需创建新对象）或 `astype` 重新指定填充值。
- 注意：修改填充值会改变稀疏表示，但不会改变底层数据（即原来的填充值可能变成有效值）。

```python
s = pd.Series([0, 1, 0]).astype(pd.SparseDtype('int', fill_value=0))
s2 = s.astype(pd.SparseDtype('int', fill_value=1))
# 现在原来的 0 都变成了有效值，原来的 1 变成填充值
```

## 6. 密度与内存

### 6.1 密度

- 密度 = 有效值数量 / 总元素数量。
- 通过 `.density` 属性获取（SparseArray、Series.sparse、DataFrame.sparse）。
- 密度越低，稀疏存储的优势越明显。

### 6.2 内存效率

- 稀疏存储只保存有效值及其索引，当密度很低时，内存占用远小于稠密存储。
- 可通过 `.memory_usage(deep=True)` 对比稀疏和稠密对象的内存。

```python
import numpy as np

dense = pd.Series(np.zeros(1000000))
sparse = dense.astype(pd.SparseDtype('float64', fill_value=0))
print(dense.memory_usage(deep=True))   # 8000000 字节左右
print(sparse.memory_usage(deep=True))  # 可能只有几千字节
```

- 但要注意：如果数据密度很高（接近 1），稀疏存储反而可能增加内存开销（因为需要额外存储索引）。

### 6.3 存储格式

- `SparseArray` 内部索引类型：
  - `IntIndex`：适用于有效值分布无规律。
  - `BlockIndex`：适用于有效值成块出现（可节省索引内存）。
- pandas 会自动选择最优索引类型，也可在创建时指定 `index` 参数。

## 7. 稀疏方法

### 7.1 专用方法

- `to_dense()`：转换为稠密数组。
- `to_coo()`：转换为 `scipy.sparse.coo_matrix`。
- `from_spmatrix()`：从 `scipy.sparse` 矩阵创建。

### 7.2 通用函数兼容性

- 大多数 NumPy 函数可作用于稀疏对象，但返回结果可能是稠密数组或标量。
- 对于 Series/DataFrame 的方法，pandas 会尝试保持稀疏性，但某些操作会返回稠密结果（如 `groupby`、`apply` 等）。

```python
s = pd.Series([0, 1, 0]).astype('Sparse[int]')
s.sum()   # 返回 1，标量
s + 1     # 返回普通 Series（可能失去稀疏性）
```

### 7.3 与其他稀疏工具互操作

- 与 `scipy.sparse` 矩阵互转。
- 与 `sklearn` 等库兼容，可直接传入稀疏 DataFrame/Series。

## 8. 与其他 dtype 交互

### 8.1 类型转换

- 稀疏 dtype 与普通 dtype 之间可以互相转换。
- 从稀疏到稠密：`.astype('int64')` 或 `.to_dense()`。
- 从稠密到稀疏：`.astype(pd.SparseDtype(...))`。

### 8.2 算术运算

- 稀疏 Series 与普通 Series 运算时，结果通常为普通 Series（稠密），因为稀疏性可能难以维持。
- 稀疏 Series 与标量运算：如果标量不是填充值，结果可能变为稠密；若标量与填充值一致，可能保持稀疏。

```python
s = pd.Series([0, 1, 0]).astype('Sparse[int]')
s + 0    # 保持稀疏
s + 2    # 可能变为稠密
```

### 8.3 与其他扩展类型

- 可以与 `Categorical`、`Int64` 等扩展 dtype 共存于 DataFrame 中。
- 但同一列不能同时是稀疏和其他扩展类型（即一列只能有一个 dtype）。

### 8.4 分组与聚合

- 对稀疏列进行 `groupby` 时，分组键必须是稠密类型；聚合结果可能是稠密或稀疏取决于操作。
- 稀疏列参与 `merge`、`concat` 等操作时，通常保持稀疏性（如果可能）。

> [!tip] 总结
> 稀疏数据是处理高维稀疏特征的利器，尤其在独热编码、推荐系统等场景。pandas 通过 `SparseArray`、`SparseDtype` 和对应的访问器提供了完整的稀疏存储、转换和计算方法，同时与 `scipy.sparse` 无缝集成。理解密度与内存的关系有助于在实际应用中做出合适的存储选择。