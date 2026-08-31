---
title: 6 数据清洗与预处理
---
# 六、数据清洗与预处理

> [!abstract] 章节总览
> 本章系统地讲解 pandas 2.3.3 中数据清洗与预处理的完整工具链，涵盖缺失值处理、重复值处理、数据类型转换、替换与裁剪、排序与排名、函数应用以及数据变换七大主题。

## 📑 章节地图

| 小节 | 核心内容 | 状态 |
| --- | --- | --- |
| [[6.1 缺失值\|1. 缺失值]] | isna / notna / dropna / fillna / interpolate / ffill / bfill / replace / NA 标识 | ✅ |
| [[6.2 重复值\|2. 重复值]] | duplicated / drop_duplicates | ✅ |
| [[6.3 数据类型转换\|3. 数据类型转换]] | astype / convert_dtypes / infer_objects / to_numeric / to_datetime / to_timedelta / Categorical / cut / qcut | ✅ |
| [[6.4 替换与裁剪\|4. 替换与裁剪]] | replace / where / mask / clip / round / floor / ceil | ✅ |
| [[6.5 排序与排名\|5. 排序与排名]] | sort_index / sort_values / rank / nlargest / nsmallest | ✅ |
| [[6.6 函数应用\|6. 函数应用]] | map / apply / applymap / pipe / agg / aggregate / transform / eval / query | ✅ |
| [[6.7 数据变换\|7. 数据变换]] | shift / diff / pct_change / 累积 / rank / dot / transpose / 算术 / 比较 / 合并 | ✅ |

> [!tip] 学习建议
> - 缺失值与重复值是数据清洗的「第一步」，建议优先掌握。
> - 数据类型转换是后续所有分析的基础。
> - 函数应用与数据变换提供灵活的数据加工能力，可组合使用。

## 🔗 相关章节

- [[一、pandas 基础]]：视图与副本、链式赋值等底层概念
- [[二、数据结构]]：Series / DataFrame / Index 基础操作
- [[五、数据选择与索引]]：loc / iloc / query 等选择方法
- [[七、文本处理]]：字符串数据清洗


> [!abstract] 章节导读
> 数据清洗是数据分析流程中最耗时的环节，也是保证数据质量的关键。本章围绕**缺失值、重复值、异常值、数据类型、文本**五大主题，系统介绍 pandas 提供的全套清洗工具与最佳实践。

本章包含：

- [[6.1 缺失值处理-2]] – 缺失值检测、删除、填充、插值、替换与传播
- [[6.2 重复值处理-2]] – 重复行检测与删除
- [[6.3 异常值处理-2]] – 基于统计方法的异常值识别与处理
- [[6.4 数据类型转换]] – 类型强制转换、日期/数值/时间差解析与自动推断
- [[6.5 文本清洗-2]] – 字符串方法、拆分替换与正则提取

---

## 清洗流程建议

```mermaid
graph LR
    A[原始数据] --> B[缺失值处理]
    B --> C[重复值处理]
    C --> D[异常值处理]
    D --> E[数据类型转换]
    E --> F[文本清洗]
    F --> G[高质量数据]
```

---

## 常用工具速查

| 任务 | 推荐方法 |
|------|---------|
| 检测缺失值 | `isna()` / `notna()` |
| 删除缺失值 | `dropna()` |
| 填充缺失值 | `fillna()` / `ffill()` / `bfill()` / `interpolate()` |
| 替换缺失值 | `replace()` |
| 检测重复行 | `duplicated()` |
| 删除重复行 | `drop_duplicates()` |
| 截断异常值 | `clip()` |
| 替换异常值 | `replace()` / `where()` / `mask()` |
| 类型转换 | `astype()` / `pd.to_*()` / `convert_dtypes()` |
| 文本清洗 | `.str` 访问器方法 |


> [!success] 本章小结
> - **缺失值**：`isna` 检测，`dropna` 删除，`fillna` / `ffill` / `interpolate` 填充，`replace` 统一标记
> - **重复值**：`duplicated` 检测，`drop_duplicates` 删除，支持 `subset` / `keep`
> - **异常值**：Z-score 或 IQR 识别，`clip` 截断，`replace` 替换，按业务决定保留
> - **类型转换**：`astype` 强制，`to_numeric` / `to_datetime` / `to_timedelta` 解析，`convert_dtypes` 自动推断
> - **文本清洗**：`.str` 方法 + 正则，完成大小写、去除空格、替换、拆分、提取等操作
