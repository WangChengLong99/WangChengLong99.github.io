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

# 空行

在excel数据生成table时可以看到空行，即为空但是算作一行数据，导入其他系统时空行可能导致报错。



# 取一个数组的非连续列

在Excel中引用Table中非连续列组成新数组，有以下几种方法：

## 方法1：使用CHOOSE函数（推荐）
```excel
=CHOOSE({1,2}, 表1[第一列名], 表1[第三列名])
```

例如，如果列名分别是"姓名"和"部门"：
```excel
=CHOOSE({1,2}, 表1[姓名], 表1[部门])
```

## 方法2：使用HSTACK函数（Excel 365/2021+）
```excel
=HSTACK(表1[第一列名], 表1[第三列名])
```

## 方法3：使用INDEX函数
```excel
=INDEX(表1, SEQUENCE(ROWS(表1)), {1,3})
```

如果需要按列名引用：
```excel
=LET(
    data, 表1,
    col1, MATCH("第一列名", 表1[#标题], 0),
    col2, MATCH("第三列名", 表1[#标题], 0),
    INDEX(data, SEQUENCE(ROWS(data)), {col1, col2})
)
```

## 注意事项：

1. **CHOOSE函数**中 `{1,2}` 表示新数组的列顺序
2. 如果需要在其他函数中使用这个新数组，直接嵌套即可：
   ```excel
   =FILTER(CHOOSE({1,2}, 表1[姓名], 表1[部门]), 表1[工资]>5000)
   ```
3. 新数组的行数与原Table相同，列数为2

选择哪种方法取决于你的Excel版本和个人偏好，**CHOOSE函数**兼容性最好，适用于大多数Excel版本。
