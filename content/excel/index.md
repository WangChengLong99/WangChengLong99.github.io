---
tags:
  - 首页
title: excel实际问题
---
# 跨工作簿引用

跨工作簿引用表格使用绝对地址的格式为`'C:\Users\Administrator\Desktop\[sksso_test_system_users.xlsx]Result'!$A$2:$D$1665`,`"地址[表名]"sheet名!表范围`，单纯引用表格不需要加引号。

# 判断某个值是否在某个集合内

## 1. **COUNTIF函数（最常用）**

```excel
=COUNTIF(集合范围, 要判断的值) > 0
```

**示例**：判断A1是否在{"a","b","c"}中

```excel
=COUNTIF({"a","b","c"}, A1) > 0
```

或使用区域：

```excel
=COUNTIF($D$1:$D$3, A1) > 0
```

## 2. **MATCH函数 + ISNUMBER**

```excel
=ISNUMBER(MATCH(要判断的值, 集合范围, 0))
```

**示例**：

```excel
=ISNUMBER(MATCH(A1, {"a","b","c"}, 0))
=ISNUMBER(MATCH(A1, $D$1:$D$3, 0))
```

## 空行

在excel数据生成table时可以看到空行，即为空但是算作一行数据，导入其他系统时空行可能导致报错。

