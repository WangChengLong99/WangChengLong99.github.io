---
index: 总览
---

> [!info] 章节总览
> 本部分覆盖 pandas 2.3.3 中日期时间与时间序列的完整知识体系，包含时间对象、创建方法、时间索引、`.dt` 访问器、时间序列操作、时区处理与日期偏移七大模块。

## 📚 子笔记导航

| 子章节 | 笔记 | 核心内容 |
| --- | --- | --- |
| 1. 时间对象 | [[9.1 时间对象]] | Timestamp、Timedelta、Period、DateOffset、NaT |
| 2. 创建方法 | [[9.2 创建方法]] | to_datetime、date_range、period_range 等 |
| 3. 时间索引 | [[9.3 时间索引]] | DatetimeIndex、TimedeltaIndex、PeriodIndex、频率字符串 |
| 4. .dt 访问器 | [[9.4 .dt 访问器]] | 日期组件、日历属性、星期属性、判断属性与方法 |
| 5. 时间序列操作 | [[9.5 时间序列操作]] | shift、asfreq、resample、rolling、at_time 等 |
| 6. 时区处理 | [[9.6 时区处理]] | tz_localize、tz_convert、夏令时、UTC |
| 7. 日期偏移 | [[9.7 日期偏移]] | DateOffset、BDay、CustomBusinessDay、节假日日历 |

## 🧭 核心概念速览

> [!tip] 一句话总结
> pandas 提供完整的时间序列工具链：**时间对象** 表示单个时刻或区间，**创建方法** 批量生成时间序列，**时间索引** 作为轴标签实现对齐与切片，**`.dt` 访问器** 提取时间组件，**时区处理** 统一标准时间，**日期偏移** 实现日历规则下的频率计算。

## 🔗 相关章节

- [[2.3 Index]] - DatetimeIndex、TimedeltaIndex、PeriodIndex 的详细属性与方法
- [[content/python/pandas/3 数据类型与扩展数组/index|index]] datetime64、timedelta64、`period[D]` 等 dtype
- [[十六、分组与聚合]] - 时间重采样（`groupby(pd.Grouper(freq=...))`）
- [[十七、窗口计算]] - 时间窗口 `rolling()` 与 `ewm()`
- [[十八、统计与数学计算]] - 时间序列上的统计计算


