---
title: 23 数据导出与序列化
---


> [!info] 章节总览
> 本章介绍 pandas 的 **数据导出与序列化** 体系，涵盖文本格式、二进制格式、数据库与 Excel、对象转换、格式参数、多 sheet 导出与压缩。

## 目录

| 笔记 | 内容 |
|------|------|
| [[23.6 文本格式导出]] | to_csv / to_json / to_html / to_latex / to_markdown / to_xml |
| [[23.2 二进制格式导出]] | to_pickle / to_feather / to_parquet / to_hdf |
| [[23.4 数据库与Excel导出]] | to_sql / to_excel |
| [[23.5 通用字符串与剪贴板]] | to_clipboard / to_string |
| [[23.1 对象转换]] | to_dict / to_records / to_numpy / to_list |
| [[23.3 格式参数与高级功能]] | 格式参数、导出多 sheet、压缩 |

## 核心速览

- **CSV** 是通用性最高的文本格式；**Parquet** 是性能最优的列式二进制格式
- `to_excel()` 支持多 sheet，`to_hdf()` / `to_parquet()` 支持多数据集
- 所有文本类导出均支持 `compression` 参数（gzip / bz2 / zip / xz / zstd）
- `to_dict()` / `to_records()` / `to_numpy()` / `to_list()` 用于内存中的对象转换

## 相关章节

- [[Pandas-二十二-样式与格式化-MOC]] - Styler 的 `to_html` / `to_excel` / `to_latex`
- [[Pandas-四-数据输入输出]] - 对应的读取函数
- [[Pandas-二十一-性能优化与大数据]] - 大数据导出时的性能与压缩策略
