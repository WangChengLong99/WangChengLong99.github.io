---
title: 总览
---


> [!info] 章节概述
> 本章讲解 pandas 中数据重塑（Reshaping）与透视（Pivoting）的核心工具。重塑是指改变数据的布局结构，透视则是按维度聚合数据的操作。掌握本章内容，能够灵活地在**长格式**与**宽格式**之间转换数据，为数据分析和可视化做准备。

## 📑 子章节导航

| 子章节 | 核心内容 | 关键函数 |
| ------ | -------- | -------- |
| [[15.1 堆叠与取消堆叠]] | 行/列索引互转 | `stack()`、`unstack()` |
| [[15.2 透视]] | 创建数据透视表 | `pivot()`、`pivot_table()` |
| [[15.3 长宽转换]] | 宽表↔长表 | `melt()`、`wide_to_long()` |
| [[15.4 交叉表]] | 频数统计表 | `crosstab()` |
| [[15.5 其他重塑操作]] | 行展开、哑变量等 | `explode()`、`get_dummies()`、`transpose()` 等 |

## 🧭 快速选择指南

| 需求场景 | 推荐工具 |
| -------- | -------- |
| 将行索引转为列（宽表） | `unstack()` 或 `pivot()` |
| 将列转为行索引（长表） | `stack()` 或 `melt()` |
| 带聚合的透视（分组统计） | `pivot_table()` |
| 统计频数/交叉分析 | `crosstab()` |
| 宽表转长表（多指标） | `wide_to_long()` |
| 将列表型单元格拆为多行 | `explode()` |
| 类别变量转为 0/1 哑变量 | `get_dummies()` |
| 行列转置 | `transpose()` / `T`、`swapaxes()` |
| 交换多层索引层级 | `swaplevel()` |

## 📌 关键词

- 长格式（Long Format）：每行一个观测值，适合 `groupby`、`plot`、`seaborn`
- 宽格式（Wide Format）：每行一个实体，每列一个变量，适合展示和部分建模
- 重塑（Reshape）：`stack` / `unstack` / `melt` / `pivot` / `wide_to_long`
- 透视（Pivot）：以指定维度为行、列，聚合第三维度

## 相关章节

- [[十六、分组与聚合]]：数据透视表的底层逻辑基于分组聚合
- [[15.2 透视]]：`pivot_table` 参数 `<aggfunc>` 详解
- [[9.8 日期时间与时间序列]]：时间序列重塑中常用 `stack/unstack`

---

> [!tip] 核心记忆
> - `stack` = 列 → 行；`unstack` = 行 → 列
> - `melt` = 宽 → 长；`pivot` = 长 → 宽（但要求无重复索引）
> - `pivot_table` = `pivot` + 聚合（可处理重复值）
> - `crosstab` = 两个或多个因子的频数 `pivot_table`
