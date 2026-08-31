---
title: 总览
---


> [!info] 章节总览
> 本章介绍 pandas 的全局配置系统，涵盖 **选项类别**、**获取与设置 API**、**显示格式化**、**浮点格式** 与 **表格样式** 五大核心主题。

## 目录

| 笔记 | 内容 |
|------|------|
| [[20.5 选项类别]] | display / mode / compute / plotting / future 五大选项类别 |
| [[20.3 获取与设置]] | get_option / set_option / reset_option / describe_option / option_context |
| [[20.4 显示格式化]] | 控制 DataFrame / Series 的输出显示效果 |
| [[20.2 浮点格式]] | 浮点数显示精度与自定义格式化 |
| [[20.1 表格样式]] | 表格样式的全局控制与 Styler 入口 |

## 核心速览

- `option_context()` 是临时修改选项的最佳方式，避免污染全局状态
- `display.max_columns` 是最常用的显示选项
- `future.copy_on_write` 控制 pandas 2.x 的 Copy-on-Write 行为
- `plotting.backend` 可切换 matplotlib / plotly 等绘图后端

## 相关章节

- [[Pandas-二十一-性能优化与大数据-MOC]] - `compute.*` 与 `future.*` 选项直接影响性能
- [[Pandas-二十二-样式与格式化-MOC]] - Styler 与显示格式化互补
- [[Pandas-二十三-数据导出与序列化-MOC]] - 导出时的编码、格式参数与全局设置关联
