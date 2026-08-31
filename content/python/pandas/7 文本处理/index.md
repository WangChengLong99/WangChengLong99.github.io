---
title: 总览
---

> [!abstract] 章节导读
> pandas 的文本处理核心是 **`.str` 访问器**，它将 Python 字符串方法与正则表达式向量化，使整列字符串操作无需编写循环。本章从访问器机制出发，系统覆盖字符串的基础操作、查找判断、拆分连接、正则提取、类型转换与性能优化。

本章包含：

- [[7.1 字符串访问器 .str]] – .str 访问器机制、适用类型与缺失值行为
- [[7.2 字符串基础方法]] – 大小写、空白处理、填充、长度、对齐
- [[7.3 字符串查找、判断与替换]] – contains / startswith / endswith / replace 等
- [[7.4 字符串拆分与连接]] – split / rsplit / cat / join 及列拼接
- [[7.5 正则表达式处理]] – extract / extractall / findall / count 等
- [[7.6 文本转换与编码]] – 文本与数值互转、编码与哑变量化
- [[7.7 文本处理实践与性能优化]] – 综合案例、向量化、内存优化与最佳实践

---

## 快速索引

### .str 方法总览

| 类别 | 方法 |
|------|------|
| 大小写 | `lower`、`upper`、`title`、`capitalize`、`swapcase`、`casefold` |
| 空白处理 | `strip`、`lstrip`、`rstrip` |
| 填充对齐 | `pad`、`center`、`zfill`、`ljust`、`rjust` |
| 长度与计数 | `len`、`count` |
| 判断与搜索 | `contains`、`startswith`、`endswith`、`match`、`fullmatch` |
| 替换 | `replace` |
| 拆分 | `split`、`rsplit`、`slice`、`get`、`slice_replace` |
| 连接 | `cat`、`join` |
| 提取 | `extract`、`extractall`、`findall` |
| 格式化 | `format` |
| 编码 | `encode`、`decode` |
| 翻译 | `translate` |

### 工作流程建议

```mermaid
graph LR
    A[原始文本列] --> B[统一格式]
    B --> C[查找定位]
    C --> D[拆分结构]
    D --> E[提取目标字段]
    E --> F[类型转换]
    F --> G[分析与建模]
```

---

## 与 6.5 文本清洗的关系

| 章节 | 角度 |
|------|------|
| [[6.5 文本清洗-2]] | 面向数据清洗流程，解决脏数据问题（空格、缺失标记、统一格式） |
| 本章（七、文本处理） | 面向文本处理全体系，提供完整、系统化的字符串与正则方法 |

> [!tip] 学习建议
> - 若只需快速清理文本，可直接查阅 [[6.5 文本清洗-2]]
> - 若需完整掌握字符串向量化操作、正则提取或文本转换，以本章为主线
