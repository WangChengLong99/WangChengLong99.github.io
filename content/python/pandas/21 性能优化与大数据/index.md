---
title: 21 性能优化与大数据
---


> [!info] 章节总览
> 本章介绍 pandas 的性能优化技巧与大数据处理方案，涵盖 **向量化操作**、**避免循环**、**eval()/query()**、**数据类型优化**、**内存分析**、**分块读取**、**加速工具**、**Copy-on-Write**、**并行处理** 与 **第三方库集成**。

## 目录

| 笔记 | 内容 |
|------|------|
| [[21.5 向量化操作与避免循环]] | 向量化操作、避免循环、eval()/query() |
| [[21.4 数据类型优化]] | category、int32/float32、sparse、Arrow |
| [[21.3 内存分析与分块读取]] | 内存分析、chunksize、iterator、迭代器 |
| [[21.2 加速工具]] | NumPy、bottleneck、numba、PyArrow 字符串 |
| [[21.6 Copy-on-Write与并行处理]] | CoW 减少拷贝、并行处理工具 |
| [[21.1 第三方库集成]] | Dask、Modin、Vaex、Polars |

## 核心速览

- 优先使用**向量化操作**替代显式循环
- `pd.eval()` / `df.query()` 借助 numexpr 加速表达式计算
- 将 object 列转换为 `category` 可大幅降低内存
- 大文件使用 `chunksize` 分块读取
- 启用 `future.copy_on_write` 减少不必要的拷贝

## 相关章节

- [[20.5 选项类别]] - `compute.*` 选项控制加速开关
- [[Pandas-二十三-数据导出与序列化-MOC]] - 大数据导出时的压缩与分块
- [[Pandas-二十七-pandas 2.x 新特性与变更]] - CoW 与 PyArrow 字符串变更
