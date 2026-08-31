---
title: 3 数据类型与扩展数据
---

> [!abstract] 章节导读
> pandas 的数据类型体系分为**基础 dtype** 和**扩展类型（ExtensionArray）**两类。理解它们之间的区别，有助于正确选择存储方式、避免类型陷阱并提高性能。

本章包含：

- [[3.1 dtypes]] – 所有 pandas 内置与扩展数据类型
- [[3.2 类型工具]] – 类型判断、转换函数及缺失值标识

## 为什么需要关注 dtype？

- 决定内存占用
- 决定运算规则
- 决定缺失值行为
- 决定与其他库（如 NumPy、PyArrow）的交互方式

> [!note] 快速查看数据类型
> - `df.dtypes` 返回每一列的类型
> - `df.info()` 概述各列类型与内存
> - `series.dtype` 查看单个 Series 的类型

## 小结


> [!success] 小结
> - **dtypes**：涵盖基础数值、布尔、对象、时间及可空/扩展类型，理解它们能更好地控制数据和内存。
> - **类型工具**：`pd.api.types` 提供精准类型判断；`astype`、`convert_dtypes`、`to_datetime` 等完成灵活转换。
> - **缺失值**：`pd.NA`、`pd.NaT`、`np.nan`、`None` 各有适用场景，推荐使用 `pd.isna()` 统一判断。