---
title: 总览
---


> [!info] 章节定位
> 分组与聚合（groupby）是 pandas 最强大、最常用的数据操作模式之一，通常遵循 **拆分 → 应用 → 合并（Split-Apply-Combine）** 范式。

## 章节地图

| 子章节 | 核心内容 | 笔记链接 |
| :--- | :--- | :--- |
| 1. groupby 创建 | `groupby()` 参数、按列/列表/函数/层级分组、`Grouper` 对象 | [[分组与聚合-1-groupby创建]] |
| 2. 迭代与分组 | 迭代组、`groups`、`indices`、`get_group()`、`size()`、`ngroups` | [[分组与聚合-2-迭代与分组]] |
| 3. 聚合 | `agg`/`aggregate`、内置/命名/自定义/多列聚合 | [[分组与聚合-3-聚合]] |
| 4. 变换 | `transform()`、与聚合区别、填充缺失、标准化 | [[分组与聚合-4-变换]] |
| 5. 过滤 | `filter()` | [[分组与聚合-5-过滤]] |
| 6. 应用 | `apply()`、`pipe()` | [[分组与聚合-6-应用]] |
| 7. 累积与排名 | `cumsum`/`cumprod` 等、`cumcount()`、`rank()`、`shift()`、`diff()` | [[分组与聚合-7-累积与排名]] |
| 8. 分组后操作 | `head`/`tail`/`sample`、`describe`、分组内 `resample`/`rolling` | [[分组与聚合-8-分组后操作]] |
| 9. 特殊分组 | 时间重采样、分类 `observed`、缺失值 `dropna`、多索引结果 | [[分组与聚合-9-特殊分组]] |

---

## 核心思想：Split-Apply-Combine

> [!example] 三阶段流程
> 1. **Split（拆分）**：根据 `by` 将 DataFrame 拆分为多个组；
> 2. **Apply（应用）**：对每组应用聚合（`agg`）、变换（`transform`）、过滤（`filter`）或通用函数（`apply`）；
> 3. **Combine（合并）**：将各组结果合并为最终输出。

```python
import pandas as pd

df = pd.DataFrame({
    "部门": ["A", "A", "B", "B", "C"],
    "员工": ["张三", "李四", "王五", "赵六", "孙七"],
    "薪资": [8000, 9500, 12000, 11000, 9000],
})

df.groupby("部门")["薪资"].sum()
```

> [!tip] 必备速查
> - `df.groupby(by=...)` ：创建分组对象；
> - `df.groupby(...).agg(funcs)` ：聚合；
> - `df.groupby(...).transform(func)` ：按组广播变换；
> - `df.groupby(...).filter(cond)` ：按组过滤；
> - `df.groupby(...).apply(func)` ：通用组级操作。

## 相关章节
- [[知识大纲|pandas 知识大纲]]
- [[分组与聚合-1-groupby创建]] → [[分组与聚合-9-特殊分组]]
- 依赖前置：[[pandas 2.3.3 数据结构|二、数据结构]]
