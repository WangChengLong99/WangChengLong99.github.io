---
title: 12 可空数据类型
---

> [!abstract] 本节大纲
> - pd.NA
> - Int64、Int32、Int16、Int8
> - UInt64、UInt32、UInt16、UInt8
> - Float64、Float32
> - Boolean
> - String
> - 算术运算中的传播
> - 与 NumPy 转换
> - 与 groupby 交互
> - 与 ArrowDtype 交互
> - pd.options.future.infer_string

## 1. pd.NA

`pd.NA` 是 pandas 引入的**通用缺失值标识符**，用于表示可空数据类型中的缺失值。

### 1.1 特性
- 与 `np.nan` 不同，`pd.NA` 可以用于整数、布尔、字符串等可空类型，而不会引发类型转换错误。
- `pd.NA` 是 `pandas._libs.missing.NAType` 的实例，全局唯一。
- 使用 `pd.isna(pd.NA)` 返回 `True`。

### 1.2 与 np.nan 和 None 的区别
| 类型 | np.nan | None | pd.NA |
|------|--------|------|-------|
| 类型 | float | NoneType | NAType |
| 可用于整数列 | 会强制转为 float | 会强制转为 object 或 float | 保持整数可空类型 |
| 可用于布尔列 | 会转为 object | 会转为 object | 保持布尔可空类型 |
| 可用于字符串列 | 转为 object | 转为 object | 保持字符串可空类型 |

### 1.3 示例
```python
import pandas as pd

s = pd.Series([1, pd.NA, 3], dtype="Int64")
print(s)
# 0       1
# 1    <NA>
# 2       3
# dtype: Int64
```

## 2. Int64、Int32、Int16、Int8

这些是 pandas 提供的**可空整数类型**，允许整数列中包含缺失值，同时保持整数存储。

### 2.1 类型列表
| 类型字符串 | 对应 pandas dtype | 对应 NumPy dtype |
|-----------|-------------------|------------------|
| `'Int64'` | `Int64Dtype()` | `np.int64` |
| `'Int32'` | `Int32Dtype()` | `np.int32` |
| `'Int16'` | `Int16Dtype()` | `np.int16` |
| `'Int8'`  | `Int8Dtype()`  | `np.int8` |

### 2.2 创建可空整数 Series
```python
s = pd.Series([1, 2, None], dtype="Int64")
print(s)
# 0       1
# 1       2
# 2    <NA>
# dtype: Int64
```

### 2.3 优势
- 避免将整数列提升为 `float64`。
- 支持缺失值 `pd.NA`（或 `None`、`np.nan`，它们会被转换为 `pd.NA`）。
- 可与其他可空类型混合。

### 2.4 注意事项
- 大小写敏感：`'Int64'` 可空，`'int64'` 不可空（NumPy 类型）。
- 运算时缺失值传播规则见下文。

## 3. UInt64、UInt32、UInt16、UInt8

这些是 pandas 提供的**无符号可空整数类型**，允许无符号整数列包含缺失值。

### 3.1 类型列表
| 类型字符串 | 对应 pandas dtype | 对应 NumPy dtype |
|-----------|-------------------|------------------|
| `'UInt64'` | `UInt64Dtype()` | `np.uint64` |
| `'UInt32'` | `UInt32Dtype()` | `np.uint32` |
| `'UInt16'` | `UInt16Dtype()` | `np.uint16` |
| `'UInt8'`  | `UInt8Dtype()`  | `np.uint8` |

### 3.2 示例
```python
s = pd.Series([1, 2, pd.NA], dtype="UInt8")
print(s)
# 0       1
# 1       2
# 2    <NA>
# dtype: UInt8
```

### 3.3 用途
- 需要无符号整数且可能包含缺失值的场景（例如像素值、计数等）。

## 4. Float64、Float32

pandas 提供可空浮点类型 `Float64` 和 `Float32`，与 NumPy 的 `float64`/`float32` 类似，但显式支持 `pd.NA`。

### 4.1 类型列表
| 类型字符串 | 对应 pandas dtype | 对应 NumPy dtype |
|-----------|-------------------|------------------|
| `'Float64'` | `Float64Dtype()` | `np.float64` |
| `'Float32'` | `Float32Dtype()` | `np.float32` |

### 4.2 示例
```python
s = pd.Series([1.5, 2.5, pd.NA], dtype="Float64")
print(s)
# 0     1.5
# 1     2.5
# 2    <NA>
# dtype: Float64
```

### 4.3 与普通浮点类型的区别
- 普通 `'float64'` 类型使用 `np.nan` 表示缺失值，而 `'Float64'` 使用 `pd.NA`。
- 运算中 `pd.NA` 的传播行为与 `np.nan` 类似，但更一致（详见“算术运算中的传播”）。

## 5. Boolean

`Boolean` 是 pandas 提供的可空布尔类型，允许布尔列中包含 `True`、`False` 和缺失值 `pd.NA`。

### 5.1 创建
```python
s = pd.Series([True, False, pd.NA], dtype="boolean")
print(s)
# 0     True
# 1    False
# 2     <NA>
# dtype: boolean
```

### 5.2 逻辑运算
- 可空布尔类型支持逻辑运算（`&`、`|`、`~`），但遵循三值逻辑（Kleene 逻辑）：
  - `True & pd.NA` -> `pd.NA`
  - `False & pd.NA` -> `False`
  - `True | pd.NA` -> `True`
  - `False | pd.NA` -> `pd.NA`
  - `~pd.NA` -> `pd.NA`

### 5.3 与普通 bool 的区别
- 普通 `'bool'` 类型无法存储缺失值，一旦出现缺失会转为 `object` 或 `float64`。
- `'boolean'` 类型保持布尔语义并支持缺失。

## 6. String

pandas 提供可空字符串类型 `'string'`，用于存储文本数据并支持缺失值。

### 6.1 创建
```python
s = pd.Series(["a", "b", pd.NA], dtype="string")
print(s)
# 0       a
# 1       b
# 2    <NA>
# dtype: string
```

### 6.2 优势
- 比 `object` 类型更明确，支持 `StringDtype`。
- 支持 `Series.str` 访问器进行字符串操作。
- 与 `pd.NA` 集成良好。

### 6.3 字符串存储后端
- 默认使用 Python 对象存储（`'python'`）。
- 可通过 `pd.options.mode.string_storage = 'pyarrow'` 切换到 PyArrow 后端，提升性能。
- 未来 `pd.options.future.infer_string = True` 可自动推断字符串为 `string` 类型。

## 7. 算术运算中的传播

可空数据类型在算术运算中遵循**缺失值传播规则**：任何涉及 `pd.NA` 的运算结果均为 `pd.NA`。

### 7.1 基本规则
- `pd.NA + 1` -> `pd.NA`
- `pd.NA * 0` -> `pd.NA`（与 `np.nan` 不同，`np.nan * 0` 为 `np.nan`，而 `pd.NA` 严格传播缺失）
- `pd.NA / 0` -> `pd.NA`

### 7.2 比较运算
- 比较结果通常为 `pd.NA`，例如 `pd.NA > 1` -> `pd.NA`。
- 但 `pd.NA == pd.NA` 返回 `pd.NA`（不是 `True`），因为缺失值不可比较。

### 7.3 聚合函数中的处理
- 大多数聚合函数（如 `sum`、`mean`）默认忽略 `pd.NA`，使用 `skipna=True`。
- 如果所有值都是缺失，则结果通常为 `pd.NA`。
- 可以通过 `skipna=False` 让聚合传播缺失。

### 7.4 与 NumPy 函数的交互
- 当可空数组传递给 NumPy 函数时，可能会转换为 object 或普通类型，导致意外的 `pd.NA` 处理。建议使用 pandas 提供的方法。

## 8. 与 NumPy 转换

可空数据类型与 NumPy 原生类型之间的转换需要特别注意缺失值的处理。

### 8.1 可空 -> NumPy
- 使用 `Series.to_numpy()` 或 `np.asarray()`：
  - 对于整数可空类型，缺失值会被转换为 `np.nan`（导致数组 dtype 变为 `float64`）。
  - 对于布尔可空类型，缺失值转换为 `np.nan`，dtype 变为 `object` 或 `float64`。
  - 对于字符串可空类型，缺失值转换为 `None`，dtype 变为 `object`。

```python
s = pd.Series([1, pd.NA], dtype="Int64")
s.to_numpy()          # array([1., nan])
s.to_numpy(dtype=object)  # array([1, <NA>], dtype=object)
```

- 使用 `Series.to_numpy(dtype=...)` 可以控制输出类型，但缺失值处理需谨慎。

### 8.2 NumPy -> 可空
- 使用 `pd.array()` 或 `Series.astype()` 转换：
  - 如果 NumPy 数组包含 `np.nan`，转换为可空整数类型时，`np.nan` 会被视为缺失值并转为 `pd.NA`。
  - 布尔数组转换为可空布尔类型时，只有 `True`/`False` 保留，其他值（如 `np.nan`）转换为 `pd.NA`。

```python
arr = np.array([1, np.nan, 3])
s = pd.Series(arr, dtype="Int64")  # 第二个元素变为 <NA>
```

### 8.3 注意事项
- 转换可能产生意想不到的缺失值（例如原数组中的 `np.nan` 或 `None`）。
- 建议显式使用 `pd.array(..., dtype='Int64')` 并检查缺失情况。

## 9. 与 groupby 交互

可空类型列在 `groupby` 操作中的行为与普通列类似，但缺失值处理更一致。

### 9.1 分组键
- 可空列作为分组键时，`pd.NA` 被视为一个独立的分组（默认 `dropna=True` 会排除缺失值分组）。
- 使用 `groupby(dropna=False)` 可以保留 `pd.NA` 分组。

```python
df = pd.DataFrame({'key': pd.Series(['a', pd.NA, 'a'], dtype='string'),
                   'value': [1, 2, 3]})
df.groupby('key', dropna=False).sum()
#       value
# key
# a        4
# <NA>     2
```

### 9.2 聚合
- 对可空数值列进行聚合（如 `sum`、`mean`）时，缺失值按 `skipna=True` 忽略。
- 聚合结果仍保留可空 dtype（如果可能）。

### 9.3 转换与过滤
- `transform`、`filter` 等方法对缺失值的处理与普通类型一致。

## 10. 与 ArrowDtype 交互

pandas 2.0+ 引入了 `ArrowDtype`，支持 Apache Arrow 内存格式。可空类型与 ArrowDtype 有紧密关系。

### 10.1 ArrowDtype 简介
- `pd.ArrowDtype` 是基于 PyArrow 的扩展 dtype，可用于存储字符串、整数、浮点等，并支持缺失值。
- 使用 `dtype='string[pyarrow]'`、`'int64[pyarrow]'` 等指定。
- 可空整数类型（`Int64`）在 PyArrow 后端下等价于 `int64[pyarrow]`。

### 10.2 转换
- 可空类型可以与 ArrowDtype 互相转换。
```python
s = pd.Series([1, pd.NA], dtype="Int64")
s_arrow = s.astype("int64[pyarrow]")
# 或者 s.convert_dtypes(dtype_backend='pyarrow')
```

### 10.3 优势
- ArrowDtype 提供更高效的内存使用和 I/O。
- 可空类型和 ArrowDtype 都使用 `pd.NA` 作为缺失值标识。

### 10.4 注意事项
- 使用 ArrowDtype 需要安装 `pyarrow`。
- 部分操作可能在 ArrowDtype 下返回不同 dtype，需留意。

## 11. pd.options.future.infer_string

`pd.options.future.infer_string` 是一个**未来行为开关**，控制是否自动将对象字符串推断为 `string` 类型。

### 11.1 默认值
- 当前（pandas 2.3.3）默认值为 `False`，保持向后兼容：字符串数据默认使用 `object` dtype。

### 11.2 设置方法
```python
pd.options.future.infer_string = True
# 或使用 pd.set_option('future.infer_string', True)
```

### 11.3 效果
- 当 `infer_string=True` 时，`pd.read_csv()`、`DataFrame()` 等会自动将字符串列推断为 `string` dtype（而非 `object`）。
- 这会影响字符串列的行为，例如更严格的类型、更好的性能（如果使用 PyArrow 后端）。

### 11.4 应用场景
- 提前适应 pandas 3.0 中可能成为默认的行为。
- 在需要明确字符串类型、避免 object 类型带来的意外行为时使用。

### 11.5 注意事项
- 启用后，字符串列可能不再与某些旧代码完全兼容（例如依赖 `object` 类型的场景）。
- 建议在开发新项目时开启，并测试兼容性。

> [!tip] 总结
> 可空数据类型（nullable dtypes）是 pandas 2.x 中处理缺失值的重要工具。它们通过 `pd.NA` 提供了统一的缺失值表示，解决了传统 NumPy 类型无法在整数、布尔等列中存储缺失值的问题。理解可空类型的创建、传播、转换和与 ArrowDtype 的交互，有助于编写更健壮、高效的数据处理代码。
```