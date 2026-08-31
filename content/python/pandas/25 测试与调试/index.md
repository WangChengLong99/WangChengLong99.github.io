---
title: 25 测试与调试
---


> [!abstract] 本节大纲
> - pandas.testing
>     - assert_frame_equal()
>     - assert_series_equal()
>     - assert_index_equal()
>     - assert_extension_array_equal()
>     - 参数
>         - check_dtype、check_index_type、check_column_type、check_frame_type、check_names、check_exact、rtol、atol、check_like、obj
> - 调试技巧
>     - df.info()
>     - df.head()
>     - pd.set_option('display.max_columns', None)
>     - df.dtypes
>     - df.describe(include='all')
>     - 常见错误排查

## 1. pandas.testing

`pandas.testing` 模块提供了一系列用于**单元测试和断言**的函数，帮助开发者验证 DataFrame、Series、Index 等对象的结构和值是否符合预期。

### 1.1 assert_frame_equal()

- **用途**：断言两个 DataFrame 相等。
- **签名**：`pandas.testing.assert_frame_equal(left, right, check_dtype=True, check_index_type='equiv', check_column_type='equiv', check_frame_type=True, check_names=True, check_exact=False, rtol=1e-05, atol=1e-08, check_like=False, obj='DataFrame')`
- **参数详解**：
  - `left` / `right`：要比较的两个 DataFrame。
  - `check_dtype`：是否检查 dtype 完全一致，默认 `True`。
  - `check_index_type`：索引类型检查级别，可选 `'equiv'`（等价即可，如 `RangeIndex` 与 `Int64Index` 可等价）或 `'exact'`（类型严格一致）。
  - `check_column_type`：列索引类型检查级别，同 `check_index_type`。
  - `check_frame_type`：是否检查对象类型必须是 DataFrame，默认 `True`（防止传入 Series 等）。
  - `check_names`：是否检查行索引和列索引的 `name` 属性，默认 `True`。
  - `check_exact`：是否精确比较数值，默认 `False`（使用 `rtol` 和 `atol` 进行近似比较）。
  - `rtol` / `atol`：相对容差和绝对容差，用于浮点数比较。
  - `check_like`：如果为 `True`，则忽略列的顺序（按标签对齐），默认 `False`。
  - `obj`：错误消息中显示的对象名称，默认 `'DataFrame'`。
- **示例**：
  ```python
  import pandas as pd
  from pandas.testing import assert_frame_equal

  df1 = pd.DataFrame({'A': [1.0, 2.0], 'B': [3.0, 4.0]})
  df2 = pd.DataFrame({'A': [1.0000001, 2.0], 'B': [3.0, 4.0]})
  assert_frame_equal(df1, df2, check_exact=False, rtol=1e-5)  # 通过
  assert_frame_equal(df1, df2, check_exact=True)               # 失败，因为值不完全相等
  ```

### 1.2 assert_series_equal()

- **用途**：断言两个 Series 相等。
- **签名**：`pandas.testing.assert_series_equal(left, right, check_dtype=True, check_index_type='equiv', check_series_type=True, check_names=True, check_exact=False, check_datetimelike_compat=False, check_categorical=True, check_category_order=True, check_freq=True, check_flags=True, rtol=1e-05, atol=1e-08, obj='Series', check_like=False)`
- **重要参数**（除与 `assert_frame_equal` 共有的外）：
  - `check_series_type`：是否检查对象必须是 Series，默认 `True`。
  - `check_datetimelike_compat`：是否比较时间相关 dtype 的等价性（如 `datetime64[ns]` 与 `datetime64[us]` 可能视为等价），默认 `False`。
  - `check_categorical`：是否检查分类类型，默认 `True`。
  - `check_category_order`：是否检查分类顺序，默认 `True`。
  - `check_freq`：对于时间索引，是否检查频率，默认 `True`。
  - `check_flags`：是否检查 `flags` 属性，默认 `True`。
- **示例**：
  ```python
  from pandas.testing import assert_series_equal

  s1 = pd.Series([1, 2, 3], name='s')
  s2 = pd.Series([1, 2, 3], name='s')
  assert_series_equal(s1, s2)  # 通过
  ```

### 1.3 assert_index_equal()

- **用途**：断言两个 Index 相等。
- **签名**：`pandas.testing.assert_index_equal(left, right, exact='equiv', check_names=True, check_exact=False, check_categorical=True, check_order=True, rtol=1e-05, atol=1e-08, obj='Index')`
- **参数**：
  - `exact`：类型检查级别，`'equiv'` 或 `True`（严格）或 `False`（不检查）。
  - `check_names`：是否检查 `name` 属性。
  - `check_exact`：是否精确比较数值。
  - `check_categorical`：是否检查分类索引。
  - `check_order`：是否检查元素顺序，默认 `True`（若设为 `False` 则只比较集合，忽略顺序）。
  - `rtol` / `atol`：浮点容差。
  - `obj`：错误消息中显示的对象名称。
- **示例**：
  ```python
  from pandas.testing import assert_index_equal

  idx1 = pd.Index([1, 2, 3])
  idx2 = pd.Index([1, 2, 3])
  assert_index_equal(idx1, idx2)  # 通过
  ```

### 1.4 assert_extension_array_equal()

- **用途**：断言两个扩展数组（ExtensionArray）相等，例如 `Categorical`、`SparseArray`、可空类型数组等。
- **签名**：`pandas.testing.assert_extension_array_equal(left, right, check_dtype=True, index=None, check_exact=False, rtol=1e-05, atol=1e-08, obj='ExtensionArray')`
- **参数**：
  - `left` / `right`：要比较的扩展数组。
  - `check_dtype`：是否检查 dtype。
  - `index`：可选，用于提供额外的索引信息进行对齐（若数组有索引）。
  - `check_exact`、`rtol`、`atol`：同其他断言。
  - `obj`：错误消息中的对象名称。
- **示例**：
  ```python
  from pandas.testing import assert_extension_array_equal

  arr1 = pd.array([1, 2, None], dtype='Int64')
  arr2 = pd.array([1, 2, None], dtype='Int64')
  assert_extension_array_equal(arr1, arr2)  # 通过
  ```

### 1.5 参数汇总

> [!note] 常用参数总结
> | 参数 | 用途 | 默认值 |
> |------|------|--------|
> | `check_dtype` | 是否检查 dtype 完全一致 | `True` |
> | `check_index_type` | 索引类型检查：`'equiv'` 或 `'exact'` | `'equiv'` |
> | `check_column_type` | 列索引类型检查（仅 DataFrame） | `'equiv'` |
> | `check_frame_type` | 检查对象类型必须为 DataFrame | `True` |
> | `check_names` | 检查索引的 `name` 属性 | `True` |
> | `check_exact` | 是否精确比较数值（`False` 则用容差） | `False` |
> | `rtol` | 相对容差 | `1e-05` |
> | `atol` | 绝对容差 | `1e-08` |
> | `check_like` | 忽略列顺序（按标签对齐） | `False` |
> | `obj` | 错误消息中显示的对象名称 | 各函数默认不同 |
>
> 这些参数在 `assert_frame_equal`、`assert_series_equal`、`assert_index_equal` 等函数中含义基本一致。

## 2. 调试技巧

调试 pandas 代码时，以下方法和技巧可以帮助快速了解数据结构和定位问题。

### 2.1 df.info()

- **用途**：打印 DataFrame 的简要信息，包括行数、列数、每列的非空值数量、dtype 和内存使用情况。
- **语法**：`DataFrame.info(verbose=None, buf=None, max_cols=None, memory_usage=None, show_counts=None)`
- **详细说明**：
  - 显示每列的名称、非空数量、dtype。
  - 可选显示内存占用（`memory_usage='deep'` 可包含对象列的实际内存）。
  - 适用于快速了解数据规模、缺失情况和类型。
- **示例**：
  ```python
  df.info()
  # <class 'pandas.core.frame.DataFrame'>
  # RangeIndex: 1000 entries, 0 to 999
  # Data columns (total 4 columns):
  #  #   Column  Non-Null Count  Dtype
  # ---  ------  --------------  -----
  #  0   A       1000 non-null   int64
  #  1   B       950 non-null    float64
  #  2   C       1000 non-null   object
  #  3   D       800 non-null    datetime64[ns]
  # dtypes: datetime64[ns](1), float64(1), int64(1), object(1)
  # memory usage: 31.4+ KB
  ```

### 2.2 df.head()

- **用途**：查看 DataFrame 的前几行数据，默认显示 5 行。
- **语法**：`DataFrame.head(n=5)`
- **参数**：`n` 为显示的行数。
- **详细说明**：快速预览数据内容、列名、值格式等。
- **示例**：
  ```python
  df.head(3)
  ```

### 2.3 pd.set_option('display.max_columns', None)

- **用途**：设置 pandas 显示选项，取消列数限制，让所有列都显示在输出中。
- **详细说明**：
  - 默认情况下，pandas 会截断列数（如只显示前 20 列），中间用省略号。
  - 将 `display.max_columns` 设为 `None` 可以显示全部列，方便查看宽表。
  - 同时还可以调整 `display.max_rows`、`display.width`、`display.max_colwidth` 等。
- **示例**：
  ```python
  pd.set_option('display.max_columns', None)  # 显示所有列
  print(df)
  ```
- **恢复默认**：`pd.reset_option('display.max_columns')`。

### 2.4 df.dtypes

- **用途**：查看 DataFrame 每列的数据类型。
- **语法**：`DataFrame.dtypes`（属性）
- **详细说明**：返回一个 Series，索引为列名，值为该列的 dtype。对于混合类型，显示 `object` 或具体扩展类型（如 `Int64`、`string`）。
- **示例**：
  ```python
  print(df.dtypes)
  # A             int64
  # B           float64
  # C            object
  # D    datetime64[ns]
  # dtype: object
  ```

### 2.5 df.describe(include='all')

- **用途**：生成描述性统计摘要，`include='all'` 会同时包含数值列和非数值列。
- **语法**：`DataFrame.describe(percentiles=None, include=None, exclude=None)`
- **参数**：
  - `include`：指定要包含的数据类型，如 `'all'`、`'number'`、`'object'`、`'category'` 或列表。
  - `percentiles`：自定义百分位数列表，默认 `[.25, .5, .75]`。
  - `exclude`：要排除的类型。
- **详细说明**：
  - 对于数值列：显示 count、mean、std、min、25%、50%、75%、max。
  - 对于非数值列：显示 count、unique、top（最高频值）、freq（频率）。
  - `include='all'` 时，输出包含所有列，数值列和非数值列分开显示统计量。
- **示例**：
  ```python
  df.describe(include='all')
  ```

### 2.6 常见错误排查

以下是一些 pandas 编程中常见的错误、警告及其排查方法。

#### 2.6.1 SettingWithCopyWarning

- **现象**：进行链式赋值或切片赋值时触发警告。
- **原因**：试图在可能为副本的对象上赋值，修改可能不会影响原始数据。
- **排查**：
  - 使用 `.loc` 或 `.iloc` 一次性完成选择与赋值。
  - 启用 Copy-on-Write（pandas 2.3 默认启用）后，链式赋值会直接报错。
  - 如果确实需要副本，显式调用 `.copy()`。

#### 2.6.2 KeyError / IndexError

- **现象**：使用不存在的列名或索引标签时抛出。
- **排查**：
  - 检查 `df.columns` 和 `df.index` 确认标签是否存在。
  - 注意列名可能包含空格或特殊字符。
  - 使用 `df.get('col')` 安全获取列（不存在时返回 `None` 或默认值）。
  - 对于位置索引，确认索引范围。

#### 2.6.3 ValueError: cannot reindex from a duplicate axis

- **现象**：索引中存在重复标签，某些操作（如 `reindex`、`groupby`）无法确定对应关系。
- **排查**：
  - 使用 `df.index.is_unique` 检查索引是否唯一。
  - 使用 `df.reset_index()` 或 `df.drop_duplicates()` 处理重复索引。

#### 2.6.4 DtypeWarning

- **现象**：读取大型 CSV 时，pandas 可能无法统一推断列类型，返回 `DtypeWarning`。
- **排查**：
  - 使用 `pd.read_csv(..., dtype=...)` 手动指定列类型。
  - 使用 `low_memory=False` 强制完整读取以准确推断。
  - 使用 `pd.options.future.infer_string = True` 优化字符串推断。

#### 2.6.5 内存不足（MemoryError）

- **现象**：处理超大 DataFrame 时内存耗尽。
- **排查**：
  - 使用 `df.info(memory_usage='deep')` 查看实际内存占用。
  - 优化数据类型：将 `float64` 转为 `float32`，`object` 转为 `category` 或 `string[pyarrow]`。
  - 使用分块读取（`chunksize`）或迭代处理。
  - 考虑使用 PyArrow 后端或与其他框架（如 Dask）结合。

#### 2.6.6 数据类型意外改变

- **现象**：运算后列类型从整数变为浮点，或出现 `object`。
- **原因**：缺失值导致类型提升，或混合类型列。
- **排查**：
  - 使用 `convert_dtypes()` 自动推断更合适的可空类型。
  - 使用 `df.select_dtypes()` 筛选特定类型。
  - 检查是否存在 `None` 或 `np.nan` 被意外引入。

> [!tip] 调试最佳实践
> - 始终先使用 `df.info()` 和 `df.head()` 了解数据。
> - 设置合适的 `display` 选项以避免输出截断。
> - 使用 `df.dtypes` 快速检查类型问题。
> - 对于复杂数据，使用 `df.describe(include='all')` 获取全面统计。
> - 遇到警告时，不要忽略，尤其是 `SettingWithCopyWarning`，应调整代码以避免潜在 bug。
> - 在单元测试中使用 `pandas.testing` 模块验证 DataFrame 是否符合预期。
