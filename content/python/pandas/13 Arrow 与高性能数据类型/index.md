---
title: 13 Arrow 与高性能数据类型
---

> [!abstract] 本节大纲
> - pyarrow 集成
> - pd.ArrowDtype
> - dtype_backend='pyarrow'
> - Series.astype('int64[pyarrow]')
> - 字符串操作性能
> - I/O 与 pyarrow
> - 与其他 DataFrame 库互操作

## 1. pyarrow 集成

### 1.1 什么是 PyArrow？
PyArrow 是 Apache Arrow 的 Python 实现，提供了一个跨语言的内存列式数据格式。它优化了数据分析的性能和内存使用，支持高效的序列化、I/O 和零拷贝数据共享。

### 1.2 pandas 与 PyArrow 的关系
从 pandas 2.0 开始，pandas 引入了对 PyArrow 的深度集成，允许用户将 DataFrame 或 Series 的数据类型指定为 Arrow 后端类型。这意味着数据在内存中以 Arrow 格式存储，而不是传统的 NumPy 数组或 Python 对象。

- **安装**：需要单独安装 `pyarrow` 包。
  ```bash
  pip install pyarrow
  ```
- **检查版本**：`pd.show_versions()` 会显示 pyarrow 版本。

### 1.3 集成带来的优势
- **更高效的内存使用**：Arrow 使用列式存储，对字符串和嵌套类型尤其节省内存。
- **更快的 I/O**：读写 Parquet、Feather 等格式时可直接与 Arrow 互操作，减少转换开销。
- **跨语言互操作**：Arrow 格式可以被多种语言（如 R、Java、C++）直接使用，无需序列化。
- **更好的字符串性能**：Arrow 字符串类型使用 UTF-8 编码和字典编码优化，比 Python 对象更紧凑。

### 1.4 注意事项
- 并非所有 pandas 功能都完全支持 Arrow 类型，部分操作可能回退到 object 类型或引发错误。
- 使用 Arrow 后端时，缺失值统一使用 `pd.NA` 表示，而不是 `np.nan` 或 `None`。
- 某些第三方库可能不识别 Arrow 类型，需要转换为普通类型。

## 2. pd.ArrowDtype

`pd.ArrowDtype` 是 pandas 提供的**扩展数据类型**，用于表示一个 PyArrow 类型。它允许 DataFrame 的列使用 Arrow 内存布局。

### 2.1 创建 ArrowDtype
- 通过字符串简写：`'int64[pyarrow]'`、`'string[pyarrow]'`、`'double[pyarrow]'` 等。
- 直接使用 `pd.ArrowDtype(pa_type)`，其中 `pa_type` 是 PyArrow 类型对象。

```python
import pandas as pd
import pyarrow as pa

# 使用字符串
dtype1 = pd.ArrowDtype(pa.int64())
# 等效于
dtype2 = pd.ArrowDtype('int64')

# 字符串简写
dtype3 = pd.ArrowDtype('string')
```

### 2.2 ArrowDtype 的属性
| 属性 | 说明 |
|------|------|
| `type` | 返回底层的 PyArrow 类型（如 `pa.int64()`）。 |
| `name` | 返回字符串表示，如 `'int64[pyarrow]'`。 |
| `kind` | 返回类型的种类，如 `'i'`、`'f'`、`'U'`。 |

```python
dt = pd.ArrowDtype(pa.string())
print(dt.type)   # DataType(string)
print(dt.name)   # string[pyarrow]
```

### 2.3 识别 ArrowDtype
- 使用 `pd.api.types.is_arrow_dtype(dtype)` 判断。
- 或者检查 `isinstance(dtype, pd.ArrowDtype)`。

```python
s = pd.Series([1, 2, 3], dtype='int64[pyarrow]')
pd.api.types.is_arrow_dtype(s.dtype)   # True
```

### 2.4 支持的 PyArrow 类型
- 数值类型：`int8, int16, int32, int64, uint8, uint16, uint32, uint64, float32, float64`
- 字符串类型：`string`（等效于 `large_string`？pandas 默认使用 `string`）
- 布尔类型：`bool`
- 时间类型：`timestamp[ns]`, `date32`, `date64`, `time32`, `time64`, `duration[ns]`
- 其他：`decimal128`, `list`, `struct`, `map` 等（有限支持）

## 3. dtype_backend='pyarrow'

`dtype_backend` 参数出现在多个 pandas 函数中，用于指定数据类型推断或转换时使用 PyArrow 后端。

### 3.1 常见使用场景
- `read_csv(..., dtype_backend='pyarrow')`：读取 CSV 时将推断的字符串等类型使用 Arrow 后端。
- `convert_dtypes(dtype_backend='pyarrow')`：将 DataFrame 转换为更适合的 dtype，优先使用 Arrow 类型。
- `read_parquet(..., dtype_backend='pyarrow')`：读取 Parquet 文件时直接使用 Arrow 类型。
- `merge`, `concat` 等操作后，可以通过 `convert_dtypes` 重新推断。

### 3.2 示例
```python
df = pd.read_csv('data.csv', dtype_backend='pyarrow')
print(df.dtypes)
# col1    string[pyarrow]
# col2     int64[pyarrow]
```

### 3.3 工作原理
- 当指定 `dtype_backend='pyarrow'` 时，pandas 会尝试使用 `pyarrow` 推断出合适的 Arrow 类型，而不是默认的 `object`、`int64` 等。
- 如果无法推断出 Arrow 类型（例如混合类型），则回退到普通类型或引发错误（取决于具体函数）。

### 3.4 与 `convert_dtypes()` 结合
```python
df = df.convert_dtypes(dtype_backend='pyarrow')
```
该方法会分析每列数据，尽量将其转换为 Arrow 类型（如字符串转为 `string[pyarrow]`，整数转为 `int64[pyarrow]`）。

## 4. Series.astype('int64[pyarrow]')

通过 `.astype()` 方法，可以将 Series 或 DataFrame 列转换为 Arrow 后端类型。

### 4.1 语法
```python
s.astype('int64[pyarrow]')
df.astype({'col1': 'string[pyarrow]', 'col2': 'int64[pyarrow]'})
```

### 4.2 支持的类型字符串
- 数值：`'int8[pyarrow]'`, `'int16[pyarrow]'`, `'int32[pyarrow]'`, `'int64[pyarrow]'`, 相应的无符号和浮点类型。
- 字符串：`'string[pyarrow]'`
- 布尔：`'bool[pyarrow]'`
- 时间：`'timestamp[ns][pyarrow]'`, `'date32[pyarrow]'` 等。

### 4.3 示例
```python
s = pd.Series([1, 2, 3])
s_arrow = s.astype('int64[pyarrow]')
print(s_arrow.dtype)   # int64[pyarrow]

s_str = pd.Series(['a', 'b', None])
s_str_arrow = s_str.astype('string[pyarrow]')
print(s_str_arrow.dtype)   # string[pyarrow]
```

### 4.4 注意事项
- 转换时，缺失值会转换为 `pd.NA`。
- 如果数据中包含无法转换的值（如字符串列转换为整数），会引发错误。
- 转换后，Series 的 `values` 属性可能返回一个 `ArrowExtensionArray`，而不是 NumPy 数组。

## 5. 字符串操作性能

Arrow 字符串类型（`string[pyarrow]`）相比传统的 Python 对象字符串（`object`）在内存和性能上有显著优势。

### 5.1 内存效率
- Python 对象存储每个字符串为独立的 Python 对象，每个对象有额外开销（约 49 字节以上）。
- Arrow 字符串使用连续内存存储 UTF-8 字节，并配合偏移量数组，大幅减少内存占用。
- 对于包含大量重复字符串的列，Arrow 还可以使用字典编码进一步压缩。

### 5.2 操作性能
- 字符串方法（如 `.str.upper()`, `.str.contains()`）在 Arrow 后端下由 PyArrow 内核执行，通常比 Python 对象上的操作更快。
- 筛选、分组等操作也因内存紧凑而提升缓存命中率。

### 5.3 示例对比
```python
import numpy as np
import pandas as pd

# 创建大型字符串 Series
n = 1_000_000
s_obj = pd.Series(['abc' + str(i % 1000) for i in range(n)], dtype=object)
s_arrow = s_obj.astype('string[pyarrow]')

print('object memory:', s_obj.memory_usage(deep=True))
print('arrow memory:', s_arrow.memory_usage(deep=True))

# 性能测试（示例）
%timeit s_obj.str.upper()
%timeit s_arrow.str.upper()
```

### 5.4 注意事项
- 某些复杂的字符串操作可能尚未在 Arrow 后端完全优化，可能会回退到 Python 对象计算。
- 如果代码依赖于 `object` 类型的特定行为，切换到 Arrow 可能产生细微差异。

## 6. I/O 与 pyarrow

PyArrow 后端显著提升了 pandas 在读写特定文件格式时的性能和兼容性。

### 6.1 Parquet
- **读取**：`pd.read_parquet(path, engine='pyarrow', dtype_backend='pyarrow')`
  - 直接返回 Arrow 类型，无需额外转换。
- **写入**：`df.to_parquet(path, engine='pyarrow')`
  - 保存为 Parquet 格式，支持高效压缩和列式存储。

### 6.2 Feather
- **读取**：`pd.read_feather(path, dtype_backend='pyarrow')`
- **写入**：`df.to_feather(path)`
- Feather 格式专为 Arrow 设计，零拷贝读取速度极快。

### 6.3 CSV
- 使用 `dtype_backend='pyarrow'` 可以让 `read_csv()` 推断 Arrow 类型。
- 但 CSV 解析本身不是 Arrow 原生操作，所以性能提升主要来自后续处理，而非解析速度。

### 6.4 其他格式
- Arrow 类型也可以用于 `to_json()`、`to_sql()` 等，但优势不明显。
- 使用 `engine='pyarrow'` 在 `read_json()` 中可获得加速。

### 6.5 与原生 Arrow 的互操作
- 可以直接将 DataFrame 转换为 Arrow Table：`df.to_parquet()` 内部使用 Arrow。
- 也可以从 Arrow Table 创建 DataFrame：
  ```python
  import pyarrow.parquet as pq
  table = pq.read_table('data.parquet')
  df = table.to_pandas()   # 默认转换为普通类型
  df_arrow = table.to_pandas(types_mapper=pd.ArrowDtype)  # 保留 Arrow 类型
  ```

## 7. 与其他 DataFrame 库互操作

Arrow 格式作为通用数据交换标准，使得 pandas 能更高效地与其它数据科学库协作。

### 7.1 Polars
- Polars 是一个基于 Arrow 的 DataFrame 库，与 pandas 可以零拷贝转换。
- `polars.from_pandas(df)` 默认保留 Arrow 类型（如果 df 是 Arrow 后端）。
- `df.to_pandas()` 从 Polars 转换回 pandas，可通过参数保留 Arrow 类型。

```python
import polars as pl
pdf = pl.DataFrame({'a': [1, 2, 3]}).to_pandas(types_mapper=pd.ArrowDtype)
```

### 7.2 DuckDB
- DuckDB 支持直接查询 pandas DataFrame，Arrow 类型有助于加速传输。
- `duckdb.query_df()` 或 `duckdb.from_df()` 可识别 Arrow 后端。

### 7.3 Apache Spark
- Spark 与 pandas 的转换（如 `toPandas()`）可以通过 Arrow 加速（`spark.sql.execution.arrow.pyspark.enabled=true`）。
- 使用 Arrow 类型可减少转换开销。

### 7.4 Vaex / Dask
- Vaex 和 Dask 也支持与 Arrow 互操作，通过共享 Arrow 内存避免复制。

### 7.5 与 NumPy 的交互
- 虽然 Arrow 类型不是 NumPy 原生，但 `Series.to_numpy()` 会尝试转换（字符串列转换为 object 数组，数值列转换为 NumPy 数组）。
- 对于数值 Arrow 列，转换通常是零拷贝的（底层共享内存）。

> [!tip] 总结
> PyArrow 集成为 pandas 带来了高性能的数据类型和 I/O 能力。通过 `pd.ArrowDtype` 和 `dtype_backend='pyarrow'`，用户可以轻松利用 Arrow 的内存优势和跨语言互操作性。字符串操作性能显著提升，文件读写（尤其是 Parquet 和 Feather）更加高效，同时还能与 Polars、DuckDB 等现代 DataFrame 库无缝协作，构建高性能数据分析流水线。
```