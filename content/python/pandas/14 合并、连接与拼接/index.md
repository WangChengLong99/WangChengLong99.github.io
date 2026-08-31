---
title: 14 合并、连接与拼接
---


> [!info] 章节总览
> 本部分覆盖 pandas 2.3.3 中合并、连接与拼接的完整知识体系，包含 `concat`、`merge`、`join`、`merge_asof`、`merge_ordered`、`compare` 以及 `combine` / `combine_first` / `update` 等组合操作。

## 📚 子笔记导航

| 子章节 | 笔记 | 核心内容 |
| --- | --- | --- |
| 1. concat() | [[14.1 concat()]] | 沿轴向拼接、MultiIndex、ignore_index |
| 2. merge() | [[14.2 merge()]] | 数据库风格合并、how、on、indicator、validate |
| 3. join() | [[14.3 join()]] | DataFrame.join、按索引连接、suffixes |
| 4. merge_asof() | [[14.4 merge_asof()]] | 时间序列近似匹配、tolerance、direction |
| 5. merge_ordered() | [[14.5 merge_ordered()]] | 有序数据合并、fill_method、分组外键 |
| 6. compare() | [[14.6 compare()]] | DataFrame/Series 差异比较、结果样式 |
| 7. 其他组合与更新 | [[14.7 其他组合与更新]] | combine、combine_first、update、align |

## 🧭 核心概念速览

> [!tip] 一句话总结
> pandas 提供四种合并路径：
> - **`concat()`** 沿轴向（行/列）直接**拼接**多个对象；
> - **`merge()`** 按**公共键**进行数据库风格的对齐合并；
> - **`join()`** 基于**索引**的便捷合并；
> - **`merge_asof()` / `merge_ordered()`** 处理**时间/有序数据**的近似与有序合并。
>
> 此外，`compare()` 用于**对比差异**，`combine()*` 系列用于**元素级组合**，`update()` 用于**原地修改**。

## 🔗 相关章节

- [[二、数据结构]] - 合并操作的数据载体
- [[五、数据选择与索引]] - 合并后的索引处理
- [[十三、重塑与透视表]] - 与合并相反的维度变换
- [[十五、分组与聚合]] - 合并与聚合的配合使用
