---
title: 18 统计与数学计算
---

> [!abstract] 本节大纲
> - 描述统计
>     - count、sum、mean、median、mode、std、var、sem、skew、kurt、min、max、quantile
> - 相关性
>     - corr()
>     - corrwith()
>     - autocorr()
> - 协方差
>     - cov()
> - 排名
>     - rank()
> - 差异与百分比变化
>     - diff()
>     - pct_change()
> - 累积运算
>     - cumsum、cumprod、cummin、cummax
> - 滚动统计
> - 加权统计
> - 聚合
> - 线性代数
>     - dot()
>     - transpose()
> - 元素级算术
> - 标量广播
> - 缺失值处理
> - 数值舍入
>     - round()、floor()、ceil()

## 1. 描述统计

描述统计用于总结数据的集中趋势、离散程度和分布形状。pandas 的 `Series` 和 `DataFrame` 提供了丰富的描述统计方法，这些方法通常默认跳过缺失值（`skipna=True`）。

### 1.1 count()
- **用途**：计算非缺失值的数量。
- **语法**：`Series.count()` / `DataFrame.count(axis=0)`
- **参数**：
  - `axis`：对于 DataFrame，`0` 或 `'index'` 按列计算，`1` 或 `'columns'` 按行计算。
  - `numeric_only`：是否仅包含数值列（默认 `False`，在 pandas 2.x 中建议显式指定）。
- **示例**：
  ```python
  s = pd.Series([1, 2, None, 4])
  s.count()   # 3
  df = pd.DataFrame({'A': [1, None, 3], 'B': [4, 5, None]})
  df.count()  # A: 2, B: 2
  ```

### 1.2 sum()
- **用途**：计算数值总和。
- **语法**：`Series.sum()` / `DataFrame.sum(axis=0)`
- **参数**：`axis`、`skipna`、`numeric_only`、`min_count`（要求的最少非缺失值数量）。
- **示例**：
  ```python
  pd.Series([1, 2, None]).sum()   # 3.0（默认跳过缺失）
  pd.Series([1, 2, None]).sum(min_count=3)  # NaN，因为只有2个非缺失
  ```

### 1.3 mean()
- **用途**：计算算术平均值。
- **语法**：`Series.mean()` / `DataFrame.mean(axis=0)`
- **参数**：`axis`、`skipna`、`numeric_only`。
- **示例**：
  ```python
  pd.Series([1, 2, 3, 4]).mean()   # 2.5
  ```

### 1.4 median()
- **用途**：计算中位数（第50百分位数）。
- **语法**：`Series.median()` / `DataFrame.median(axis=0)`
- **参数**：`axis`、`skipna`、`numeric_only`。
- **示例**：
  ```python
  pd.Series([1, 2, 100]).median()   # 2.0
  ```

### 1.5 mode()
- **用途**：计算众数（出现频率最高的值）。返回 Series 或 DataFrame，因为众数可能不唯一。
- **语法**：`Series.mode()` / `DataFrame.mode(axis=0)`
- **参数**：`axis`、`numeric_only`、`dropna`。
- **示例**：
  ```python
  pd.Series([1, 2, 2, 3]).mode()   # 返回 [2]
  ```

### 1.6 std()
- **用途**：计算样本标准差（默认使用 N-1 作为分母）。
- **语法**：`Series.std()` / `DataFrame.std(axis=0)`
- **参数**：`axis`、`skipna`、`ddof`（自由度，默认 1）、`numeric_only`。
- **示例**：
  ```python
  pd.Series([1, 2, 3]).std()   # 1.0（样本标准差）
  pd.Series([1, 2, 3]).std(ddof=0)   # 0.8165（总体标准差）
  ```

### 1.7 var()
- **用途**：计算样本方差。
- **语法**：`Series.var()` / `DataFrame.var(axis=0)`
- **参数**：`axis`、`skipna`、`ddof`、`numeric_only`。
- **示例**：
  ```python
  pd.Series([1, 2, 3]).var()   # 1.0
  ```

### 1.8 sem()
- **用途**：计算均值的标准误差（标准差 / sqrt(样本数)）。
- **语法**：`Series.sem()` / `DataFrame.sem(axis=0)`
- **参数**：`axis`、`skipna`、`ddof`、`numeric_only`。
- **示例**：
  ```python
  pd.Series([1, 2, 3, 4]).sem()   # 0.6455
  ```

### 1.9 skew()
- **用途**：计算偏度（分布的不对称性）。偏度 > 0 右偏，< 0 左偏。
- **语法**：`Series.skew()` / `DataFrame.skew(axis=0)`
- **参数**：`axis`、`skipna`、`numeric_only`。
- **示例**：
  ```python
  pd.Series([1, 2, 3, 4, 100]).skew()   # 正值，右偏
  ```

### 1.10 kurt()
- **用途**：计算峰度（分布尾部厚度）。正态分布的峰度约为 0（使用 Fisher 定义）。
- **语法**：`Series.kurt()` / `DataFrame.kurt(axis=0)`
- **参数**：`axis`、`skipna`、`numeric_only`。
- **示例**：
  ```python
  pd.Series([1, 2, 3, 4, 5]).kurt()   # 接近 -1.2 等，取决于定义
  ```

### 1.11 min()
- **用途**：计算最小值。
- **语法**：`Series.min()` / `DataFrame.min(axis=0)`
- **参数**：`axis`、`skipna`、`numeric_only`。
- **示例**：
  ```python
  pd.Series([3, 1, 2]).min()   # 1
  ```

### 1.12 max()
- **用途**：计算最大值。
- **语法**：`Series.max()` / `DataFrame.max(axis=0)`
- **参数**：`axis`、`skipna`、`numeric_only`。
- **示例**：
  ```python
  pd.Series([3, 1, 2]).max()   # 3
  ```

### 1.13 quantile()
- **用途**：计算指定分位数。
- **语法**：`Series.quantile(q=0.5)` / `DataFrame.quantile(q=0.5, axis=0)`
- **参数**：
  - `q`：分位数（0 到 1 之间），可以是单个值或列表。
  - `axis`、`numeric_only`、`interpolation`（插值方法，如 `'linear'`、`'lower'`、`'higher'`、`'midpoint'`、`'nearest'`）。
- **示例**：
  ```python
  s = pd.Series([1, 2, 3, 4, 5])
  s.quantile(0.25)   # 2.0
  s.quantile([0.25, 0.75])   # 返回 Series
  ```

## 2. 相关性

相关性衡量两个变量之间的线性关系强度和方向。

### 2.1 corr()
- **用途**：计算列之间（或 Series 之间）的相关系数。
- **语法**：
  - `DataFrame.corr(method='pearson', min_periods=1)`
  - `Series.corr(other, method='pearson', min_periods=1)`
- **参数**：
  - `method`：相关系数类型，可选 `'pearson'`（默认，线性相关）、`'kendall'`（秩相关）、`'spearman'`（秩相关）。
  - `min_periods`：所需的最小非缺失值数量。
  - `numeric_only`：是否仅数值列。
- **示例**：
  ```python
  df = pd.DataFrame({'A': [1, 2, 3], 'B': [4, 5, 6]})
  df.corr()   # A 与 B 的相关系数接近 1
  s1 = pd.Series([1, 2, 3])
  s2 = pd.Series([2, 4, 6])
  s1.corr(s2)   # 1.0
  ```

### 2.2 corrwith()
- **用途**：计算 DataFrame 的每一列（或行）与另一个 Series 或 DataFrame 的相关系数。
- **语法**：`DataFrame.corrwith(other, axis=0, drop=False, method='pearson')`
- **参数**：
  - `other`：Series 或 DataFrame。
  - `axis`：按行（0）或按列（1）对齐。
  - `drop`：是否丢弃不匹配的索引。
  - `method`：同 `corr()`。
- **示例**：
  ```python
  df = pd.DataFrame({'A': [1, 2, 3], 'B': [4, 6, 8]})
  s = pd.Series([1, 2, 3])
  df.corrwith(s)   # A: 1.0, B: 1.0
  ```

### 2.3 autocorr()
- **用途**：计算时间序列的自相关（序列与其自身滞后版本的相关性）。
- **语法**：`Series.autocorr(lag=1)`
- **参数**：
  - `lag`：滞后期数（默认 1）。
- **示例**：
  ```python
  s = pd.Series([1, 2, 3, 4, 5])
  s.autocorr(lag=1)   # 1.0（完全正相关）
  s.autocorr(lag=2)   # 1.0
  ```

## 3. 协方差

### cov()
- **用途**：计算协方差矩阵（DataFrame）或两个 Series 的协方差。
- **语法**：
  - `DataFrame.cov(min_periods=None, ddof=1)`
  - `Series.cov(other, min_periods=None, ddof=1)`
- **参数**：
  - `min_periods`：所需的最小观测数。
  - `ddof`：自由度调整，默认 1（样本协方差）。
- **示例**：
  ```python
  df = pd.DataFrame({'A': [1, 2, 3], 'B': [4, 5, 6]})
  df.cov()   # 返回 2x2 协方差矩阵
  s1 = pd.Series([1, 2, 3])
  s2 = pd.Series([4, 5, 6])
  s1.cov(s2)   # 1.0
  ```

## 4. 排名

### rank()
- **用途**：为数据分配排名（从 1 开始）。默认将最小值排名为 1，升序排列。
- **语法**：`Series.rank()` / `DataFrame.rank(axis=0)`
- **参数**：
  - `axis`：按行或列排名。
  - `method`：处理并列排名的方式，可选：
    - `'average'`（默认，平均排名）
    - `'min'`（取最小排名）
    - `'max'`（取最大排名）
    - `'first'`（按出现顺序排名）
    - `'dense'`（密集排名，排名连续无间隙）
  - `ascending`：升序或降序，默认 `True`。
  - `na_option`：缺失值处理，可选 `'keep'`（保留为 NaN）、`'top'`（缺失排最前）、`'bottom'`（缺失排最后）。
  - `pct`：是否返回百分位排名（0 到 1），默认 `False`。
- **示例**：
  ```python
  s = pd.Series([10, 20, 20, 30])
  s.rank()               # [1.0, 2.5, 2.5, 4.0]（平均排名）
  s.rank(method='min')   # [1.0, 2.0, 2.0, 4.0]
  s.rank(pct=True)       # [0.25, 0.625, 0.625, 1.0]
  ```

## 5. 差异与百分比变化

### 5.1 diff()
- **用途**：计算相邻元素之间的差值（一阶差分）。
- **语法**：`Series.diff(periods=1)` / `DataFrame.diff(periods=1, axis=0)`
- **参数**：
  - `periods`：差分的滞后期数（默认 1，即当前值减去前一个值）。
  - `axis`：对于 DataFrame，沿行（0）或列（1）计算。
- **示例**：
  ```python
  s = pd.Series([1, 3, 6, 10])
  s.diff()    # [NaN, 2, 3, 4]
  s.diff(2)   # [NaN, NaN, 5, 7]
  ```

### 5.2 pct_change()
- **用途**：计算百分比变化（当前值与前一个值相比的变化率）。
- **语法**：`Series.pct_change(periods=1, fill_method='pad', limit=None, freq=None)` / DataFrame 类似。
- **参数**：
  - `periods`：滞后期数，默认 1。
  - `fill_method`：如何处理缺失值（如 `'pad'` 前向填充）。
  - `limit`：填充的最大数量。
  - `freq`：当索引为时间序列时，可按时间频率对齐。
- **示例**：
  ```python
  s = pd.Series([100, 110, 121, 133.1])
  s.pct_change()   # [NaN, 0.1, 0.1, 0.1]
  ```

## 6. 累积运算

累积运算返回与输入相同形状的对象，每个位置是到当前位置为止的累积结果。

### 6.1 cumsum()
- **用途**：累积求和。
- **语法**：`Series.cumsum(axis=0, skipna=True)` / DataFrame 类似。
- **示例**：
  ```python
  pd.Series([1, 2, 3, 4]).cumsum()   # [1, 3, 6, 10]
  ```

### 6.2 cumprod()
- **用途**：累积乘积。
- **语法**：`Series.cumprod(axis=0, skipna=True)` / DataFrame 类似。
- **示例**：
  ```python
  pd.Series([1, 2, 3, 4]).cumprod()   # [1, 2, 6, 24]
  ```

### 6.3 cummin()
- **用途**：累积最小值（到当前位置为止的最小值）。
- **语法**：`Series.cummin(axis=0, skipna=True)` / DataFrame 类似。
- **示例**：
  ```python
  pd.Series([3, 1, 2, 0]).cummin()   # [3, 1, 1, 0]
  ```

### 6.4 cummax()
- **用途**：累积最大值。
- **语法**：`Series.cummax(axis=0, skipna=True)` / DataFrame 类似。
- **示例**：
  ```python
  pd.Series([3, 1, 2, 0]).cummax()   # [3, 3, 3, 3]
  ```

## 7. 滚动统计

滚动统计（窗口计算）用于在滑动窗口上计算统计量，常用于时间序列分析。pandas 提供 `rolling()`、`expanding()` 和 `ewm()` 方法。

### 7.1 rolling()
- **用途**：在固定大小的窗口上计算统计量。
- **语法**：`Series.rolling(window, min_periods=None, center=False, win_type=None, on=None, axis=0, closed=None)`
- **常用方法**：`sum()`, `mean()`, `std()`, `var()`, `min()`, `max()`, `count()`, `quantile()`, `apply()`, `corr()`, `cov()` 等。
- **示例**：
  ```python
  s = pd.Series([1, 2, 3, 4, 5])
  s.rolling(window=3).mean()   # [NaN, NaN, 2.0, 3.0, 4.0]
  ```

### 7.2 expanding()
- **用途**：在扩展窗口上计算统计量（从起始位置到当前位置）。
- **语法**：`Series.expanding(min_periods=1)`，方法与 rolling 类似。
- **示例**：
  ```python
  s.expanding().sum()   # [1, 3, 6, 10, 15]
  ```

### 7.3 ewm()
- **用途**：指数加权移动统计量，为近期数据赋予更大权重。
- **语法**：`Series.ewm(com=None, span=None, halflife=None, alpha=None, adjust=True, ignore_na=False, min_periods=0)`
- **常用方法**：`mean()`, `std()`, `var()`, `corr()`, `cov()` 等。
- **示例**：
  ```python
  s.ewm(span=3).mean()   # 指数加权移动平均
  ```

## 8. 加权统计

加权统计指在计算统计量时对不同数据点赋予不同权重。pandas 没有内置统一的加权统计方法，但可以通过自定义函数或结合 `rolling`、`ewm` 实现。

### 8.1 使用 ewm 进行加权
- `ewm(alpha=...)` 本质上是一种加权方法，权重指数衰减。
- 示例：`s.ewm(alpha=0.5).mean()` 计算加权移动平均。

### 8.2 自定义加权函数
- 使用 `apply` 配合权重向量：
  ```python
  weights = np.array([0.1, 0.2, 0.3, 0.4])
  weighted_mean = (s * weights).sum() / weights.sum()
  ```
- 也可在 `rolling().apply()` 中传入自定义加权函数。

### 8.3 加权分位数
- 没有内置方法，需要借助 `numpy` 或 `scipy` 实现。

## 9. 聚合

聚合指将多个值合并为一个汇总值。pandas 提供了多种聚合方式。

### 9.1 内置聚合方法
- 前面描述统计中的 `sum`、`mean`、`min`、`max` 等都可以直接调用。
- `agg()` / `aggregate()` 方法支持一次应用多个聚合函数：
  ```python
  df.agg(['sum', 'mean', 'std'])
  df.agg({'A': 'sum', 'B': ['mean', 'max']})
  ```

### 9.2 使用 groupby 聚合
- `groupby().agg()` 是常见的分组聚合方式。
- 支持命名聚合：
  ```python
  df.groupby('key').agg(total=('value', 'sum'), avg=('value', 'mean'))
  ```

### 9.3 使用 transform 进行聚合转换
- `transform()` 返回与原始数据相同形状的聚合结果，可用于标准化等。

## 10. 线性代数

pandas 支持一些基本的线性代数操作。

### 10.1 dot()
- **用途**：计算矩阵乘法或向量点积。
- **语法**：`DataFrame.dot(other)` 或 `Series.dot(other)`。
- **参数**：`other` 可以是 DataFrame、Series 或数组。
- **示例**：
  ```python
  df = pd.DataFrame({'A': [1, 2], 'B': [3, 4]})
  s = pd.Series([5, 6])
  df.dot(s)   # 结果：1*5+3*6=23, 2*5+4*6=34
  ```

### 10.2 transpose()
- **用途**：转置 DataFrame（行列互换）。
- **语法**：`DataFrame.transpose()` 或简写 `DataFrame.T`。
- **示例**：
  ```python
  df.T
  ```

## 11. 元素级算术

pandas 支持对 Series 和 DataFrame 进行元素级算术运算，运算符包括 `+`、`-`、`*`、`/`、`//`、`%`、`**` 等。

### 11.1 基本运算
- `df + 2`：每个元素加 2。
- `df1 + df2`：按索引对齐后相加。
- 对应的方法：`add()`, `sub()`, `mul()`, `div()`, `floordiv()`, `mod()`, `pow()`。

### 11.2 反向运算
- `radd()`, `rsub()`, `rmul()`, `rdiv()` 等，用于处理 `2 - df` 等场景。

### 11.3 比较运算
- `==`, `!=`, `<`, `<=`, `>`, `>=` 返回布尔 DataFrame/Series。
- 方法：`eq()`, `ne()`, `lt()`, `le()`, `gt()`, `ge()`。

## 12. 标量广播

标量与 Series 或 DataFrame 运算时，标量会自动广播到所有元素，无需循环。

### 12.1 标量广播规则
- 标量与 Series/DataFrame 的算术运算：每个元素都与该标量进行运算。
- 标量与 DataFrame 的比较运算类似。
- 示例：
  ```python
  df = pd.DataFrame({'A': [1, 2], 'B': [3, 4]})
  df + 10   # 每个元素加 10
  ```

### 12.2 缺失值处理
- 广播运算中，缺失值会保留，结果中对应位置为缺失值。

## 13. 缺失值处理

统计和数学运算中，pandas 默认跳过缺失值（`skipna=True`），但可以控制。

### 13.1 skipna 参数
- 大多数统计方法（如 `sum`、`mean`）都有 `skipna` 参数，默认 `True`，忽略缺失值。
- 设置 `skipna=False` 时，任何包含缺失值的位置都会导致结果为缺失值。

### 13.2 缺失值在运算中的传播
- 算术运算中，缺失值与任何值运算结果均为缺失值。
- 比较运算中，缺失值结果为 `False`（对于 `==`）或 `pd.NA`（可空类型）。

### 13.3 填充缺失值
- 可以使用 `fillna()` 在计算前填充缺失值。
- 在滚动、累积等运算中，缺失值可能被跳过或传播，取决于具体实现。

## 14. 数值舍入

### 14.1 round()
- **用途**：将数值四舍五入到指定小数位数。
- **语法**：`Series.round(decimals=0)` / `DataFrame.round(decimals=0)`。
- **参数**：`decimals` 可以是整数（统一小数位）或字典（按列指定）。
- **示例**：
  ```python
  s = pd.Series([1.234, 2.345])
  s.round(2)   # [1.23, 2.35]
  df.round({'A': 1, 'B': 2})
  ```

### 14.2 floor()
- **用途**：向下取整（不大于原值的最大整数）。
- **语法**：`Series.floor()` / `DataFrame.floor()`。
- **示例**：
  ```python
  pd.Series([1.9, -1.1]).floor()   # [1.0, -2.0]
  ```

### 14.3 ceil()
- **用途**：向上取整（不小于原值的最小整数）。
- **语法**：`Series.ceil()` / `DataFrame.ceil()`。
- **示例**：
  ```python
  pd.Series([1.1, -1.9]).ceil()   # [2.0, -1.0]
  ```

> [!tip] 总结
> pandas 提供了全面的统计与数学计算工具，从基本的描述统计到复杂的相关性、滚动窗口和线性代数操作。理解这些方法及其参数，能够高效地进行数据探索、特征工程和时间序列分析。同时，注意缺失值的处理方式和广播规则，以避免计算错误。
```