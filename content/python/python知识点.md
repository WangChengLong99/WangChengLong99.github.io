---
title: python知识点
tags:
  - python
---

# 导入数据

下面分别说明 Python 导入 Excel 数据时的类型对应，以及从 MySQL 查询数据时的类型对应。

---

## 一、Python 导入 Excel 数据的类型对应关系

Excel 中的数据类型并不像数据库那样严格，它通过“单元格格式”来表达数值、文本、日期、布尔等。Python 读取 Excel 时，**使用的库不同，映射规则也会不同**，常见的有 `pandas.read_excel()`（背后引擎可以是 `openpyxl` 或 `xlrd`）和直接使用 `openpyxl`。

### 1. 用 `pandas.read_excel()` 读取时的类型映射

`pandas` 会推断整列的数据类型，返回的 `DataFrame` 的 `dtype` 是列的通用类型。单个值的实际 Python 类型依赖于此列 `dtype`。

| Excel 中的内容/格式               | pandas 列类型 (dtype)                                                                                 | 单元格内的 Python 类型                           | 备注                                           |
| --------------------------- | -------------------------------------------------------------------------------------------------- | ----------------------------------------- | -------------------------------------------- |
| 常规数字、数值                     | `int64`（无缺失/空）<br>`float64`（有空值或含小数）                                                               | `int` <br>`float`                         | 若包含空值，列会提升为 `float64`，空值显示 `NaN`             |
| 文本 / 字符串                    | `object`<br>文本型数字视作常规数字、数值<br>空字符串和空白都被读为`NaN`                                                     | `str`<br>`int`或`float`<br><br>`float`     | 混合类型列通常也是 `object`                           |
| 日期（如 `2024-01-01`）          | 若 `parse_dates=True` 或自动识别(当excel中的日期类型为文本时不被识别，为日期时可以自动识别) → `datetime64[ns]`<br>若包含空值，空值类型为`NaT` | `pd.Timestamp`<br><br><br><br>` NaTType,` | 本质是 `numpy.datetime64` 的封装，可当作 `datetime` 使用 |
| 日期+时间（如 `2024-01-01 14:30`） | `datetime64[ns]`                                                                                   | `pd.Timestamp`                            | 同上                                           |
| 纯时间（如 `14:30:00`）           | object                                                                                             | `datetime.time`                           | 整列不是时间类型，但是内部是                               |
| 布尔值 `TRUE`/`FALSE`          | `bool`（无缺失）<br>`float64`（有缺失）                                                                      | `bool` <br>`float`（缺失值为float类型)           | 含有空单元格时可能退化为 `float64`，值为 `float`            |
| 空白单元格                       | 同列数字列 → `float64` 下的 `NaN` <br>同列文本列 → `object` 下的 `NaN` (实为 `float`)                              | `float`                                   | 空值统一为缺失值<br>空字符串也为缺失值                        |
| 公式                          | 计算结果的值，按值类型对应上表                                                                                    | 与计算结果类型一致                                 | 需要 Excel 计算过的值（读取时使用 `data_only=True` 的引擎参数） |
| 错误值 (`#DIV/0!` 等)           | `object`                                                                                           | `str` (如 `'#DIV/0!'`)                     | pandas 默认将错误当作文本                             |

> [!important] 日期类型读取
> 日期和时间能不能读取为日期和时间类型，主要还是看excel中数据有没有被标为日期时间，如果被标为文本，则该列以及列中的数据都不会为日期时间类型

> [!note] **混合类型列**
> `pandas` 会将整个列当作 `object` 类型，每个单元格保留其原始 Python 类型（如同时包含 `int` 和 `str`）。当单元格格式能跟数据格式保持一致，或者常规（无特定格式），pandas可以识别每个单元格本来的格式。如果整列被设置为文本，那么在混合列中就不能识别除文本外的其他格式

---

### 2. 用 `openpyxl` 直接读取单元格 `.value` 时的类型映射

这种方式可以获取**每个单元格**的精确 Python 类型。

| Excel 单元格类型（数字格式） | `openpyxl` 返回的 Python 类型 | 说明 |
|--------------------------|-----------------------------|------|
| 整数数字 | `int` | 如 Excel 中的 100 |
| 小数 / 浮点数字 | `float` | 如 3.14 |
| 日期格式（如 `yyyy-mm-dd`） | `datetime.datetime` | Excel 序列号转换为 `datetime`（日期部分有效，时间默认 00:00:00） |
| 日期+时间格式 | `datetime.datetime` | 完整的日期时间 |
| 纯时间格式（如 `hh:mm:ss`） | `datetime.time` | Excel 内部为小数，openpyxl 转为 `time` |
| 时长格式（如 `[hh]:mm:ss`） | `datetime.timedelta` | 适用于超过 24 小时的时间差 |
| 文本 / 字符串 | `str` | |
| 布尔值 | `bool` | `True` 或 `False` |
| 空单元格 | `None` | |
| 错误 (`#N/A`, `#VALUE!` 等) | `str`（如 `'#N/A'`） | 需注意，不是异常而是字符串 |
| 公式（工作簿未计算） | 公式字符串（如 `'=A1+B1'`） | 若 `data_only=True` 且缓存了结果，则返回计算结果值 |

---

### 小结与注意点

- **Excel 没有独立的“日期”或“时间”类型**，它们本质是数字（序列号），通过数字格式显示。`openpyxl` 能根据格式智能转换为 `datetime`/`time`/`timedelta`，而 `pandas` 默认只自动处理常见的日期时间列，纯时间列可能需要手动 `pd.to_timedelta` 或自定义转换。
- **公式**：如果 Excel 文件未保存计算结果，`openpyxl` 只能读到公式字符串，`pandas` 可能无法读取到期望的值。

---

## 二、查询 MySQL 数据时 Python 数据类型的对应关系

同样取决于你使用的 Python 库。最常见的是 **DB-API 驱动（如 `pymysql`、`mysql-connector-python`）** 和 **`pandas.read_sql()`**，下面以 `pymysql` 为例展示标准 DB-API 的映射，再补充 `pandas` 的差异。

### 1. `pymysql`（及大多数 MySQL Python 驱动）的类型映射

默认情况下，游标返回的行中的字段已经转换为对应的 Python 对象。

| MySQL 数据类型 | Python 类型 | 备注 |
|--------------|------------|------|
| `TINYINT`, `SMALLINT`, `MEDIUMINT`, `INT`, `INTEGER`, `BIGINT` | `int` | Python 3 的 int 无上限，能容纳 BIGINT |
| `FLOAT`, `DOUBLE` | `float` | |
| `DECIMAL`, `NUMERIC` | `decimal.Decimal` | 保留精度，适合金融计算 |
| `BIT` | `bytes` | BIT(1) 返回 `b'\x00'` 或 `b'\x01'` |
| `BOOL`, `BOOLEAN` | `int` (0 或 1) | 实际上是 TINYINT(1)，默认返回 int；可通过 `conv` 参数转为 `bool` |
| `CHAR`, `VARCHAR`, `TINYTEXT`, `TEXT`, `MEDIUMTEXT`, `LONGTEXT` | `str` | |
| `ENUM`, `SET` | `str` | 返回枚举/集合的字符串值 |
| `BINARY`, `VARBINARY`, `TINYBLOB`, `BLOB`, `MEDIUMBLOB`, `LONGBLOB` | `bytes` | 二进制数据 |
| `DATE` | `datetime.date` | |
| `DATETIME`, `TIMESTAMP` | `datetime.datetime` | |
| `TIME` | `datetime.timedelta` | MySQL 的 TIME 范围较大（-838:59:59 ~ 838:59:59），timedelta 可完整表示 |
| `YEAR` | `int` | 返回年份整数（如 2024） |
| `JSON` | `str` | MySQL 返回 JSON 文本，需手动 `json.loads()` 解析 |
| `GEOMETRY` | `bytes` | 返回 WKB（Well-Known Binary）格式字节串 |
| `NULL` | `None` | |

#### 一些可调整的地方

- **布尔值**：`pymysql` 默认将 `TINYINT(1)` 转为 `int`。若想直接得到 `bool`，可在连接时设置 `conv=pymysql.converters.conversions` 或添加自定义转换器。
- **TIME 作为字符串**：某些驱动或设置下，`TIME` 会返回 `str`（如 `'14:30:00'`），可检查驱动文档或连接参数（如 `pymysql` 的 `converter_class`）。

---

### 2. 使用 `pandas.read_sql()` 时的类型映射

`pandas.read_sql()` 底层利用 SQLAlchemy 或 DB-API 游标，并将结果构造成 `DataFrame`。它会再次调整类型以适合数据分析。

| MySQL 类型 | pandas 列类型 (dtype) | 说明 |
|-----------|----------------------|------|
| 整数列 (`INT`, `BIGINT` 等) | `int64`（有空值则变为 `float64`） | pandas 要求整数列不能有 `NaN`，有缺失值会提升为 float |
| `FLOAT`, `DOUBLE` | `float64` | |
| `DECIMAL` | `float64` 或 `object` | 往往被转为 float，可能丢失精度；也可通过 SQLAlchemy 保留为 Decimal（列类型为 `object`） |
| `CHAR`, `VARCHAR`, `TEXT` 等 | `object` (元素为 `str`) | |
| `DATE` | `object` (元素为 `datetime.date`) 或 `datetime64[ns]` | 若连接时加了 `parse_dates` 参数，则转为 `datetime64` |
| `DATETIME`, `TIMESTAMP` | `datetime64[ns]` | |
| `TIME` | `object` (元素为 `timedelta` 或 `str`) | 取决于驱动返回的类型，pandas 不会自动转为时间类型 |
| `BOOLEAN` / `BOOL` | `int64` 或 `bool` | 若驱动返回 0/1，则是 int；若返回 True/False 则为 bool |
| `ENUM`, `SET` | `object` (字符串) | |
| `JSON` | `object` (字符串) | 需手动解析 JSON |
| 空值 `NULL` | `NaN` 或 `None` | 取决于列类型，数值列 `NaN`，对象列可能是 `None` 或 `NaN` |

---

### 小结与实用建议

- **从 MySQL 拿数据做精确计算**：使用 `pymysql` 直接获取 `Decimal` 而不要用 `pandas` 将其转为 float。
- **处理时间**：`pymysql` 的 `TIME` → `timedelta` 很方便；而 `pandas` 读取时通常需要额外的 `pd.to_timedelta` 进行转换。
- **布尔值**：如果数据库使用 `TINYINT(1)` 表示布尔，记得根据需要转换 `int` → `bool`。
- **JSON 字段**：从 MySQL 查出来默认是字符串，一般需要 `json.loads()` 解析成字典或列表。

总的来说，Python 与 Excel 的类型映射高度依赖读取库及其参数，Python 与 MySQL 的映射则由驱动和是否使用 `pandas` 决定，但大体遵循上述规则。搞清楚这些对应关系，在数据清洗和格式转换时就能避免很多隐式错误。

# python空值类型判断与转换

在数据清洗，特别是处理从 Excel 或数据库导入的数据时，经常会遇到各种“空值”。Python 中的空值类型不止一种，判断和转换方法也各不相同。下面系统梳理常见空值的判断，以及在不同场景下的转换方式。

---

## 一、Python 中常见的空值类型

| 类型 | 来源 | 典型值 | 特点 |
|------|------|--------|------|
| `None` | Python 内置 | `None` | 单例对象，表示“无”，常用于对象列或数据库 NULL |
| `float('nan')` | Python 内置 | `float('nan')` | 浮点数，不等于自身（`nan != nan`） |
| `numpy.nan` (`np.nan`) | NumPy | `np.nan` | 同上，但类型为 `np.float64` |
| `pandas.NaT` | Pandas | `pd.NaT` | 表示缺失的时间，兼容 `datetime64` / `timedelta64` |
| `pandas.NA` | Pandas (≥1.0) | `pd.NA` | 实验性标量，可用于可空整数、字符串、布尔类型，避免类型提升 |
| 空字符串 `''` | Python 内置 | `''` | 长度为 0 的字符串，不是严格意义上的缺失值，但在业务中常当作空值处理 |
| 空白字符串 `'   '` | Python 内置 | `'   '` | 全是空白符，也常被当作空值 |
| 空容器 `[]`, `{}` | Python 内置 |  | 空列表、字典等 |
| 缺失值占位符 | 业务数据 | 如 `'NA'`, `'NULL'`, `'N/A'`, `'-'` | 需要识别并转换为真正的缺失值 |

---

## 二、单值判断：如何识别一个变量是否为空

### 1. `None` 的判断
```python
x = None
x is None   # True
```
- **注意**：`x == None` 虽然可行，但不符合 PEP8，且对 `numpy` 数组会返回逐元素结果，推荐用 `is`。

### 2. `float('nan')` / `np.nan` 的判断
```python
import math
import numpy as np

x = float('nan')
# 不能用 ==，因为 nan != nan
math.isnan(x)   # True
np.isnan(x)     # True (如果是 np.nan 或 float nan)

# 对于非浮点类型会报错，如 np.isnan(None) 会引发 TypeError
```
- `np.nan` 和 `float('nan')` 在 `math.isnan` 和 `np.isnan` 下等价，但 `np.isnan` 可接受数组。

### 3. `pd.NaT` 的判断
```python
import pandas as pd
x = pd.NaT
pd.isna(x)      # True
pd.isnull(x)    # True (isnull 是 isna 的别名)
# x is pd.NaT   # 也成立，但 pd.NaT 是单例，推荐使用 pd.isna
```

### 4. `pd.NA` 的判断
```python
x = pd.NA
pd.isna(x)      # True
```

### 5. 空字符串 / 空白字符串 / 占位符的判断
```python
s = ''
if s == '':
    # 空字符串
if s.strip() == '':
    # 空或只有空白
```
- 对于 `None` 或 `np.nan`，直接 `.strip()` 会报错，需先判断。

### 通用判断函数对比

| 方法 | 识别 None | 识别 np.nan | 识别 pd.NaT | 识别 pd.NA | 识别 '' | 备注 |
|------|-----------|-------------|-------------|------------|---------|------|
| `x is None` | ✔️ | ❌ | ❌ | ❌ | ❌ | 仅 None |
| `math.isnan(x)` | ❌ (报错) | ✔️ | ❌ (报错) | ❌ (报错) | ❌ | 仅 float nan |
| `np.isnan(x)` | ❌ (报错) | ✔️ | ❌ (报错) | ❌ (报错) | ❌ | 仅浮点 nan，可处理数组 |
| `pd.isna(x)` | ✔️ | ✔️ | ✔️ | ✔️ | ❌ | **推荐，覆盖最广** |
| `pd.isnull(x)` | ✔️ | ✔️ | ✔️ | ✔️ | ❌ | 与 isna 相同 |

```python
# 4. 综合判断：None / 空字符串 / NaN / 空列表等
def is_empty(v):
    if v is None:
        return True
    if isinstance(v, float) and math.isnan(v):
        return True
    if isinstance(v, str) and v.strip() == "":
        return True
    if isinstance(v, (list, tuple, dict, set)) and len(v) == 0:
        return True
    return False
```


> **最佳实践**：在数据清洗时，统一使用 `pd.isna()` / `pd.isnull()` 来判断各种缺失值标量。

---

## 三、批量判断：Pandas DataFrame / Series 中的空值

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    'A': [1, None, 3],
    'B': [np.nan, 'text', ''],
    'C': [pd.NaT, pd.Timestamp('2020-01-01'), pd.NaT],
    'D': [pd.NA, 2, pd.NA],
})
```

### 1. 检测所有缺失值
```python
df.isna()    # 返回布尔 DataFrame，对 None/np.nan/NaT/NA 均为 True，对 '' 为 False
df.isnull()  # 完全等价
```

### 2. 检测非缺失值
```python
df.notna()
```

### 3. 统计缺失值
```python
df.isna().sum()        # 每列缺失值个数
df.isna().sum().sum()  # 总缺失值个数
```

### 4. 将空字符串也视为缺失值
```python
# 替换所有空字符串为 NaN
df.replace('', np.nan, inplace=True)
# 或
df['B'] = df['B'].replace('', np.nan)
# 空白字符串同理
df.replace(r'^\s*$', np.nan, regex=True, inplace=True)
```

### 5. 判断某列是否存在缺失值
```python
df['A'].hasnans   # True 如果列中包含 NaN 或 NaT （仅对数值/日期列有效）
# 通用
df['A'].isna().any()
```

---

## 四、空值的转换与清洗

### 1. 统一缺失值表示：将各种“空”变成标准的 `NaN` / `None`

一个常见场景：从 Excel 或 CSV 读入后，缺失值可能有 `None`、`np.nan`、`'NA'`、`'NULL'`、`''`、`'-'` 等。

```python
# 定义需要替换的占位符列表
na_values = ['NA', 'NULL', 'N/A', '-', '']

# 读取时直接处理（以 read_csv 为例）
df = pd.read_csv('file.csv', na_values=na_values, keep_default_na=True)

# 或读取后替换
df.replace(na_values, np.nan, inplace=True)

# 同时去除字符串首尾空白后视为缺失
df = df.applymap(lambda x: np.nan if isinstance(x, str) and x.strip() == '' else x)
```

### 2. 填充缺失值 (`fillna`)

```python
# 用常数填充
df['A'].fillna(0, inplace=True)

# 用均值/中位数填充
df['A'].fillna(df['A'].mean(), inplace=True)

# 用前一个值填充（向前填充）
df['B'].ffill()

# 用字典对不同列指定不同填充值
df.fillna({'A': 0, 'B': 'unknown', 'C': pd.Timestamp.now()}, inplace=True)

# 对于 pd.NA 兼容的可空类型，可直接填充 None 或 pd.NA
df['D'].fillna(0, inplace=True)  # pd.NA 也可以接受数值填充
```

### 3. 删除缺失值 (`dropna`)

```python
# 删除任何包含缺失值的行
df.dropna(inplace=True)

# 仅当某列缺失才删除该行
df.dropna(subset=['A', 'C'], inplace=True)

# 删除全为空的行或列
df.dropna(how='all', inplace=True)
df.dropna(axis=1, how='all', inplace=True)

# 保留至少有 n 个非空值的行
df.dropna(thresh=2, inplace=True)
```

### 4. 类型兼容：让整型列支持缺失值

普通的 `int64` 列无法容纳 `NaN`，一旦出现缺失值就会被自动提升为 `float64`。解决方法：

```python
# 使用 Pandas 的可空整数类型
df['A'] = df['A'].astype('Int64')   # 注意 I 大写，这是 pd.Int64Dtype()
# 现在缺失值显示为 <NA>，列为 Int64 类型，不再是 float
```

对于布尔类型、字符串类型也有类似的扩展类型：
- `boolean` （可空布尔）
- `string` （可空字符串，与 object 不同，存 `pd.NA` 而非 `NaN`）

```python
df['B'] = df['B'].astype('string')
```

### 5. 写出数据时的空值转换

#### 写入 Excel
```python
# pandas 默认会将 NaN/NaT 写为空单元格，将 None 写为空单元格
df.to_excel('output.xlsx', na_rep='')  # 可以用 na_rep 指定替代文本
# 对于 Int64 等可空类型，<NA> 也会写为空单元格
```

#### 写入数据库 (MySQL)
```python
# 用 to_sql 时，NaN/NaT 自动映射为 SQL NULL
from sqlalchemy import create_engine
engine = create_engine('mysql+pymysql://...')
df.to_sql('table_name', engine, if_exists='replace', index=False)

# 如果自己组装 SQL，需要将 None 作为 NULL 插入
import pymysql
conn = pymysql.connect(...)
cursor = conn.cursor()
# 将 np.nan / pd.NaT / pd.NA 转换为 None
value = None if pd.isna(value) else value
cursor.execute("INSERT INTO table (col) VALUES (%s)", (value,))
```

通用转换函数示例：
```python
def to_sql_value(val):
    """将 Python 空值统一转换为 None，用于插入数据库"""
    if val is None:
        return None
    if isinstance(val, float) and math.isnan(val):
        return None
    if isinstance(val, pd.Timestamp) and pd.isna(val):
        return None
    if hasattr(val, '__module__') and 'pandas' in val.__module__:
        if pd.isna(val):
            return None
    return val
```

### 6. 其他转换技巧

- **用 `numpy.where` 或 `Series.where`**：
```python
df['A'] = df['A'].where(df['A'].notna(), other=0)  # 缺失值替换为0
```

- **布尔值列，将 NaN 填充为 False**：
```python
df['flag'] = df['flag'].fillna(False).astype(bool)
```

- **使用 `coalesce` 逻辑选择第一个非空值**：
```python
# 相当于 SQL 的 COALESCE(A, B)
df['A'] = df['A'].combine_first(df['B'])
# 或
df['A'] = df['A'].fillna(df['B'])
```

---

## 五、注意事项总结

1. **`np.nan` 具有传染性**：任何与 `np.nan` 的算术运算结果都是 `np.nan`。
2. **整型列缺失值会自动提升为 `float64`**，需要可空整数类型用 `'Int64'`。
3. **`None` 在某些情况下自动转为 `np.nan`**：当 `None` 被放入 `float64` 或 `datetime64` 数组时。
4. **`pd.isna('')` 返回 `False`**，需要显式处理空字符串。
5. **判断不要用 `== np.nan`**，始终用 `math.isnan`、`np.isnan` 或 `pd.isna`。
6. **输出到数据库时**，记得将 `np.nan`、`pd.NaT` 等转为 `None`，否则可能报错或写入非预期的值。

掌握这些判断和转换方法后，处理 Excel、数据库导入的数据中的空值就会非常得心应手。

# python数据类型转换

在数据处理流程中，理解并正确转换数据类型是极关键的一环。从 **Python 内置类型转换** 到 **NumPy 类型**，再重点展开 **Pandas DataFrame 的列类型转换**与**元素级的值转换**。

---

## 一、Python 内建数据类型及其转换

### 1. 基础类型

- 数字：`int`、`float`、`complex`
- 布尔：`bool`（`True`/`False`，实际上是 `int` 的子类）
- 文本：`str`
- 二进制：`bytes`、`bytearray`
- 空值：`NoneType`（只有 `None` 一个值）
- 容器：`list`、`tuple`、`dict`、`set`、`frozenset`

### 2. 类型检查

```python
type(x)           # 返回类型对象
isinstance(x, int) # 推荐，支持继承关系
```

### 3. 显式转换（构造函数）

| 转换目标 | 函数 | 说明 |
|---------|------|------|
| 整数 | `int(x)` | 字符串 `"10"` → `10`；浮点数 `3.9` → `3`（截断） |
| 浮点数 | `float(x)` | 整数 `10` → `10.0`；字符串 `"3.14"` → `3.14` |
| 布尔值 | `bool(x)` | 绝大多数非空/非零 → `True`；`0`、`""`、`[]`、`None` 等 → `False` |
| 字符串 | `str(x)` | 几乎所有对象都有 `__str__` |
| 列表/元组 | `list(x)`/`tuple(x)` | 从可迭代对象转换 |

**常见陷阱**：
- `int("abc")` 抛出 `ValueError`
- `int(None)` 抛出 `TypeError`
- `float("3.14.15")` 抛出 `ValueError`
- `bool("False")` 返回 `True`，因为非空字符串为真

### 4. 隐式转换

- 整数与浮点数运算：`3 + 4.0` → `7.0`（int 自动转 float）
- 字符串与数字拼接：`"value: " + 5` 报错，需显式 `"value: " + str(5)`
- 布尔值参与算术：`True + 1` → `2`（True 视为 1）

### 5. 与空值相关的转换

从 Excel/MySQL 导入的数据常混合 `None` 和 `np.nan`，转换前务必用 `pd.isna()` 或 `math.isnan()` 判空，参考你前一个问题中的空值判断表。

---

## 二、NumPy 数据类型（Pandas 的基石）

Pandas 列底层是 `numpy.ndarray` 或 `ExtensionArray`，因此了解 NumPy 类型很重要。

| 类型名 | 描述 | 对应 Python 类型 |
|--------|------|-----------------|
| `int64` | 64 位有符号整数 | `int`（但不能存 NaN） |
| `float64` | 双精度浮点 | `float`（可存 `np.nan`） |
| `bool_` | 布尔值 | `bool` |
| `object` | 任意 Python 对象 | `str`、`list` 等混合类型 |
| `datetime64[ns]` | 纳秒精度时间戳 | `pd.Timestamp`（类似 `datetime.datetime`） |
| `timedelta64[ns]` | 时间差 | `pd.Timedelta` |

**转换**：`np.array([1,2,3]).astype('float64')`

---

## 三、Pandas DataFrame 的**列**类型转换

这是数据处理中最常用的操作——**将整列的数据类型统一调整**。

### 1. 查看列类型
```python
df.dtypes
# 或单列
df['A'].dtype
```

### 2. 主要转换方法

#### (1) `Series.astype()` —— 最通用的方法

```python
# 转换为内置或 numpy 类型
df['int_col'] = df['float_col'].astype(int)     # 会截断小数
df['str_col'] = df['num_col'].astype(str)       # 数字 -> 字符串
df['bool_col'] = df['flag'].astype(bool)

# 转换为 pandas 可空类型（支持缺失值，不会将 NaN 强转为 float）
df['int_col'] = df['float_col'].astype('Int64') # 注意大写 I
df['str_col'] = df['obj_col'].astype('string')
df['bool_col'] = df['flag'].astype('boolean')

# 转换为 category（节省内存）
df['cat_col'] = df['str_col'].astype('category')

# 转换为 datetime64
df['date_col'] = pd.to_datetime(df['date_str'])
# 再用 astype 统一格式（如有需要）
df['date_col'] = df['date_col'].astype('datetime64[ns]')
```

**注意**：`astype()` 会忽略缺失值（NaN），但如果列里原本有 `pd.NA`，转成 `int` 会报错，此时应转为 `Int64`。若列里含有无法转换的字符串，`astype(int)` 会直接报错。

#### (2) 专门的安全转换函数

- **转数值**：`pd.to_numeric()`
```python
# errors='coerce' 将无法转换的值变为 NaN
df['price'] = pd.to_numeric(df['price'], errors='coerce')
# downcast 可缩小内存占用
df['price'] = pd.to_numeric(df['price'], downcast='integer')
```
- **转日期时间**：`pd.to_datetime()`
```python
df['date'] = pd.to_datetime(df['date_str'], format='%Y-%m-%d', errors='coerce')
# 从 Excel 序列号转换（1899-12-30 为基准）
df['date'] = pd.to_datetime(df['excel_serial'], unit='D', origin='1899-12-30')
```
- **转时间差**：`pd.to_timedelta()`
```python
df['duration'] = pd.to_timedelta(df['time_str'])
```

#### (3) 将 object 列中的多种空值占位符统一处理后再转类型

结合你之前的空值判断，典型流程：
```python
# 先将各种空值占位符替换为 NaN
na_vals = ['', 'NA', 'NULL', 'N/A', '-']
df.replace(na_vals, np.nan, inplace=True)
# 再安全转为数值
df['col'] = pd.to_numeric(df['col'], errors='coerce')
# 最后转为可空整型（避免 float）
df['col'] = df['col'].astype('Int64')
```

#### (4) 多列同时转换

```python
df = df.astype({
    'id': 'Int64',
    'name': 'string',
    'score': 'float',
    'active': 'boolean',
    'created_at': 'datetime64[ns]'
})
```

---

## 四、Pandas DataFrame **值**级别的类型转换

有时不是整列转类型，而是对每个单元格进行自定义转换。

### 1. Series 上的逐元素操作

- **`Series.apply(func)`**：对每个元素应用函数
```python
df['new'] = df['old'].apply(lambda x: x * 2 if isinstance(x, (int,float)) else x)
```
- **`Series.map(func/dict)`**：可用于查找替换或函数变换
```python
# 映射转换
df['code'] = df['status'].map({'pass':1, 'fail':0})
```

### 2. DataFrame 上的逐元素操作

- **`DataFrame.applymap(func)`**（新版本推荐用 `DataFrame.map(func)`）
```python
# 将所有数字四舍五入到两位小数
df = df.applymap(lambda x: round(x, 2) if isinstance(x, float) else x)
```
- **向量化条件转换**：直接用布尔索引或 `np.where`
```python
df['new'] = np.where(df['score'] > 60, 'pass', 'fail')
```

### 3. 分箱转换（连续值 → 类别）

```python
df['age_group'] = pd.cut(df['age'], bins=[0,18,35,60,100], labels=['child','youth','adult','senior'])
# 结果自动为 category 类型
```

### 4. 处理混合类型列的坑

从 Excel 读取时可能一列中混合数字和文本，其 dtype 为 `object`。直接 `.apply()` 通常安全，但性能低；若想转为数值，务必先用 `pd.to_numeric(errors='coerce')` 将非数字转 NaN，否则会报错。

---

## 五、实战：Excel / MySQL 数据导入后的类型修正示例

结合你之前询问的 Excel 和 MySQL 类型对应，这里给出几个典型修正场景：

### 从 Excel 读取后
```python
import pandas as pd
df = pd.read_excel('data.xlsx')

# 1. 日期序列号转日期（如列中为数字）
df['date'] = pd.to_datetime(df['date'], unit='D', origin='1899-12-30', errors='coerce')

# 2. 带有千位分隔符的数字字符串 (如 "1,234") 转为数值
df['amount'] = df['amount'].str.replace(',', '').astype(float)

# 3. 百分比字符串 (如 "85%") 转为浮点数
df['pct'] = df['pct'].str.rstrip('%').astype(float) / 100.0

# 4. 空值占位符清理后转为 Int64
df.replace(['', 'NULL', 'N/A'], pd.NA, inplace=True)
df['id'] = df['id'].astype('Int64')
```

### 从 MySQL 读取后
```python
from sqlalchemy import create_engine
engine = create_engine('mysql+pymysql://...')
df = pd.read_sql("SELECT * FROM products", engine)

# 1. Decimal 类型列可能被读为 object（字符串或 Decimal 对象）
# 转为 float，注意精度，或保留 Decimal 用于后续精确计算
from decimal import Decimal
df['price'] = df['price'].apply(lambda x: float(x) if x is not None else None)

# 2. TINYINT(1) 布尔标志：pymysql 返回 0/1，转为 bool
df['active'] = df['active'].astype(bool)

# 3. JSON 字符串解析为字典
import json
df['props'] = df['props'].apply(lambda x: json.loads(x) if pd.notna(x) else {})
```

---

## 六、总结与最佳实践

1. **先诊断，后转换**：用 `df.dtypes`、`df.head()`、`df['col'].unique()` 了解数据实际类型和异常值。
2. **优先使用列级转换**：`pd.to_numeric`、`pd.to_datetime` 配合 `errors='coerce'`，比循环 `apply` 快得多。
3. **缺失值友好**：想让整数列带 NaN，用 pandas 的可空类型 `Int64`、`boolean`、`string`，不要用 `numpy` 的 `int64`。
4. **精确度控制**：金融数据避免 `float` → `Decimal`，或读取 MySQL 时直接保留 `Decimal` 对象。
5. **链式操作**：结合 `astype`、`replace`、`fillna` 形成清洗流水线，最终得到干净、类型正确的 DataFrame。

掌握了这些，无论数据来自 Excel、MySQL 还是其他源，你都能游刃有余地完成类型整理和后续分析了。

# 日期时间操作

在数据分析与业务开发中，日期时间的计算无处不在。从合同到期日、工龄、工作天数到时间序列聚合，Python 标准库和 Pandas 提供了强大的支持。这里系统梳理日期时间对象、转换、运算以及常见业务场景的实现。

---

## 一、Python 标准库核心类与操作

`datetime` 模块是基石，主要类：

| 类 | 说明 | 示例 |
|---|------|------|
| `date` | 日期（年、月、日） | `date(2025, 5, 20)` |
| `time` | 时间（时、分、秒、微秒） | `time(14, 30, 45)` |
| `datetime` | 日期 + 时间 | `datetime(2025, 5, 20, 14, 30, 45)` |
| `timedelta` | 两个日期/时间之间的差值 | `timedelta(days=5, hours=3)` |
| `timezone` | 时区（Python 3.2+） | `timezone(timedelta(hours=8))` |

### 1. 创建对象
```python
from datetime import date, time, datetime, timedelta

d = date.today()                        # 当前日期
dt = datetime.now()                     # 当前日期时间（本地）
dt_utc = datetime.utcnow()              # UTC 时间（不含时区信息）
specific = datetime(2025, 5, 20, 14, 30)
```

### 2. 字符串 ↔ 日期时间
```python
# 字符串 → 日期时间
dt = datetime.strptime("2025-05-20 14:30:00", "%Y-%m-%d %H:%M:%S")
d = datetime.strptime("20/05/25", "%d/%m/%y").date()

# 日期时间 → 字符串
dt.strftime("%Y年%m月%d日 %H:%M")    # '2025年05月20日 14:30'
```

常用格式码：
- `%Y` 四位年，`%y` 两位年
- `%m` 月（01-12），`%d` 日（01-31）
- `%H` 24小时制小时，`%I` 12小时制，`%M` 分，`%S` 秒
- `%A` 星期全名，`%a` 缩写

### 3. 时间加减与差值
```python
dt + timedelta(days=30)                # 30天后
dt - timedelta(hours=2)                # 2小时前

diff = datetime(2025,6,1) - datetime(2025,5,20)  # timedelta(days=12)
diff.days            # 12
diff.total_seconds() # 转换为总秒数
```

### 4. 字段替换
```python
dt = datetime(2025, 5, 20, 14, 30)
dt.replace(year=2026, day=1)  # 只改年份和日，其他不变
```

### 5. 日期比较与排序
```python
date1 < date2    # 直接比较
max(dates)       # 获取最大日期
sorted(dates)    # 排序
```

---

## 二、时区处理

Python 3.9+ 推荐用内置 `zoneinfo`，更早版本可用 `pytz`。

### 1. 添加时区
```python
from datetime import datetime, timezone, timedelta
from zoneinfo import ZoneInfo

# 创建带时区的当前时间
dt_cn = datetime.now(ZoneInfo("Asia/Shanghai"))

# 给无时区日期时间附加时区
dt_utc = datetime(2025,5,20,14,30, tzinfo=timezone.utc)
dt_cn = dt_utc.astimezone(ZoneInfo("Asia/Shanghai"))  # 转换为中国时间
```

### 2. 时区转换
```python
dt_us = dt_cn.astimezone(ZoneInfo("America/New_York"))
```

### 3. 获取 UTC 时间戳
```python
dt.timestamp()  # 返回 POSIX 时间戳（秒）
```

---

## 三、Pandas 中的日期时间操作

Pandas 将日期时间抽象为 `Timestamp` 和 `Timedelta`，列的类型可以是 `datetime64[ns]`。

### 1. 创建与解析
```python
import pandas as pd

# 字符串数组转为 DatetimeIndex 或 Series
dates = pd.to_datetime(['2025-05-20', '2025-06-01', 'invalid'], errors='coerce')
# 处理 Excel 序列号（假设 44535 代表 2021-12-15）
pd.to_datetime(44535, unit='D', origin='1899-12-30')
```

### 2. 日期属性提取
```python
s = pd.Series(pd.date_range('2025-01-01', periods=3, freq='D'))
s.dt.year          # 年
s.dt.month         # 月
s.dt.day           # 日
s.dt.weekday       # 星期几（0=周一，6=周日）
s.dt.dayofyear     # 一年中的第几天
s.dt.quarter       # 季度
s.dt.is_month_start # 是否月初
s.dt.is_month_end   # 是否月末
```

### 3. 日期偏移与计算
```python
from pandas.tseries.offsets import MonthEnd, BusinessDay, DateOffset

s + MonthEnd()               # 移到本月末
s + BusinessDay(10)          # 10个工作日后的日期
s + DateOffset(months=3)     # 3个月后（会保持月末逻辑）
```

### 4. 时间序列重采样
```python
ts = pd.Series(np.random.randn(10), 
               index=pd.date_range('2025-01-01', periods=10, freq='H'))
# 按天汇总
ts.resample('D').mean()
# 按周汇总，从周日开始
ts.resample('W').sum()
# 按月，取最大值
ts.resample('M').max()
```

---

## 四、常用业务场景计算

结合前面的类型知识和空值处理，以下是可直接使用的实战代码。

### 1. 计算年龄/工龄（精确到年）
```python
from datetime import date

def calculate_age(birth: date, as_of: date = None) -> int:
    """计算周岁"""
    today = as_of or date.today()
    age = today.year - birth.year
    # 如果今年生日还没过，减1
    if (today.month, today.day) < (birth.month, birth.day):
        age -= 1
    return age

# 使用：calculate_age(date(1990, 8, 15))
```

### 2. 合同到期日计算（加上N个月/年，保持月末逻辑）
```python
from dateutil.relativedelta import relativedelta  # 需 pip install python-dateutil

expiry = datetime(2025, 1, 31) + relativedelta(months=6)  # 2025-07-31
# 如果是月末则保持月末
expiry = datetime(2025, 1, 31) + relativedelta(months=1)  # 2025-02-28
```
若不使用 dateutil，纯 datetime 加月需自行处理：
```python
def add_months(dt: datetime, months: int) -> datetime:
    month = dt.month - 1 + months
    year = dt.year + month // 12
    month = month % 12 + 1
    day = min(dt.day, [31,29 if year%4==0 and (year%100!=0 or year%400==0) else 28,31,30,31,30,31,31,30,31,30,31][month-1])
    return dt.replace(year=year, month=month, day=day)
```

### 3. 计算两个日期之间的工作日天数
```python
import numpy as np

workdays = np.busday_count('2025-05-01', '2025-05-31')  # 不含起，含止？默认不含结束日，需 +1
# 若包含起始日
workdays = np.busday_count('2025-05-01', '2025-05-31') + 1

# 自定义节假日
holidays = ['2025-05-01', '2025-05-02']
workdays = np.busday_count('2025-05-01', '2025-05-31', holidays=holidays)
```

用 Pandas：
```python
import pandas as pd
bd_range = pd.bdate_range(start='2025-05-01', end='2025-05-31', freq='C', holidays=holidays)
len(bd_range)
```

### 4. 找出下一个工作日（跳过周末和节假日）
```python
def next_business_day(start_date, holidays=None):
    holidays = holidays or []
    day = start_date + timedelta(days=1)
    while day.weekday() >= 5 or day in holidays:
        day += timedelta(days=1)
    return day
```

### 5. 获取某月最后一天 / 季度末
```python
# 当月最后一天
import calendar
_, last_day = calendar.monthrange(2025, 2)  # 返回(第一天星期, 最后一天)
date(2025, 2, last_day)

# 或用 Pandas
pd.Timestamp('2025-02-01') + pd.offsets.MonthEnd(1)  # 2025-02-28
# 季度末
pd.Timestamp('2025-05-20') + pd.offsets.QuarterEnd(startingMonth=1)  # 3月底
```

### 6. 日期分组（按周/月/季度/年）并汇总
```python
df['order_date'] = pd.to_datetime(df['order_date'])
# 增加月份标签
df['month'] = df['order_date'].dt.to_period('M')   # 2025-05
# 按周聚合销售额
weekly_sales = df.groupby(pd.Grouper(key='order_date', freq='W-MON'))['amount'].sum()
```

### 7. 计算时间间隔（小时、分钟）
```python
start = datetime(2025,5,20,9,0)
end = datetime(2025,5,20,17,30)
diff = end - start
hours = diff.total_seconds() / 3600   # 8.5 小时
```

### 8. 从 Excel 序列号转换日期
Excel 整数日期序列号是基于 1900-01-01 的天数（有 1900 年闰年 bug 注意）：
```python
def excel_serial_to_date(serial):
    """将 Excel 序列号转为 date，支持整数和浮点（含时间）"""
    if pd.isna(serial):
        return pd.NaT
    # Excel 日期基准：1899-12-30
    return pd.Timestamp('1899-12-30') + pd.Timedelta(days=serial)
```

### 9. 两个时间段是否重叠
```python
def is_overlap(start1, end1, start2, end2):
    return max(start1, start2) < min(end1, end2)
```

### 10. 时间序列填充缺失日期（补全日历）
```python
all_dates = pd.date_range(start=df['date'].min(), end=df['date'].max(), freq='D')
df_full = df.set_index('date').reindex(all_dates).fillna(0).reset_index()
```

---

## 五、注意事项与最佳实践

- **时区意识**：从 MySQL 读取的 `DATETIME` 一般无时区，跨时区应用时务必用 `zoneinfo` 显式设置，防止比较出错。
- **月末加减**：`relativedelta(months=1)` 会自动处理月末边界（如 1-31 → 2-28），而 `timedelta(days=30)` 不会。
- **性能**：大批量日期运算优先用 Pandas 向量化方法，避免逐行 `apply`。
- **Excel 日期**：读取 Excel 时推荐用 `pd.to_datetime` 配合 `origin='1899-12-30'` 或 `openpyxl` 的自动转换，注意纯时间会变成 `timedelta`。
- **空值**：日期列中的 `NaT` 用 `pd.isna()` 判断，不要用 `==`。

掌握以上内容后，无论是合同、考勤、财务周期还是时间序列分析，你都可以游刃有余地实现。

# 字符串操作及正则表达式

在数据处理全流程中，字符串操作与正则表达式是清洗、提取、校验信息的核心能力。下面系统梳理 Python 内置字符串方法、格式化、`re` 模块及在 `pandas` 中的实战用法。

---

## 一、Python 字符串核心操作

### 1. 基础特性

- 不可变序列，索引从 0 开始，支持切片 `s[start:stop:step]`
- 单引号、双引号、三引号（多行）均可
- 转义符 `\n` `\t`，原始字符串 `r"..."` 不转义

### 2. 常用方法速查表

| 分类 | 方法 | 说明 | 示例 |
|------|------|------|------|
| **大小写转换** | `upper()` / `lower()` | 全转大写/小写 | `'Abc'.lower()` → `'abc'` |
| | `capitalize()` | 首字母大写，其余小写 | `'aBC'.capitalize()` → `'Abc'` |
| | `title()` | 每个单词首字母大写 | `'hello world'.title()` → `'Hello World'` |
| | `swapcase()` | 大小写反转 | `'AbC'.swapcase()` → `'aBc'` |
| **判断** | `startswith(prefix)` / `endswith(suffix)` | 是否以某字符串开头/结尾 | `'data.csv'.endswith('.csv')` |
| | `isalpha()` / `isdigit()` / `isalnum()` | 是否全为字母/数字/字母+数字 | `'123'.isdigit()` → `True` |
| | `isspace()` | 是否全为空白字符 | `'   '.isspace()` → `True` |
| | `islower()` / `isupper()` | 是否全为小写/大写 | |
| | `isnumeric()` / `isdecimal()` | 是否全为数字字符（更宽泛） | `'Ⅷ'.isnumeric()` → `True` |
| **查找** | `find(sub)` / `rfind(sub)` | 返回索引，找不到返 -1 | `'abcabc'.find('a')` → 0 |
| | `index(sub)` / `rindex(sub)` | 同上，找不到抛 ValueError | |
| | `count(sub)` | 子串出现次数 | `'banana'.count('a')` → 3 |
| **替换** | `replace(old, new, count)` | 替换子串，count 限制次数 | `'a,b,c'.replace(',','-')` → `'a-b-c'` |
| | `translate(table)` | 结合 `str.maketrans` 进行字符级替换 | `'abc'.translate(str.maketrans('a','1'))` → `'1bc'` |
| **拆分与合并** | `split(sep, maxsplit)` | 按分隔符拆成列表，默认空白 | `'a,b,c'.split(',')` → `['a','b','c']` |
| | `rsplit(sep, maxsplit)` | 从右侧拆分 | `'a,b,c'.rsplit(',',1)` → `['a,b','c']` |
| | `splitlines(keepends)` | 按行拆分 | `'a\nb'.splitlines()` → `['a','b']` |
| | `partition(sep)` | 拆成三元组 (前, 分隔符, 后) | `'a,b'.partition(',')` → `('a',',','b')` |
| | `join(iterable)` | 用字符串连接可迭代对象 | `','.join(['a','b'])` → `'a,b'` |
| **修剪填充** | `strip(chars)` / `lstrip` / `rstrip` | 去除两端指定字符（默认空白） | `'  hi  '.strip()` → `'hi'` |
| | `center(width, fillchar)` | 居中对齐填充 | `'hi'.center(5,'-')` → `'-hi--'` |
| | `ljust(width, fillchar)` / `rjust` | 左/右对齐填充 | |
| | `zfill(width)` | 右对齐，左填充 `0` | `'42'.zfill(5)` → `'00042'` |
| **编码** | `encode(encoding)` | 转为 bytes | `'你好'.encode('utf-8')` |

### 3. 字符串格式化（三剑客）

**① %-格式化（旧式）**
```python
name = "World"
"Hello, %s!" % name              # 字符串
"pi = %.2f" % 3.14159           # 保留两位小数
```

**② `str.format()`**
```python
"{} {} {}".format(1,2,3)        # 按位置
"{1} {0}".format("a","b")       # 按索引
"name: {name}".format(name="X") # 按关键字
"pi: {:.2f}".format(3.14159)    # 格式化
```
格式规范：`[[fill]align][sign][#][0][width][,][.precision][type]`  
常用：`{:<10}` 左对齐宽10，`{:>10}` 右对齐，`{:^10}` 居中，`{:0>5}` 左补0，`{:,}` 千位分隔，`{:.2f}` 保留两位小数。

**③ f-string（推荐，Python 3.6+）**
```python
name = "Tom"
age = 25
f"{name} is {age} years old."
f"圆周率：{3.14159:.2f}"
f"二进制：{age:#b}"       # 带 0b 前缀
f"日期：{datetime.now():%Y-%m-%d}"
```
支持表达式 `f"{a+b}"`，甚至调用函数，但不能用反斜杠。

---

## 二、正则表达式 (re 模块)

正则表达式是处理非结构化文本的利器。使用**原始字符串** `r"..."` 避免多重转义。

### 1. re 核心函数

| 函数 | 作用 | 返回值 |
|------|------|--------|
| `re.search(pattern, string)` | 扫描整个字符串，找到**第一个**匹配 | Match 对象或 None |
| `re.match(pattern, string)` | 从**字符串开头**匹配，相当于 `^` | Match 对象或 None |
| `re.fullmatch(pattern, string)` | 整个字符串完全匹配模式 | Match 对象或 None |
| `re.findall(pattern, string)` | 返回所有非重叠匹配的**列表**（若模式有分组，返回元组列表） | list |
| `re.finditer(pattern, string)` | 返回所有匹配的**迭代器**，产生 Match 对象 | iterator |
| `re.sub(pattern, repl, string, count=0)` | 替换匹配项，repl 可为函数 | 新字符串 |
| `re.subn(pattern, repl, string)` | 同 sub，同时返回替换次数 | (新字符串, 次数) |
| `re.split(pattern, string, maxsplit=0)` | 按模式分割字符串 | list |
| `re.compile(pattern, flags)` | 预编译模式，提高反复使用效率 | Pattern 对象 |

**Match 对象常用方法**：

- `group(n)`：第 n 个分组捕获内容（也就是第几个括号中的内容）（0 为整个匹配）
- `groups()`：所有分组组成的元组（每个括号中的内容组成的元组）
- `groupdict()`：命名分组字典
- `start()` / `end()`：匹配起始/结束索引
- `span()`：(start, end)

```python
import re
s = "1a2bd45y93g"
re.match(r"(\d).+(\w)",s),re.match(r"(\d).+(\w)",s).groups(),re.match(r"(\d).+(\w)",s).group(0)
re.match(r"(\d).+(\w)",s).group(2)
```

```
(<re.Match object; span=(0, 11), match='1a2bd45y93g'>,
 ('1', 'g'),
 '1a2bd45y93g')
 'g'
```


### 2. 模式语法速查

#### 基本字符

- 大多数字符匹配自身
- `.` 匹配除换行符外的任意字符（`re.DOTALL` 下匹配所有）
- `\` 转义特殊字符，如 `\.` 匹配句点

#### 字符类
| 符号 | 含义 |
|------|------|
| `[abc]` | 匹配 a、b 或 c 中任一 |
| `[^abc]` | 匹配**除** a、b、c 外的字符 |
| `[a-z]` | 范围 |
| `\d` | 数字 `[0-9]` |
| `\D` | 非数字 |
| `\w` | 单词字符 `[a-zA-Z0-9_]` |
| `\W` | 非单词字符 |
| `\s` | 空白字符（空格、制表、换行等） |
| `\S` | 非空白 |

#### 量词（贪婪默认）
| 符号 | 次数 | 懒惰形式（尽可能少）|
|------|------|--------------------|
| `*` | 0 或多次 | `*?` |
| `+` | 1 或多次 | `+?` |
| `?` | 0 或 1 次 | `??` |
| `{n}` | 恰好 n 次 | `{n}?` |
| `{n,}` | 至少 n 次 | `{n,}?` |
| `{n,m}` | n 到 m 次 | `{n,m}?` |

#### 边界/锚点
- `^` 字符串开头（多行模式下每行开头）
- `$` 字符串结尾（多行模式下每行结尾）
- `\b` 单词边界
- `\B` 非单词边界

#### 分组与引用
| 语法 | 含义 |
|------|------|
| `( ... )` | 捕获分组，自动编号从 1 开始 |
| `(?: ... )` | 非捕获分组，只做逻辑分组不保存 |
| `(?P<name> ... )` | 命名捕获分组，可通过 groupdict 或 `\g<name>` 访问 |
| `\n` | 引用第 n 个分组（如 `\1`） |
| `(?(id)yes|no)` | 条件匹配 |

#### 断言（前瞻/后顾，不消耗字符）
| 语法 | 含义 |
|------|------|
| `(?=...)` | 正向前瞻，后面是...才匹配 |
| `(?!...)` | 负向前瞻，后面不是...才匹配 |
| `(?<=...)` | 正向后顾，前面是...才匹配（python 要求固定宽度） |
| `(?<!...)` | 负向后顾，前面不是...才匹配 |

#### 常用标志（可组合）
- `re.I` / `re.IGNORECASE`：忽略大小写
- `re.M` / `re.MULTILINE`：`^` `$` 匹配每行首尾
- `re.S` / `re.DOTALL`：`.` 匹配包括换行符
- `re.X` / `re.VERBOSE`：允许模式中添加空白和注释
- `re.A` / `re.ASCII`：`\w` `\b` 等仅匹配 ASCII

### 3. 常见业务场景示例

#### 提取信息
```python
text = "联系方式：张三 (zhangsan@mail.com)，电话：13800138000"
# 提取邮箱
email = re.search(r'[\w.-]+@[\w.-]+', text).group()
# 提取手机号（1开头的11位数字）
mobile = re.search(r'1[3-9]\d{9}', text).group()
# 提取所有数字
numbers = re.findall(r'\d+', text)
```

#### 替换与清洗
```python
# 脱敏手机号中间4位
re.sub(r'(1[3-9]\d)\d{4}(\d{4})', r'\1****\2', '13800138000')  # 138****8000
# 删除多余空白
re.sub(r'\s+', ' ', 'a   b  c').strip()   # 'a b c'
# 将驼峰命名转为下划线
re.sub(r'(?<!^)(?=[A-Z])', '_', 'myVarName').lower()  # 'my_var_name'
```

#### 验证格式
```python
def is_valid_email(s):
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return re.fullmatch(pattern, s) is not None
```

#### 高级分割
```python
# 按逗号分割但忽略引号内的逗号
re.split(r',(?=(?:[^"]*"[^"]*")*[^"]*$)', 'a,"b,c",d')
```

### 4. 性能与最佳实践
- **编译复用**：频繁使用相同模式时，用 `p = re.compile(r'...'); p.search(s)` 更快。
- **非捕获分组**：`(?: )` 可减少不必要的捕获开销。
- **避免“灾难性回溯”**：如 `(a+)+b` 匹配大量 “a” 会指数级耗时，谨慎使用嵌套量词。
- **原始字符串**：始终用 `r` 前缀，以免 `\b` 被解释为退格符。

---

## 三、Pandas 中的字符串操作（Series.str 访问器）

在数据处理中，通常对整个列批量应用字符串操作，避免循环。

前提：`s = df['column'].astype('string')` 或保持 object 类型。

| 方法 | 对应 Python/正则 | 示例 |
|------|------------------|------|
| `str.lower()` / `str.upper()` | 同字符串方法 | `df['name'].str.lower()` |
| `str.strip()` / `str.lstrip()` / `str.rstrip()` | 去空白/字符 | `df['text'].str.strip()` |
| `str.split(pat, expand=True)` | 分割，expand=True 直接拆成多列 | `df['name'].str.split(' ', expand=True)` |
| `str.replace(pat, repl, regex=True)` | 替换，可用正则 | `df['phone'].str.replace(r'(\d{3})\d{4}(\d{4})', r'\1****\2', regex=True)` |
| `str.contains(pat, na=False)` | 返回布尔序列 | `df['email'].str.contains(r'@')` |
| `str.extract(pat)` | 提取第一个捕获分组，返回列 | `df['text'].str.extract(r'(\d+)')` |
| `str.extractall(pat)` | 提取所有匹配，多级索引 | 用于一行多次匹配 |
| `str.findall(pat)` | 返回每个单元格匹配项组成的列表 | |
| `str.len()` | 字符串长度 | `df['comment'].str.len()` |
| `str.cat(sep)` | 连接 Series | `df['first'].str.cat(df['last'], sep=' ')` |
| `str.match(pat)` | 完全等同于 `re.match`，从开头匹配 | |
| `str.fullmatch(pat)` | 完全匹配 | |
| `str.slice(start,stop)` / `str[]` | 切片 | `df['code'].str[0:3]` |
| `str.zfill(width)` | 左补0 | `df['id'].str.zfill(6)` |

**实战清洗流水线**：
```python
# 读取 Excel 后，清洗姓名和手机号列
df['姓名'] = df['姓名'].str.strip().str.replace(r'\s+', ' ', regex=True)
df['手机'] = df['手机'].astype('string').str.extract(r'(1[3-9]\d{9})')  # 提取标准手机号
df['邮箱'] = df['邮箱'].str.lower().str.strip()
# 过滤无效邮箱
df = df[df['邮箱'].str.fullmatch(r'[\w.-]+@[\w.-]+\.\w+', na=False)]
```

---

## 四、总结

- **字符串方法**处理简单、明确的格式转换，如去除空格、大小写、拆分拼接。
- **正则表达式**处理复杂模式匹配、提取和验证，是文本处理的瑞士军刀。
- **Pandas .str** 将两种能力向量化，使数据列清洗效率极高，且能优雅处理缺失值（`na` 参数）。
- 最佳实践：优先使用 Python 内置字符串方法（可读性好），需要模式匹配时用正则，在 DataFrame 中始终通过 `.str` 访问器批量操作，并预编译高频率正则模式。

掌握这套组合拳，无论是从 Excel、数据库导入的脏数据清洗，还是日常文本分析需求，你都能快速实现。

# dataframe常用方法和操作

在数据处理的核心环节，Pandas DataFrame 的操作涉及创建、查询、清洗、变换、聚合和合并等多个方面。下面系统梳理最常用、最高频的方法和技巧，并结合你之前学到的**类型转换**、**空值处理**、**字符串与日期时间操作**，形成完整的知识网络。

---

## 一、创建与基本属性

### 1. 创建 DataFrame
```python
import pandas as pd
import numpy as np

# 从字典
df = pd.DataFrame({'A': [1,2,3], 'B': ['a','b','c']})

# 从列表/NumPy 数组
df = pd.DataFrame(np.random.randn(10, 4), columns=list('ABCD'))

# 从 Series
s1 = pd.Series([1,2,3], name='X')
s2 = pd.Series([4,5,6], name='Y')
df = pd.concat([s1, s2], axis=1)

# 从外部数据（Excel/CSV/SQL）见之前的总结，这里不再重复
```

### 2. 基本属性与概览
```python
df.shape          # 行列数 (rows, cols)
df.columns        # 列名
df.index          # 行索引
df.dtypes         # 各列数据类型
df.info()         # 内存、列、非空值数量、类型摘要
df.describe()     # 数值列的统计摘要（计数、均值、标准差、四分位数等）
df.describe(include='object')  # 字符串列摘要（计数、唯一值数、最高频值等）
df.head(n) / df.tail(n)        # 查看前/后 n 行
df.sample(n, random_state=42)  # 随机抽样 n 行
```

### 3. 设置与重置索引
```python
df.set_index('id', inplace=True)         # 将某列设为索引
df.reset_index(drop=False, inplace=True) # 将索引恢复为列，drop=True 则丢弃索引
```

---

## 二、数据选择与过滤

### 1. 列选择
```python
df['A']                    # 单列，返回 Series
df[['A', 'B']]             # 多列，返回 DataFrame
df.A                       # 点号访问，仅当列名是有效 Python 标识符时可用
```

### 2. 行选择（按位置或标签）
```python
# 按位置切片
df[0:3]                    # 前3行

# 按标签或布尔数组
df[df['A'] > 0]            # 条件过滤

# 使用 .loc[行标签, 列标签]
df.loc[2]                  # 索引为 2 的行
df.loc[:, 'A']             # 所有行的 A 列
df.loc[df['A'] > 0, ['A','B']]

# 使用 .iloc[行位置, 列位置]
df.iloc[0:3, 1:3]         # 前3行，第2到第3列
df.iloc[-1]                # 最后一行
```

### 3. 条件过滤（布尔索引）
```python
mask = (df['age'] > 18) & (df['city'] == 'NY')
df[mask]

# 使用 .query() 方法，更简洁
df.query('age > 18 and city == "NY"')

# 过滤 NaN
df[df['column'].notna()]

# 使用 .between(), .isin(), .str.contains()
df[df['age'].between(20, 30)]
df[df['code'].isin(['A1','A2'])]
df[df['name'].str.contains('John', na=False)]
```

### 4. 随机抽样与条件选取
```python
df.sample(frac=0.2)               # 随机抽取20%行
df.nlargest(5, 'score')           # score 最高的5行
df.nsmallest(3, 'price')          # price 最低的3行
```

---

## 三、数据清洗与转换（重点）

你之前已经掌握了很多零散点，这里整合成标准流程。

### 1. 缺失值处理
```python
# 检测
df.isna().sum()
df[df['A'].isna()]

# 删除
df.dropna(subset=['A','B'], inplace=True)   # 删除A或B有空值的行
df.dropna(how='all', inplace=True)          # 删除全为空的行
df.dropna(thresh=3, inplace=True)           # 至少3个非空值才保留

# 填充
df['A'].fillna(0, inplace=True)
df['B'].fillna(method='ffill', inplace=True)  # 前向填充
df.fillna({'A':0, 'B':'unknown'}, inplace=True)

# 空字符串转 NaN
df.replace('', np.nan, inplace=True)
df['col'] = df['col'].replace(r'^\s*$', np.nan, regex=True)
```

### 2. 类型转换
```python
# 单列
df['A'] = pd.to_numeric(df['A'], errors='coerce')
df['B'] = df['B'].astype('string')  # pandas 可空字符串
df['C'] = df['C'].astype('Int64')   # 可空整数

# 多列一次性
df = df.astype({'id':'Int64', 'name':'string', 'created':'datetime64[ns]'})

# 日期解析
df['date'] = pd.to_datetime(df['date_str'], errors='coerce')
df['time'] = pd.to_timedelta(df['duration_str'])
```

### 3. 重复值处理
```python
df.duplicated().sum()           # 重复行数
df.drop_duplicates(inplace=True)
df.drop_duplicates(subset=['id'], keep='first', inplace=True)
```

### 4. 数据替换
```python
df['status'].replace([1,0], ['active','inactive'], inplace=True)
df.replace({'A': {np.nan: -1}}) # 字典形式替换
```

### 5. 列的重命名与顺序调整
```python
df.rename(columns={'old':'new'}, inplace=True)
df.columns = ['col1','col2','col3']          # 直接赋值全部
df = df[['B', 'A', 'C']]                     # 重新排序列
```

### 6. 行列的删除
```python
df.drop('column_name', axis=1, inplace=True) # 删除列
df.drop([0,2], axis=0, inplace=True)         # 按索引删除行
df.drop(df[df['A']<0].index, inplace=True)   # 按条件删除行
```

---

## 四、排序与排名

```python
# 排序
df.sort_values('A', ascending=False, inplace=True)
df.sort_values(['A','B'], ascending=[True, False], inplace=True)
df.sort_index(ascending=False, inplace=True)       # 按索引排序

# 排名
df['rank'] = df['score'].rank(ascending=False, method='min')
```

---

## 五、聚合与分组（GroupBy）

这是数据分析的核心。
```python
# 简单聚合
df.groupby('category')['sales'].sum()
df.groupby('category')['sales'].agg(['sum', 'mean', 'count', 'std'])

# 对多列不同聚合
df.groupby('category').agg({
    'sales': 'sum',
    'profit': ['mean', 'max'],
    'customer': 'nunique'
})

# 自定义聚合函数
df.groupby('year').apply(lambda x: x['price'].max() - x['price'].min())

# 分组后重置索引
df.groupby('region')['sales'].sum().reset_index()

# 添加汇总行/列
pd.pivot_table(df, values='sales', index='region', columns='quarter', aggfunc='sum', margins=True)
```

### 高级分组方法
- `groupby().transform()` : 保持原形状，例如填充组内均值
```python
df['avg_sales'] = df.groupby('region')['sales'].transform('mean')
```
- `groupby().filter()` : 按组过滤
```python
df.groupby('city').filter(lambda g: len(g) >= 10)  # 保留记录数≥10的组
```

---

## 六、数据重塑：透视、堆叠与合并

### 1. 透视表与交叉表
```python
# pivot_table
pd.pivot_table(df, index='date', columns='product', values='sales', aggfunc='sum', fill_value=0)

# crosstab 统计频次
pd.crosstab(df['gender'], df['age_group'], margins=True)
```

### 2. melt – 宽表转长表
```python
pd.melt(df, id_vars=['id'], value_vars=['math','english'], var_name='subject', value_name='score')
```

### 3. stack / unstack
```python
df.set_index(['date','product']).unstack()  # 行索引的某一级转为列
stacked.unstack(level=0)                    # 列转回行
```

### 4. 合并表（join, merge, concat）
```python
# merge (类似 SQL JOIN)
pd.merge(left, right, on='key', how='inner')       # 内连接
pd.merge(left, right, left_on='lkey', right_on='rkey', how='left') # 左连接
pd.merge(left, right, on='key', how='outer', indicator=True) # 全外连接并标记来源

# join (基于索引合并)
left.join(right, how='inner')

# concat 纵向或横向拼接
pd.concat([df1, df2], axis=0, ignore_index=True)   # 纵向追加
pd.concat([df1, df2], axis=1)                      # 横向拼接
```

---

## 七、应用函数与向量化运算

### 1. 列/行级运算
```python
# 向量化操作（优先）
df['total'] = df['price'] * df['qty']
df['discount'] = np.where(df['member'], 0.9, 1.0)

# apply 对行/列应用函数
df['grade'] = df['score'].apply(lambda x: 'A' if x >= 90 else 'B')

# applymap / map 对每个元素
df[['col1','col2']] = df[['col1','col2']].applymap(lambda x: x.strip() if isinstance(x,str) else x)

# 使用 pipe 链式操作
def clean(df):
    return df.dropna().rename(...)
df_clean = (df.pipe(clean)
             .query('age>18')
             .assign(new_col = lambda d: d['A'] * 2))
```

### 2. 条件赋值
```python
df.loc[df['score'] > 90, 'level'] = 'excellent'
df['type'] = df['value'].mask(df['value'] < 0, 'negative').where(df['value'] != 0, 'zero')
```

### 3. 使用 .assign() 添加新列（链式）
```python
df.assign(
    ratio = df['profit'] / df['sales'],
    flag = lambda d: d['ratio'] > 0.2
)
```

---

## 八、时间序列操作（回顾）

前面专门讲过，这里只列几个最常用方法在 DataFrame 上的应用：
```python
# 设置为时间索引
df.set_index('date', inplace=True)

# 重采样
df.resample('M')['sales'].sum()

# 滚动窗口
df['rolling_avg'] = df['sales'].rolling(window=7).mean()

# 时间属性提取
df['year'] = df.index.year
df['month'] = df.index.month
df['weekday'] = df.index.weekday

# 日期范围生成
pd.date_range('2025-01-01', periods=10, freq='D')
```

---

## 九、性能优化与实用技巧

- **避免循环**：优先使用向量化运算和 `apply`，不推荐 `iterrows()`（性能差）。
- **类型优化**：读取时用 `dtype` 参数；转换数值用 `pd.to_numeric(downcast='integer')` 减少内存。
- **分类数据**：重复字符串列转为 `category` 类型，`df['col'].astype('category')`。
- **链式操作**：结合 `assign`, `pipe`, `query`, `loc` 写出清晰的转换流程。
- **大规模数据**：使用 `chunksize` 分块读取，或考虑 `dask`。

---

## 十、输出与保存

```python
# 写入 Excel
df.to_excel('output.xlsx', index=False, sheet_name='Sheet1')

# 写入 CSV
df.to_csv('output.csv', index=False, encoding='utf-8-sig')

# 写入数据库（需 sqlalchemy）
from sqlalchemy import create_engine
engine = create_engine('mysql+pymysql://...')
df.to_sql('table_name', engine, if_exists='replace', index=False)

# 写入剪贴板（方便粘贴到 Excel）
df.to_clipboard(index=False)
```

---

## 十一、总结：清洗与分析的典型流水线

```python
import pandas as pd
import numpy as np

# 1. 读取
df = pd.read_excel('raw.xlsx', dtype={'id':str}, na_values=['N/A','-'])

# 2. 初探
print(df.shape, df.dtypes, df.describe())

# 3. 清洗
df.drop_duplicates(subset='id', inplace=True)
df.columns = df.columns.str.strip().str.lower()
df['date'] = pd.to_datetime(df['date'], errors='coerce')
df['price'] = pd.to_numeric(df['price'], errors='coerce')
df['name'] = df['name'].str.strip().replace('', np.nan)
df.dropna(subset=['id','date','price'], inplace=True)
df['category'] = df['category'].astype('category')

# 4. 变换与特征
df['year_month'] = df['date'].dt.to_period('M')
df['amount'] = df['price'] * df['qty']
df['discount_flag'] = np.where(df['discount'] > 0, 1, 0)

# 5. 聚合分析
summary = (df.groupby(['year_month', 'category'], observed=True)
           .agg(total_sales=('amount','sum'),
                avg_price=('price','mean'),
                orders=('id','count'))
           .reset_index())

# 6. 保存结果
summary.to_excel('summary.xlsx', index=False)
```

这一套覆盖了 DataFrame 80% 以上的日常操作，配合你之前学到的类型转换、空值处理、字符串/正则、日期时间知识，可以高效完成绝大多数数据分析任务。



