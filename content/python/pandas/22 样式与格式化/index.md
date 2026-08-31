---
title: 总览
---


> [!info] 章节总览
> 本章介绍 `DataFrame.style` 返回的 **Styler 对象**，涵盖 **格式化方法** 与 **样式函数**，以及最终的 **导出** 能力。

## 目录

| 笔记 | 内容 |
|------|------|
| [[22.4 DataFrame.style]] | Styler 对象概述 |
| [[22.2 格式化方法]] | format / hide / set_properties / apply 等 |
| [[22.3 样式函数]] | highlight_null / bar / background_gradient 等 |
| [[22.1 导出]] | to_html / to_excel / to_latex / to_string |

## 核心速览

- `df.style` 返回 Styler 对象，所有方法返回自身，支持**链式调用**
- `format()` 控制单元格显示格式，不改动底层数据
- `apply()` / `map()` 支持自定义样式函数
- 导出时可保留样式（HTML / Excel / LaTeX）

## 相关章节

- [[20.4 显示格式化]] - 控制台显示与 Styler 互补
- [[Pandas-二十三-数据导出与序列化-MOC]] - Styler 导出与普通导出对比
